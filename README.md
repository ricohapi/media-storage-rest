# Ricoh Media Storage REST API

Ricoh Media Storage service is a cloud-based storage service provided by Ricoh Company, Ltd.

Developers can access Ricoh Media Storage service features listed below, via its APIs.

* Uploading media data to the cloud storage
* Downloading media data from the cloud storage
* Fetching the index of all uploaded media data in the cloud storage
* Fetching metadata of each uploaded media data in the cloud storage

## Requirements

You  need

* Ricoh API Client Credentials (client_id & client_secret)
* Ricoh ID (user_id & password)

If you don't have them, please register them at [THETA Developers Website](http://contest.theta360.com/).

## Access Token
Before a client can call a Media Storage API, the client needs to obtain an access token. <br> Please refer to the following document.

* [Authorization and Discovery Service](http://docs.ricohapi.com/docs/authorization-and-discovery-service)

An obtained access token should be set in the Authorization header of every API request.

```JavaScript
curl --request GET 'https://mss.ricohapi.com/v1/media' \
    --header 'Authorization: Bearer <access token>'
```

## APIs

* [Media](media.md)

## SDK & Sample Apps

* [SDK for Ruby](https://github.com/ricohapi/media-storage-rb)
* [SDK for JavaScript](https://github.com/ricohapi/media-storage-js)
* [Sample App](https://github.com/ricohapi/media-storage-sample-app)

## API Console

You can play with the Ricoh Media Storage REST API using its [API Console](http://docs.ricohapi.com/docs/media-storage-rest).
