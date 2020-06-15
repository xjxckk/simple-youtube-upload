## Simple upload to YouTube example
A barebones script to upload to YouTube with Python

## Setup
    pip install --upgrade google-api-python-client oauth2client pickle
Download the script

* Go to the [Google console](https://console.developers.google.com/).
* Create a project (this doesn't need to be on the same Google account as your YouTube channel)
* Side menu: APIs & Auth -> APIs
* Top menu: Enabled API(s): Enable all Youtube APIs
* Side menu: APIs & Auth -> Credentials
* Create a Client ID: Add credentials -> OAuth 2.0 Client ID -> Other -> Name: youtube-upload -> Create -> OK
* Download JSON: Under the section "OAuth 2.0 client IDs"
* Save the file in the same directory as the script

## Code breakdown
This will guide you through the code to help you understand it.

#### Authentication
```python
with open("token", "rb") as token:
    creds = pickle.load(token)
```
This opens/creates a file "token" which stores your authentication credentials.

```python
if not creds or not creds.valid:
```
Checks if the credentials are in the file and are valid.
```python
    if creds and creds.expired and creds.refresh_token:
        creds.refresh(Request())
```
If the credentials exist and are expired refresh them.
```python
    else:
        flow = InstalledAppFlow.from_client_secrets_file(
            "client_secret.json",
            ["https://www.googleapis.com/auth/youtube.upload"]
            )
        creds = flow.run_console()
```
This will run when you run the script for your first time.

Connect to authentication flow and run_console().

This will give you a link to open in a browser and connect your YouTube account.

It will give you an authentication code to enter.
```python
    with open("token", "wb") as token:
        pickle.dump(creds, token)
```
Saves your credentials to "token" file for next time.

#### Connect to the API
```python
youtube = build("youtube", "v3", credentials=creds).videos()
```
Creates an instance with the YouTube v3 Videos API.

#### Set video options
```python
body = {
    "snippet": {
        "title": "Test",
        "description": "Testing",
        "tags": None,
        "categoryId": "20"
    },
    "status": {
        "privacyStatus": "unlisted"
    }
}
```
Create a dictionary in the required format for the request.

[Category ID list](https://gist.github.com/dgp/1b24bf2961521bd75d6c)

#### Sending the first request
```python
request = youtube.insert(
    part="snippet, status",
    body=body,
    media_body=MediaFileUpload("sample.mp4", chunksize=-1)
)
```
Call the insert method on the YouTube API instance.

part indicates the items that are included in your previous body request.

body is your request body.

media_body is your file and options (chunksize=-1 means it will upload your file in one continuous go).

#### Handling the upload
```python
response = None
while response is None:
    print("Uploading video...")
    response = request.next_chunk()
    if "id" in response:
        print("Successfully uploaded.")
        print("https://www.youtube.com/watch?v=" + response["id"])
```
Sends your video in a chunk and waits for the response.

If the video ID is in the response the video is succesfully uploaded.

## Full code:
```python
import random
import time
import pickle
import google.oauth2.credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from google.auth.transport.requests import Request
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload

with open("token", "rb") as token:
    creds = pickle.load(token)
    # If there are no (valid) credentials available, let the user log in.
if not creds or not creds.valid:
    if creds and creds.expired and creds.refresh_token:
        creds.refresh(Request())
    else:
        flow = InstalledAppFlow.from_client_secrets_file(
            "client_secret.json",
            ["https://www.googleapis.com/auth/youtube.upload"]
            )
        creds = flow.run_console()
    with open("token", "wb") as token:
        pickle.dump(creds, token)

youtube = build("youtube", "v3", credentials = creds).videos()

body = {
    "snippet": {
        "title":"Test",
        "description": "Testing",
        "tags": None,
        "categoryId": "20" # Gaming
    },
    "status": {
        "privacyStatus": "unlisted"
    }
}

# Call the API"s videos.insert method to create and upload the video.
request = youtube.insert(
    part="snippet, status",
    body=body,
    media_body=MediaFileUpload("sample.mp4", resumable=True, chunksize=-1)
)

response = None
while response is None:
    print("Uploading video...")
    response = request.next_chunk()
    if "id" in response:
        print("Successfully uploaded.")
        print("https://www.youtube.com/watch?v=" + response["id"])
```

## Further reading
https://developers.google.com/youtube/v3/guides/uploading_a_video
