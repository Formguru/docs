## Errors

The Guru API uses the following error globally across all endpoints:


Error Code | Meaning
---------- | -------
400 | Bad Request -- The request did not contain the required fields, or some were invalid. Consult the endpoint's documentation.
401 | Unauthorized -- The request could not be authenticated. Ensure you are [specifying a valid authentication token](#authentication).
404 | Not Found -- The resource requested no longer exists. If fetching an analysis then the video may have been deleted.
405 | Method Not Allowed -- The endpoint you called does not support the HTTP method specified.
406 | Not Acceptable -- Requested an unsupported format. All endpoints currently serve JSON.
429 | Too Many Requests -- Your service is making too many calls and has been throttled. Please wait before retrying and lower your call volume.
500 | Internal Server Error -- An unexpected server-side error. Please reach out to Guru for resolution.

<aside class="notice">
If an endpoint varies its error codes then that will be noted explicitly in that endpoint's documentation. 
</aside>
