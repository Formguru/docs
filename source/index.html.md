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

If you would like to integrate and require assistance then please [contact us](https://www.formguru.fitness/contact).

# Authentication

Authentication with the Guru API occurs using Bearer tokens. Include your authentication token as a request header:

`Authorization: Bearer <token>`

# Videos

Uploading a video for analysis is a three-step process:

1. Call the [Create API](#create-video) to specify the video's metadata. This will tell Guru some basic information about the video such as its size, and also include optional additional information such as the activity being performed in the video. This information helps deliver more accurate analysis results. The API will return a URL that specifies where the video should be uploaded to.
1. Upload the video content to the URL returned in step 1. The video will be encoded as `multipart/form-data` in the request.
1. Poll the Analysis API until the video is ready. It will typically take 30 seconds for analysis to complete, though longer wait times may be experienced for larger videos.

See below for details on each individual API call.

## Create Video

```javascript
axios({
    method: 'post',
    url: 'https://api.formguru.fitness/videos',
    headers: {
        Authorization: 'Bearer ' + token
    }, 
    data: {
        filename: 'workout.mp4',
        size: 1234,
        domain: 'weightlifting',
        activity: 'squat',
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

`POST https://api.formguru.fitness/videos`

### Request

Parameter | Required | Default | Description
--------- | ------- | ------- | -----------
filename | Yes | None | The name of the video file, including extension.
size | Yes | None | The size of the video file, in bytes.
source | No | None | The source of the video. If the video was captured by your service then enter your service's name for this field.
domain | No | None | The category of exercise being performed in the video. See the table below for accepted values.
activity | No | None | The movement being performed in the video. See the table below for accepted values.

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
