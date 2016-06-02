## Media

All API endpoints are listed below.

| Method | Description |
|--------|-------------|
| [GET  /media](#GetMediaList) | Fetch the index of all uploaded media data |
| [POST /media](#Upload) | Upload a media data |
| [POST /media (multipart)](#UploadMulti) | Upload a media data |
| [POST /media/search](#SearchMedia) | Search for uploaded media data |
| [DELETE /media/{id}](#DeleteMedia) | Delete a media data |
| [GET /media/{id}](#GetMediaInfo) | Fetch information of a media data |
| [GET /media/{id}/content](#GetMedia) | Download a media data |
| [GET /media/{id}/meta](#GetMediaMeta) | Fetch metadata (Exif, Photo Sphere XMP etc.) of a media data |
| [GET /media/{id}/meta/exif](#GetMediaMetaExif) | Fetch Exif of a media data |
| [GET /media/{id}/meta/gpano](#GetMediaMetaGpano) | Fetch Photo Sphere XMP of a media data |
| [DELETE /media/{id}/meta/user](#DeleteMediaMetaUser) | Delete all user metadata from a media data |
| [GET  /media/{id}/meta/user](#GetMediaMetaUser) | Fetch all user metadata on a media data |
| [DELETE /media/{id}/meta/user/{key}](#DeleteMediaMetaUserKey) | Delete a user metadata from a media data |
| [GET  /media/{id}/meta/user/{key}](#GetMediaMetaUserKey) | Fetch a user metadata on a media data |
| [PUT  /media/{id}/meta/user/{key}](#PutMediaMetaUserKey) | Add a user metadata on a media data |

* The default encoding is UTF-8, 4 bytes or fewer per each character.
* The maximum request body size is 1MB, except for `POST /media` call.

<a name="GetMediaList"></a>
## GET /media

Returns the index of all uploaded media data and returns in a list.

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
| paging/next | string | The URI of the next page. <br> If the next page does not exist, this key won't be included. |
| paging/previous | string | The URI of the previous page. <br> If the previous page does not exist, this key won't be included. |

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

Receives the uploaded media data from the request payload.<br>
The received data is stored as a new media data.  A Media ID is newly allocated for the media data, then returned in the response.

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

The maximum payload size is defined per each content type.

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
| 411 | Length Required. Content-Length header is missing. |
| 413 | Payload Too Large. Exceeded the content size limit. |
| 415 | Unsupported Media Type. The Content-Type is not supported. |
| 429 | Too Many Requests. |
| 500 | Internal server error. |


<a name="UploadMulti"></a>
## POST /media (multipart)

Receives the uploading media data from the multipart request payload.<br>
The received data is stored as a new media data.  A Media ID is newly allocated for the media data, then returned in the response.

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

| Chunk name | Description |
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


<a name="SearchMedia"></a>
## POST /media/search

Searches the media data and returns the result as a list.<br>
Up to 100 media data can be returned in one response.<br>
If the size of the result list exceeds 100, retry with more query terms.

The search supports schema version `"2016-06-01"`.<br>
This API allows query terms on user metadata only.

### URL Structure
```
https://mss.ricohapi.com/v1/media/search
```

### Example request
```
curl --request POST 'https://mss.ricohapi.com/v1/media/search' \
    --header 'Authorization: Bearer <access token>' \
    --data '{"search_version": "2016-06-01","query": {"meta.user.key1":"value1"}}'
```

### Parameters

| Parameter | Description | Parameter Type | Data Type |
|------|----|:----:|------|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| | Search Expression | body | Model Schema |

#### Model Schema
```
{
  "search_version": "2016-06-01",
  "query":
    {
      "meta.user.key1":"value1",
      "meta.user.key2":"value2"
        ・
        ・
        ・
    }
}
```
| Key | Type | Description |
|----|:----:|-------------|
| search_version | string | Version of the search model schema: `"2016-06-01"` |
| query/meta.user. | string | Search condition in list of _query terms_ (key-value pairs).  <br>Each query term represents an exact match against a single user metadata value. When multiple query terms are specified, they are combined as AND (there is no option for OR). Use `"meta.user"` in a key prefix to denote this is a user metadata query. |

### Response

```JavaScript
{
  "media": [
    {
      "id": "string"
    }
  ]
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
  ]
}
```
| Header | Description |
|----|-------------|
| Content-Type | application/json; charset=utf-8 |

| Key | Type | Description |
|----|:----:|-------------|
| media/id | string | Media ID |

### Status Codes
| Code | Reason |
|------|--------|
| 200 | OK. |
| 400 | Invalid query expression. |
| 401 | Unauthorized. The access token is invalid or expired. |
| 403 | Forbidden. |
| 404 | Not found. |
| 429 | Too Many Requests. |
| 500 | Internal server error. |


<a name="DeleteMedia"></a>
## DELETE /media/{id}

Deletes the specified media data.<br>
Once deleted, media data can not be restored.

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

Returns the information of the media data.

### URL Structure
```
https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472
```

### Example request
```
curl --request GET 'https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472' \
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

Returns the media data for download.

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

Returns the metadata (Exif, Google Photo Sphere XMP etc.) of the media data.

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
  },
  "user": {
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
  },
  "user": {
    "key1": "value1",
    "key2": "value2",
    "key3": "value3"
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
| user | object | User Metadata |

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


<a name="GetMediaMetaExif"></a>
## GET /media/{id}/meta/exif

Returns the Exif of the media data.

### URL Structure
```
https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/meta/exif
```

### Example request
```
curl --request GET 'https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/meta/exif' \
    --header 'Authorization: Bearer <access token>'
```

### Parameters
| Parameter | Description | Parameter Type | Data Type |
|------|:----:|:----:|------|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| id | Media ID | path | string |


### Example response
```JavaScript
{
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
}
```
| Header | Description |
|----|-------------|
| Content-Type | application/json; charset=utf-8 |

| Key | Type | Description |
|------|:----:|-------------|
|  | object | EXIF Metadata |

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


<a name="GetMediaMetaGpano"></a>
## GET /media/{id}/meta/gpano

Returns the Google Photo Sphere XMP of the media data.

### URL Structure
```
https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/meta/gpano
```

### Example request
```
curl --request GET 'https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/meta/gpano' \
    --header 'Authorization: Bearer <access token>'
```

### Parameters
| Parameter | Description | Parameter Type | Data Type |
|------|:----:|:----:|------|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| id | Media ID | path | string |


### Example response
```JavaScript
{
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
```
| Header | Description |
|----|-------------|
| Content-Type | application/json; charset=utf-8 |

| Key | Type | Description |
|------|:----:|-------------|
| | object | Photo Sphere XMP Metadata |

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


<a name="DeleteMediaMetaUser"></a>
## DELETE /media/{id}/meta/user

Deletes all the user metadata attached to the media data.

### URL Structure
```
https://mss.ricohapi.com/v1/media/03das393-a8f6-4959-80ec-23039485304d/meta/user
```

### Example request
```
curl --request DELETE 'https://mss.ricohapi.com/v1/media/03das393-a8f6-4959-80ec-23039485304d/meta/user' \
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


<a name="GetMediaMetaUser"></a>
## GET /media/{id}/meta/user

Returns the user metadata attached to the media data.

### URL Structure
```
https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/meta/user
```

### Example request
```
curl --request DELETE 'https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/meta/user' \
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
  "key1": "value1",
  "key2": "value2",
  "key3": "value3"
}
```
| Header | Description |
|----|-------------|
| Content-Type | application/json; charset=utf-8 |

| Key | Type | Description |
|------|:----:|-------------|
| key name of user metadata | string | value of user metadata |

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


<a name="DeleteMediaMetaUserKey"></a>
## DELETE /media/{id}/meta/user/{key}

Deteles the user metadata under the specified key.

### URL Structure
```
https://mss.ricohapi.com/v1/media/03das393-a8f6-4959-80ec-23039485304d/meta/user/key
```

### Example request
```
curl --request DELETE 'https://mss.ricohapi.com/v1/media/03das393-a8f6-4959-80ec-23039485304d/meta/user/key' \
    --header 'Authorization: Bearer <access token>'
```

### Parameters

| Parameter | Description | Parameter Type | Data Type |
|------|:----:|:----:|------|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| id | Media ID | path | string |
| key | key name of user metadata | path | string |

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


<a name="GetMediaMetaUserKey"></a>
## GET /media/{id}/meta/user/{key}

Returns the user metadata under the specified key.

### URL Structure
```
https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/meta/user/key1
```

### Example request
```
curl --request DELETE 'https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/meta/user/key1' \
    --header 'Authorization: Bearer <access token>'
```

### Parameters
| Parameter | Description | Parameter Type | Data Type |
|------|:----:|:----:|------|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| id | Media ID | path | string |
| key | key name of user metadata | path | string |

### Response
```JavaScript
{
  "key1": "value1"
}
```
| Header | Description |
|----|-------------|
| Content-Type | application/json; charset=utf-8 |

| Key | Type | Description |
|------|:----:|-------------|
| key name of user metadata | string | value of user metadata |

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


<a name="PutMediaMetaUserKey"></a>
## PUT /media/{id}/meta/user/{key}

Attaches a user metadata to the media data under the specified key.<br>
Existing metadata value for the same key will be overwritten.<br>
Up to 10 user metadata can be attached to a media data.

### URL Structure
```
https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/meta/user/key1
```

### Example request
```
curl --request PUT 'https://mss.ricohapi.com/v1/media/a39jcbc5-053c-4873-a7c0-0b20c3948472/meta/user/key1' \
    --header 'Authorization: Bearer <access token>' \
    --data 'value1'
```

### Parameters
| Parameter | Description | Parameter Type | Data Type |
|------|:----:|:----:|------|
| Authorization | Authorization Header<br>Put the resource owner's OAuth2 access token | header | string |
| id | Media ID | path | string |
| key | User metadata key.<br> Key can be a string of 1 to 256 bytes, composed by characters: a~z A~Z 0~9 _ (0x5F) - (0x2D)| path | string |
| | User metadata value.<br> Value can be a string of 1 to 1024 bytes. | body | string |

### Response
| Header | Description |
|----|-------------|
| Content-Type | application/octet-stream |

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
