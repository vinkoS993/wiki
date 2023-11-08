# Apillon API

<div class="divider"></div>

## Endpoints

List of endpoints the API is available at:

| Environment | URL                    |
| ----------- | ---------------------- |
| Production  | https://api.apillon.io |
| Testing     | Coming soon...         |

## API to Web3

Apillon API is a set of RESTful API endpoints allowing developers to integrate Apillon modules into their Web3 applications.

Unless clearly marked as public, all routes are private and require an API key.

### Requests

The server speaks [JSON](https://en.wikipedia.org/wiki/JSON). It is recommended that every call to the server includes a `Content-Type` header set to `application/json;`.

### Authentication and authorization

API routes restrict public access and require authentication.

Requests must include a [basic auth](https://en.wikipedia.org/wiki/Basic_access_authentication) HTTP header field in the form of `Authorization: Basic <credentials>`, where credentials represent the Base64 encoding of API key and API key secret joined by a single colon `:`.

API keys could be generated on the [developer dashboard](https://app.apillon.io/dashboard/api-keys) under Project settings.

#### Authentication errors

Every request goes through authentication middleware, where following errors can occur:
| Status | Message | Description
|-|-|-
|400|Missing Authorization header|Request is missing authorization header.
|400|Malformed Authorization header|Authorization header field has invalid form.
|401|Invalid API key or API key secret|Authorization header is valid but credentials in it are not.

#### Authorization errors

Each endpoint requires a certain role or permission from the API key.

There are three types of permissions that could be assigned to an API key:

| Code | Name        | Description                                    |
| ---- | ----------- | ---------------------------------------------- |
| 50   | KEY_EXECUTE | Permission to execute certain actions          |
| 51   | KEY_WRITE   | Permission to create, modify or delete records |
| 52   | KEY_READ    | Permission to read record                      |

These permissions could be assigned to an API key for every attached service (e. g., Web3 Storage (Crust), Web3 Authentication (KILT), etc.).

If a request is made with an API key that lacks permission for called endpoint, the following errors can occur:

| Status | Message                                                         | Description                                                                                                                                                |
| ------ | --------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 403    | Insufficient permissions - missing `permission name` permission | API key lacks required permission for called service.                                                                                                      |
| 403    | Insufficient permissions to access this record                  | API key has required permissions for endpoint, but it does not have the right to access addressed record (e. g., a record belongs to a different project). |

### Responses

<div class="split_content">
	<div class="split_side">

Every response has a unique ID which helps identify potential issues. It also includes a status code that can help identify the cause of a potential problem.

Query requests through `GET` method can return status codes `200`, `400`, `401`, `403`, or `500`. Mutations through `POST`, `PUT` and `DELETE` can also return codes `201` and `422`. Invalid routes return status code `404`.

A successful request includes a `data` key, which holds a valid response object.

  </div>
  <div class="split_side">

  <CodeGroup>
  <CodeGroupItem title="Response" active>

```json
{
  "id": ...,
  "status": ...,
  "data": { ... },
}
```

  </CodeGroupItem>
  </CodeGroup>
	</div>
</div>

List of responses:

- **200**: Success
- **201**: Creation successful
- **400**: Bad request
- **401**: Unauthenticated access
- **403**: Unauthorized access
- **404**: Path not found
- **422**: Data validation failed
- **500**: System error

### Error handling

Request fails if response code is not 200 or 201. The Apillon API returns two types of errors.

#### Code exception

Errors include a unique code number, a property which caused the error, and an error message. The code number helps identify potential issues and points to their exact position in the system.

Fields in code exception:

<div class="split_content">
	<div class="split_side">

| Field     | Description                                                                  |
| --------- | ---------------------------------------------------------------------------- |
| id        | Unique ID of request                                                         |
| code      | Apillon API internal error code pointing to the exact position in the system |
| message   | Message describing the error                                                 |
| path      | Endpoint that threw the error                                                |
| timestamp | Date when the error occurred                                                 |

  </div>
	<div class="split_side">
  <CodeGroup>
  <CodeGroupItem title="Response" active>

```json
{
  "id": "c46821e7-a6c3-4377-bc32-0001e348ceb4",
  "code": 40406005,
  "message": "FILE_DOES_NOT_EXIST",
  "path": "/storage/cee9f151-a371-495b-acd2-4362fbb87780/file/xxx/detail",
  "timestamp": "2022-12-29T11:54:47.555Z"
}
```

  </CodeGroupItem>
  </CodeGroup>
	</div>
</div>

#### Validation exception

<div class="split_content">
	<div class="split_side">

Unprocessable entity `422 Error status` includes an `errors` key, which holds a list of error objects.

This error typically occurs when the request body is not valid (i. e., it is invalid or missing keys).

Errors include a unique code number, a property which caused the error, and an error message. The code number helps identify potential issues and points to their exact position in the system

  </div>
	<div class="split_side">
  <CodeGroup>
  <CodeGroupItem title="Response" active>

```json
{
  ...
  "errors": [
    {
        "code": 42200008,
        "property": "fileName",
        "message": "FILE_NAME_NOT_PRESENT"
    },
  ]
}
```

  </CodeGroupItem>
  </CodeGroup>
	</div>
</div>

Fields in validation exception:
| Field | Description
|-|-
| id | Unique ID of request
| model | Apillon API model used to validate request payload
| errors | Array of errors
| path | Endpoint that threw the error
| timestamp | Date when the error occurred

## Web3 Storage API

**Note:** To use Apillon Web3 Storage APIs, you should first create a bucket on the [Apillon dashboard](app.apillon.io/service/storage).

In all cURL examples, parameters with a colon as a prefix should be replaced with real values.

**File upload process through Apillon Web3 Storage API**

1. Request signed URL(s) for upload.
2. File is uploaded to Apillon central server.
3. File is transferred to IPFS and available through the Apillon gateway.
4. File is replicated to different IPFS nodes globally via Crust Network.

### Upload to bucket

> API that creates file upload requests and returns URLs for file upload along with `sessionUuid`.

<div class="request-url">POST /storage/:bucketUuid/upload</div>

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name       | Description                                                            | Required |
| ---------- | ---------------------------------------------------------------------- | -------- |
| bucketUuid | Unique key of storage bucket. Key is displayed on developer dashboard. | true     |

#### Body fields

| Name  | Type    | Description              | Required |
| ----- | ------- | ------------------------ | -------- |
| files | `array` | Array of files metadata. | true     |

Each file metadata object in `files` array, contain below properties.

| Name        | Type     | Description                                                                                                                                                                                                                                                                                                                                                                                               | Required |
| ----------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| fileName    | `string` | Full name (name and extension) of file to be uploaded                                                                                                                                                                                                                                                                                                                                                     | true     |
| contentType | `string` | File [MIME type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types)                                                                                                                                                                                                                                                                                                | false    |
| path        | `string` | Virtual file path. Empty for root. Must not contain `fileName`. <br><br> The `path` field can be used to place file in virtual directories inside a bucket. If directories do not yet exist, they will be automatically generated.<br><br>For example, an `images/icons` path creates `images` directory in a bucket and `icons` directory inside it. File will then be created in the `icons` directory. | false    |

#### Possible errors

| Code     | Description                                                                    |
| -------- | ------------------------------------------------------------------------------ |
| 40406002 | Bucket does not exist.                                                         |
| 42200040 | Request body is missing a `files` field.                                       |
| 42200008 | Request body file object is missing a `fileName` field.                        |
| 40006002 | Bucket has reached max size limit.                                             |
| 40406009 | Bucket is marked for deletion. It is no longer possible to upload files to it. |
| 50006003 | Internal error - Apillon was unable to generate upload URL.                    |

#### Response

| Name        | Type     | Description                                                                        |
| ----------- | -------- | ---------------------------------------------------------------------------------- |
| sessionUuid | `string` | Session unique key, which is later used to end upload and transfer files to bucket |
| files       | `array`  | Array of files metadata.                                                           |

Files in request body are returned in response `data.files` property. Each file is equipped with `url` and `fileUuid`. All properties are displayed below.

| Field       | Type     | Description                                                                                                                                                                                                                                                                                                                                 |
| ----------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| url         | `string` | URL for file upload. Signed URL is unique for each file and is valid only for a limited time (1 min), so you should start with file upload as soon as possible.<br><br>Request should use `PUT` method and `binary` body.<br><br>Binary data should be sent in body as-is, but with the appropriate Content-Type header (e.g., text/plain). |
| fileUuid    | `string` | File unique identifier used to query file status, etc.                                                                                                                                                                                                                                                                                      |
| fileName    | `string` | Full name (name and extension) of file to be uploaded                                                                                                                                                                                                                                                                                       |
| contentType | `string` | File [MIME type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types)                                                                                                                                                                                                                                  |
| path        | `string` | File path.                                                                                                                                                                                                                                                                                                                                  |

  </div>
  <div class="split_side">
    <br>
      <CodeGroup>
      <CodeGroupItem title="cURL" active>

```sh
curl --location --request POST "https://api.apillon.io/storage/:bucketUuid/upload" \
--header "Authorization: Basic :credentials" \
--header "Content-Type: application/json" \
--data-raw "{
    \"files\": [
        {
            \"fileName\": \"My test file\",
            \"contentType\": \"text/html\"
        }
    ]

}"
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
  "id": "cbdc4930-2bbd-4b20-84fa-15daa4429952",
  "status": 201,
  "data": {
    "sessionUuid": "3b6113bc-f265-4662-8cc5-ea86f06cc74b",
    "files": [
      {
        "path": null,
        "fileName": "My test file",
        "contentType": "text/html",
        "url": "https://sync-to-ipfs-queue.s3.eu-west-1.amazonaws.com/STORAGE_sessions/73/3b6113bc-f265-4662-8cc5-ea86f06cc74b/My%20test%20file?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAQIMRRA6GJRL57L7G%2F20230215%2Feu-west-1%2Fs3%2Faws4_request&X-Amz-Date=20230215T114524Z&X-Amz-Expires=900&X-Amz-Signature=499367f6c6bff5be50686724475ac2fa6307b77b94fd1a25584c092fe74b0a58&X-Amz-SignedHeaders=host&x-id=PutObject",
        "fileUuid": "4ef1177b-f7c9-4434-be56-a559cec0cc18"
      }
    ]
  }
}
```

**Example for uploading to signed URL:**

  </CodeGroupItem>
  </CodeGroup>

  <CodeGroup>
  <CodeGroupItem title="cURL binary" active>

```sh
curl --location --request PUT "https://sync-to-ipfs-queue.s3.eu-west-1.amazonaws.com/STORAGE_sessions/73/3b6113bc-f265-4662-8cc5-ea86f06cc74b/My%20test%20file?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAQIMRRA6GJRL57L7G%2F20230215%2Feu-west-1%2Fs3%2Faws4_request&X-Amz-Date=20230215T114524Z&X-Amz-Expires=900&X-Amz-Signature=499367f6c6bff5be50686724475ac2fa6307b77b94fd1a25584c092fe74b0a58&X-Amz-SignedHeaders=host&x-id=PutObject" \
--data-binary "My test content"
```

  </CodeGroupItem>
  <CodeGroupItem title="cURL file from disk" active>

```sh
curl --location --request PUT "https://sync-to-ipfs-queue.s3.eu-west-1.amazonaws.com/STORAGE_sessions/73/3b6113bc-f265-4662-8cc5-ea86f06cc74b/My%20test%20file?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAQIMRRA6GJRL57L7G%2F20230215%2Feu-west-1%2Fs3%2Faws4_request&X-Amz-Date=20230215T114524Z&X-Amz-Expires=900&X-Amz-Signature=499367f6c6bff5be50686724475ac2fa6307b77b94fd1a25584c092fe74b0a58&X-Amz-SignedHeaders=host&x-id=PutObject" \
--header "Content-Type: text/plain" \
--data-binary ":full path to file"
```

  </CodeGroupItem>
  </CodeGroup>

  </div>
</div>

### End upload session

> Once files are uploaded to cloud server via received URL, trigger sync of files to IPFS and CRUST.

<div class="request-url">POST /storage/:bucketUuid/upload/:sessionUuid/end</div>

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name        | Description                                                     | Required |
| ----------- | --------------------------------------------------------------- | -------- |
| bucketUuid  | Unique key of bucket. Key is displayed in developer dashboard.  | true     |
| sessionUuid | Session uuid, recieved in [upload to bucket](#upload-to-bucket) | true     |

#### Possible errors

| Code     | Description                                    |
| -------- | ---------------------------------------------- |
| 40406004 | Session does not exists                        |
| 40006001 | Files in this session were already transferred |

#### Response

Api respond with status `200 OK` , if operation is successfully executed.

  </div>
  <div class="split_side">
    <br>
      <CodeGroup>
          <CodeGroupItem title="cURL" active>

```sh
curl --location --request POST "https://api.apillon.io/storage/:bucketUuid/upload/:sessionUuid/end" \
--header "Authorization: Basic :credentials" \
--header "Content-Type: application/json" \
--data-raw "{
    \"directSync\": true
}"
```

  </CodeGroupItem>
</CodeGroup>
<CodeGroup>
  <CodeGroupItem title="Response">

```json
{
  "id": "b64b1c07-1a8a-4b05-9e3b-3c6a519d6ff7",
  "status": 200,
  "data": true
}
```

</CodeGroupItem>
</CodeGroup>
  </div>
</div>

### Get bucket content

> Gets directories and files in bucket. Items are paginated and can be filtered and ordered through query parameters.

**Note: This endpoint returns files, that are successfully transferred to IPFS node. I.e. files with [fileStatus](#file-statuses) 3 or 4.**

<div class="request-url">GET /storage/:bucketUuid/content</div>

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name       | Description                                                    | Required |
| ---------- | -------------------------------------------------------------- | -------- |
| bucketUuid | Unique key of bucket. Key is displayed on developer dashboard. | true     |

#### Query parameters

| Name        | Description                                                                               | Required |
| ----------- | ----------------------------------------------------------------------------------------- | -------- |
| search      | Filters file by file name.                                                                | false    |
| directoryId | Gets files inside a specific directory.                                                   | false    |
| page        | Files are paginated by default. This parameter is used to get files from a specific page. | false    |
| limit       | Number of files on a page (default: 20).                                                  | false    |
| orderBy     | One or multiple properties, separated by a comma, used to order data.                     | false    |
| desc        | `Boolean` values, mapped to the index of the `orderBy` parameter. Defaults to false.      | false    |

#### Possible errors

| Code     | Description            |
| -------- | ---------------------- |
| 40406002 | Bucket does not exist. |

#### Response fields

The `Data` property of API response contains two properties: `items` (records that match the current query) and `total` (number of all records. This information should be used for pagination: Round up (`total` / `limit`) = number of pages.

Properties of each item:

| Field             | Type       | Description                                                        |
| ----------------- | ---------- | ------------------------------------------------------------------ |
| id                | `integer`  | Item internal ID                                                   |
| type              | `integer`  | Item type with possible values `1`(directory) and `2`(file)        |
| name              | `string`   | Item (directory or file) name                                      |
| createTime        | `DateTime` | Item create time                                                   |
| updateTime        | `DateTime` | Item last update time                                              |
| contentType       | `string`   | Item content type (MIME type)                                      |
| size              | `integer`  | Item size in bytes                                                 |
| parentDirectoryId | `integer`  | ID of directory where a file is located                            |
| fileUuid          | `string`   | File unique identifier                                             |
| CID               | `string`   | File content identifier - label used to point to material in IPFS. |
| link              | `string`   | File link on Apillon IPFS gateway.                                 |

  </div>
  <div class="split_side">
    <br>
      <CodeGroup>
      <CodeGroupItem title="cURL basic" active>

```sh
curl --location --request GET "https://api.apillon.io/storage/:bucketUuid/content" \
--header "Authorization: Basic :credentials"
```

  </CodeGroupItem>
  <CodeGroupItem title="cURL with params" active>

```sh
curl --location --request GET "https://api.apillon.io/storage/:bucketUuid/content?orderBy=name&desc=false&limit=5&page=1" \
--header "Authorization: Basic :credentials"
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
    "id": "c8c50b3b-91ff-42c7-b0af-f866ce23f18a",
    "status": 200,
    "data": {
        "items": [
            ...
            {
                "type": 1,
                "id": 11,
                "status": 5,
                "name": "My directory",
                "CID": null,
                "createTime": "2022-12-08T13:27:00.000Z",
                "updateTime": "2023-01-10T12:18:55.000Z",
                "contentType": null,
                "size": null,
                "parentDirectoryId": null,
                "fileUuid": null,
                "link": null
            },
            {
                "type": 2,
                "id": 397,
                "status": 5,
                "name": "My file.txt",
                "CID": "QmcG9r6Rdw9ZdJ4imGBWc6mi5VzWHQfkcLDMe2aP74eb42",
                "createTime": "2023-01-19T10:10:01.000Z",
                "updateTime": "2023-01-19T10:10:31.000Z",
                "contentType": "text/plain",
                "size": 68,
                "parentDirectoryId": null,
                "fileUuid": "0a775bfa-a0d0-4e0b-9a1e-e909e426bd11",
                "link": "https://ipfs.apillon.io/ipfs/QmcG9r6Rdw9ZdJ4imGBWc6mi5VzWHQfkcLDMe2aP74eb42"
            }
            ...
        ],
        "total": 10
    }
}
```

  </CodeGroupItem>
  </CodeGroup>
	</div>
</div>

### Get file details

> Gets details of a specific file inside a bucket.

<div class="request-url">GET /storage/:bucketUuid/file/:id/detail</div>

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name       | Description                                                      | required |
| ---------- | ---------------------------------------------------------------- | -------- |
| bucketUuid | Unique key of a bucket. Key is displayed on developer dashboard. | true     |
| id         | File internal ID, UUID or CID.                                   | true     |

#### Possible errors

| Code     | Description          |
| -------- | -------------------- |
| 40406005 | File does not exist. |

#### Response fields

Response `data` property contains two properties: `fileStatus` and `file`. File status tells the current status of the file relative to the entire flow the file goes through to be fully loaded and pinned on Crust Network, while `file` property contains file metadata fields.

##### File statuses

| Number | Description                                                       |
| ------ | ----------------------------------------------------------------- |
| 1      | Request for upload to Apillon storage was generated.              |
| 2      | File is uploaded to Apillon central server.                       |
| 3      | File is transferred to IPFS node.                                 |
| 4      | File is replicated to different IPFS nodes through Crust Network. |

##### File metadata

`CID`, `size`, and `downloadLink` are present if file is already loaded to IPFS.

| Field                                                                    | Type      | Description                                          |
| ------------------------------------------------------------------------ | --------- | ---------------------------------------------------- |
| id                                                                       | `integer` | Apillon internal file ID                             |
| status                                                                   | `integer` | Apillon internal file status                         |
| fileUuid                                                                 | `string`  | File unique identifier                               |
| name                                                                     | `string`  | File name                                            |
| contentType                                                              | `string`  | File content type (MIME type)                        |
| [CID](https://docs.ipfs.tech/concepts/content-addressing/#what-is-a-cid) | `string`  | File content identifier pointing to material in IPFS |
| size                                                                     | `integer` | File size in bytes                                   |
| downloadLink                                                             | `string`  | File link on Apillon IPFS gateway                    |

  </div>
  <div class="split_side">
    <br>
      <CodeGroup>
      <CodeGroupItem title="cURL" active>

```sh
curl --location --request GET "https://api.apillon.io/storage/:bucketUuid/file/:id/detail" \
--header "Authorization: Basic :credentials"
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
  "id": "5be33c54-2cc9-46f4-8f50-debc98866810",
  "status": 200,
  "data": {
    "fileStatus": 4,
    "file": {
      "id": 397,
      "status": 5,
      "fileUuid": "0a775bfa-a0d0-4e0b-9a1e-e909e426bd11",
      "CID": "QmcG9r6Rdw9ZdJ4imGBWc6mi5VzWHQfkcLDMe2aP74eb42",
      "name": "My file.txt",
      "contentType": "text/plain",
      "size": 68,
      "fileStatus": 4,
      "downloadLink": "https://ipfs.apillon.io/ipfs/QmcG9r6Rdw9ZdJ4imGBWc6mi5VzWHQfkcLDMe2aP74eb42"
    }
  }
}
```

  </CodeGroupItem>
  </CodeGroup>
	</div>
</div>

### Delete file

> Marks a file inside bucket for deletion by `id`, `fileUuid`, or `CID`. File will be completely deleted from the Apillon system and Apillon IPFS node after 3 months.
> If file is marked for deletion, it will not be renewed on Crust Network.

<div class="request-url">DELETE /storage/:bucketUuid/file/:id</div>

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name       | Description                                                    | required |
| ---------- | -------------------------------------------------------------- | -------- |
| bucketUuid | Unique key of bucket. Key is displayed on developer dashboard. | true     |
| id         | File internal ID, UUID, or CID.                                | true     |

#### Possible errors

| Code     | Description                  |
| -------- | ---------------------------- |
| 40406005 | File does not exist.         |
| 40006009 | File is marked for deletion. |

#### Response fields

The response of delete function is a record that has been marked for deletion.

Returned fields are the same as fields that are returned in [GET file details API](#file-metadata).

**Note:** The `status` property of file is 8. This means the file is marked for deletion and will be deleted after a certain period.

  </div>
  <div class="split_side">
    <br>
      <CodeGroup>
      <CodeGroupItem title="cURL" active>

```sh
curl --location --request DELETE "https://api.apillon.io/storage/:bucketUuid/file/:id" \
--header "Authorization: Basic :credentials" \
--data-raw ""
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
  "id": "bc92ff8d-05f2-4380-bb13-75a1b6b7f388",
  "status": 200,
  "data": {
    "id": 397,
    "status": 8,
    "fileUuid": "0a775bfa-a0d0-4e0b-9a1e-e909e426bd11",
    "CID": "QmcG9r6Rdw9ZdJ4imGBWc6mi5VzWHQfkcLDMe2aP74eb42",
    "name": "My file.txt",
    "contentType": "text/plain",
    "size": 68,
    "fileStatus": 4
  }
}
```

  </CodeGroupItem>
  </CodeGroup>
	</div>
</div>

## Web3 Hosting API

Hosting API provides endpoints, that can be used to implement [CI/CD](https://en.wikipedia.org/wiki/CI/CD).
To deploy page through Apillon API, follow below steps:

1. Upload website files to Apillon cloud server.

- Request URLs for files upload
- Upload files to cloud server
- Trigger transfer into website

2. Execute deployment to staging or production environment.

**Note:** To use Apillon Web3 Hosting APIs, you should first create a website on the [Apillon dashboard](https://app.apillon.io/dashboard/service/hosting).

In all cURL examples, parameters with a colon as a prefix should be replaced with real values.

### Get URLs for files upload

> API that creates file upload requests and returns URLs for files upload.

<div class="request-url">POST /hosting/websites/:websiteUuid/upload</div>

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name        | Description                                                            | Required |
| ----------- | ---------------------------------------------------------------------- | -------- |
| websiteUuid | Unique key of website bucket. Key is displayed on developer dashboard. | true     |

#### Body fields

| Name  | Type    | Description              | Required |
| ----- | ------- | ------------------------ | -------- |
| files | `array` | Array of files metadata. | true     |

Each file metadata object in `files` array, contain below properties.

| Name        | Type     | Description                                                                                                | Required |
| ----------- | -------- | ---------------------------------------------------------------------------------------------------------- | -------- |
| fileName    | `string` | Full name (name and extension) of file to be uploaded                                                      | true     |
| contentType | `string` | File [MIME type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types) | false    |
| path        | `string` | File path inside website. Empty for root. Must not contain `fileName`.                                     | false    |

#### Possible errors

| Code     | Description                                                 |
| -------- | ----------------------------------------------------------- |
| 40406010 | Website does not exists                                     |
| 42200040 | Request body is missing a `files` field.                    |
| 42200008 | Request body file object is missing a `fileName` field.     |
| 40006002 | Website has reached max size limit.                         |
| 50006003 | Internal error - Apillon was unable to generate upload URL. |

#### Response

| Name        | Type     | Description                                                                         |
| ----------- | -------- | ----------------------------------------------------------------------------------- |
| sessionUuid | `string` | Session unique key, which is later used to end upload and transfer files ti website |
| files       | `array`  | Array of files metadata.                                                            |

Files in request body are returned in response `data.files` property. Each file is equipped with `url` and `fileUuid`. All properties are displayed below.

| Field       | Type     | Description                                                                                                                                                                                                                                                                                                                                 |
| ----------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| url         | `string` | URL for file upload. Signed URL is unique for each file and is valid only for a limited time (1 min), so you should start with file upload as soon as possible.<br><br>Request should use `PUT` method and `binary` body.<br><br>Binary data should be sent in body as-is, but with the appropriate Content-Type header (e.g., text/plain). |
| fileUuid    | `string` | File unique identifier used to query file status, etc.                                                                                                                                                                                                                                                                                      |
| fileName    | `string` | Full name (name and extension) of file to be uploaded                                                                                                                                                                                                                                                                                       |
| contentType | `string` | File [MIME type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types)                                                                                                                                                                                                                                  |
| path        | `string` | File path.                                                                                                                                                                                                                                                                                                                                  |

  </div>
  <div class="split_side">
    <br>
      <CodeGroup>
      <CodeGroupItem title="cURL" active>

```sh
curl --location --request POST "https://api.apillon.io/hosting/websites/:websiteUuid/upload" \
--header "Authorization: Basic :credentials" \
--header "Content-Type: application/json" \
--data-raw "{
    \"files\": [
        {
            \"fileName\": \"index.html\",
            \"contentType\": \"text/html\"
        },
        {
            \"fileName\": \"styles.css\",
            \"contentType\": \"text/css\",
            \"path\": \"assets/\"
        }

    ]

}"
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
  "id": "7dd011ec-20e2-4c28-b585-da6c6f7fce8d",
  "status": 201,
  "data": {
    "sessionUuid": "29ef6ca2-b171-440c-b934-db8aa88c3424",
    "files": [
      {
        "path": null,
        "fileName": "index.html",
        "contentType": "text/html",
        "url": "https://sync-to-ipfs-queue.s3.eu-west-1.amazonaws.com/HOSTING_sessions/70/29ef6ca2-b171-440c-b934-db8aa88c3424/index.html?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAQIMRRA6GJRL57L7G%2F20230215%2Feu-west-1%2Fs3%2Faws4_request&X-Amz-Date=20230215T105030Z&X-Amz-Expires=900&X-Amz-Signature=187035d2307bc089101eff3abbdd7baa3e8691b4d5d3bafa5aebb87e589e8c0c&X-Amz-SignedHeaders=host&x-id=PutObject",
        "fileUuid": "e17436a1-5292-4380-ad91-eaac02a862b1"
      },
      {
        "path": "assets/",
        "fileName": "styles.css",
        "contentType": "text/css",
        "url": "https://sync-to-ipfs-queue.s3.eu-west-1.amazonaws.com/HOSTING_sessions/70/29ef6ca2-b171-440c-b934-db8aa88c3424/assets/styles.css?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAQIMRRA6GJRL57L7G%2F20230215%2Feu-west-1%2Fs3%2Faws4_request&X-Amz-Date=20230215T105030Z&X-Amz-Expires=900&X-Amz-Signature=f91b03a951fe3f99291802306be3e79812ca64e39effbb7dea1c19bb7cd1e42b&X-Amz-SignedHeaders=host&x-id=PutObject",
        "fileUuid": "358c2942-4ced-421e-9a6f-edbf94c55dff"
      }
    ]
  }
}
```

**Example for uploading to signed URL:**

  </CodeGroupItem>
  </CodeGroup>

  <CodeGroup>
  <CodeGroupItem title="cURL binary" active>

```sh
curl --location --request PUT "https://sync-to-ipfs-queue.s3.eu-west-1.amazonaws.com/HOSTING_sessions/70/29ef6ca2-b171-440c-b934-db8aa88c3424/index.html?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAQIMRRA6GJRL57L7G%2F20230215%2Feu-west-1%2Fs3%2Faws4_request&X-Amz-Date=20230215T105030Z&X-Amz-Expires=900&X-Amz-Signature=187035d2307bc089101eff3abbdd7baa3e8691b4d5d3bafa5aebb87e589e8c0c&X-Amz-SignedHeaders=host&x-id=PutObject" \
--header "Content-Type: text/plain" \
--data-raw "<h1>
Welcome to my awesome website
</h1>"
```

  </CodeGroupItem>
  <CodeGroupItem title="cURL file from disk" active>

```sh
curl --location --request PUT "https://sync-to-ipfs-queue.s3.eu-west-1.amazonaws.com/HOSTING_sessions/70/29ef6ca2-b171-440c-b934-db8aa88c3424/index.html?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAQIMRRA6GJRL57L7G%2F20230215%2Feu-west-1%2Fs3%2Faws4_request&X-Amz-Date=20230215T105030Z&X-Amz-Expires=900&X-Amz-Signature=187035d2307bc089101eff3abbdd7baa3e8691b4d5d3bafa5aebb87e589e8c0c&X-Amz-SignedHeaders=host&x-id=PutObject" \
--header "Content-Type: text/plain" \
--data-binary ":full path to file"
```

  </CodeGroupItem>
  </CodeGroup>

  </div>
</div>

### End upload session

> Transfer files to website bucket, which is used as source for deploy to staging(preview) environment.

<div class="request-url">POST /hosting/websites/:websiteUuid/upload/:sessionUuid/end</div>

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name        | Description                                                                           | Required |
| ----------- | ------------------------------------------------------------------------------------- | -------- |
| websiteUuid | Unique key of website. Key is displayed in developer dashboard.                       | true     |
| sessionUuid | Session uuid, recieved in [get URL for upload request](#post-storagebucketuuidupload) | true     |

#### Possible errors

| Code     | Description                                    |
| -------- | ---------------------------------------------- |
| 40406004 | Session does not exists                        |
| 40006001 | Files in this session were already transferred |

#### Response

Api respond with status `200 OK` , if operation is successfully executed.

  </div>
  <div class="split_side">
    <br>
    <CodeGroup>
        <CodeGroupItem title="cURL" active>

```sh
curl --location --request POST "https://api.apillon.io/hosting/websites/:websiteUuid/upload/:sessionUuid/end" \
--header "Authorization: Basic :credentials" \
--header "Content-Type: application/json" \
--data-raw "{
    \"directSync\": true
}"
```

  </CodeGroupItem>
</CodeGroup>
<CodeGroup>
  <CodeGroupItem title="Response">

```json
{
  "id": "b64b1c07-1a8a-4b05-9e3b-3c6a519d6ff7",
  "status": 200,
  "data": true
}
```

</CodeGroupItem>
</CodeGroup>
  </div>
</div>

### Deploy website

> Endpoint to trigger website deployment into specific environment.

<div class="request-url">POST /hosting/websites/:websiteUuid/deploy</div>

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name        | Description                                                 | Required |
| ----------- | ----------------------------------------------------------- | -------- |
| websiteUuid | Website UUID, visible in developer console website overview | true     |

#### Body fields

| Name        | Type     | Description                                       | Required |
| ----------- | -------- | ------------------------------------------------- | -------- |
| environment | `number` | Possible `environment` values are explained below | true     |

##### Environments

| Status | Description                                                                                                                                                      |
| ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1      | Uploaded files are deployed to staging environment. Website will be available through staging IPNS link                                                          |
| 2      | Files from current staging environment are deployed to production environment. Website is pinned to CRUST, replicated and available through production IPNS link |
| 3      | Same as `2`, only that the source are uploaded files, not files in staging environment.                                                                          |

#### Possible errors

| Code     | Description                                     |
| -------- | ----------------------------------------------- |
| 42200039 | Request body is missing an `environment` field. |
| 40406010 | Website does not exists                         |
| 40006016 | There are no files to deploy.                   |
| 40006017 | There are no changes to deploy.                 |

#### Response

Endpoint triggers deployment of website to specific environment. As result, deployment record with below field is returned.
This deployment is now waiting to be processed.

**Note:** Deployment is processed in background, which may take several minutes.
When deploying to `staging` environment, files are added to IPFS and wrapped to directory, which is then accessible in IPFS via IPNS or CID.
In `production`, this CID is pinned to CRUST and replicated to other nodes.

| Field            | Type     | Description                                                                                   |
| ---------------- | -------- | --------------------------------------------------------------------------------------------- |
| id               | `number` | Deployment internal number                                                                    |
| status           | `number` | Deployment record status                                                                      |
| bucketId         | `number` | Internal ID of bucket, which contain files for this environment                               |
| environment      | `number` | Environment to where website will be deployed                                                 |
| deploymentStatus | `number` | Current status of deployment. Possible values are listed below.                               |
| cid              | `string` | When deployment is successful, CID points to directory on IPFS, where this page is accessible |
| size             | `number` | Size of website                                                                               |
| number           | `number` | Deployment serial number - for this environment                                               |

Deployment goes through different stages and each stage updates `deploymentStatus`. Possible deployment statuses:

| Status | Description           |
| ------ | --------------------- |
| 0      | Deployment initiated  |
| 1      | In processing         |
| 10     | Deployment successful |
| 100    | Deployment failed     |

  </div>
  <div class="split_side">
    <br>
      <CodeGroup>
      <CodeGroupItem title="cURL" active>

```sh
curl --location --request POST "https://api.apillon.io/hosting/websites/:websiteUuid/deploy" \
--header "Authorization: Basic :credentials" \
--header "Content-Type: application/json" \
--data-raw "{
    \"environment\": 1
}"
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
  "id": "32eff81a-6b0b-4a92-a5cb-e5cebf6d6c28",
  "status": 200,
  "data": {
    "id": 51,
    "status": 5,
    "websiteId": 4,
    "bucketId": 62,
    "environment": 1,
    "deploymentStatus": 0,
    "cid": null,
    "size": null,
    "number": 11
  }
}
```

  </CodeGroupItem>
  </CodeGroup>
  </div>
</div>

### Get deployment

> Endpoint to get deployment.

<div class="request-url">GET /hosting/websites/:websiteUuid/deployments/:deploymentId</div>

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name         | Description                                                         | Required |
| ------------ | ------------------------------------------------------------------- | -------- |
| websiteUuid  | Website UUID, visible in developer console website overview         | true     |
| deploymentId | Deployment internal number, returned from `deploy` website endpoint | true     |

#### Possible errors

| Code     | Description                |
| -------- | -------------------------- |
| 40406011 | Deployment does not exists |

#### Response fields

| Field            | Type     | Description                                                                                   |
| ---------------- | -------- | --------------------------------------------------------------------------------------------- |
| id               | `number` | Deployment internal number                                                                    |
| status           | `number` | Deployment DB status                                                                          |
| bucketId         | `number` | Internal ID of bucket, which contain files for this environment                               |
| environment      | `number` | Environment to where website will be deployed                                                 |
| deploymentStatus | `number` | Current status of deployment                                                                  |
| cid              | `string` | When deployment is successful, CID points to directory on IPFS, where this page is accessible |
| size             | `number` | Size of website                                                                               |
| number           | `number` | Deployment serial number - for this environment                                               |

Deployment goes through different stages and each stage updates `deploymentStatus`. Possible deployment statuses:

| Status | Description           |
| ------ | --------------------- |
| 0      | Deployment initiated  |
| 1      | In processing         |
| 10     | Deployment successful |
| 100    | Deployment failed     |

  </div>
  <div class="split_side">
    <br>
      <CodeGroup>
      <CodeGroupItem title="cURL" active>

```sh
curl --location --request GET "https://api.apillon.io/hosting/websites/:websiteUuid/deployments/:deploymentId" \
--header "Authorization: Basic :credentials"
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
  "id": "2d7d1b0c-15b1-4816-9aec-857182c7b617",
  "status": 200,
  "data": {
    "id": 51,
    "status": 5,
    "bucketId": 62,
    "environment": 1,
    "deploymentStatus": 10,
    "cid": "QmY3KF4F6Ap5HbrwWpq8dTVaR65Cd674jYXKBayNynDASJ",
    "size": 289,
    "number": 11
  }
}
```

  </CodeGroupItem>
  </CodeGroup>
  </div>
</div>

### Get website

> Endpoint to get website. Endpoint returns basic website data, along with IPNS links.

<div class="request-url">GET /hosting/websites/:websiteUuid</div>

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name        | Description                                                 | Required |
| ----------- | ----------------------------------------------------------- | -------- |
| websiteUuid | Website UUID, visible in developer console website overview | true     |

#### Possible errors

| Code     | Description             |
| -------- | ----------------------- |
| 40406010 | Website does not exists |

#### Response fields

| Field              | Type     | Description                                                 |
| ------------------ | -------- | ----------------------------------------------------------- |
| id                 | `number` | Website id                                                  |
| status             | `number` | Website DB status                                           |
| name               | `string` | Website name                                                |
| description        | `string` | Website description                                         |
| domain             | `string` | Domain for production environment                           |
| bucketUuid         | `string` | UUID of bucket, for file upload                             |
| ipnsStagingLink    | `string` | IPNS address of staging version, on Apillon IPFS gateway    |
| ipnsProductionLink | `string` | IPNS address of production version, on Apillon IPFS gateway |

  </div>
  <div class="split_side">
    <br>
      <CodeGroup>
      <CodeGroupItem title="cURL" active>

```sh
curl --location --request GET "https://api.apillon.io/hosting/websites/:websiteUuid" \
--header "Authorization: Basic :credentials"
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
  "id": "0eb223ce-51c0-4a9b-96ce-331a1cd99603",
  "status": 200,
  "data": {
    "id": 4,
    "status": 5,
    "name": "My test page",
    "description": null,
    "domain": "",
    "bucketUuid": "57aef0fc-84cb-4564-9af2-0f7bfc0ef729",
    "ipnsStagingLink": "https://ipfs.apillon.io/ipns/k2k4r8p6fvcyq5qogaqdtvmqn5vyvyy3khut1llkrz13ls16ocp4gojx",
    "ipnsProductionLink": "https://ipfs.apillon.io/ipns/k2k4r8ng8nqexubrmbwnhsuu1d6n4ebgnxsbdcu9gxk8uv3l67098hge"
  }
}
```

  </CodeGroupItem>
  </CodeGroup>
  </div>
</div>



## Web3 NFTs API

API is for creating and managing NFTs. To prepare images and metadata you can use storage API. To learn more about metadata standards you can visit: https://docs.opensea.io/docs/metadata-standards

### Get NFT Collection

> Get NFT collection by UUID

#### GET /nfts/collections/:uuid

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name  | Description                                                        | Required |
|-------|--------------------------------------------------------------------| -------- |
| uuid  | Unique key of collection. Key is displayed in developer dashboard. | true     |

#### Possible errors

| Code     | Description                       |
| -------- |-----------------------------------|
| 40300000 | Not allowed to access collection. |
| 50012009 | Collection does not exist.        |

#### Response Fields

| Name             | Type        | Description                                                                                              |
|------------------|-------------|----------------------------------------------------------------------------------------------------------|
| createTime       | `DateTime`  | Collection create time.                                                                                  |
| updateTime       | `DateTime`  | Collection last update time.                                                                             |
| collectionType   | `number`    | Type of smart contract to use for collection. Available types are described [here](#collection-types).   |
| collectionUuid   | `string`    | Unique key of a collection.                                                                              |
| symbol           | `string`    | NFT collection symbol (usually 3-4 characters long).                                                     |
| name             | `string`    | NFT collection name.                                                                                     |
| description      | `string`    | NFT collection description.                                                                              |
| maxSupply        | `number`    | Maximal number of NFTs ever in existence (0 stands for unlimited).                                       |
| bucketUuid       | `string`    | UUID of the bucket where metadata is stored.                                                             |
| baseUri          | `string`    | Base URI for collection metadata (token id and file extension is appended to it).                        |
| baseExtension    | `string`    | File extension that is auto appended after token id to form a full URL.                                  |
| isSoulbound      | `boolean`   | Soul bound tokens are NFTs that are bounded to wallet and not transferable.                              |
| isRevokable      | `boolean`   | For revocable collection owner can destroy NFTs at any time.                                             |
| royaltiesFees    | `number`    | Percentage (between 0 and 100) of each NFT sale sent to wallet specified under royalties address.        |
| royaltiesAddress | `string`    | Address where royalties are sent to.                                                                     |
| collectionStatus | `number`    | Apillon internal/database collection status.                                                             |
| contractAddress  | `string`    | Smart address of contract for deployed collection.                                                       |
| transactionHash  | `string`    | Deployment transaction hash/id.                                                                          |
| deployerAddress  | `string`    | Wallet address of deployer.                                                                              |
| chain            | `number`    | Blockchain id on which you want to release your collection.                                              |
| drop             | `boolean`   | Determines if collection is mintable by public.                                                          |
| dropStart        | `number`    | UNIX timestamp which determines public mint opening date and time.                                       |
| dropPrice        | `number`    | Price of NFT at mint stage in token that is used on `chain`.                                             |
| dropReserve      | `number`    | Amount of NFTs reserved by owner.                                                                        |

  </div>
  <div class="split_side">

<CodeGroup>
    <CodeGroupItem title="cURL" active>

```sh
curl --location 'https://api.apillon.io/nfts/collections/:uuid' \
--header 'Authorization: Basic :credentials'
```

  </CodeGroupItem>
</CodeGroup>
<CodeGroup>
  <CodeGroupItem title="Response">

```json
{
    "id": "b5935c73-204d-4365-9f9a-6a1792adab5b",
    "status": 200,
    "data": {
        "createTime": "2023-06-13T10:15:58.000Z",
        "updateTime": "2023-06-13T10:15:58.000Z",
        "collectionType": 1,
        "collectionUuid": "d6355fd3-640d-4803-a4d9-79d875abcb5a",
        "symbol": "NFT",
        "name": "NFT Collection",
        "description": "NFT Collection Description",
        "maxSupply": 1000,
        "bucketUuid": "a9425ff7-4802-4a38-b771-84a790112c30",
        "baseUri": "https://ipfs.apillon.io/metadata/",
        "baseExtension": ".json",
        "isSoulbound": false,
        "isRevokable": true,
        "royaltiesFees": 0.1,
        "royaltiesAddress": "0x4156edbafc5091507de2dd2a53ded551a346f83b",
        "collectionStatus": 0,
        "contractAddress": "0x452101C96A1Cf2cBDfa5BB5353e4a7F235241557",
        "transactionHash": "0x6b97424de3367cd0335b08265787b83053b62bee2d1c8bec1f776936bea4fb26",
        "deployerAddress": "0x4156edbafc5091507de2dd2a53ded551a346f83b",
        "chain": 1287,
        "drop": true,
        "dropStart": 1687251003,
        "dropPrice": 0.1,
        "dropReserve": 5
    }
}
```

</CodeGroupItem>
</CodeGroup>
  </div>
</div>


##### Collection Type

| Number | Description              |
|--------|--------------------------|
| 0      | Generic NFT collection.  |
| 1      | Nestable NFT collection. |

##### Collection Statuses

| Number | Description                              |
|--------|------------------------------------------|
| 0      | Collection was created.                  |
| 1      | Deploying collection was initiated.      |
| 2      | Collection is being deployed.            |
| 3      | Collection was deployed successfully.    |
| 4      | Collection was transferred successfully. |
| 5      | Failed deploying collection.             |


### List NFT Collections

> List NFT collections. Items are paginated and can be filtered and ordered through query parameters.

#### GET /nfts/collections

<div class="split_content">
	<div class="split_side">

#### Query parameters

| Name             | Description                                                                                           | Required |
|------------------|-------------------------------------------------------------------------------------------------------|----------|
| collectionStatus | Collection status. Find available statuses [here](#collection-statuses).                              | false    |
| search           | Search by collection name.                                                                            | false    |
| page             | Collections are paginated by default. This parameter is used to get collections from a specific page. | false    |
| limit            | Number of files on a page (default: 20).                                                              | false    |
| orderBy          | One or multiple properties, separated by a comma, used to order data.                                 | false    |
| desc             | `Boolean` values, mapped to the index of the `orderBy` parameter. Defaults to false.                  | false    |

#### Response
Response is a list of items described [under Response Fields above](#get-nft-collection).

  </div>
  <div class="split_side">

<CodeGroup>
    <CodeGroupItem title="cURL" active>

```sh
curl --location 'https://api.apillon.io/nfts/collections' \
--header 'Authorization: Basic :credentials'
```

  </CodeGroupItem>
</CodeGroup>
<CodeGroup>
  <CodeGroupItem title="Response">

```json
{
    "id": "b5935c73-204d-4365-9f9a-6a1792adab5b",
    "status": 200,
    "data": {
        "items": [
            {
                "createTime": "2023-06-13T10:15:58.000Z",
                "updateTime": "2023-06-13T10:15:58.000Z",
                "collectionType": 1,
                "collectionUuid": "d6355fd3-640d-4803-a4d9-79d875abcb5a",
                "symbol": "NFT",
                "name": "NFT Collection",
                "description": "NFT Collection Description",
                "maxSupply": 1000,
                "bucketUuid": "a9425ff7-4802-4a38-b771-84a790112c30",
                "baseUri": "https://ipfs.apillon.io/metadata/",
                "baseExtension": ".json",
                "isSoulbound": false,
                "isRevokable": true,
                "royaltiesFees": 0.1,
                "royaltiesAddress": "0x4156edbafc5091507de2dd2a53ded551a346f83b",
                "collectionStatus": 0,
                "contractAddress": "0x452101C96A1Cf2cBDfa5BB5353e4a7F235241557",
                "transactionHash": "0x6b97424de3367cd0335b08265787b83053b62bee2d1c8bec1f776936bea4fb26",
                "deployerAddress": "0x4156edbafc5091507de2dd2a53ded551a346f83b",
                "chain": 1287,
                "drop": true,
                "dropStart": 1687251003,
                "dropPrice": 0.1,
                "dropReserve": 5
            }
        ],
        "total": 1
    }
}
```

</CodeGroupItem>
</CodeGroup>
  </div>
</div>


### List Collection Transactions

> List NFT collections. Items are paginated and can be filtered and ordered through query parameters.

#### GET /nfts/collections/:uuid/transactions

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name  | Description                                                        | Required |
|-------|--------------------------------------------------------------------| -------- |
| uuid  | Unique key of collection. Key is displayed in developer dashboard. | true     |

#### Query parameters

| Name              | Description                                                                                             | Required |
|-------------------|---------------------------------------------------------------------------------------------------------|----------|
| transactionStatus | Transaction status.                                                                                     | false    |
| transactionType   | Transaction type.                                                                                       | false    |
| search            | Search by transaction hash.                                                                             | false    |
| page              | Transactions are paginated by default. This parameter is used to get transactions from a specific page. | false    |
| limit             | Number of transactions on a page (default: 20).                                                         | false    |
| orderBy           | One or multiple properties, separated by a comma, used to order data.                                   | false    |
| desc              | `Boolean` values, mapped to the index of the `orderBy` parameter. Defaults to false.                    | false    |

#### Response Fields

| Name              | Type        | Description                                                 |
|-------------------|-------------|-------------------------------------------------------------|
| chainId           | `number`    | Blockchain id on which you want to release your collection. |
| transactionType   | `number`    | Transaction type.                                           |
| transactionStatus | `number`    | Transaction status                                          |
| transactionHash   | `number`    | Transaction hash/id.                                        |
| updateTime        | `DateTime`  | Transaction last update time.                               |
| createTime        | `DateTime`  | Transaction create time.                                     |

  </div>
  <div class="split_side">

<CodeGroup>
    <CodeGroupItem title="cURL" active>

```sh
curl --location 'https://api.apillon.io/nfts/collections/:uuid/transactions' \
--header 'Authorization: Basic :credentials'
```

  </CodeGroupItem>
</CodeGroup>
<CodeGroup>
  <CodeGroupItem title="Response">

```json
{
    "id": "b5935c73-204d-4365-9f9a-6a1792adab5b",
    "status": 200,
    "data": {
        "items": [
            {
                "chainId": 1287,
                "transactionType": 1,
                "transactionStatus": 1,
                "transactionHash": "0xb988c8cda7ec8b441611b208360e0aace9c294e1ca5672375b21e815890a54d1",
                "updateTime": "2023-06-13T10:15:58.000Z",
                "createTime": "2023-06-13T10:15:58.000Z"
            }
        ],
        "total": 1
    }
}
```

</CodeGroupItem>
</CodeGroup>
  </div>
</div>

##### Transaction Types

| Number | Description                 |
|--------|-----------------------------|
| 1      | Deploy Contract             |
| 2      | Transfer Contract Ownership |
| 3      | Mint NFT                    |
| 4      | Set Collection Base URI     |
| 5      | Burn NFT                    |
| 6      | Nest mint NFT               |

##### Transaction Status

| Number | Description |
|--------|-------------|
| 1      | Pending     |
| 2      | Confirmed   |
| 3      | Failed      |
| 4      | Error       |



### Create NFT Collection

> API endpoint that creates NFT collection and deploys it on selected network.

Collection can be created with a few features/functionalities:

- drop: collection can be minted/purchased by users
- revokable: NFTs can be revoked by collection owner who can burn them
- soulbound: NFTs are bound to wallet address and can't be transferred
- royalties: owner can enable royalties to earn specified percentage per each NFT trade

2 types of collections are supported:
1. Generic collection (based on [OpenZeppelins ERC-721](https://docs.openzeppelin.com/contracts/3.x/erc721) NFT standard)
2. Nestable collection which allows nesting NFTs under each other (based on [RMRKs ERC-7401](https://evm.rmrk.app/general-overview/rmrk-legos/nestable) NFT standard)

#### POST /nfts/collections

<div class="split_content">
	<div class="split_side">

#### Body fields

| Name             | Type      | Description                                                                              | Required |
|------------------|-----------|------------------------------------------------------------------------------------------|----------|
| collectionType   | `number`  | Type of smart contract to use when deploying collection (1 for generic, 2 for nestable). | true     |
| chain            | `number`  | Blockchain id on which you want to release your collection.                              | true     |
| symbol           | `string`  | NFT collection symbol (usually 3-4 characters long).                                     | true     |
| name             | `string`  | NFT collection name.                                                                     | true     |
| description      | `string`  | NFT collection description.                                                              | false    |
| maxSupply        | `number`  | Maximal number of NFTs ever in existence (0 stands for unlimited).                       | true     |
| baseUri          | `string`  | Base URI for collection metadata (token id and file extension is appended to it).        | true     |
| baseExtension    | `string`  | File extension that is auto appended after token id to form a full URL.                  | true     |
| isRevokable      | `boolean` | For revocable collection owner can destroy NFTs at any time.                             | true     |
| isSoulbound      | `boolean` | Soul bound tokens are NFTs that are bound to wallet and not transferable.                | true     |
| royaltiesAddress | `string`  | Address where royalties are sent to.                                                     | true     |
| royaltiesFees    | `number`  | Percentage of royalties earned per each NFT trade.                                       | true     |
| drop             | `boolean` | Determines if collection is mintable by public.                                          | true     |
| dropStart*       | `number`  | UNIX timestamp (in seconds) which determines public mint opening date and time.          | true     |
| dropPrice*       | `number`  | Price of NFT at mint stage.                                                              | true     |
| dropReserve*     | `number`  | Amount of NFTs reserved by owner.                                                        | true     |


**Notes:**

*`dropStart`, `dropPrice` and `dropReserve` are only used if `drop` is set to boolean `true`.

#### Possible errors
Beside validation errors (with 422 http status code) these are the error codes may be returned:

| Code     | Description                                   |
| -------- |-----------------------------------------------|
| 40012002 | Collection quota reached                      |
| 50012003 | Failed deploying NFT contract on chain.       |
| 50012010 | Failed to create bucket for storing metadata. |


#### Response
Response payload is described [under Response Fields above](#get-nft-collection).

  </div>
  <div class="split_side">

<CodeGroup>
    <CodeGroupItem title="cURL" active>

```sh
curl --location 'https://api.apillon.io/nfts/collections' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic :credentials' \
--data '{
    "collectionType": 1,
    "chain": 1287,
    "symbol": "NFT",
    "name": "NFT Collection",
    "description": "NFT Collection description",
    "maxSupply": 1000,
    "baseUri": "https://ipfs.apillon.io/metadata/",
    "baseExtension": "json",
    "isRevokable": true,
    "isSoulbound": false,
    "royaltiesAddress": "0x4156edbafc5091507de2dd2a53ded551a346f83b",
    "royaltiesFees": 10,
    "drop": true,
    "dropStart": 1687251003,
    "dropReserve": 5,
    "dropPrice": 0.1
}'
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
    "id": "b5935c73-204d-4365-9f9a-6a1792adab5b",
    "status": 200,
    "data": {
        "createTime": "2023-06-13T10:15:58.000Z",
        "updateTime": "2023-06-13T10:15:58.000Z",
        "collectionType": 1,
        "collectionUuid": "d6355fd3-640d-4803-a4d9-79d875abcb5a",
        "symbol": "NFT",
        "name": "NFT Collection",
        "description": "NFT Collection Description",
        "maxSupply": 1000,
        "bucketUuid": "a9425ff7-4802-4a38-b771-84a790112c30",
        "baseUri": "https://ipfs.apillon.io/metadata/",
        "baseExtension": ".json",
        "isSoulbound": false,
        "isRevokable": true,
        "royaltiesFees": 0.1,
        "royaltiesAddress": "0x4156edbafc5091507de2dd2a53ded551a346f83b",
        "collectionStatus": 0,
        "contractAddress": "0x452101C96A1Cf2cBDfa5BB5353e4a7F235241557",
        "transactionHash": "0x6b97424de3367cd0335b08265787b83053b62bee2d1c8bec1f776936bea4fb26",
        "deployerAddress": "0x4156edbafc5091507de2dd2a53ded551a346f83b",
        "chain": 1287,
        "drop": true,
        "dropStart": 1687251003,
        "dropPrice": 0.1,
        "dropReserve": 5
    }
}
```
  </CodeGroupItem>
  </CodeGroup>
  </div>
</div>


### Transfer Collection

> Transfer collection ownership from a wallet owned by caller to a new wallet address.

#### POST /nfts/collections/:uuid/transfer

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name  | Description                                                        | Required |
|-------|--------------------------------------------------------------------| -------- |
| uuid  | Unique key of collection. Key is displayed in developer dashboard. | true     |

#### Body fields

| Name    | Type     | Description                    | Required |
|---------|----------|--------------------------------| -------- |
| address | `string` | Wallet address of a new owner. | true     |

#### Possible errors
Beside validation errors (with 422 http status code) these are the error codes may be returned:

| Code     | Description                                                                         |
| -------- |-------------------------------------------------------------------------------------|
| 40012003 | Contract can't be transferred to wallet address that already owns this collection.  |
| 40012004 | Transfer transaction already exists.                                                |
| 40300000 | Not allowed to access collection                                                    |
| 50012002 | Collection doesn't exist, wasn't deployed or was already transferred.               |
| 50012004 | Collection transfer failed.                                                         | 

#### Response

Response payload is described [under Response Fields above](#get-nft-collection).

  </div>
  <div class="split_side">

  <CodeGroup>
  <CodeGroupItem title="cURL" active>

```sh
curl --location 'https://api.apillon.io/nfts/collections/:uuid/transfer' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic :credentials' \
--data '{"address": "0x452101C96A1Cf2cBDfa5BB5353e4a7F235241551"}'
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
    "id": "b5935c73-204d-4365-9f9a-6a1792adab5b",
    "status": 200,
    "data": {
        "createTime": "2023-06-13T10:15:58.000Z",
        "updateTime": "2023-06-13T10:15:58.000Z",
        "collectionType": 1,
        "collectionUuid": "d6355fd3-640d-4803-a4d9-79d875abcb5a",
        "symbol": "NFT",
        "name": "NFT Collection",
        "description": "NFT Collection Description",
        "maxSupply": 1000,
        "bucketUuid": "a9425ff7-4802-4a38-b771-84a790112c30",
        "baseUri": "https://ipfs.apillon.io/metadata/",
        "baseExtension": ".json",
        "isSoulbound": false,
        "isRevokable": true,
        "royaltiesFees": 0.1,
        "royaltiesAddress": "0x4156edbafc5091507de2dd2a53ded551a346f83b",
        "collectionStatus": 0,
        "contractAddress": "0x452101C96A1Cf2cBDfa5BB5353e4a7F235241557",
        "transactionHash": "0x6b97424de3367cd0335b08265787b83053b62bee2d1c8bec1f776936bea4fb26",
        "deployerAddress": "0x4156edbafc5091507de2dd2a53ded551a346f83b",
        "chain": 1287,
        "drop": true,
        "dropStart": 1687251003,
        "dropPrice": 0.1,
        "dropReserve": 5
    }
}
```

  </CodeGroupItem>
  </CodeGroup>
  </div>
</div>


### Mint Collection NFTs

> Mint specified amount of NFTs to a wallet address provided in request.

**Note:** if the collection is set as `drop` this endpoint can only mint reserved NFTs.

#### POST /nfts/collections/:uuid/mint

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name  | Description                                                        | Required |
|-------|--------------------------------------------------------------------| -------- |
| uuid  | Unique key of collection. Key is displayed in developer dashboard. | true     |

#### Body fields

| Name             | Type     | Description                     | Required |
|------------------|----------|---------------------------------| -------- |
| receivingAddress | `string` | Wallet address of NFT receiver. | true     |
| quantity         | `number` | Number of NFTs to mint.         | true     |


#### Possible errors
Beside validation errors (with 422 http status code) these are the error codes may be returned:

| Code     | Description                                                              |
| -------- |--------------------------------------------------------------------------|
| 40300000 | Not allowed to access collection.                                        |
| 50012002 | Collection doesn't exist, wasn't deployed or was already transferred.    |
| 50012005 | Error minting NFT.                                                       |
| 50012007 | Total number of minted NFTs would exceed max supply for this collection. |
| 50012008 | All of the reserved NFTs were already minted.                            |

#### Response Fields

| Field            | Type      | Description       |
| ---------------- |-----------|-------------------|
| success          | `boolean` | Status of action. |

  </div>
  <div class="split_side">

  <CodeGroup>
  <CodeGroupItem title="cURL" active>

```sh
curl --location 'https://api.apillon.io/nfts/collections/:uuid/mint' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic :credentials' \
--data '{"receivingAddress": "0x452101C96A1Cf2cBDfa5BB5353e4a7F235241557", "quantity": 1}'
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
    "id": "b5935c73-204d-4365-9f9a-6a1792adab5b",
    "status": 200,
    "data": {
        "success": true
    }
}
```

  </CodeGroupItem>
  </CodeGroup>
  </div>
</div>


### Nest Mint Collection NFTs

> Nest mint specified amount of NFTs under a parent NFT defined by the parent collection UUID and token id.

#### POST /nfts/collections/:uuid/nest-mint

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name  | Description                                      | Required |
|-------|--------------------------------------------------| -------- |
| uuid  | Unique key of (child) collection we are minting. | true     |

#### Body fields

| Name                 | Type     | Description                                       | Required |
|----------------------|----------|---------------------------------------------------| -------- |
| parentCollectionUuid | `string` | Collection UUID of NFT receiving nest-minted NFT. | true     |
| parentNftId          | `number` | Token id of NFT receiving nest-minted NFT.        | true     |
| quantity             | `number` | Number of NFTs to nest-mint.                      | true     |


#### Possible errors
Beside validation errors (with 422 http status code) these are the error codes may be returned:

| Code     | Description                                                              |
| -------- |--------------------------------------------------------------------------|
| 40300000 | Not allowed to access collection.                                        |
| 50012002 | Collection doesn't exist, wasn't deployed or was already transferred.    |
| 50012007 | Total number of minted NFTs would exceed max supply for this collection. |
| 50012008 | All of the reserved NFTs were already minted.                            |
| 50012013 | Parrent collection doesn't support nesting.                              |
| 50012014 | Parrent and child collection chain missmatch.                            |

#### Response Fields

| Field            | Type      | Description       |
| ---------------- |-----------|-------------------|
| success          | `boolean` | Status of action. |

  </div>
  <div class="split_side">

  <CodeGroup>
  <CodeGroupItem title="cURL" active>

```sh
curl --location 'https://api.apillon.io/nfts/collections/:uuid/next-mint' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic :credentials' \
--data '{"parentCollectionUuid": "d6355fd3-640d-4803-a4d9-79d875abcb5a", "parentNftId": 1, "quantity": 1}'
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
    "id": "b5935c73-204d-4365-9f9a-6a1792adab5b",
    "status": 200,
    "data": {
        "success": true
    }
}
```

  </CodeGroupItem>
  </CodeGroup>
  </div>
</div>


### Burn Collection NFT

> Burn specific NFT belonging to collection specified.

**Note:** burning NFTs is only available if `isRevokable` is enabled on collection.

#### POST /nfts/collections/:uuid/burn

<div class="split_content">
	<div class="split_side">

#### URL parameters

| Name  | Description                                                        | Required |
|-------|--------------------------------------------------------------------| -------- |
| uuid  | Unique key of collection. Key is displayed in developer dashboard. | true     |

#### Body fields

| Name     | Type     | Description                                | Required |
|----------|----------|--------------------------------------------| -------- |
| tokenId  | `number` | Non fungible token id that we are burning. | true     |

#### Possible errors
Beside validation errors (with 422 http status code) these are the error codes may be returned:

| Code     | Description                                                           |
| -------- |-----------------------------------------------------------------------|
| 40300000 | Not allowed to access collection.                                     |
| 50012002 | Collection doesn't exist, wasn't deployed or was already transferred. |
| 50012012 | Burning NFT failed.                                                   |

#### Response fields

| Field            | Type      | Description       |
| ---------------- |-----------|-------------------|
| success          | `boolean` | Status of action. |

  </div>
  <div class="split_side">

  <CodeGroup>
  <CodeGroupItem title="cURL" active>

```sh
curl --location 'https://api.apillon.io/nfts/collections/:uuid/burn' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic :credentials' \
--data '{"tokenId": 1}'
```

  </CodeGroupItem>
  </CodeGroup>
  <CodeGroup>
  <CodeGroupItem title="Response">

```json
{
    "id": "b5935c73-204d-4365-9f9a-6a1792adab5b",
    "status": 200,
    "data": {
        "status": true
    }
}
```

  </CodeGroupItem>
  </CodeGroup>
  </div>
</div>