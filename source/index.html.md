---
title: Guru API

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript
  - typescript
  - python

toc_footers:
  #- <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Guru API
---

# Introduction

Welcome to the Guru API. This API allows you to upload and perform analysis on your workouts and exercise.
See below to find out how to authenticate your calls and start working with the API.

If you would like to integrate and require assistance then please [contact us](mailto:support@getguru.fitness).

# Authentication

Authentication with the Guru API occurs using OAuth tokens. You must include your authentication token as a header on each request you make:

`Authorization: <token>`

```javascript
var request = require("request");

var options = { method: 'POST',
  url: 'https://customer-console-prod.auth.us-west-2.amazoncognito.com/oauth2/token',
  headers: { 'content-type': 'application/x-www-form-urlencoded' },
  body: 'grant_type=client_credentials&client_id=' + client_id + '&client_secret=' + client_secret + '&scope=https://api.getguru.fitness/default' };

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});
```

Your service will obtain its authentication token using an OAuth Client-Credential Flow.
If you are a newly-integrating service then you will need to create an account with Guru via the [Console](https://console.getguru.fitness) to access your credentials.
Please see the [Getting Started with Guru](https://blog.getguru.fitness/2022/03/24/getting-started-with-guru.html) guide for more details on account creation.

Once you have your access credentials, the authentication flow will be:

1. Exchange your client ID and secret with `https://customer-console-prod.auth.us-west-2.amazoncognito.com` for an access token. The value of the `expires_in` field in the response is the number of seconds until this token expires.
1. Store the token along with its expiration date in persistent storage so that it can be re-used on each call. It is important to not request new tokens on each call as your application will be rate limited.
1. Before making a call to the Guru API, check if the token is expired and, if so, refresh it.
1. Make the call to the Guru API using the access token.

See the example on this page for working code to perform the credential exchange.
See [here](https://auth0.com/docs/get-started/authentication-and-authorization-flow/client-credentials-flow) for more details on implementing the Client-Credential flow.

# Videos

Uploading a video for analysis is a three-step process:

1. Call the [Create API](#create-video) to specify the video's metadata. This will tell Guru some basic information about the video such as its size, and also include optional additional information such as the activity being performed in the video. This information helps deliver more accurate analysis results. The API will return a URL that specifies where the video should be uploaded to.
1. Upload the video content to the URL returned in step 1. The video will be encoded as `multipart/form-data` in the request.
1. Poll the [Analysis API](#get-analysis) until the video is ready. It will typically take 30 seconds for analysis to complete, though longer wait times may be experienced for larger videos.

See below for details on each individual API call.

## Create Video

```javascript
axios({
    method: 'post',
    url: 'https://api.getguru.fitness/videos',
    headers: {
        Authorization: token
    },
    data: {
        filename: 'workout.mp4',
        size: <<video-size-in-bytes>>,
        domain: 'weightlifting',
        activity: 'squat',
        repCount: 12,
        source: 'my-service'
    }
}).then(function (response) {
    const formData = new FormData();
    Object.keys(response.data.fields).forEach((key) => {
        formData.append(key, response.data.fields[key]);
    });
    formData.append("file", video);

    axios.post(
        response.data.url,
        formData,
        {
            headers: { "Content-Type": "multipart/form-data", "Content-Length": <<video-size-in-bytes>> }
        }
    ).catch(function (error) {
        //...
    });
}).catch(function (error) {
    //...
});
```

```typescript
//assuming variable 'file' of type Express.Multer.File
axios({
  method: 'post',
  url: 'https://api.getguru.fitness/videos',
  headers: {
    Authorization: token
  },
  data: {
    filename: 'workout.mp4',
    size: 1234,
    domain: 'weightlifting',
    activity: 'squat',
    repCount: 12,
    source: 'my-service'
  }
}).then(function (response) {
  const formData = new FormData();
  Object.keys(response.data.fields).forEach((key) => {
    formData.append(key, response.data.fields[key]);
  });
  formData.append("file",  file.buffer , file.originalname);
  let headers = formData.getHeaders()
  formData.getLength(function(err,length){
    headers["content-length"] = length
    axios.post(
      response.data.url,
      formData,
      {
        headers: headers
      }
    ).then(function (response) {
      //...
    }).catch(function (error) {
      //...
    });
  });
}).catch(function (error) {
  //...
});
```

```python
import os
import requests


def create(video_path, access_token, domain, activity, rep_count = 3):
    return requests.post(
        "https://api.getguru.fitness/videos",
        json = {
            "filename": os.path.basename(video_path),
            "size": os.path.getsize(video_path),
            "domain": domain,
            "activity": activity,
            "repCount": rep_count,
        },
        headers = {
            "Content-Type": "application/json",
            "Authorization": access_token
        }
    )


def upload(video_path, create_response):
    json = create_response.json()
    url = json["url"]
    fields = json["fields"]

    with open(video_path, "rb") as file:
        return requests.post(
            url,
            data=fields,
            files={"file": file},
        )

# access_token = ...
video_path = "path/to/video.mp4"
create_response = create(video_path, access_token, "weightlifting", "squat", 1)
upload_response = upload(video_path, create_response)

```

`POST https://api.getguru.fitness/videos`

### Request

Parameter | Required | Default | Description
--------- | ------- | ------- | -----------
filename | Yes | None | The name of the video file, including extension.
size | Yes | None | The size of the video file, in bytes.
source | No | None | The source of the video. If the video was captured by your service then enter your service's name for this field.
domain | No | None | The category of exercise being performed in the video. See the table below for accepted values.
activity | No | None | The movement being performed in the video. See the table below for accepted values.
repCount | No | None | The number of reps expected to be performed in the video. Omit if unknown.

The currently accepted values for `domain` and `activity` are:

 Domain | Activity
 ------ | ------- |
 weightlifting | bench_press, clean_and_jerk, deadlift, snatch, squat  
 calisthenics | bodyweight_squat, burpee, chin_up, lunge, push_up, sit_up
 martial_arts | punch, front_kick
 mobility | knee_to_chest
 running | sprint
 yoga | downward_dog

### Response
The response is JSON and contains the following data:

Field | Description
--------- | -----------
id | Unique identifier for your video. You will use it to make calls to the API to fetch results or perform other operations on the video.
url | Location to which your video content will be uploaded. This upload must be `multipart/form-data` encoded.
fields | The signing fields which must be included in your form when you upload the video. Take a look at the example to see how to combine these fields with your video content.

## Get Video

```javascript
axios({
    url: 'https://api.getguru.fitness/videos/' + videoId,
    headers: {
        Authorization: token
    }
}).then(function (response) {
    //...
});
```

`GET https://api.getguru.fitness/videos/{id}?include=j2p,analysis`

### Request

Parameter | Required | Default | Description
--------- | ------- | ------- | -----------
id | Yes | None | The ID of the video you wish to fetch data for.
include | No | None | A comma-separated list of additional fields you wish to return. Accepted values are `j2p`, `analysis`, and `objects`. 

### Response
The response is JSON and contains the following data:

Field | Description
--------- | -----------
status | Indicating whether the video has been uploaded to Guru. Possible values are: `Pending` (if the video has not been uploaded yet), `Success`, or `Failed`.
reason | The reason that the analysis failed. Only present when status is `Failed`
uri | The location from which the raw video can be downloaded.
overlays | Contains information about the overlays (e.g. wireframes) Guru has built for this video. The object will map the type of overlay to an object that has a `status` field. If the overlay has been built then it will also contain a `uri` field that is a link to download the overlayed video.
fps | The frame rate (in frames per second) of the uploaded video
analysis | Only present if specified in `include`. See the [Get Analysis](#get-analysis) endpoint for the structure of this object. Contains a `status` field to indicate whether processing has completed.
j2p | Only present if specified in `include`. See the [Get Joint Data](#get-joint-data) endpoint for the structure of this object. Contains a `status` field to indicate whether processing has completed.
objects | Only present if specified in `include`. Contains an array of each object detected in the video. Each object will contain an array of `boundingBoxes`, showing the location of that object at particular frames in the video.

The currently supported overlay types are:

Type | Description
--------- | -----------
skeleton | Contains a wireframe drawing of the joints and major landmarks identified on the person.
all | Contains all supported overlay elements, including wireframes, rep counting, and analytics about the movement.

## Update Video

```javascript
axios({
    method: 'put',
    url: 'https://api.getguru.fitness/videos/' + videoId,
    headers: {
        Authorization: token
    },
    data: {
        repCount: 10,
    }
}).then(function (response) {
    //...
});
```

`PUT https://api.getguru.fitness/videos/{id}`

### Request

The request payload should be in a JSON-encoded body. All of the fields are optional.
If a field is omitted, then the existing value will be preserved.

Parameter | Required | Default | Description
--------- | ------- | ------- | -----------
repCount | No | None | The number of reps that were performed in the video.
domain | No | None | The category of exercise being performed in the video. See the table in Create Video for accepted values.
activity | No | None | The movement being performed in the video. See the table in Create Video for accepted values.

### Response
The response is JSON and contains the ID of the video.

## Get Analysis

```javascript
axios({
    url: 'https://api.getguru.fitness/videos/' + videoId + '/analysis',
    headers: {
        Authorization: token
    }
}).then(function (response) {
    //...
}).catch(function (error) {
    //...
});
```

> A successful response would look something like this:

```json
{
    "status": "Complete",
    "domain": "weightlifting",
    "activity": "squat",
    "reps": [
        {
            "startTimestampMs": 123,
            "midTimestampMs": 456,
            "endTimestampMs": 789,
            "analyses": [
                {
                    "analysisType": "HIP_KNEE_ANGLE_DEGREES",
                    "analysisScalar": 12.34
                }
            ]
        }
    ]
}
```

`GET https://api.getguru.fitness/videos/{id}/analysis`

### Request

Parameter | Required | Default | Description
--------- | ------- | ------- | -----------
id | Yes | None | The ID of the video you wish to fetch analysis for.

### Response
The response is JSON and contains the following data:

Field | Description
--------- | -----------
status | Indicating where analysis was successfully performed. Possible values are: `Pending`, `Complete`, or `Failed`.
reason | The reason that the analysis failed. Only present when status is `Failed`
domain | The category of exercise being performed in the video.
activity | The movement being performed in the video.
reps | An array of objects, each one an individual rep detected in the video. Each rep indicates the timestamp offsets within the video where it can be found (via `startTimestampMs`, `midTimestampMs`, and `endTimestampMs`). It also specifies an `analyses` array of objects, which contains the individual insights generated by the analysis.

Some of the analyses on reps will include an `analysisOpinion` field. 
This is the opinion of the Guru platform on the quality of the rep on this particular metric.
For example, for the hip/knee angle analysis of a squat, 
Guru will have a better opinion of a user who sits lower in a squat as opposed to
one who sits higher. Possible values for this field are `good` or `bad`.

When set, `reason` will be one of the following values:

Value | Description
------ | ----------
`LOW_QUALITY_POSE_ESTIMATE` | Guru couldn't confidently detect the body's position throughout the video

### Details - Sprints

For sprints, the analysis field contains some additional fields:

Field | Description
----- | -----------
fieldMarkers | Only present if the runner is on an American football field with 5-yard markers or if 10 yards have been marked with start and end cones. See below for a description of the values.
reps | A list containing one entry for each stride. A "stride" begins when the toe leaves the ground and ends when the same foot contacts the ground. See below for details on the fields in each rep.
runnerProgress | A list which represents a time-series of the distance the runner has traveled. Each entry in the list contains three fields: `distanceFromStart` (in meters), `frameIndex` and `timestamp` (in milliseconds). Note that `distanceFromStart` is negative when the runner hasn't yet crossed the starting line (e.g., if the runner is 1 meter _behind_ the starting line 1 second into the video, then `distanceFromStart=-1` @ `timestamp=1000`)

#### Field Markers

The `fieldMarkers` object has 4 possible keys: `startLine`, `middleLine`,
`finishLine`, and `cones`. The start, middle, and finish lines correspond to 0,
5, and 10 yards on an American football field. The start line is the line that
the runner crosses first and the end line is the line that they cross last.

Field | Description
----- | -----------
type | Either `YARD_LINE` or `CONE`
position | Contains fields `x1`, `y1`, `x2`, and `y2`. If `YARD_LINE`, these fields represent the unnormalized coordinates of the line segment. If `CONE`, these fields represent a bounding-box that circumscribes the cone.
frame_idx | The frame index of the video corresponding to the detection timestamp: The timestamp (in milliseconds) corresponding to the detection

#### Reps

Each entry in `reps` corresponds to a stride and contains the following fields:

Field | Description
--------- | -----------
startTimestampMs | The timestamp at which the toe leaves the ground
midTimestampMs | The timestamp corresponding to the middle of the stride (defined as peak flexion at the hip)
endTimestampMs | The timestamp at which the foot touches back down onto the ground
analyses | A list of objects containing two fields: `analysisType` and `anaylsisScalar`. See below for details.

Each entry in the `analyses` list contains these fields:

Analysis Type | Description
--------- | -----------
IS_LEFT_LEG | 1 if the left leg is the leg swinging forward in this stride, 0 if it's the right leg
STRIDE_AIR_TIME_MS | The time that the runner is in the air during this stride, equal to the duration between toe-off (`startTimestampMs` of this stride) and the touch-down of the opposite foot (`endTimestampMs` of the previous stride). This will be `null` for the first stride in the list since it isn't well defined without a previous stride.
STRIDE_GROUND_TIME_MS | The time that the runner is on the ground until the next stride begins. Ground time begins at the same instant that the air time ends, at touch-down of the opposite foot. This will also be `null` for the first stride in the list.
STRIDE_PEAK_FLEXION_ANGLE | The peak flexion angle of the leg during the stride
STRIDE_PEAK_FLEXION_ANGLE_TIMESTAMP | The timestamp at which the leg reaches the peak flexion angle
STRIDE_PEAK_EXTENSION_ANGLE | The peak extension angle of the leg during the stride
STRIDE_PEAK_EXTENSION_ANGLE_TIMESTAMP | The timestamp at which the leg reaches the peak extension angle


## Get Joint Data

```javascript
axios({
    url: 'https://api.getguru.fitness/videos/' + videoId + '/j2p',
    headers: {
        Authorization: token
    }
}).then(function (response) {
    //...
}).catch(function (error) {
    //...
});
```

> A successful response would look something like this:

```json
{
  "status": "Complete",
  "resolutionHeight": 1280,
  "resolutionWidth": 720,
  "jointToPoints": {
    "leftAnkle": [{
        "frame_idx": 0,
        "part": "leftAnkle",
        "position": {
          "x": 0.4,
          "y": 0.7
        },
        "score": 0.8,
        "timestamp": 0
      },
      {
        "frame_idx": 4,
        "part": "leftAnkle",
        "position": {
          "x": 0.5,
          "y": 0.75
        },
        "score": 0.86,
        "timestamp": 0.1
      }
    ],
    "leftElbow": [...],
    "leftEye": [...],
    "leftHand": [....],
    "leftHip": [...],
    "leftKnee": [...],
    "leftShoulder": [...],
    "leftWrist": [...],
    "rightElbow": [...],
    "rightEye": [...],
    "rightHand": [....],
    "rightHip": [...],
    "rightKnee": [...],
    "rightShoulder": [...],
    "rightWrist": [...]
  },
  "analysis": {
    "reps": [{
      "startTimestampMs": 123,
      "midTimestampMs": 456,
      "endTimestampMs": 789,
      "analyses": [{
        "analysisType": "HIP_KNEE_ANGLE_DEGREES",
        "analysisScalar": 12.34
      }]
    }]
  }
}
```

`GET https://api.getguru.fitness/videos/{id}/j2p`

### Request

Parameter | Required | Default | Description
--------- | ------- | ------- | -----------
id | Yes | None | The ID of the video you wish to fetch joint-to-point (j2p) for.

### Response
The response is JSON and contains the following data:

Field | Description
--------- | -----------
status | Indicating where analysis was successfully performed. Possible values are: `Pending`, `Complete`, or `Failed`.
reason | The reason that the analysis failed. Only present when status is `Failed`
resolutionHeight | The height of the video.
resolutionWidth | The width of the video.
analysis | This contains the rep information equivalent to that returned from the `Get Analysis` endpoint.
jointToPoints | An object containing one attribute for each joint being tracked. Each joint is an array of `JointFrame` objects, detailed below.

The semantics of the `reason` field are identical to those of the [the
Analysis endpoint](#get-analysis).

`JointFrame` objects define the information for a single joint for a single frame within the video.
They have the following structure:

Field | Description
--------- | -----------
frame_idx | The index of the frame within the video.
part | The name of the joint.
position | An object containing an `x` and `y`, the 2D location of that joint within the video, relative to the resolution of the video. Each coordinate will be >= 0 and <= 1.
score | The confidence the model has in its prediction of this joint location within the frame. Value is >= 0 and <= 1.
timestamp | The timestamp of this frame within the video, measured in seconds.
