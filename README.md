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
API_KEY = "AASdgSyAgT9FBXbDMHOMC6kyJfUurScD7hffFpVJas"
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




    54




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

    ['Rtc6PRKG47o', 'MtCevOQzKfI', 'mw0NmDMb-DE', 'LAaAXgKLfJY', 'ODuz6pIWUA4', 'UqClTujadU8', 'UIFtGbJFrKE', 'd8WsOajiHjY', 'F6YeI9AHoac', '504mmqxxv9M', 'UumTx2DwQ7Q', '-R0vnpDL6qw', 'JP7USCQPffI', 'L0dSzD3eLeI', 'PKIDu6uOO1w', 'oRd8T8sVsWE', 'Pzpvr2hk7rY', 'dlXQEmzfxV8', 't2St1jogCrI', 'Kiy8HvgOwyQ', '_CZlzdJgH2I', '_KGfQ1zEi3w', 'EivefwSAwqs', 'KXvU6KJ2uUQ', 'LRid1JJAXpI', 'n64gIQSPRkc', 'Ny-Tl8RUYj0', 'UlEDHsKr8PE', 'TxzgivOrJ94', 'b3zLRpDBcMQ', 'Ay6O4pwIaHo', 'J80GqCkD-FA', 'H2Jnm1jLwk8', 'OfKumVgVv3I', '5q-FRLUCBk8', 'aC1kHynH2Zo', 'yZMExQfPcj8', 'A-oSzZ6Vvdk', 's-rYDwSebbA', 'KdoZn8xkGnM', '8nVIBBddHyU', 'D5UQePDUMUU', 'Im8ls1qi-iU', 'D5UQePDUMUU', 'Im8ls1qi-iU', '6rQeSfXaQm0', 'kJuZ5AyYdbg', '1l_npl6InTw', 'MLovx7CTowk', 'agHCZ_qMs_o', 'QsP-n54qBEU', 'oJVkME2cy6E', '3mTxY70OQxA', 'Z7yjQcCdge4', '-aRBKwZJ7CE', '1DuIofdpAEM', 'G3V3e-oYbqY', 'WehQ4yTNz5g']



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

    ['KIycMXs2O2I', 'HcGDD4DH6mU', 'sdYmRp2K19g', 'BBVVHzVbZYY', 'dVcq6-GZ5ls', '1fsKvW8fcDw', 'djT5HHcTk10', 'SC5H4vVwW2M', '5DaVV221-yw', 'V7n1Kt88UEU', 'xyaDBM0ToXQ', '94yxIq9hUuM', 'bK21nG7df0M', '5EK9zz8DfbI', '2sGbt0w2IsU', '4VMfMu3GHOw', '5IYeyZ2nTsA', 'Rii0qZte1IU', '8iqSLtJ8t2o', 'R_xrZRUeaqk', 'TH_ACRZTMMo', 'CKVsyQMblFg', 'jvQluLtn7fY', 'j1cp2OLTS5w', 'x_DDQEVt3to', 'UC5SjkM-K-Q', 'AQ0rYI-3P-E', 'zEIQI1uE-s8', 'UtbA5pMc8jE', 'oLueBjkSObE', 'X1DL53cVJd0', '1aeppdXQwT4', 'y2yveI8a_Pk', 'K2muzjEJG-k', 'cNigZxNFCok', '8mat-AzlKIM', 'CcKyYlISZlg', 'gFM2Evm_z34', 'AwZFYeQezKk', 'Ji1mD2V_2RA', 'kv756wOiKak', 'WVkmKDpSxdg', '4k1iYKpUaKU', 'OVrtzg2nnEM', 'hnvxqu4BGTE', '1q1jqbJM2v4', 'oVsMpkjDDhI', 'mTm4MGHH1N8', 'WpXQTZSZv-M', 'UlsNRy5W3ME', 'b5P3xyCZ3Ws', 'BQPZ3sNUd64', 'clSBnWW1zHk', 'ES9PFJRcni0']



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




    {'video_id': 'sdYmRp2K19g',
     'comment_id': 'UgznP5NfZXlGWRwbSsl4AaABAg',
     'text': 'Price gouging is the major factor in inflation.  they blame things on gas price, so they increase their products&#39; prices.  but when gas prices went down, did they ever reduce their products prices?  No.  This is true, especially for food items.',
     'published_at': '2024-01-29T19:06:36Z',
     'likeCount': 218}



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


```python
fox_merged_df = pd.merge(df_comments_fox, df_fox_video, left_on='video_id', right_on='id')
fox_merged_df.head()
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    /tmp/ipykernel_1374/823332505.py in ?()
    ----> 1 fox_merged_df = pd.merge(df_comments_fox, df_fox_video, left_on='video_id', right_on='id')
          2 fox_merged_df.head()


    /opt/conda/lib/python3.10/site-packages/pandas/core/reshape/merge.py in ?(left, right, how, on, left_on, right_on, left_index, right_index, sort, suffixes, copy, indicator, validate)
        165             validate=validate,
        166             copy=copy,
        167         )
        168     else:
    --> 169         op = _MergeOperation(
        170             left_df,
        171             right_df,
        172             how=how,


    /opt/conda/lib/python3.10/site-packages/pandas/core/reshape/merge.py in ?(self, left, right, how, on, left_on, right_on, left_index, right_index, sort, suffixes, indicator, validate)
        787             self.right_join_keys,
        788             self.join_names,
        789             left_drop,
        790             right_drop,
    --> 791         ) = self._get_merge_keys()
        792 
        793         if left_drop:
        794             self.left = self.left._drop_labels_or_levels(left_drop)


    /opt/conda/lib/python3.10/site-packages/pandas/core/reshape/merge.py in ?(self)
       1265                         # Then we're either Hashable or a wrong-length arraylike,
       1266                         #  the latter of which will raise
       1267                         rk = cast(Hashable, rk)
       1268                         if rk is not None:
    -> 1269                             right_keys.append(right._get_label_or_level_values(rk))
       1270                         else:
       1271                             # work-around for merge_asof(right_index=True)
       1272                             right_keys.append(right.index._values)


    /opt/conda/lib/python3.10/site-packages/pandas/core/generic.py in ?(self, key, axis)
       1840             values = self.xs(key, axis=other_axes[0])._values
       1841         elif self._is_level_reference(key, axis=axis):
       1842             values = self.axes[axis].get_level_values(key)._values
       1843         else:
    -> 1844             raise KeyError(key)
       1845 
       1846         # Check for duplicates
       1847         if values.ndim > 1:


    KeyError: 'id'


For cnn
transfer list into data frame by using pandas


```python
df_cnn_video=pd.DataFrame(cnn_news_election_videos)
df_comments_cnn=pd.DataFrame(list_video_comments_cnn)
```


```python
cnn_merged_df = pd.merge(df_comments_cnn, df_cnn_video, left_on='video_id', right_on='id')
cnn_merged_df.head()
```

Combine the two data frame cnn_merged_df and fox_merged_df


```python
yt_comments = pd.concat([cnn_merged_df, fox_merged_df])
yt_comments.head()
```

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
APP_PASSWORD = "abcd-7m32-v2xg-7ob7"
```

Create a `Client` object and login using credentials


```python
client = Client()
client.login(USERNAME, APP_PASSWORD)
```

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


```python
len(WP_reply_info)
```


```python
WP_reply_info[0]
```

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


```python
len(NYT_reply_info)
```


```python
NYT_reply_info[0]
```

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

For New York Times


```python
df_NYT_posts=pd.DataFrame(NYT_relevant_posts)
df_NYT_reply=pd.DataFrame(NYT_reply_info)
NYT_merged_df = pd.merge(df_NYT_posts, df_NYT_reply, left_on='uri', right_on='post_id')
NYT_merged_df.head()
```

## Make and save the csv file
1. Turn the data frame into a CSV
2. save it in a file called “bsky_replies.csv”


```python
yt_comments = pd.concat([WP_merged_df, NYT_merged_df])
yt_comments.head()
```


```python
yt_comments.to_csv('bsky_replies.csv', index=False)
```


```python

```
