---
title: Guru API

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript

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

Authentication with the Guru API occurs using Bearer tokens. You must include your authentication token as a header on each request you make:

`Authorization: Bearer <token>`

```javascript
axios({
    method: 'post',
    url: 'https://guru-prod.us.auth0.com/oauth/token',
    headers: {
        'content-type': 'application/x-www-form-urlencoded'
    }, 
    data: {
        client_id: 'YOUR_CLIENT_ID',
        client_secret: 'YOUR_CLIENT_SECRET',
        audience: 'https://api.formguru.fitness/',
        grant_type: 'client_credentials'
    }
}).then(function (response) {
    const authToken = response.data.access_token;

    //...
}).catch(function (error) {
    //...
});
```

Your service will obtain its authentication token using an OAuth Client-Credential Flow.
If you are a newly-integrating service then you will need to [reach out to Guru](mailto:support@getguru.fitness) to have your client credentials created.
Once you have received these credentials, you will write code that exchanges them for an authentication token when you want to make a request to our API.
The flow will be:

1. Exchange your client ID and secret with `https://guru-prod.us.auth0.com` for an access token.
1. Include the access token on each request to the Guru API.

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
        Authorization: 'Bearer ' + token
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
    formData.append("file", video);
    
    axios.post(
        response.data.url, 
        formData, 
        {
            headers: { "Content-Type": "multipart/form-data" }
        }
    ).catch(function (error) {
        //...
    });
}).catch(function (error) {
    //...
});
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
 weightlifting | squat, deadlift, bench_press
 calisthenics | push_up

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
        Authorization: 'Bearer ' + token
    }
}).then(function (response) {
    //...
});
```

`GET https://api.getguru.fitness/videos/{id}` 

### Request

Parameter | Required | Default | Description
--------- | ------- | ------- | -----------
id | Yes | None | The ID of the video you wish to fetch data for.

### Response
The response is JSON and contains the following data:

Field | Description
--------- | -----------
status | Indicating where analysis was successfully performed. Possible values are: `Pending` (if they video has not been uploaded yet), or `Success`.
uri | The location from which the raw video can be downloaded.

## Update Video

```javascript
axios({
    method: 'put',
    url: 'https://api.getguru.fitness/videos/' + videoId,
    headers: {
        Authorization: 'Bearer ' + token
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
        Authorization: 'Bearer ' + token
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
domain | The category of exercise being performed in the video.
activity | The movement being performed in the video.
reps | An array of objects, each one an individual rep detected in the video. Each rep indicates the timestamp offsets within the video where it can be found (via `startTimestampMs`, `midTimestampMs`, and `endTimestampMs`). It also specifies an `analyses` array of objects, which contains the individual insights generated by the analysis.
