# Spotify Streaming History Dashboard
### Objective
The main objective of this project, tackled in the first part of the dashboard, is to visually explore your mood is reflected in your music streaming habits over the course of a year. 
- From Spotify's persepctive, this would be valuable information that could help provide better song/playlist reccommendations by being able to predict the user's mood based on the month/day/hour/etc. 
- From an advertising perspective, the insights could help the advertiser choose more receptive ads by predicting the user's mood. 
- From a personal persepctive, it could serve as informative insights that could compliment the annual Spotify Wrapped. Since I am interested in seeing different information represented in my Spotify Wrapped, the goal of the second part of the dashboard is to provide a visual to represent your mood through music for the year, as well as a visual to represent how much you appreciate your top artist for the year. 

### Data Acquisition
**Part 1 - Personal Data**
Your personal streaming data must be requested through your Spotify account. Instructions can be found [here](https://support.spotify.com/us/article/data-rights-and-privacy-settings/). A few days after requesting my personal data, Spotify sent me a folder containing severeal json files containing my streaming habits. Once obtained, I combined my streaming history json files into one Pandas dataframe:
```python
df_file0 = pd.read_json('StreamingHistory0.json')
df_file1 = pd.read_json('StreamingHistory1.json')
df_file2 = pd.read_json('StreamingHistory2.json')
df_file3 = pd.read_json('StreamingHistory3.json')
df_file4 = pd.read_json('StreamingHistory4.json')

df_streams = pd.concat([df_file0, df_file1, df_file2, df_file3, df_file4]).reset_index(drop=True)
```
**Part 2 - Track Audio Features**

### Data Cleaning

### Data Visualization



### Documentation / Resources
-  [spotipy](https://spotipy.readthedocs.io/en/2.19.0/#examples)
-  [Spotify authorization scopes for Web API](https://developer.spotify.com/documentation/general/guides/authorization/scopes/)
-  [Spotify personal data overview](https://support.spotify.com/us/article/understanding-my-data/)
-  [plotly express](https://plotly.com/python/plotly-express/)
-  [dash html components](https://dash.plotly.com/dash-html-components)
-  [dash core components](https://dash.plotly.com/dash-core-components)
