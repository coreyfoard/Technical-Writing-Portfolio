#### Base URL for all API requests is:

```javascript
https://api.zotero.org
```

## Getting all item types
Retrieves all requested item types. An item type will distinguish whether the item is a "book," "note," "report," or any of the other broad categories used to help cite items.

```javascript
GET /itemTypes
```

#### Parameters

| Parameter   | Value       | Description |
| ----------- | ----------- | ----------- |
| format      |“atom”, “bib”, “keys”, “versions”,       |Will return request in the assigned format. |
|key         | string      |If a valid API key is provided, the request will be processed with the identity of the key's owner and the authority granted to the key.
| newer       | integer     |Return only objects modified after the specified library verson.          |
| locale| query| Will request names in other languages.|

#### <summary>Sample Request</summary>

```javascript
curl --location --request GET 'https://api.zotero.org/itemTypes'
```

#### <summary>Sample Response</summary>

<div style="height:300px;width:px;border:1px solid #ccc;font:16px/26px Georgia, Garamond, Serif;overflow:auto;">

```javascript
[
    {
        "itemType": "artwork",
        "localized": "Artwork"
    },
    {
        "itemType": "audioRecording",
        "localized": "Audio Recording"
    },
    {
        "itemType": "bill",
        "localized": "Bill"
    },
    {
        "itemType": "blogPost",
        "localized": "Blog Post"
    },
    {
        "itemType": "book",
        "localized": "Book"
    }
]
```

</div>

## Get updated group metadata
Group metadata includes group titles and descriptions as well as member/role/permissions information. It is separate from group library data.

```javascript
GET /users/<userID>/groups?format=versions
```

#### Parameters

| Parameter   | Value       | Description |
| ----------- | ----------- | ----------- |
| Last-Modified-Version      |header       |Indicates the current version of either a library (for multi-object requests) or an individual object (for single-object requests). If changes are made to a library in a write request, the library's version number will be increased, any objects modified in the same request will be set to the new version number, and the new version number will be returned in the ```Last-Modified-Version``` header. Since modified objects always receive the newly increased library version, the returned ```Last-Modified-Version``` will be the same whether an item is modified as part of a multi-object or single-object request. |
| If-Modified-Since-Version       | header     |Checks for new data. If ```If-Modified-Since-Version: <libraryVersion>``` is passed with a multi-object read request and data has not changed in the library since the specified version, the API will return ```304 Not Modified```. If ```If-Modified-Since-Version: <objectVersion>``` is passed with a single-object read request, a ```304 Not Modified``` will be returned if the individual object has not changed.       |
| If-Unmodified-Since-Version     | header      |Used to ensure that existing data won't be overwritten by a client with out-of-date data. All write requests that modify existing objects must include either the ```If-Unmodified-Since-Version: <version>``` header or a JSON version property for each object. If both are omitted, the API will return a ```428 Precondition Required.```
| newer       | integer     |Return only objects modified after the specified library verson.          |
format      |“atom”, “bib”, “keys”, “versions”,       |Will retrun request in the assigned format. |

#### Sample Request


```javascript
curl --location --request GET 'https://api.zotero.org//users/<userid>/groups?format=versions'
```
#### Sample Response

```javascript
{
  "<groupID>": "<version>",
  "<groupID>": "<version>",
  "<groupID>": "<version>"
}
```
