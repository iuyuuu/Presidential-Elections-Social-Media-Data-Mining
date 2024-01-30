# Data Mining 

The US presidential elections in 2024 is coming to the corner and much of the conversations about the elections are happening on social media. In this project, I tried to collect data from two YouTube channels (CNN and FoxNews) and two Blue Sky accounts (The Washington Post and the New York Times).


# Keyword selection


liberal,
conservative,
presidential,
democrats,
republicans,
election,
Biden,
Trump,
bill

The chosen keywords are essential for understanding the intricacies of the US presidential elections in 2024.

Liberal and Conservative: Liberal and Conservative represent the ideological spectrum prevalent in American politics. Understanding the positions and policies of liberal and conservative factions provides insight into the contrasting viewpoints and approaches to governance, influencing voter attitudes and candidate strategies.

Presidential: "Presidential" ensures that our exploration centers specifically on the dynamics, campaigns, and outcomes of the presidential race in 2024, offering a targeted analysis of this critical electoral process.

Democrats:"Democrats" encapsulates the party that held the presidency leading up to the 2024 

Trump: "Trump" represents the incumbent or former president who may have significant influence or involvement in the political landscape.

Election : The inclusion of "election" emphasizes the broader context of the electoral process. 

Biden:"Biden" signifies the Democratic candidate or incumbent president during the 2024 election, providing insights into his policies, campaign strategies, and potential impacts on the electoral landscape.

Bill: This keyword expands the scope to encompass legislative initiatives, proposals, or policy agendas that may influence the electoral discourse or voter preferences. 

# YouTube data collection
1. Get a YouTube API key
2. Install the Google API Python client
3. Then, to import the relevant Python packages
4. Initialize the YouTube API


```python
API_KEY = "AIzaSyAgT9FBXbDMHOMC6kyJfUurScD65FpVJas"
```


```python
!pip install --upgrade google-api-python-client --quiet
```


```python
import json
import googleapiclient
import googleapiclient.discovery
import googleapiclient.errors
from googleapiclient.discovery import build

```


```python
youtube = googleapiclient.discovery.build("youtube", "v3", developerKey=API_KEY)
```

## Get the video playlist of FoxNews and CNN
1. Get the channel ID.

2. Use the YouTube API to list all the videos uploaded on the channel starting with the most recent.
   
3. Go over the videos and check whether the video title contains any of the keywords selected.

4. Make sure to extract at least 50 election-related videos per channel.

For FoxNews

Fetch at least 50 videos that include the key words selected


```python
request = youtube.channels().list(
        part='id',
        forUsername="FoxNews"
    )
response = request.execute()
channel_id= response['items'][0]['id']
channel_id
```




    'UCp6yMWT7VTXdHRxGQ3gOpLw'




```python
def fetch_videos(channel_id):
    youtube = build('youtube', 'v3', developerKey=API_KEY)

    videos = []
    next_page_token = None

    while True:
        request = youtube.search().list(
            part='snippet',
            channelId=channel_id,
            maxResults=100,
            order='date',
            pageToken=next_page_token
        )
        response = request.execute()

        for item in response['items']:
            videos.append({
                'channelTitle':item['snippet']['channelTitle'],
                'id': item['id'],
                'title': item['snippet']['title'],
                'description': item['snippet']['description'],
                'published_at': item['snippet']['publishedAt']
                })

        next_page_token = response.get('nextPageToken')

        if not next_page_token:
            break

    return videos
   


```


```python
election_keywords = ["liberal", "conservative", "presidential", "democrats", "republicans", "election" ,"Biden" ,"Trump","bill"]
def contains_election_keywords(title):
    for keyword in election_keywords:
        if keyword in title.lower():
            return True
    return False
```


```python
def save_election_videos(channel_name, videos):
    relevant_videos = []

    for video in videos:
        title = video['title']
        description = video['description']
        videoId= video['id']
        channelTitle= video['channelTitle']
        publishedAt= video['published_at']
        if contains_election_keywords(title):
            relevant_videos.append({
                'channelTitle':channelTitle,
                'id': videoId['videoId'],
                'title': title,
                'publishedAt':publishedAt,
                'description': description
            })

    return relevant_videos


```


```python
fox_news_videos = fetch_videos('UCXIJgqnII2ZOINSWNOGFThA')
fox_news_election_videos = save_election_videos("FoxNews", fox_news_videos)

```


```python
len(fox_news_election_videos)
```




    58




```python
fox_news_election_videos[0]
```




    {'channelTitle': 'Fox News',
     'id': 'Rtc6PRKG47o',
     'title': 'RFK Jr.: The Democrats aren&#39;t pretending anymore',
     'publishedAt': '2024-01-30T14:45:01Z',
     'description': "Presidential candidate RFK Jr. joined 'Fox & Friends' to discuss his plan to mitigate the border surge and his reaction to a growing ..."}



For CNN

Get at least 50 videos that include the key words selected


```python
request1 = youtube.channels().list(
        part='id',
        forUsername="CNN"
    )
response1 = request1.execute()
channel_id1 = response1['items'][0]['id']
channel_id1
```




    'UCupvZG-5ko_eiXAupbDfxWw'




```python
cnn_news_videos = fetch_videos('UCupvZG-5ko_eiXAupbDfxWw') 
```


```python
cnn_news_election_videos = save_election_videos("CNN", cnn_news_videos)
```


```python
len(cnn_news_election_videos)
```




    53




```python
cnn_news_election_videos[0]
```




    {'channelTitle': 'CNN',
     'id': 'KIycMXs2O2I',
     'title': 'Retired conservative federal judge urges Supreme Court to disqualify Trump from office',
     'publishedAt': '2024-01-30T15:30:11Z',
     'description': 'Retired conservative Judge J. Michael Luttig argued that the Supreme Court should keep Donald Trump off the ballot in 2024.'}



## For each video fetch the 30 relevant comments. 
1. extract the video_id from the previous step
2. make a list of election video IDs
3. iterate through each video ID and fetch comments information(30 comments for each video)

For foxnews 

fetch 30 election relevant comments


```python
fox_news_election_video_ids = [video["id"] for video in fox_news_election_videos]
print(fox_news_election_video_ids)

```

    ['Rtc6PRKG47o', 'MtCevOQzKfI', 'mw0NmDMb-DE', 'ODuz6pIWUA4', 'UqClTujadU8', 'UIFtGbJFrKE', 'd8WsOajiHjY', 'F6YeI9AHoac', '504mmqxxv9M', 'UumTx2DwQ7Q', '-R0vnpDL6qw', 'JP7USCQPffI', 'L0dSzD3eLeI', 'PKIDu6uOO1w', 'oRd8T8sVsWE', 'Pzpvr2hk7rY', 'dlXQEmzfxV8', 't2St1jogCrI', 'Kiy8HvgOwyQ', '_CZlzdJgH2I', '_KGfQ1zEi3w', 'EivefwSAwqs', 'KXvU6KJ2uUQ', 'LRid1JJAXpI', 'n64gIQSPRkc', 'Ny-Tl8RUYj0', 'UlEDHsKr8PE', 'TxzgivOrJ94', 'b3zLRpDBcMQ', 'Ay6O4pwIaHo', 'J80GqCkD-FA', 'H2Jnm1jLwk8', 'OfKumVgVv3I', '5q-FRLUCBk8', 'aC1kHynH2Zo', 'yZMExQfPcj8', 'A-oSzZ6Vvdk', 's-rYDwSebbA', 'KdoZn8xkGnM', '8nVIBBddHyU', 'D5UQePDUMUU', 'Im8ls1qi-iU', '6rQeSfXaQm0', 'kJuZ5AyYdbg', '1l_npl6InTw', 'MLovx7CTowk', 'CghcdD2kMwo', 'agHCZ_qMs_o', 'QsP-n54qBEU', 'oJVkME2cy6E', '3mTxY70OQxA', 'Z7yjQcCdge4', '3mTxY70OQxA', 'Z7yjQcCdge4', '-aRBKwZJ7CE', '1DuIofdpAEM', 'G3V3e-oYbqY', 'WehQ4yTNz5g']



```python
fox_news_election_videos = [
    'LAaAXgKLfJY', 'UqClTujadU8', 'UIFtGbJFrKE', 'd8WsOajiHjY', 'F6YeI9AHoac', '504mmqxxv9M', 'UumTx2DwQ7Q', '-R0vnpDL6qw', 'JP7USCQPffI',  'PKIDu6uOO1w', 'oRd8T8sVsWE', 'Pzpvr2hk7rY', 'dlXQEmzfxV8', 't2St1jogCrI',  '_CZlzdJgH2I', '_KGfQ1zEi3w', 'EivefwSAwqs', 'KXvU6KJ2uUQ', 'LRid1JJAXpI', 'n64gIQSPRkc', 'Ny-Tl8RUYj0', 'UlEDHsKr8PE', 'TxzgivOrJ94',  'H2Jnm1jLwk8',  '5q-FRLUCBk8', 'aC1kHynH2Zo', 'yZMExQfPcj8', 'A-oSzZ6Vvdk', 's-rYDwSebbA', 'KdoZn8xkGnM', '8nVIBBddHyU', 'D5UQePDUMUU', 'Im8ls1qi-iU', '6rQeSfXaQm0', 'Im8ls1qi-iU', '6rQeSfXaQm0', 'kJuZ5AyYdbg', '1l_npl6InTw', 'MLovx7CTowk', 'CghcdD2kMwo', 'agHCZ_qMs_o', 'QsP-n54qBEU', '1DuIofdpAEM', 'G3V3e-oYbqY'
    ]
```


```python
# Function to fetch comments for a video

def fetch_video_comments(video_id):
    comments = []

    # Fetch comments for the video
    request = youtube.commentThreads().list(
        part="snippet",
        videoId=video_id,
        maxResults=30, 
        order="relevance"  
    )
    response = request.execute()

    # Extract comment information
    for item in response['items']:
            comments.append({
            "video_id": video_id,
            "comment_id": item["id"],
            "text": item["snippet"]["topLevelComment"]["snippet"]["textDisplay"],
            "published_at": item["snippet"]["topLevelComment"]["snippet"]["publishedAt"],
            "likeCount":item["snippet"]["topLevelComment"]["snippet"]["likeCount"]
                             })

    return comments

```


```python
video_comments_fox = {}

for video_id in fox_news_election_videos:
    comments_info = fetch_video_comments(video_id)
    video_comments_fox[video_id] = comments_info

```


```python
video_comments_fox
list_video_comments_fox = [item[0] for item in video_comments_fox.values()]
```


```python
list_video_comments_fox [0]
```




    {'video_id': 'LAaAXgKLfJY',
     'comment_id': 'UgyQ_6_3Tue-lWf6Q3J4AaABAg',
     'text': 'The fact that anyone would even listen to AOC at this point is absolutely insane',
     'published_at': '2024-01-29T19:05:41Z',
     'likeCount': 125}



For CNN

fetch 30 election relevant comments


```python
cnn_news_election_video_ids = [video["id"] for video in cnn_news_election_videos]
print(cnn_news_election_video_ids)
```


```python
cnn_news_election_videos = [
    'sdYmRp2K19g', 'BBVVHzVbZYY', 'dVcq6-GZ5ls', '1fsKvW8fcDw', 'djT5HHcTk10', '5DaVV221-yw', 'vxn6kBnxLEQ', 'V7n1Kt88UEU', 'xyaDBM0ToXQ', '94yxIq9hUuM', 'bK21nG7df0M', '5EK9zz8DfbI', '6mO5fOAnAUE', '2sGbt0w2IsU', '4VMfMu3GHOw', '5IYeyZ2nTsA', 'Rii0qZte1IU', '8iqSLtJ8t2o', 'R_xrZRUeaqk', 'TH_ACRZTMMo', 'CKVsyQMblFg', 'jvQluLtn7fY', 'j1cp2OLTS5w', 'x_DDQEVt3to', 'UC5SjkM-K-Q', 'AQ0rYI-3P-E', 'zEIQI1uE-s8', 'UtbA5pMc8jE', 'oLueBjkSObE', 'X1DL53cVJd0', '1aeppdXQwT4', 'y2yveI8a_Pk', 'K2muzjEJG-k', 'cNigZxNFCok', '8mat-AzlKIM', 'CcKyYlISZlg', 'gFM2Evm_z34', 'AwZFYeQezKk', 'Ji1mD2V_2RA', 'kv756wOiKak', 'WVkmKDpSxdg', '4k1iYKpUaKU', 'OVrtzg2nnEM', 'hnvxqu4BGTE', '1q1jqbJM2v4', 'oVsMpkjDDhI', 'WpXQTZSZv-M', 'UlsNRy5W3ME', 'b5P3xyCZ3Ws', 'BQPZ3sNUd64', 'clSBnWW1zHk', 'ES9PFJRcni0'
]

```


```python
video_comments_cnn = {}

for video_id in cnn_news_election_videos:
    comments_info = fetch_video_comments(video_id)
    video_comments_cnn[video_id] = comments_info

```


```python
video_comments_cnn
list_video_comments_cnn = [item[0] for item in video_comments_cnn.values()]
```


```python
list_video_comments_cnn[0]
```

## Combine the information collected above and make a data frame
1. Create a Pandas data frame that contains the relevant information about the data you extracted.
2. Call the video list and make it a data frame
3. Call the comment dictionary and make it a data frame
4. Combine the two data frames using video_id=id



```python
import pandas as pd
```

For foxnews
transfer list into data frame by using pandas


```python
df_fox_video=pd.DataFrame(fox_news_election_videos)
df_comments_fox=pd.DataFrame(list_video_comments_fox)

```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Cell In[72], line 1
    ----> 1 df_fox_video=pd.DataFrame(fox_news_election_videos)
          2 df_comments_fox=pd.DataFrame(list_video_comments_fox)


    NameError: name 'pd' is not defined



```python
fox_merged_df = pd.merge(df_comments_fox, df_fox_video, left_on='video_id', right_on='id')
fox_merged_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>video_id</th>
      <th>comment_id</th>
      <th>text</th>
      <th>published_at</th>
      <th>likeCount</th>
      <th>channelTitle</th>
      <th>id</th>
      <th>title</th>
      <th>publishedAt</th>
      <th>description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>LAaAXgKLfJY</td>
      <td>UgyQ_6_3Tue-lWf6Q3J4AaABAg</td>
      <td>The fact that anyone would even listen to AOC ...</td>
      <td>2024-01-29T19:05:41Z</td>
      <td>125</td>
      <td>Fox News</td>
      <td>LAaAXgKLfJY</td>
      <td>AOC admits Biden could be doing more to advanc...</td>
      <td>2024-01-29T19:00:10Z</td>
      <td>OutKick founder Clay Travis joined 'America's ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>UqClTujadU8</td>
      <td>UgzkCpbma6tluB3w67x4AaABAg</td>
      <td>Political strategist Drew Littman: &amp;quot;Secre...</td>
      <td>2024-01-28T20:58:13Z</td>
      <td>99</td>
      <td>Fox News</td>
      <td>UqClTujadU8</td>
      <td>Democrats and Republicans can ‘work together’ ...</td>
      <td>2024-01-28T20:45:01Z</td>
      <td>Political strategist Drew Littman and Holt Com...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>UIFtGbJFrKE</td>
      <td>UgyWKX15NQ98ZrNWTSJ4AaABAg</td>
      <td>I trust president Trump 💯 FJB and his whole ad...</td>
      <td>2024-01-28T15:36:57Z</td>
      <td>45</td>
      <td>Fox News</td>
      <td>UIFtGbJFrKE</td>
      <td>Biden is poised to blame this on Republicans: ...</td>
      <td>2024-01-28T14:17:16Z</td>
      <td>'Fox News Sunday' anchor Shannon Bream joins '...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>d8WsOajiHjY</td>
      <td>UgyeI7ZeSeq6RIjGTdl4AaABAg</td>
      <td>You are right about Reagan. I remember, he was...</td>
      <td>2024-01-28T06:16:21Z</td>
      <td>130</td>
      <td>Fox News</td>
      <td>d8WsOajiHjY</td>
      <td>Mark Levin warns about RINOs, liberals, modera...</td>
      <td>2024-01-28T03:00:21Z</td>
      <td>'Life, Liberty &amp; Levin' host Mark Levin gives ...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>F6YeI9AHoac</td>
      <td>UgztLxQFc2Zh_yLKaGh4AaABAg</td>
      <td>You should have left Trumps policies in place....</td>
      <td>2024-01-28T00:18:28Z</td>
      <td>32</td>
      <td>Fox News</td>
      <td>F6YeI9AHoac</td>
      <td>Kamala claims Republicans want to ‘run’ on the...</td>
      <td>2024-01-27T23:00:08Z</td>
      <td>Subscribe to Fox News! https://bit.ly/2vBUvAS ...</td>
    </tr>
  </tbody>
</table>
</div>



For cnn
transfer list into data frame by using pandas


```python
df_cnn_video=pd.DataFrame(cnn_news_election_videos)
df_comments_cnn=pd.DataFrame(list_video_comments_cnn)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>channelTitle</th>
      <th>id</th>
      <th>title</th>
      <th>publishedAt</th>
      <th>description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CNN</td>
      <td>KIycMXs2O2I</td>
      <td>Retired conservative federal judge urges Supre...</td>
      <td>2024-01-30T15:30:11Z</td>
      <td>Retired conservative Judge J. Michael Luttig a...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>CNN</td>
      <td>HcGDD4DH6mU</td>
      <td>What Al Franken wants ex-top Trump officials t...</td>
      <td>2024-01-30T12:41:36Z</td>
      <td>Former Democratic Sen. Al Franken tells CNN's ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>CNN</td>
      <td>sdYmRp2K19g</td>
      <td>Sen. Warren weighs in on Trump&amp;#39;s desire to...</td>
      <td>2024-01-29T18:38:34Z</td>
      <td>Sen. Elizabeth Warren (D-MA) speaks with CNN's...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CNN</td>
      <td>BBVVHzVbZYY</td>
      <td>Joe Manchin could upend Biden’s reelection cam...</td>
      <td>2024-01-29T01:30:08Z</td>
      <td>Sen. Joe Manchin says he “absolutely” can see ...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>CNN</td>
      <td>dVcq6-GZ5ls</td>
      <td>&amp;#39;Appalling&amp;#39;: Romney accuses Trump of t...</td>
      <td>2024-01-25T18:41:42Z</td>
      <td>Sen. Mitt Romney (R-UT) says that President Tr...</td>
    </tr>
  </tbody>
</table>
</div>




```python
cnn_merged_df = pd.merge(df_comments_cnn, df_cnn_video, left_on='video_id', right_on='id')
cnn_merged_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>video_id</th>
      <th>comment_id</th>
      <th>text</th>
      <th>published_at</th>
      <th>likeCount</th>
      <th>channelTitle</th>
      <th>id</th>
      <th>title</th>
      <th>publishedAt</th>
      <th>description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>sdYmRp2K19g</td>
      <td>UgznP5NfZXlGWRwbSsl4AaABAg</td>
      <td>Price gouging is the major factor in inflation...</td>
      <td>2024-01-29T19:06:36Z</td>
      <td>214</td>
      <td>CNN</td>
      <td>sdYmRp2K19g</td>
      <td>Sen. Warren weighs in on Trump&amp;#39;s desire to...</td>
      <td>2024-01-29T18:38:34Z</td>
      <td>Sen. Elizabeth Warren (D-MA) speaks with CNN's...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BBVVHzVbZYY</td>
      <td>Ugx2m4xtDX1Csny3us94AaABAg</td>
      <td>Easily one of the worst politicians alive today.</td>
      <td>2024-01-29T04:51:32Z</td>
      <td>48</td>
      <td>CNN</td>
      <td>BBVVHzVbZYY</td>
      <td>Joe Manchin could upend Biden’s reelection cam...</td>
      <td>2024-01-29T01:30:08Z</td>
      <td>Sen. Joe Manchin says he “absolutely” can see ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>dVcq6-GZ5ls</td>
      <td>Ugy2oLmWBkl9uf7FE794AaABAg</td>
      <td>Hard to imagine anything getting accomplished....</td>
      <td>2024-01-25T18:47:31Z</td>
      <td>1868</td>
      <td>CNN</td>
      <td>dVcq6-GZ5ls</td>
      <td>&amp;#39;Appalling&amp;#39;: Romney accuses Trump of t...</td>
      <td>2024-01-25T18:41:42Z</td>
      <td>Sen. Mitt Romney (R-UT) says that President Tr...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1fsKvW8fcDw</td>
      <td>Ugw2pekCjZ8JOpRxhzl4AaABAg</td>
      <td>The thought of the public “deciding “ an elect...</td>
      <td>2024-01-24T19:37:23Z</td>
      <td>56</td>
      <td>CNN</td>
      <td>1fsKvW8fcDw</td>
      <td>Democratic presidential candidate: &amp;#39;My par...</td>
      <td>2024-01-23T14:04:36Z</td>
      <td>Presidential candidate Rep. Dean Phillips (D-M...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>djT5HHcTk10</td>
      <td>Ugx42GGg2YyJBHj9GyZ4AaABAg</td>
      <td>You&amp;#39;re not suspending your campaign, you&amp;#...</td>
      <td>2024-01-21T21:01:31Z</td>
      <td>966</td>
      <td>CNN</td>
      <td>djT5HHcTk10</td>
      <td>Ron DeSantis ends his 2024 presidential campaign</td>
      <td>2024-01-21T20:55:44Z</td>
      <td>Florida Gov. Ron DeSantis announces he is endi...</td>
    </tr>
  </tbody>
</table>
</div>



Combine the two data frame cnn_merged_df and fox_merged_df


```python
yt_comments = pd.concat([cnn_merged_df, fox_merged_df])
yt_comments.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>video_id</th>
      <th>comment_id</th>
      <th>text</th>
      <th>published_at</th>
      <th>likeCount</th>
      <th>channelTitle</th>
      <th>id</th>
      <th>title</th>
      <th>publishedAt</th>
      <th>description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>sdYmRp2K19g</td>
      <td>UgznP5NfZXlGWRwbSsl4AaABAg</td>
      <td>Price gouging is the major factor in inflation...</td>
      <td>2024-01-29T19:06:36Z</td>
      <td>214</td>
      <td>CNN</td>
      <td>sdYmRp2K19g</td>
      <td>Sen. Warren weighs in on Trump&amp;#39;s desire to...</td>
      <td>2024-01-29T18:38:34Z</td>
      <td>Sen. Elizabeth Warren (D-MA) speaks with CNN's...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BBVVHzVbZYY</td>
      <td>Ugx2m4xtDX1Csny3us94AaABAg</td>
      <td>Easily one of the worst politicians alive today.</td>
      <td>2024-01-29T04:51:32Z</td>
      <td>48</td>
      <td>CNN</td>
      <td>BBVVHzVbZYY</td>
      <td>Joe Manchin could upend Biden’s reelection cam...</td>
      <td>2024-01-29T01:30:08Z</td>
      <td>Sen. Joe Manchin says he “absolutely” can see ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>dVcq6-GZ5ls</td>
      <td>Ugy2oLmWBkl9uf7FE794AaABAg</td>
      <td>Hard to imagine anything getting accomplished....</td>
      <td>2024-01-25T18:47:31Z</td>
      <td>1868</td>
      <td>CNN</td>
      <td>dVcq6-GZ5ls</td>
      <td>&amp;#39;Appalling&amp;#39;: Romney accuses Trump of t...</td>
      <td>2024-01-25T18:41:42Z</td>
      <td>Sen. Mitt Romney (R-UT) says that President Tr...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1fsKvW8fcDw</td>
      <td>Ugw2pekCjZ8JOpRxhzl4AaABAg</td>
      <td>The thought of the public “deciding “ an elect...</td>
      <td>2024-01-24T19:37:23Z</td>
      <td>56</td>
      <td>CNN</td>
      <td>1fsKvW8fcDw</td>
      <td>Democratic presidential candidate: &amp;#39;My par...</td>
      <td>2024-01-23T14:04:36Z</td>
      <td>Presidential candidate Rep. Dean Phillips (D-M...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>djT5HHcTk10</td>
      <td>Ugx42GGg2YyJBHj9GyZ4AaABAg</td>
      <td>You&amp;#39;re not suspending your campaign, you&amp;#...</td>
      <td>2024-01-21T21:01:31Z</td>
      <td>966</td>
      <td>CNN</td>
      <td>djT5HHcTk10</td>
      <td>Ron DeSantis ends his 2024 presidential campaign</td>
      <td>2024-01-21T20:55:44Z</td>
      <td>Florida Gov. Ron DeSantis announces he is endi...</td>
    </tr>
  </tbody>
</table>
</div>



## Make and save the csv file
1. Turn the data frame into a CSV
2. save it in a file called “yt_comments.csv”.


```python
yt_comments.to_csv('yt_comments.csv', index=False)
```

# BlueSky data collection

1. For each of the two Blue Sky accounts (The Washington Post and the New York Times) use the Blue Sky API to fetch all posts posted by each account. 

2. Then, go over all posts and check whether the posts’ text contains the election-related keywords, including liberal, conservative, presidential, democrats, republicans, election, Biden, Trump, bill

3. Check and save relevant information about the posts that are about the election

Pre-requisites


```python
!pip install atproto --quiet
```


```python
import json
from atproto import Client, models
```


```python
USERNAME = "iuyu.bsky.social"
APP_PASSWORD = "rzay-7m32-v2xg-7ob7"
```

Create a `Client` object and login using credentials


```python
client = Client()
client.login(USERNAME, APP_PASSWORD)
```




    ProfileViewDetailed(did='did:plc:v66go4gm4owc5to6c7wiba6d', handle='iuyu.bsky.social', avatar=None, banner=None, description=None, display_name=None, followers_count=6, follows_count=11, indexed_at='0001-01-01T00:00:00.000Z', labels=[], posts_count=2, viewer=ViewerState(blocked_by=False, blocking=None, blocking_by_list=None, followed_by=None, following=None, muted=False, muted_by_list=None, py_type='app.bsky.actor.defs#viewerState'), py_type='app.bsky.actor.defs#profileViewDetailed')



For Washingtonpost


```python
WP_data = client.get_author_feed(actor='washingtonpost.com',cursor= None)
WP_data.model_dump()
cursor=WP_data.cursor
WP_dict=[]

for i in range(len(WP_data.feed)):
    post=dict()
    post['account_name']=WP_data.feed[i].post.author.handle
    post['uri']=WP_data.feed[i].post['uri']
    post['text']=WP_data.feed[i].post.record.text
    post['created_at']=WP_data.feed[i].post.record.created_at
    post['like_count']=WP_data.feed[i].post["like_count"]
    post['reply_count']=WP_data.feed[i].post["reply_count"]
    post['repost_count']=WP_data.feed[i].post["repost_count"]
    WP_dict.append(post)

while cursor!= None :
    WP_data=client.get_author_feed(actor='washingtonpost.com',cursor= cursor)
    cursor=WP_data.cursor
    for i in range(len(WP_data.feed)):
        post=dict()
        post['account_name']=WP_data.feed[i].post.author.handle
        post['uri']=WP_data.feed[i].post['uri']
        post['text']=WP_data.feed[i].post.record.text
        post['created_at']=WP_data.feed[i].post.record.created_at
        post['like_count']=WP_data.feed[i].post["like_count"]
        post['reply_count']=WP_data.feed[i].post["reply_count"]
        post['repost_count']=WP_data.feed[i].post["repost_count"]
        WP_dict.append(post)
```


```python
election_keywords = ["liberal", "conservative", "presidential", "democrats", "republicans", "election","Biden" ,"Trump","bill"]
WP_relevant_posts = []

for post in WP_dict:
    post_text = post["text"]
    if any(keyword.lower() in post_text.lower() for keyword in election_keywords):
        WP_relevant_posts.append(post)

```


```python
len(WP_relevant_posts)
```




    310



For NYtimes


```python
NYT_data = client.get_author_feed(actor='nytimes.com',cursor= None)
NYT_data.model_dump()
cursor=NYT_data.cursor
NYT_dict=[]

for i in range(len(NYT_data.feed)):
    post=dict()
    post['account_name']=NYT_data.feed[i].post.author.handle
    post['uri']=NYT_data.feed[i].post['uri']
    post['text']=NYT_data.feed[i].post.record.text
    post['created_at']=NYT_data.feed[i].post.record.created_at
    post['like_count']=NYT_data.feed[i].post["like_count"]
    post['reply_count']=NYT_data.feed[i].post["reply_count"]
    post['repost_count']=NYT_data.feed[i].post["repost_count"]
    NYT_dict.append(post)

while cursor!= None :
    NYT_data=client.get_author_feed(actor='nytimes.com',cursor= cursor)
    cursor=NYT_data.cursor
    for i in range(len(NYT_data.feed)):
        post=dict()
        post['account_name']=NYT_data.feed[i].post.author.handle
        post['uri']=NYT_data.feed[i].post['uri']
        post['text']=NYT_data.feed[i].post.record.text
        post['created_at']=NYT_data.feed[i].post.record.created_at
        post['like_count']=NYT_data.feed[i].post["like_count"]
        post['reply_count']=NYT_data.feed[i].post["reply_count"]
        post['repost_count']=NYT_data.feed[i].post["repost_count"]
        NYT_dict.append(post)
```


```python
election_keywords = ["liberal", "conservative", "presidential", "democrats", "republicans", "election","Biden" ,"Trump","bill"]
NYT_relevant_posts = []

for post in NYT_dict:
    post_text = post["text"]
    if any(keyword.lower() in post_text.lower() for keyword in election_keywords):
        NYT_relevant_posts.append(post)

```


```python
len(NYT_relevant_posts)
```




    455



## BlueSky data collection

1. For each of the election-related post identified above, collect all of their replies
2. Save the replies into a list

For the washington post


```python
WP_uri = [post["uri"] for post in WP_relevant_posts]
```


```python
# create a list where you will save the results, i.e., information about the replies
WP_reply_info = []

for uri in WP_uri:
    feed_data=client.get_post_thread(uri=uri)
    first_reply=feed_data["thread"]["replies"]
    next_reply=[]
    next_reply.extend(first_reply)
    
    while next_reply:
    # get the reply at the end of the queue
    # (you can use pop: it removes the last element in the list and returns it)
        current_reply = next_reply.pop()
    # extract the relevant fields of this reply and save them to a dictionary
        reply_dict = {
            'post_id':current_reply.post.record.reply.parent.uri,
            'reply_id':current_reply.post.uri,
            'reply_uri':current_reply.post.record.reply.root.uri,
            'reply_text':current_reply.post.record.text,
            'reply_created_at':current_reply.post.record.created_at,
            'like_count':current_reply.post.like_count,
            'reply_count':current_reply.post.reply_count,
            'repost_count':current_reply.post.repost_count
        }
        WP_reply_info.append(reply_dict)

    # loop through all replies of this reply (if any) and add them to the queue
    # (you can use append)
    reply=current_reply.replies
    next_reply.append(reply)
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    File /opt/conda/lib/python3.10/site-packages/pydantic/main.py:759, in BaseModel.__getattr__(self, item)
        758 try:
    --> 759     return pydantic_extra[item]
        760 except KeyError as exc:


    KeyError: 'post'

    
    The above exception was the direct cause of the following exception:


    AttributeError                            Traceback (most recent call last)

    Cell In[122], line 16
         13     current_reply = next_reply.pop()
         14 # extract the relevant fields of this reply and save them to a dictionary
         15     reply_dict = {
    ---> 16         'post_id':current_reply.post.record.reply.parent.uri,
         17         'reply_id':current_reply.post.uri,
         18         'reply_uri':current_reply.post.record.reply.root.uri,
         19         'reply_text':current_reply.post.record.text,
         20         'reply_created_at':current_reply.post.record.created_at,
         21         'like_count':current_reply.post.like_count,
         22         'reply_count':current_reply.post.reply_count,
         23         'repost_count':current_reply.post.repost_count
         24     }
         25     WP_reply_info.append(reply_dict)
         27 # loop through all replies of this reply (if any) and add them to the queue
         28 # (you can use append)


    File /opt/conda/lib/python3.10/site-packages/pydantic/main.py:761, in BaseModel.__getattr__(self, item)
        759         return pydantic_extra[item]
        760     except KeyError as exc:
    --> 761         raise AttributeError(f'{type(self).__name__!r} object has no attribute {item!r}') from exc
        762 else:
        763     if hasattr(self.__class__, item):


    AttributeError: 'NotFoundPost' object has no attribute 'post'



```python
len(WP_reply_info)
```




    530




```python
WP_reply_info[0]
```




    {'post_id': 'at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky.feed.post/3kk7qqj3ucv2r',
     'reply_id': 'at://did:plc:gocfjm77z2emznwmmtxlttm4/app.bsky.feed.post/3kk7qubppuy2m',
     'reply_uri': 'at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky.feed.post/3kk7qqj3ucv2r',
     'reply_text': '"He did the thing that should disqualify him but we don\'t think we should disqualify him."\n\nW\nT\nF.',
     'reply_created_at': '2024-01-30T18:12:24.028Z',
     'like_count': 3,
     'reply_count': 1,
     'repost_count': 0}



For the NewYorktimes


```python
NYT_uri = [post["uri"] for post in NYT_relevant_posts]
```


```python
NYT_reply_info = []

for uri in NYT_uri:
    feed_data=client.get_post_thread(uri=uri)
    first_reply=feed_data["thread"]["replies"]
    next_reply=[]
    next_reply.extend(first_reply)
    
    while next_reply:
    # get the reply at the end of the queue
    # (you can use pop: it removes the last element in the list and returns it)
        current_reply = next_reply.pop()
    # extract the relevant fields of this reply and save them to a dictionary
        reply_dict = {
            'post_id':current_reply.post.record.reply.parent.uri,
            'reply_id':current_reply.post.uri,
            'reply_uri':current_reply.post.record.reply.root.uri,
            'reply_text':current_reply.post.record.text,
            'reply_created_at':current_reply.post.record.created_at,
            'like_count':current_reply.post.like_count,
            'reply_count':current_reply.post.reply_count,
            'repost_count':current_reply.post.repost_count
        }
        NYT_reply_info.append(reply_dict)

    # loop through all replies of this reply (if any) and add them to the queue
    # (you can use append)
    reply=current_reply.replies
    next_reply.append(reply)
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    File /opt/conda/lib/python3.10/site-packages/pydantic/main.py:759, in BaseModel.__getattr__(self, item)
        758 try:
    --> 759     return pydantic_extra[item]
        760 except KeyError as exc:


    KeyError: 'post'

    
    The above exception was the direct cause of the following exception:


    AttributeError                            Traceback (most recent call last)

    Cell In[131], line 15
         12     current_reply = next_reply.pop()
         13 # extract the relevant fields of this reply and save them to a dictionary
         14     reply_dict = {
    ---> 15         'post_id':current_reply.post.record.reply.parent.uri,
         16         'reply_id':current_reply.post.uri,
         17         'reply_uri':current_reply.post.record.reply.root.uri,
         18         'reply_text':current_reply.post.record.text,
         19         'reply_created_at':current_reply.post.record.created_at,
         20         'like_count':current_reply.post.like_count,
         21         'reply_count':current_reply.post.reply_count,
         22         'repost_count':current_reply.post.repost_count
         23     }
         24     NYT_reply_info.append(reply_dict)
         26 # loop through all replies of this reply (if any) and add them to the queue
         27 # (you can use append)


    File /opt/conda/lib/python3.10/site-packages/pydantic/main.py:761, in BaseModel.__getattr__(self, item)
        759         return pydantic_extra[item]
        760     except KeyError as exc:
    --> 761         raise AttributeError(f'{type(self).__name__!r} object has no attribute {item!r}') from exc
        762 else:
        763     if hasattr(self.__class__, item):


    AttributeError: 'NotFoundPost' object has no attribute 'post'



```python
len(NYT_reply_info)
```




    2421




```python
NYT_reply_info[0]
```




    {'post_id': 'at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky.feed.post/3kk7psiwq5m2y',
     'reply_id': 'at://did:plc:l2w2iwtjyteegqzp7yfdzenk/app.bsky.feed.post/3kk7spp7xec2u',
     'reply_uri': 'at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky.feed.post/3kk7psiwq5m2y',
     'reply_text': "Isn't it the board's job to decide eligibility to be on the ballot?   They do it all the time -- not enough signatures, not living in the district, too  young.",
     'reply_created_at': '2024-01-30T18:45:41.105Z',
     'like_count': 0,
     'reply_count': 0,
     'repost_count': 0}



## Combine the information collected above and make a data frame
1. Create a Pandas data frame that contains the relevant information about the data you extracted.
2. Call the post list and make it a data frame
3. Call the comment list and make it a data frame
4. Combine the two data frames using uri=post_id



For the washington post


```python
df_WP_posts=pd.DataFrame(WP_relevant_posts)
df_WP_reply=pd.DataFrame(WP_reply_info)
WP_merged_df = pd.merge(df_WP_posts, df_WP_reply, left_on='uri', right_on='post_id')
WP_merged_df.head()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>account_name</th>
      <th>uri</th>
      <th>text</th>
      <th>created_at</th>
      <th>like_count_x</th>
      <th>reply_count_x</th>
      <th>repost_count_x</th>
      <th>post_id</th>
      <th>reply_id</th>
      <th>reply_uri</th>
      <th>reply_text</th>
      <th>reply_created_at</th>
      <th>like_count_y</th>
      <th>reply_count_y</th>
      <th>repost_count_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>washingtonpost.com</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>The Illinois State Board of Elections dismisse...</td>
      <td>2024-01-30T18:10:21.278Z</td>
      <td>18</td>
      <td>2</td>
      <td>9</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>at://did:plc:gocfjm77z2emznwmmtxlttm4/app.bsky...</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>"He did the thing that should disqualify him b...</td>
      <td>2024-01-30T18:12:24.028Z</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>washingtonpost.com</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>The Illinois State Board of Elections dismisse...</td>
      <td>2024-01-30T18:10:21.278Z</td>
      <td>18</td>
      <td>2</td>
      <td>9</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>at://did:plc:ojz7khsudq7hxkblcqwternh/app.bsky...</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>"This Republican believes there was an insurre...</td>
      <td>2024-01-30T20:11:05.146Z</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>washingtonpost.com</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>UPS plans to cut 12,000 jobs as a part of a pl...</td>
      <td>2024-01-30T15:55:20.814Z</td>
      <td>41</td>
      <td>10</td>
      <td>15</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>at://did:plc:7rmkcsoqszbotlwdamz5eztl/app.bsky...</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>Cutting costs doesn't increase revenue.</td>
      <td>2024-01-30T15:58:36.392Z</td>
      <td>6</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>washingtonpost.com</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>UPS plans to cut 12,000 jobs as a part of a pl...</td>
      <td>2024-01-30T15:55:20.814Z</td>
      <td>41</td>
      <td>10</td>
      <td>15</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>at://did:plc:ah5u5ohfmkjxwrdvgyf7schn/app.bsky...</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>Maybe try a 10% executive pay cut and realloca...</td>
      <td>2024-01-30T18:23:40.465Z</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>washingtonpost.com</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>UPS plans to cut 12,000 jobs as a part of a pl...</td>
      <td>2024-01-30T15:55:20.814Z</td>
      <td>41</td>
      <td>10</td>
      <td>15</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>at://did:plc:bhtob5etjvnccglouvzxiv7k/app.bsky...</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>They had almost $10 billion in profits last ye...</td>
      <td>2024-01-30T16:14:12.946Z</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



For New York Times


```python
df_NYT_posts=pd.DataFrame(NYT_relevant_posts)
df_NYT_reply=pd.DataFrame(NYT_reply_info)
NYT_merged_df = pd.merge(df_NYT_posts, df_NYT_reply, left_on='uri', right_on='post_id')
NYT_merged_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>account_name</th>
      <th>uri</th>
      <th>text</th>
      <th>created_at</th>
      <th>like_count_x</th>
      <th>reply_count_x</th>
      <th>repost_count_x</th>
      <th>post_id</th>
      <th>reply_id</th>
      <th>reply_uri</th>
      <th>reply_text</th>
      <th>reply_created_at</th>
      <th>like_count_y</th>
      <th>reply_count_y</th>
      <th>repost_count_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>nytimes.com</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>The Illinois State Board of Elections rejected...</td>
      <td>2024-01-30T17:53:34.429Z</td>
      <td>10</td>
      <td>3</td>
      <td>8</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>at://did:plc:l2w2iwtjyteegqzp7yfdzenk/app.bsky...</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>Isn't it the board's job to decide eligibility...</td>
      <td>2024-01-30T18:45:41.105Z</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>nytimes.com</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>The Illinois State Board of Elections rejected...</td>
      <td>2024-01-30T17:53:34.429Z</td>
      <td>10</td>
      <td>3</td>
      <td>8</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>at://did:plc:pw3f36vq5a5kwmdim3nuzitw/app.bsky...</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>But that's already been determined</td>
      <td>2024-01-30T18:26:35.508Z</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>nytimes.com</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>The Illinois State Board of Elections rejected...</td>
      <td>2024-01-30T17:53:34.429Z</td>
      <td>10</td>
      <td>3</td>
      <td>8</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>at://did:plc:zbhggfk4zlwofwclezhbbe5r/app.bsky...</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>"did not have the authority"\n\nwhat, did they...</td>
      <td>2024-01-30T17:56:12.897Z</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>nytimes.com</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>Professors at the University of Pennsylvania s...</td>
      <td>2024-01-29T22:14:44.271Z</td>
      <td>32</td>
      <td>1</td>
      <td>11</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>at://did:plc:5sze4gz5jtvlowbjnomzqh5z/app.bsky...</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>Too bad @nytimes.com 100% is on the side of th...</td>
      <td>2024-01-29T23:46:04.716Z</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>nytimes.com</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>Charles Littlejohn, a former IRS contractor ac...</td>
      <td>2024-01-29T21:53:05.765Z</td>
      <td>25</td>
      <td>11</td>
      <td>12</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>at://did:plc:2qffdvlzftjxkmgepbony3ru/app.bsky...</td>
      <td>at://did:plc:eclio37ymobqex2ncko63h4r/app.bsky...</td>
      <td>Leaks a few documents ... go directly to jail....</td>
      <td>2024-01-29T22:02:40.550Z</td>
      <td>6</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



## Make and save the csv file
1. Turn the data frame into a CSV
2. save it in a file called “bsky_replies.csv”


```python
yt_comments = pd.concat([WP_merged_df, NYT_merged_df])
yt_comments.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>account_name</th>
      <th>uri</th>
      <th>text</th>
      <th>created_at</th>
      <th>like_count_x</th>
      <th>reply_count_x</th>
      <th>repost_count_x</th>
      <th>post_id</th>
      <th>reply_id</th>
      <th>reply_uri</th>
      <th>reply_text</th>
      <th>reply_created_at</th>
      <th>like_count_y</th>
      <th>reply_count_y</th>
      <th>repost_count_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>washingtonpost.com</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>The Illinois State Board of Elections dismisse...</td>
      <td>2024-01-30T18:10:21.278Z</td>
      <td>18</td>
      <td>2</td>
      <td>9</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>at://did:plc:gocfjm77z2emznwmmtxlttm4/app.bsky...</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>"He did the thing that should disqualify him b...</td>
      <td>2024-01-30T18:12:24.028Z</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>washingtonpost.com</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>The Illinois State Board of Elections dismisse...</td>
      <td>2024-01-30T18:10:21.278Z</td>
      <td>18</td>
      <td>2</td>
      <td>9</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>at://did:plc:ojz7khsudq7hxkblcqwternh/app.bsky...</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>"This Republican believes there was an insurre...</td>
      <td>2024-01-30T20:11:05.146Z</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>washingtonpost.com</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>UPS plans to cut 12,000 jobs as a part of a pl...</td>
      <td>2024-01-30T15:55:20.814Z</td>
      <td>41</td>
      <td>10</td>
      <td>15</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>at://did:plc:7rmkcsoqszbotlwdamz5eztl/app.bsky...</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>Cutting costs doesn't increase revenue.</td>
      <td>2024-01-30T15:58:36.392Z</td>
      <td>6</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>washingtonpost.com</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>UPS plans to cut 12,000 jobs as a part of a pl...</td>
      <td>2024-01-30T15:55:20.814Z</td>
      <td>41</td>
      <td>10</td>
      <td>15</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>at://did:plc:ah5u5ohfmkjxwrdvgyf7schn/app.bsky...</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>Maybe try a 10% executive pay cut and realloca...</td>
      <td>2024-01-30T18:23:40.465Z</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>washingtonpost.com</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>UPS plans to cut 12,000 jobs as a part of a pl...</td>
      <td>2024-01-30T15:55:20.814Z</td>
      <td>41</td>
      <td>10</td>
      <td>15</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>at://did:plc:bhtob5etjvnccglouvzxiv7k/app.bsky...</td>
      <td>at://did:plc:k5nskatzhyxersjilvtnz4lh/app.bsky...</td>
      <td>They had almost $10 billion in profits last ye...</td>
      <td>2024-01-30T16:14:12.946Z</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
yt_comments.to_csv('bsky_replies.csv', index=False)
```


```python

```
