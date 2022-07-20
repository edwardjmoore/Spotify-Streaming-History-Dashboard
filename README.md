# Spotify Streaming History Dashboard
View the interactive dashboard on Heroku [here](https://spotify-streaming-dashboard.herokuapp.com/). 

### About This Project 
My goal for this project was to produce an interactive dashboard using my personal streaming history data from Spotify from the past year. There are three objectives that I had in mind: 
1. How is my mood reflected in my streaming habits and are there any trends within my streaming habits in relation to the music's audio features?
1. How can I create a personalized visualization of my mood through music for the past year? 
1. How much do I really love my top streamed artist of the year and how I can visualize it?

***
### Results
**How is my mood reflected in my streaming habits and are there any trends within my streaming habits in relation to the music's audio features?**

I mainly looked at the audio feature valence to analyze how my mood is reflected in the music I listen to. Spotify defines valence as musical positiveness conveyed by a track. Songs with high valence are supposedly happier, while songs with low valence are sadder. 

Here's what I found:

<img align="left" width="33%" src="https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/streaming_habits_month.png?raw=true">
When grouping my streaming history by month, I found that I listened to the saddest music in January (valence = 0.36) and the happiest music in June (valence = 0.47). Not only were January and June the saddest and happiest months, respectively, but they were also my most streamed and least streamed months, respectively. In January I listened to 219 hours of music, whereas in June I only listened to 67 hours of music. In general, I found that I listened to sadder music in the Fall and early Winter and happier music in the summer, which could be a hint of some seasonal depression (all part of the fun of living in Canada).<br><br>
<img align="right" width="33%" src="https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/streaming_habits_day.png?raw=true">
When grouping my streaming habits by day of the week, I found that I listened to the saddest music on Mondays and Tuesdays and the happiest music on Fridays. Although there are trends in my streaming habits by day of the week, the variance between the different days is not as strong as the variance between different months. The valence of my streaming history grouped by month varied between 0.36 and 0.47, whereas the valence of my streaming history grouped by day of the week only varied between 0.39 and 0.42. This means that the month would be a stronger predictor of my mood than the day of the week. 

***
### Data Acquisition
**Part 1 - Personal Data**<br>
Your personal streaming data must be requested through your Spotify account. Instructions can be found [here](https://support.spotify.com/us/article/data-rights-and-privacy-settings/). A few days after requesting my personal data, Spotify sent me a folder containing severeal json files containing my streaming habits. Once obtained, I combined my streaming history json files into one Pandas dataframe:
```python
df_file0 = pd.read_json('StreamingHistory0.json')
df_file1 = pd.read_json('StreamingHistory1.json')
df_file2 = pd.read_json('StreamingHistory2.json')
df_file3 = pd.read_json('StreamingHistory3.json')
df_file4 = pd.read_json('StreamingHistory4.json')

df_streams = pd.concat([df_file0, df_file1, df_file2, df_file3, df_file4]).reset_index(drop=True)
```
**Part 2 - Track Audio Features**<br>
The streaming history data that Spotify provides is quite bare and only contains the end time, track name, artist name, and time played for each track. In order to obtain the audio features for each track (danceability, energy, acousticness, etc.), you need to have the URIs (the unique code Spotify assigns to each track). The URI for each track can be found using spotipy.search(), which retrieves queried information from the Spotify Web API:
```python
trackURI = []

for index, row in df.iterrows():
    artist = row['artistName']
    track = row['trackName']
    uri = row['uri']
    if pd.isnull(uri):
        query = sp.search(q='artist:' + artist + ' track:' + track, type='track')
        try:
            track_id = query['tracks']['items'][0]['id']
            trackURI.append(track_id)
        except:
            trackURI.append(np.nan)
    else:
        trackURI.append(uri)
        
df['uri'] = trackURI
```
The audio features for each track can then be retrieved using spotipy.audio_features():
```python
danceability = []
energy = []
loudness = []
speechiness = []
acousticness = []
instrumentalness = []
liveness = []
valence = []
tempo = []
duration_ms = []

for index, row in df.iterrows():
    if index % 200 == 0:
        time.sleep(random.uniform(3, 6))
    uri = row['uri']
    query = sp.audio_features(uri)
    try:
        danceability.append(query[0]['danceability'])
    except:
        danceability.append('NA')
    try:
        energy.append(query[0]['energy'])
    except:
        energy.append('NA')
    try:
        loudness.append(query[0]['loudness'])
    except:
        loudness.append('NA')
    try:
        speechiness.append(query[0]['speechiness'])
    except:
        speechiness.append('NA')
    try:
        acousticness.append(query[0]['acousticness'])
    except:
        acousticness.append('NA')
    try:
        instrumentalness.append(query[0]['instrumentalness'])
    except:
        instrumentalness.append('NA')
    try:
        liveness.append(query[0]['liveness'])
    except:
        liveness.append('NA')
    try:
        valence.append(query[0]['valence'])
    except:
        valence.append('NA')
    try:
        tempo.append(query[0]['tempo'])
    except:
        tempo.append('NA')
    try:
        duration_ms.append(query[0]['duration_ms'])
    except:
        duration_ms.append('NA')
        
df['danceability'] = danceability
df['energy'] = energy
df['loudness'] = loudness
df['speechiness'] = speechiness
df['acousticness'] = acousticness
df['instrumentalness'] = instrumentalness
df['liveness'] = liveness
df['valence'] = valence
df['tempo'] = tempo
df['duration_ms'] = duration_ms
```
***
### Data Cleaning
Some data cleaning was done before acquiring the URI for each track. This includes:
- Dropping entries with unknown artist or unknown track
- Removing songs with 0ms listening time
- Removing extra days (we want exactly one year)

After acquiring the URI for each track and before acquiring the audio features for each track, the following updates were made to the dataframe:
- Removing entries with no URI
- Adding a startTime column by subtracting msPlayed from the endTime
- Adding a listeningSession columnm by defining a new streaming session as starting at least one hour after the end of the previous song)
- Adding day number, month, day of week, and hour of day columns in order to visualize data based on different time intervals

After acquiring the audio features for each track, the following updates were made:
- Removing rows where audio features weren't found
- Normalizing audio features that do not already range from 0 to 1

Finally, new dataframes with the specific requirements for the individual visualizations were created by grouping and modifying the master dataframe. The dataframes created include average audio features grouped by day number (1-365), by streaming session (1-1000), by month (Jan-Dec), by day of week (Mon-Sun), and by hour of day (0-24), as well as top artist streaming data grouped by day number and by streaming session. 
***
### Data Visualization
The interactive dashboard layout was created using Dash core and html components and then populated with Plotly graphs. The dashboard allows you to view the following visualizations:
- Streaming habits by month, day of week, and hour of day - with dropdown to select the audio feature you wish to explore for each graph
- All audio features compared side by side - with dropdown to select the time interval you wish to explore (by day or by session)
- Audio feature mapped out over time interval - with dropdowns to select specific audio feature and time interval (all days or all sessions)
- Visual representation of how much you listened to your top artist over the year - with dropdown to select time interval (per day or per session)

![alt text](https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/dashboard_part_1.png?raw=true)
![alt text](https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/dashboard_part_2.png?raw=true)
***
### Results
**Part 1: Streaming Habits Insights**

Streaming Habits by Month:
- January was the month with the most streaming time while June was the month with the least.
- By looking at how the various audio features are distributed over the months we can conclude that January was when the slowest/saddest/least energetic music was played, while June was when the fastest/happiest/most energetic music was played.
- The winter months have similar streaming habits as January, while the summer months have similar streaming habits as June. The spring and fall months are somewhere in between.
- Based on these insights, Spotify could recommend music for the current month that has similar audio features as the historical values for the specified month. From an advertising perspective, I could be presented with different ads based on the month and my mood as predicted my streaming habits. From a personal perspective, I could actively listen to happier music in the winter months, knowing that historically, I have listened to slower music in the winter. 

![alt text](https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/streaming_habits_month.png?raw=true)

Streaming Habits by Day of Week:
- In general, I listened to more music on weekdays than weekends. 
- I streamed faster/happier/more energetic music on Fridays, Saturdays, and Sundays and slower/sadder/less energetic music on Mondays and Tuesdays. Wednesdays and Thursdays lied somewhere in the middle. 
- Based on these insights, Spotify could recommend music for the current day of the week that has similar audio features as the historical values for that day. From an advertising perspective, I could be presented with different ads based on the day of week and my mood as predicted my streaming habits. From a personal perspective, I could listen to more upbeat music on Mondays and Tuesdays, knowing that I tend to be in a less energetic mood on those days.

![alt text](https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/streaming_habits_day.png?raw=true)

Streaming Habits by Hour of Day:
- I listened to much less music from 6am to 1pm than the rest of the day. 
- Based on my hourly listening habits, my mood is more uplifting from 2pm to 1am. After 1am and before 2pm, I tend to listen to slower, more acoustic music. 

![alt text](https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/streaming_habits_hour.png?raw=true)

Audio Features per Day and per Streaming Session Over Previous Year:
- Comparing the audio features grouped by day and by streaming session, we can see that grouping by streaming session results in a more dynamic range within each audio features. I believe this proves that analyzing the audio features by streaming session is more effective than anlyzing by day. A single streaming session is more likely to have a smaller range for each audio feature, resulting in more accurate grouping. When viewing the plot grouped by day, the audio features' ranges are "smoother", likely because one day could have many streaming sessions, each with vastly different audio features.
- The audio features that appear more monochromatic tell us that the music I listen to tends to have a smaller range of acceptable values for that feature. For example, if a song has an instrumentalness of >0.4, I would probably skip it. The features that have a more diverse range of colours show us that I have a wider range of acceptable values for that feature. For example, the acousticness of a song would not be a good predictor as to whether or not I would skip the song since I tend to listen to songs with a wide range of acousticness. 

![alt text](https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/audio_features_day.png?raw=true) ![alt text](https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/audio_features_session.png?raw=true)

**Part 2: Personalized Visualizations**

The purpose of this section is to provide unique, personalized visualizations of your streaming history for the past year. Think of the following visuals as unique representations of your streaming habits that could be provided in something like Spotify Wrapped. 

Streaming Time vs. Audio Feature:
- This graph allows you to choose an audio feature and see how it is represented in your streaming habits over the past year. You can also choose to view each bar as a listening session, instead of a day, to get a more accurate representation of your changing mood.

![alt text](https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/mood_day.png?raw=true)
![alt text](https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/mood_session.png?raw=true)

Top Artist Appreciation:
- This graph shows you how much you love your top artist. You can view by day or by streaming session. Each circle represents a day or a streaming session. An orange cirlce means you listened to your favourite artist that day or session. A blue circle means you did not listen to your favourite artist that day or session. 

![alt text](https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/artist_day.png?raw=true)
![alt text](https://github.com/edwardjmoore/Spotify-Streaming-History-Dashboard/blob/main/images/artist_session.png?raw=true)
***
### Documentation / Resources
-  [spotipy](https://spotipy.readthedocs.io/en/2.19.0/#examples)
-  [Spotify authorization scopes for Web API](https://developer.spotify.com/documentation/general/guides/authorization/scopes/)
-  [Spotify personal data overview](https://support.spotify.com/us/article/understanding-my-data/)
-  [plotly express](https://plotly.com/python/plotly-express/)
-  [dash html components](https://dash.plotly.com/dash-html-components)
-  [dash core components](https://dash.plotly.com/dash-core-components)
