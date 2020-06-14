## Simple upload to YouTube example
A barebones script to upload to YouTube with Python

## Setup
    pip install --upgrade google-api-python-client oauth2client pickle
Download the script

* Go to the [Google console](https://console.developers.google.com/).
* Create project.
* Side menu: APIs & auth -> APIs
* Top menu: Enabled API(s): Enable all Youtube APIs.
* Side menu: APIs & auth -> Credentials.
* Create a Client ID: Add credentials -> OAuth 2.0 Client ID -> Other -> Name: youtube-upload -> Create -> OK
* Download JSON: Under the section "OAuth 2.0 client IDs".
* Save the file in the same directory as the script.

## Code breakdown
Code explanation

#### Authentication
```python
# Auth
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
```

#### Connect to the API
```python
youtube = build("youtube", "v3", credentials = creds).videos()
```

#### Set video options
[Category list](https://gist.github.com/dgp/1b24bf2961521bd75d6c)
```python
body = {
    "snippet": {
        "title": "Test",
        "description": "Testing",
        "tags": None,
        "categoryId": "20" # Gaming
    },
    "status": {
        "privacyStatus": "unlisted"
    }
}
```

#### Sending the first request
```python
# Call the API"s videos.insert method to create and upload the video.
request = youtube.insert(
    part="snippet, status",
    body=body,
    media_body=MediaFileUpload("temp/h81woe.mp4", resumable=True, chunksize=-1)
)
```

#### Handling the upload
```python
response = None
while response is None:
    print("Uploading video...")
    response = request.next_chunk()
    if "id" in response:
        print("Successfully uploaded.")
        print("https://www.youtube.com/watch?v=" + response["id"])
    elif response is not None:
        print("The upload failed with an unexpected response:", response)
```

## Full code:
```python
import random
import time

# import socket

import pickle
# import apiclient.http
# import http.client
# import httplib2

# import googleapiclient.errors
# from googleapiclient.errors import HttpError
import google.oauth2.credentials
# import google_auth_oauthlib.flow
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
        flow = InstalledAppFlow.from_client_secrets_file("client_secret.json", ["https://www.googleapis.com/auth/youtube.upload"])
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
    media_body=MediaFileUpload("temp/h81woe.mp4", resumable=True, chunksize=-1)
)

response = None
while response is None:
    print("Uploading video...")
    response = request.next_chunk()
    if "id" in response:
        print("Successfully uploaded.")
        print("https://www.youtube.com/watch?v=" + response["id"])
    elif response is not None:
        print("The upload failed with an unexpected response:", response)
```
