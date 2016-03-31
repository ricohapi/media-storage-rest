## Media

All API endpoints are listed below.

| Method | Description |
|--------|-------------|
| [GET  /media](#GetMediaList) | Fetching the index of all uploaded media data |
| [POST /media](#Upload) | Uploading a media data |
| [POST /media (multipart)](#UploadMulti) | Uploading a media data |
| [DELETE /media/{id}](#DeleteMedia) | Deleting a media data |
| [GET /media/{id}](#GetMediaInfo) | Fetching information of a media data |
| [GET /media/{id}/content](#GetMedia) | Downloading a media data |
| [GET /media/{id}/meta](#GetMediaMeta) | Fetching metadata (Exif, Photo Sphere XMP etc.) of a media data |

* The default encoding is UTF-8, 4 bytes or fewer per each character.
* The maximum request body size is 1MB, except for `POST /media` call.

<a name="GetMediaList"></a>
## GET /media

Fetching the index of all uploaded media data.

The index is ordered by the uploaded datetime.

### URL Structure
```
https://mss.ricohapi.com/v1/media
```

### Example request
```
curl --request GET 'https://mss.ricohapi.com/v1/media \
    ?after=71234bad-234a-4537-8e11-54c82e234584&limit=100' \
    --header 'Authorization: Bearer <access token>'
```

### Parameters

| Parameter | Description | Parameter Type | Data Type |
|:----:|----|:----:|:----:|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| after <br> (optional)| Media ID<br>Response will include a list of media data `after` the specified id, including the media data which the id points to.<br>If both `before` and `after` are specified, `after` has priority. | query | string |
| before <br> (optional)| Media ID<br>Response will include a list of media data `before` the specified id, excluding the media data which the id points to.<br>If both `before` and `after` are specified, `after` has priority. | query | string |
| limit <br> (optional)| Number of media data returned<br>The default is 10, and maximum is 100. | query | string |

### Response

```JavaScript
{
  "media": [
    {
      "id": "string"
    }
  ],
  "paging": {
    "next": "string",
    "previous": "string"
  }
}
```

###  Example response
```JavaScript
{
  "media":[
    {
      "id":"71234bad-234a-4537-8e11-54c82e232345"
    },
          .
          .
          .
  ],
  "paging":{
    "next":"https://mss.ricohapi.com/v1/media/?after=71234bad-234a-4537-8e11-54c82e567345\u0026limit=10",
    "previous":"https://mss.ricohapi.com/v1/media?before=71234bad-234a-4537-8e11-521c2e232345\u0026limit=10"
  }
}
```
| Header | Description |
|----|-------------|
| Content-Type | application/json; charset=utf-8 |

| Key | Type | Description |
|----|:----:|-------------|
| media/id | string | Media ID |
| paging/next | string | The URI of the next page. <br> If no next page exists, this key won't be included. |
| paging/previous | string | The URI of the previous page. <br> If no previous page exists, this key won't be included. |

### Status Codes
| Code | Reason |
|------|--------|
| 200 | OK |
| 400 | Bad request. |
| 401 | Unauthorized. The access token is invalid or expired. |
| 403 | Forbidden. |
| 404 | Not found. |
| 429 | Too Many Requests. |
| 500 | Internal server error. |


<a name="Upload"></a>
## POST /media

Send the uploading media data as the request payload.

### URL Structure
```
https://mss.ricohapi.com/v1/media
```

### Example request
```
curl --request POST 'https://mss.ricohapi.com/v1/media' \
    --header 'Authorization: Bearer <access token>' \
    --header 'Content-Type: image/jpeg' \
    --header 'Content-Length: 5996544' \
    --upload-file 'sample.jpg'
```

### Parameters

| Parameter | Description | Parameter Type | Data Type |
|------|----|:----:|------|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| Content-Type | Content type of the media data <br> If the content type is not supported, 415 error will be returned. | header | string |
| Content-Length | Content size of the media data <br> If not specified, 411 error will be returned. | header | string |
| | The transfered media data. | body | file |

#### Supported Content Types

The maximum payload size is defined per each conten type.

| Content-Type | Limit | File Ext. |
|------|:----:|------|
| image/jpeg | 30MB | jpg |

### Response

```JavaScript
{
　　"id":"string",
　　"content_type":"string",
　　"bytes": 3944775,
　　"created_at":"2016-02-24T00:56:46Z"
}
```

| Header | Description |
|----|-------------|
| Content-Type | application/json; charset=utf-8 |
| Location | The URI of the generated media data |

| Key | Type | Description |
|------|:----:|-------------|
| id | string | Media ID |
| content_type | string | Media Content Type |
| bytes | integer | bytes |
| created_at | string | Created Datetime <br> ISO8601-formatted with timezone offset |

### Status Codes
| Code | Reason |
|------|--------|
| 201 | Resource created. |
| 400 | Bad request. |
| 401 | Unauthorized. The access token is invalid or expired. |
| 403 | Forbidden. |
| 404 | Not found. |
| 411	| Length Required. Content-Length header is missing. |
| 413 | Payload Too Large. Exceeded the content size limit. |
| 415 | Unsupported Media Type. The Content-Type is not supported. |
| 429 | Too Many Requests. |
| 500 | Internal server error. |


<a name="UploadMulti"></a>
## POST /media (multipart)

Send the uploading media data as the request payload.

### URL Structure
```
https://mss.ricohapi.com/v1/media
```

### Example request
```
curl --request POST 'https://mss.ricohapi.com/v1/media' \
    --header 'Authorization: Bearer <access token>' \
    --header 'Content-Type: multipart/form-data' \
    --header 'Content-Length: 5996544' \
    --form 'content=@sample.jpg'
```
```
POST /media HTTP/1.1
Authorization: 1:
Content-Type: multipart/form-data; boundary=---------------------------2096545099721798917139276116
Content-Length: 5996544

-----------------------------2096545099721798917139276116
Content-Disposition: form-data; name="content"; filename="sample.jpg"
Content-Type: image/jpeg

(binary)
-----------------------------2096545099721798917139276116--
```

### Parameters

| Parameter | Description | Parameter Type | Data Type |
|------|:----:|:----:|------|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| Content-Type | multipart/form-data | header | string |
| Content-Length | Content size of the media data <br> If not specified, 411 error will be returned. | header | string |
| | Transfered parameters and media data. | formData | file |

|　Chunk name | Description |
|------|----|
| content | Media Data <br> Content-Type header is mandatory. |

#### Supported Content Types

The maximum payload size is defined per each conten type.

| Content-Type | Limit | File Ext. |
|------|:----:|------|
| image/jpeg | 30MB | jpg |

### Response

```JavaScript
{
　　"id":"string",
　　"content_type":"string",
　　"bytes": 5996544,
　　"created_at":"2016-02-24T00:56:46Z"
}
```

| Header | Description |
|----|-------------|
| Content-Type | application/json; charset=utf-8 |
| Location | The URI of the generated media data |

| Key | Type | Description |
|------|:----:|-------------|
| id | string | Media ID |
| content_type | string | Media Content Type |
| bytes | integer | bytes |
| created_at | string | Created Datetime <br> ISO8601-formatted with timezone offset |

### Status Codes
| Code | Reason |
|------|--------|
| 201 | Resource created. |
| 400 | Bad request. |
| 401 | Unauthorized. The access token is invalid or expired. |
| 403 | Forbidden. |
| 404 | Not found. |
| 411	| Length Required. Content-Length header is missing. |
| 413 | Payload Too Large. Exceeded the content size limit. |
| 415 | Unsupported Media Type. The Content-Type is not supported. |
| 429 | Too Many Requests. |
| 500 | Internal server error. |


<a name="DeleteMedia"></a>
## DELETE /media/{id}

Delete the specified meda data.<br>
Once deleted files can not be restored.

### URL Structure
```
https://mss.ricohapi.com/v1/media/03das393-a8f6-4959-80ec-23039485304d
```

### Example request
```
curl --request DELETE 'https://mss.ricohapi.com/v1/media/03das393-a8f6-4959-80ec-23039485304d' \
    --header 'Authorization: Bearer <access token>'
```

### Parameters

| Parameter | Description | Parameter Type | Data Type |
|------|:----:|:----:|------|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| id | Media ID | path | string |

### Response

### Status Codes
| Code | Reason |
|------|--------|
| 204 | No content. |
| 400 | Bad request. |
| 401 | Unauthorized. The access token is invalid or expired. |
| 403 | Forbidden. |
| 404 | Not found. |
| 429 | Too Many Requests. |
| 500 | Internal server error. |


<a name="GetMediaInfo"></a>
## GET /media/{id}

Obtain the information of the media data.

### URL Structure
```
https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472
```

### Example request
```
curl --request DELETE 'https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472' \
    --header 'Authorization: Bearer <access token>'
```

### Parameters
| Parameter | Description | Parameter Type | Data Type |
|------|:----:|:----:|------|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| id | Media ID | path | string |

### Response
```JavaScript
{
　　"id":"string",
　　"content_type":"string",
　　"bytes": 3944773,
　　"created_at":"2016-02-24T00:56:46Z"
}
```
| Header | Description |
|----|-------------|
| Content-Type | application/json; charset=utf-8 |

| Key | Type | Description |
|------|:----:|-------------|
| id | string | Media ID |
| content_type | string | Media Content Type |
| bytes | integer | bytes |
| created_at | string | Created Datetime <br> ISO8601-formatted with timezone offset |

### Status Codes
| Code | Reason |
|------|--------|
| 200 | OK |
| 400 | Bad request. |
| 401 | Unauthorized. The access token is invalid or expired. |
| 403 | Forbidden. |
| 404 | Not found. |
| 429 | Too Many Requests. |
| 500 | Internal server error. |


<a name="GetMedia"></a>
## GET /media/{id}/content

Download the media data.

### URL Structure
```
https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/content
```

### Example request
```
curl --request GET 'https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/content' \
    --header 'Authorization: Bearer <access token>' \
    --output 'download.jpg'
```

### Parameters
| Parameter | Description | Parameter Type | Data Type |
|------|:----:|:----:|------|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| id | Media ID | path | string |

### Response
| Header | Description |
|----|-------------|
| Content-Type | Media Content Type |
| Last-Modified | The last modified date for the requested object. |

### Status Codes
| Code | Reason |
|------|--------|
| 200 | OK |
| 400 | Bad request. |
| 401 | Unauthorized. The access token is invalid or expired. |
| 403 | Forbidden. |
| 404 | Not found. |
| 429 | Too Many Requests. |
| 500 | Internal server error. |


<a name="GetMediaMeta"></a>
## GET /media/{id}/meta

Obtain the metadata (Exif, Google Photo Sphere XMP etc.) of the media data.

### URL Structure
```
https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/meta
```

### Example request
```
curl --request GET 'https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/meta' \
    --header 'Authorization: Bearer <access token>'
```

### Parameters
| Parameter | Description | Parameter Type | Data Type |
|------|:----:|:----:|------|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| id | Media ID | path | string |

### Response
```JavaScript
{
  "exif": {
  },
  "gpano": {
  }
}
```

### Example response
```JavaScript
{
  "exif": {
    "ImageWidth": "5376.0",
    "FocalLength": "1.3 mm",
    "Model": "RICOH THETA S",
    "ExposureTime": "1/6400",
    "Copyright": null,
    "Make": "RICOH",
    "ModifyDate": "2015:05:18 15:02:04",
    "ImageSize": "5376x2688",
    "ExposureMode": "Auto",
    "MaxApertureValue": "2.0",
    "ColorSpace": "sRGB",
    "SensitivityType": "Standard Output Sensitivity",
    "BrightnessValue": "10.3",
    "ExposureProgram": "Auto",
    "DateTimeOriginal": "2015:05:18 15:02:04",
    "ImageHeight": "2688.0",
    "CreateDate": "2015:05:18 15:02:04",
    "ImageDescription": "                                                               ",
    "WhiteBalance": "Auto",
    "FNumber": "2.0",
    "Flash": "No Flash",
    "ISO": "100.0",
    "Orientation": "Horizontal (normal)",
    "MeteringMode": "Multi-segment",
    "LightSource": "Unknown",
    "ApertureValue": "2.0",
    "Software": "RICOH THETA S"
    "GPSLatitudeRef": "North",
    "GPSLatitude": "+35.894469",
    "GPSLongitudeRef": "East",
    "GPSLongitude": "+137.433050",
    "GPSAltitudeRef": "Above Sea Level",
    "GPSAltitude": "830.3 m Above Sea Level",
    "GPSTimeStamp": "16:03:59"
  },
  "gpano": {
    "FullPanoHeightPixels": "2688",
    "CroppedAreaTopPixels": "0",
    "CroppedAreaImageHeightPixels": "2688",
    "PoseHeadingDegrees": "225.0",
    "PosePitchDegrees": "7.0",
    "FullPanoWidthPixels": "5376",
    "UsePanoramaViewer": "True",
    "ProjectionType": "equirectangular",
    "PoseRollDegrees": "2.6",
    "CroppedAreaImageWidthPixels": "5376",
    "CroppedAreaLeftPixels": "0"
  }
}
```
| Header | Description |
|----|-------------|
| Content-Type | application/json; charset=utf-8 |

| Key | Type | Description |
|------|:----:|-------------|
| exif | object | EXIF Metadata |
| gpano | object | Photo Sphere XMP Metadata |

### Status Codes
| Code | Reason |
|------|--------|
| 200 | OK |
| 400 | Bad request. |
| 401 | Unauthorized. The access token is invalid or expired. |
| 403 | Forbidden. |
| 404 | Not found. |
| 429 | Too Many Requests. |
| 500 | Internal server error. |
