# Spotify Streaming History Dashboard
### Objective
The main objective of this project, tackled in the first part of the dashboard, is to visually explore your mood is reflected in your music streaming habits over the course of a year. 
- From Spotify's persepctive, this would be valuable information that could help provide better song/playlist reccommendations by being able to predict the user's mood based on the month/day/hour/etc. 
- From an advertising perspective, the insights could help the advertiser choose more receptive ads by predicting the user's mood. 
- From a personal persepctive, it could serve as informative insights that could compliment the annual Spotify Wrapped. Since I am interested in seeing different information represented in my Spotify Wrapped, the goal of the second part of the dashboard is to provide a visual to represent your mood through music for the year, as well as a visual to represent how much you appreciate your top artist for the year. 

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
### Data Cleaning

### Data Visualization



### Documentation / Resources
-  [spotipy](https://spotipy.readthedocs.io/en/2.19.0/#examples)
-  [Spotify authorization scopes for Web API](https://developer.spotify.com/documentation/general/guides/authorization/scopes/)
-  [Spotify personal data overview](https://support.spotify.com/us/article/understanding-my-data/)
-  [plotly express](https://plotly.com/python/plotly-express/)
-  [dash html components](https://dash.plotly.com/dash-html-components)
-  [dash core components](https://dash.plotly.com/dash-core-components)
