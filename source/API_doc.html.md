---
title: PCD Documentation

language_tabs:
  - shell: cURL
  - php: PHP



search: true
---

# Introduction

The Predictive Content Delivery (PCD) solution provides a platform into which you can manage content to be pre-positioned on end user mobile devices. We have created this guide to help you provision content in the PCD application.

The API is currently in its first iteration. 
All calls must use the format <code>/ingest/v1/{api_call}</code>.

For example: <code> /ingest/v1/updateContentById </code>


# Authentication

> Example of Content Provider ID and Access Token authentication:

```shell
curl -H "Content-Type: application/json" -d 
'{"access_token":"5278fb26f31bcbf29594bf23bd411879d122a2508d511448d478ab81",
content_id":"5907", "content_provider_id":"cnbc"}'
-k 'https://23.79.234.237/ingest/v1/getContentById' 

curl -H "Content-Type: application/json" -d 
'{"access_token":"5278fb26f31bcbf29594bf23bd411879d122a2508d511448d478ab81",
"content_provider_id":"cnbc"}'
-k 'https://23.79.234.237/ingest/v1/getContentIds' 

curl -H "Content-Type: application/json" -d 
'{"access_token":"5278fb26f31bcbf29594bf23bd411879d122a2508d511448d478ab81",
content_id":"5907", "content_provider_id":"cnbc"}'
-k 'https://23.79.234.237/ingest/v1/getActivityStatusById' 
```

An access token and a content provider ID are provided to content providers who are authorized to use the API .

Also, to submit content to the PCD as an aggregation of multiple content providers, you must have an access token for each content provider. To ensure a successful request, you must use each access token with the corresponding provider.

You must use both values in the body of any post call to authenticate the API request.

The following provides a sample part of an API request (application/json) containing the content provider ID and the access token:

<code>
{ 
"content_provider_id": "akamai_internal",
"access_token": "ab203jduns03kd0",
...
}
</code>


## Return Codes

The response containing a "return code" value returns each API call with a JSON. The meaning of the return code depends on the context of the API call.

A sample of return codes and their interpretation are listed in the following table:

HTTP Status | Error Code | Description
----------- | ---------- | -----------
200 | Success Success. | Content Successfully.
400 | Missing data  | The request JSON is missing essential data to complete the request.
401 | Access denied | The access token validation failed.
422 | Duplicate | Provider tried to add a piece of content with a unique ID that is already in the database.
500 | Unknown | An unknown error occurred while processing the request.
403 | Bad content id  | The provided content ID is not associated with the content provider.
403 | Update not permitted  | The access token cannot update the specified content.
403 | Updating unique id error  | The request is unable to update the unique ID to a value that matches another record in the database.
403 | Purge not permitted | The access token is unable to purge the specified content.


Example of response from a request with error code:

<code>
Response 400 (application/json)
{
"return_code": "Access denied"
}
</code>


# Getting started

## GETTING A CATEGORY LIST

When using the API to add or update content, you must know the list of available categories as the initial PCD application release supports only a certain set of categories with which you can associate content.

To view a list of available content categories, you can use the getContentCategories API call.

*Note:* "Trending" is currently a PCD reserved keyword; it is not a content category.

To get a list of content categories, use the following command:

<code> POST /ingest/v1/getContentCategories </code>

*URI Parameters:* _content_provider_id,access_token_


Parameter | Required | Description
--------- | -------- | ------------
access_token | true | Access token associated with each provider
content_provider_id | true | ID associated with each provider

### RETURN CODES

The following table provides a list of possible return codes:

HTTP Status | Error Code | Description
----------- | ---------- | ------------
200 | Success Success. | Request completed.
400 | Missing data |  Request json is missing essential data to complete the request
401 | Access denied | Access Token validation failed
500 | Unknown | Unknown error occurred while processing the request


### Request

Headers

Content-Type: application/json
<br>
<code>
Body<br>
{ <br>
"content_provider_id": "the id of the content provider",<br>
"access_token": "access token as provisioned by the MNO portal"<br>
}<br>
</code>

#### Response 200

Headers

Content-Type: application/json
<br>
<code>
Body <br>
{ <br>
"content_category":"string of a supported content category" <br>
...<br>
}
</code>

#### Response 400

Headers

Content-Type: application/json
<br>
<code>
Body <br>
{<br>
"return_code": "a return code here" <br>
}
</code>

#### Response 500

Headers

Content-Type: application/json
<br>
<code>
Body<br>
{<br>
"return_code": "Unknown" <br>
}
</code>

> Example Code for Getting a Category List:

```php
<?php
$curl = curl_init();

curl_setopt_array($curl, array(
CURLOPT_URL => "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/getContentCategories",
CURLOPT_RETURNTRANSFER => true,
CURLOPT_CUSTOMREQUEST => "POST",
CURLOPT_POSTFIELDS => "{\"content_provider_id\": \"akamai_internal\",\"access_token\": \"B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE\"}",
CURLOPT_HTTPHEADER => array(
"content-type: application/json"
)
));

$response = curl_exec($curl);
?> 
```

```shell
curl -X POST -H "Content-Type: application/json" -d 
'{"content_provider_id": "akamai_internal",
"access_token": "B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE" 
}' "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/getContentCategories"
```

#### Result:

Response 200

HEADERS

Content-Type:application/json

<code>
Body: <br>
[<br>
"Ads",<br>
"Entertainment",<br>
"Business" <br>
] <br>
</code>


## ADDING CONTENT

This section will explain how to add content to the PCD platform. Currently, PCD enables you to submit video information using two formats - mp4 and m3u8 (HLS). You can add as many variants of the video as you like for a particular video.

Note: Future versions of PCD will support other video formats (for example, dash and flv) using the streams field.

To add content, use the following command:

<code> POST/ingest/v1/addContent </code>

To add an mp4 variant, add a JSON object to the streams array using the following format:

<code>{"url":"mp4 url", "size":"length of mp4 in bytes", "type":"mp4"}</code>


To add an m3u8 variant, add a JSON object to the streams array using the following format:

<code> {"preferred_stream": "bitrate to choose from the m3u8 file" , "url":"m3u8 url", "size":"length of file in bytes", "type":"m3u8"} </code>

URI Parameters: _content_provider_id,access_token_

Parameter | Required | Description
--------- | -------- | -----------
access_token | true | Access token associated with each provider
content_provider_id | true | ID associated with each provider
content_video_width | true |  integer - width in pixels
content_video_height | true | integer - height in pixels
content_bit_rate | true | integer - bits per second
content_categories | true | Content category
content_tags | true |  
thumb_width | true | thumbnail width
thumb_height | true | thumbnail height
thumb_length | true | thumbnail length in bytes
content_title | true | Title associated with the content
content_description | true | description of the content
content_duration | true | thumbnail length in bytes
content_unique_id | true | Unique id of the content as determined by the provider
streams| true | Format of objects (mp4 or m3u8)
publication_timestamp | true | Published date of the content
thumb_url | true | HTTP URL associated with the thumbnail

### RETURN CODES

The following table provides a list of possible return codes:

HTTP Status | Error Code | Description
----------- | ---------- | -----------
200 | Success Success. | Request completed.
400 | Does not conform to spec | Something in the request does not conform to specification; the request cannot continue.
400 | Missing data| The request JSON is missing essential data to complete the request.
401 | Access denied | The access token validation failed.
403 | Add not permitted | The access token cannot add content.
403 | Content category not permitted | The content category is not supported by the PCD application.
403 | No variants of content provided | No mp4 or m3u8 variants of the content were provided.
422 | Duplicate | The provider attempted to add a piece of content with a unique ID that already resides in the database.
500 | Unknown | An unknown error occurred while processing the request.

> Example Code - Adding Content (mp4 format & m3u8 format)


```php
<?php
$curl = curl_init();

curl_setopt_array($curl, array(
CURLOPT_URL => "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/addContent",
CURLOPT_RETURNTRANSFER => true,
CURLOPT_CUSTOMREQUEST => "POST",
CURLOPT_POSTFIELDS => "{\"content_provider_id\": \"akamai_internal\",\"access_token\": \"B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE\",\"content_video_width\": 1024,\"content_video_height\": 720,\"content_bit_rate\": 4096,\"content_categories\": [\"Entertainment\"],\"content_tags\": [{\"text\": \"stehd\",\"weight\": \"2\"}],\"thumb_width\": 1024,\"thumb_height\": 720,\"thumb_length\": 1280,\"content_title\": \"Akamai-Edge 2016\",\"content_description\": \"Where the world \'\s leading brands unlock the opportunities of the internet. You'll meet 2,000 Attendees from 30+ Countries and can network with innovators from brands big and small. Don't miss out on the most productive event for your online business.\",\"content_duration\": 65,\"content_unique_id\": \"a-lL66tBZZA\",\"streams\": [{\"url\": \"http://tools.watch-now.co/a-lL66tBZZA/index.mp4\",\"type\": \"mp4\",\"size\": \"12324\"}],\"publication_timestamp\": \"2016-08-09\",\"thumb_url\": \"http://tools.watch-now.co/a-lL66tBZZA/video.jpg\"}", CURLOPT_HTTPHEADER => array(
"content-type: application/json"
),
));

$response = curl_exec($curl);

?> 
```

```php
<?php 
$curl = curl_init();

curl_setopt_array($curl, array(
CURLOPT_URL => "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/addContent",
CURLOPT_RETURNTRANSFER => true,
CURLOPT_CUSTOMREQUEST => "POST",
CURLOPT_POSTFIELDS => "{\"content_provider_id\": \"akamai_internal\",\"access_token\": \"B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE\",\"content_video_width\": 1024,\"content_video_height\": 720,\"content_bit_rate\": 4096,\"content_categories\": [\"Entertainment\"],\"content_tags\": [{\"text\": \"stehd\",\"weight\": \"2\"}],\"thumb_width\": 1024,\"thumb_height\": 720,\"thumb_length\": 1280,\"content_title\": \"Akamai-Edge 2016\",\"content_description\": \"Where the world\'s leading brands unlock the opportunities of the internet. You'll meet 2,000 Attendees from 30+ Countries and can network with innovators from brands big and small. Don't miss out on the most productive event for your online business.\",\"content_duration\": 65,\"content_unique_id\": \"a-lL66tBZZA\",\"streams\": [{\"url\": \"http://tools.watch-now.co/a-lL66tBZZA/index.m3u8\",\"type\": \"m3u8\",\"size\": \"12324\"}],\"publication_timestamp\": \"2016-08-09\",\"thumb_url\": \"http://tools.watch-now.co/a-lL66tBZZA/video.jpg\"}", CURLOPT_HTTPHEADER => array(
"content-type: application/json"
),
));

$response = curl_exec($curl);

?> 
```

```shell
curl -X POST -H "Content-Type: application/json"" -d '{"content_provider_id": "akamai_internal",
"access_token": "B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE",
"content_video_width": 1024,
"content_video_height": 720,
"content_bit_rate": 4096,
"content_categories": [
"Entertainment"
],
"content_tags": [{
"text": "stehd",
"weight": "2"
}],
"thumb_width": 1024,
"thumb_height": 720,
"thumb_length": 1280,
"content_title": "Akamai-Edge 2016",
"content_description": "Where the world's leading brands unlock the opportunities of the internet. You'll meet 2,000 Attendees from 30+ Countries and can network with innovators from brands big and small. Don't miss out on the most productive event for your online business.",
"content_duration": 65,
"content_unique_id": "a-lL66tBZZA",
"streams": [{
"url": "http://tools.watch-now.co/a-lL66tBZZA/index.m3u8",
"type": "m3u8",
"size": "12324"
}],
"publication_timestamp": "2016-08-09",
"thumb_url": "http://tools.watch-now.co/a-lL66tBZZA/video.jpg"}' "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/addContent"
```
### Request

Headers

Content-Type: application/json

<code>Body
{
"content_provider_id": "the id of the content provider",
"access_token": "access token as provisioned by the MNO portal",
"content_video_width": integer - width in pixels,
"content_video_height": integer - height in pixels,
"content_bit_rate": integer - bits per second,
"content_categories": [
""
],
"content_description": "string - textual long form description of the content"
"content_duration": integer - seconds,
"content_unique_id": string - Unique id of the content as determined by the provider,
"content_tags": [
{
"text": "string - Tags associated with the Content",
"weight": "string - Weight associated with the Tag"
},
],
"content_title": "string - Title associated with the content",
"streams": [
_objects of this format_
{
"url": "string - mp4 url of content", 
"type": "mp4"
"size": "string - length of mp4 in bytes"
},
_AND/OR objects of this format_
{
"preferred_stream": "bitrate to choose from the m3u8 file",
"url": "string = url of m3u8 file",
"type": "m3u8",
"size": "length of stream in bytes"
}
]
"publication_timestamp": "string â€“ YYYY-MM-DD HH:MM:SS",
"thumb_width": integeger - thumbnail width,
"thumb_height": integer - thumbnail height,
"thumb_length": integer - thumbnail length in bytes,
"thumb_url": "string - HTTP URL associated with the thumbnail" ,
"ad_url": "OPTIONAL: string - HTTP URL of ad to play with content",
"sdk_metadata_passthrough": "OPTIONAL: string- any metadata a customer wants to pass directly through the sdk for later use (e.g. a json, xml,text as a string) ",
}
</code>

### Response 200

Headers

Content-Type: application/json

<code>Body
{
"return_code": "Success",
"content_id": "string - id of content"
}
</code>

### Response 400

Headers

Content-Type: application/json

<code> Body
{
"return_code": "a return code here"
}
</code>

### Response 500

Headers

Content-Type: application/json

<code>Body
{
"return_code": "Unknown" 
}
</code>

### Result:
### Response 200

HEADERS

Content-Type:application/json

<code>
Body:

{
"content_id": 346708,
"return_code": "Success"
}
</code>

## UPDATING CONTENT METADATA BY ID

This action enables you to update a single content entry using its unique ID.

To update content by ID, use the following command:

<code> POST/ingest/v1/updateContentById </code>

URI Parameters: content_provider_id,access_token,content_id

Parameter | Required | Description
--------- | -------- | ------------
access_token | true | Access token associated with each provider
content_provider_id |true | ID associated with each provider
content_id | true |  ID of the content created by the provider
content_video_width | true | integer - width in pixels
content_video_height | true | integer - height in pixels
content_bit_rate | true | integer â€“ bits per second
content_categories | true | Content category
content_tags  | true |  
thumb_width | true | thumbnail width
thumb_height | true |  thumbnail height
thumb_length  | true | thumbnail length in bytes
content_title | true | Title associated with the content
content_description | true | description of the content
content_duration | true | thumbnail length in bytes
content_unique_id | true | Unique id of the content as determined by the provider
streams | true | Format of objects (mp4 or m3u8)
publication_timestamp | true | Published date of the content
thumb_url | true | HTTP URL associated with the thumbnail


### RETURN CODES

The following table provides a list of possible return codes:

HTTP Status | Error Code | Description
----------- | ---------- | ------------
200 | Success Success. | Request completed.
400 | Missing data | The request JSON is missing essential data to complete the request.
401 | Access denied | The access token validation failed.
403 | Bad content id  | The provided content ID is not associated with any content.
403 | Update not permitted | The provided access token does not have permission to update content.
403 | Update unique id error | Updating the content ID results in more than one entry with the same ID; IDs must be unique.
403 | Content category not permitted | You are attempting to update the content with an unsupported category.
500 | Unknown |An unknown error occurred while processing the request.

### Request 200

Headers

Content-Type: application/json

<code>
Body
{
"content_id": "the id of content",
"content_provider_id": "the id of the content provider",
"access_token": "access token as provisioned by the MNO portal",
_At least one field from below to update:_
"content_video_width": integer - width in pixels,
"content_video_height": integer - height in pixels,
"content_bit_rate": integer â€“ bits per second,
"content_categories": [
""
],
"content_description": "string - textual long form description of the content"
"content_duration": integer â€“ seconds,
"content_unique_id": "string â€“ Unique id of the content as determined by the provider",
"content_tags": [
{
"text": "string â€“ Tags associated with the Content",
"weight": "string â€“ Weight associated with the Tag"
},
]
"content_title": "string â€“ Title associated with the content",
"publication_timestamp": "string â€“ YYYY-MM-DD HH:MM:SS",
"thumb_width": integer â€“ thumbnail width,
"thumb_height": integer â€“ thumbnail height,
"thumb_length": integer â€“ thumbnail length in bytes,
"thumb_url": "string - HTTP URL associated with the thumbnail",
"ad_url": "string- HTTP URL of ad to play with content",
"sdk_metadata_passthrough": "string- any metadata a customer
wants to pass directly through the sdk for later use (e.g. a json, xml, or text as a string) ",
"streams": [
_objects of this format_
{
"url": "string - mp4 url of content",
"type": "mp4"
"size": "string - length of mp4 in bytes"
},
_AND/OR objects of this format_
{
"preferred_stream": "bitrate to choose from the m3u8 file",
"url": "string = url of m3u8 file",
"type": "m3u8",
"size": "length of stream in bytes"
}
]
}
</code>

### Response 200

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "Success"
}
</code>

### Response 400

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "a return code here"
}
</code>

### Response 500

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "Unknown"
}
</code>


> Example Code - Updating Content Metadata By ID

```php
<?php
$curl = curl_init();
curl_setopt_array($curl, array(
CURLOPT_URL => "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/updateContentById",
CURLOPT_RETURNTRANSFER => true,
CURLOPT_CUSTOMREQUEST => "POST",
CURLOPT_POSTFIELDS => "{\"content_provider_id\": \"akamai_internal\",\"content_id\":346708,\"access_token\": \"B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE\",\"content_title\": \"Akamai-Edge 2016\"}",
CURLOPT_HTTPHEADER => array(
"content-type: application/json"
)
));

$response = curl_exec($curl);
?>
```

```shell
curl -X POST -H "Content-Type: application/json" -d '{"content_provider_id": "akamai_internal",
"content_id":346708,
"access_token": "B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE",
"content_title": "Akamai-Edge 2016"}' "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/updateContentById"
```

### Result:
### Response200

HEADERS

Content-Type:application/json

<code>
Body:
{
"return_code": "Success"
}
</code>

## PURGE BY CONTENT ID

This action enables you to purge (remove) content from PCD using the content ID.

Use the following command to purge content by ID:

<code>POST/ingest/v1/requestPurgeById</code>

URI Parameters: content_provider_id,access_token,content_id

Parameter | Required |Description
--------- | -------- |-----------
access_token | true | Access token associated with each provider
content_provider_id | true | ID associated with each provider
content_id | true | ID of the content created by the provider


### RETURN CODES

The following table provides a list of possible return codes:

HTTP Status | Error Code | Description
----------- | ---------- | -----------
200 | Success | Success. Request completed.
400 | Missing data | The request JSON is missing essential data to complete the request.
401 | Access denied | The access token validation failed.
403 | Bad content id | The provided content ID is not associated with any content.
403 | Purge not permitted | The provided access token cannot purge content.
500 | Unknown | An unknown error occurred while processing the request.

### Request Get content ids

Headers

Content-Type: application/json

<code>
Body
{
"content_id": "the id of content",
"content_provider_id": "the id of the content provider",
"access_token": "access token as provisioned by the MNO portal"
}
</code>

### Response 200

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "Success"
}
</code>

### Response 400

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "a return code here"
}
</code>

### Response 500

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "Unknown"
}
</code>

>Example Code for Purging Content By ID:

```php
<?php
$curl = curl_init();

curl_setopt_array($curl, array(
CURLOPT_URL => "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/requestPurgeById",
CURLOPT_RETURNTRANSFER => true,
CURLOPT_CUSTOMREQUEST => "POST",
CURLOPT_POSTFIELDS => "{\"content_provider_id\": \"akamai_internal\",
\"access_token\": \"B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE\",
\"content_id\":346708}",
CURLOPT_HTTPHEADER => array(
"content-type: application/json"
)
));

$response = curl_exec($curl);
?>
```
```shell
curl -X POST -H "Content-Type: application/json" -d '{
"content_provider_id": "akamai_internal",
"access_token": "B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE",
"content_id":346708
}' 
"https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/requestPurgeById"
```

### Result:

### Response200

HEADERS

Content-Type:application/json

<code>
Body:
{
"return_code": "Success"
}
</code>


## GETTING ALL CONTENT IDs

This action returns all associated content IDs.

To view all content IDs, use the following command:

<code> POST/ingest/v1/getContentIds </code>

URI Parameters: content_provider_id,access_token

Parameter | Required | Description
--------- | -------- | ------------
access_token | true | Access token associated with each provider
content_provider_id | true | ID associated with each provider

### RETURN CODES

The following table provides a list of possible return codes:

HTTP Status | Error Code | Description
----------- | ---------- | -----------
200 | Active | Success. Request completed. The content is active.
200 | Inactive | Success. Request completed. The content is inactive.
400 | Missing data | The request JSON is missing essential data to complete the request.
401 | Access denied | The access token validation failed.
403 | Bad content id | The provided content ID is not associated with any content.
500 | Unknown | An unknown error occurred while processing the request.

### Request Get content ids

Headers

Content-Type: application/json

<code>
Body
{
"content_provider_id": "the id of the content provider",
"access_token": "access token as provisioned by the MNO portal"
}
</code>

### Response 200

Headers

Content-Type: application/json

<code>
Body
[
{
"content_id": 
},
â€¦
]
</code>

### Response 400

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "a return code here"
} 
</code>

### Response 500

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "Unknown"
}
</code>

> Example Code for Getting All Content By IDs:

```php
<?php
$curl = curl_init();

curl_setopt_array($curl, array(
CURLOPT_URL => "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/getContentByIds",
CURLOPT_RETURNTRANSFER => true,
CURLOPT_CUSTOMREQUEST => "POST",
CURLOPT_POSTFIELDS => "{\"content_provider_id\": \"akamai_internal\",\"access_token\": \"B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE\"}",
CURLOPT_HTTPHEADER => array(
"content-type: application/json"
)
));
$response = curl_exec($curl);
?>
```
```shell
curl -X POST -H "Content-Type: application/json" -d '{
"content_provider_id": "akamai_internal",
"access_token": "B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE",
"content_id":346708
}' "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/getContentByIds"
```
### Result:
### Response 200

Headers

Content-Type:application/json

<code>
[
{
"content_id": 346708
}
] 
</code>

## GETTING CONTENT METADATA BY ID

This call enables you to retrieve meta-information of a particular content using the content ID.

Use the following command to view particular content by content ID:

<code> POST/ingest/v1/getContentById </code>

URI Parameters:content_provider_id,access_token,content_id

Parameter | Required | Description
--------- | -------- | -----------
access_token | true | Access token associated with each provider
content_provider_id | true | ID associated with each provider
content_id | true | ID of the content created by the provider

### RETURN CODES

The following table provides a list of possible return codes:

HTTP Status | Error Code | Description
----------- | ---------- | -----------
200 | Active  | Success. Request completed. The content is active.
200 | Inactive | Success. Request completed. The content is inactive.
400 | Missing data | The request JSON is missing essential data to complete the request.
401 | Access denied | The access token validation failed.
403 | Bad content id | The provided content ID is not associated with any content.
500 | Unknown | An unknown error occurred while processing the request.uly

### Request Get content by id

Headers

Content-Type: application/json

<code>
Body
{
"content_id": "the id of content",
"content_provider_id": "the id of the content provider",
"access_token": "access token as provisioned by the MNO portal"
}
</code>

### Response 200

Headers

Content-Type: application/json

<code>
Body
{
"content_id": "the id of content",
"content_provider_id": "the id of the content provider",
"content_video_width": integer - width in pixels,
"content_video_height": integer - height in pixels,
"content_bit_rate": integer - bits per second,
"content_categories": [
""
],
"content_description": "string - textual long form description of the content>â€
"content_duration": integer - seconds,
"content_unique_id": "string - Unique id of the content as determined by the provider",
"content_tags": [
{
"text": "string - Tags associated with the Content",
"weight": "string - Weight associated with the Tag"
},
]
"content_title": "string â€“ Title associated with the content",
"publication_timestamp": "string - YYYY-MM-DD HH:MM:SS",
"thumb_width": integer - thumbnail width,
"thumb_height": integer - thumbnail height,
"thumb_length": integer - thumbnail length in bytes,
"thumb_url": "string - HTTP URL associated with the thumbnail" ,
"ad_url": "string- HTTP URL of ad to play with content",
"sdk_metadata_passthrough": "string- any metadata a customer
wants to pass directly through the sdk for later use (e.g. a json, xml, or text as a string) ",
"streams": [
_objects of this format_
{
"url": "string - mp4 url of content", 
"type": "mp4"
"size": "string - length of mp4 in bytes"
},
_AND/OR objects of this format_
{
"preferred_stream": "bitrate to choose from the m3u8 file",
"url": "string = url of m3u8 file",
"type": "m3u8",
"size": "length of stream in bytes"
}
]
}
</code>

### Response 400

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "a return code here"
}
</code>

### Response 500

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "Unknown"
}
</code>

> Example code for Getting Content Metadata By IDs:

```php
<?php
$curl = curl_init();
curl_setopt_array($curl, array(
CURLOPT_URL => "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/getContentById",
CURLOPT_RETURNTRANSFER => true,
CURLOPT_CUSTOMREQUEST => "POST",
CURLOPT_POSTFIELDS => "{\"content_provider_id\": \"akamai_internal\",\"access_token\": \"B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE\",\"content_id\": \"346708\"}",
CURLOPT_HTTPHEADER => array(
"content-type: application/json"
)
));

$response = curl_exec($curl);
?>
```
```shell
curl -X POST -H "Content-Type: application/json" -d '{
"content_provider_id": "akamai_internal",
"access_token": "B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE",
"content_id":346708
}' "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/getContentById"
```

### Result:

### Response 200

Headers

Content-Type:application/json

<code>
Body:
{
"content_length": 12324,
"content_description": "Where the worlds leading brands unlock the opportunities of the internet.
You'll meet 2,000 Attendees from 30+ Countries and can network with innovators from brands big and small.
Don't miss out on the most productive event for your online business.",
"content_provider_id": "akamai_internal",
"sdk_metadata_passthrough": null,
"content_video_height": "720",
"thumb_height": "720",
"content_title": "Akamai Edge 2016",
"content_expiration": null,
"preselected": null,
"persistToExpiration": null,
"publication_timestamp": "2016-08-09 00:00:00",
"content_tags": [
{
"text": "stehd",
"weight": "2"
}
],
"thumb_width": "1024",
"long_lived": true,
"content_type": "video/m3u8",
"content_categories": [
"Entertainment"
],
"thumb_url": "http://tools.watch-now.co/a-lL66tBZZA/video.jpg",
"content_video_width": "1024",
"content_unique_id": "a-lL66tBZZA",
"content_bit_rate": 4096,
"thumb_length": 1280,
"streams": [
{
"url": "http://tools.watch-now.co/a-lL66tBZZA/index.m3u8",
"type": "m3u8",
"lowcost_map": {
"url": null,
"use": false,
"host": null
},
"size": "12324"
}
],
"content_duration": 65,
"ad_url": null
}
</code>


## GETTING CONTENT STATUS

The status attribute associated with all the content that is used by the PCD application, enables you to activate and deactivate content without having to add and delete content from the PCD application. You can both obtain the content status and activate the content by ID.

This call enables you to determine the active state of content using the ID.

To get the activity status of content using a specific ID, use the following command:

<code> POST/ingest/v1/getActivityStatusById </code>

URI Parameters: content_provider_id,access_token,content_id

Parameter | Required | Description
--------- | -------- | -----------
access_token | true | Access token associated with each provider
content_provider_id |true | ID associated with each provider
content_id  | true | ID of the content created by the provider

### RETURN CODES

The following table provides a list of possible return codes:

HTTP Status | Error Code |  Description
----------- | ---------- | -------------
200 | Active | Success. Request completed. The content is active.
200 | Inactive | Success. Request completed. The content is inactive.
200 | Purged | Success. The content is purged from the PCD application.
400 | Missing data | The request JSON is missing essential data to complete the request.
401 | Access denied | The access token validation failed.
403 | Bad content id | The provided content ID is not associated with any content.
500 | Unknown | An unknown error occurred while processing the request.

### Request Get status of content

Headers

Content-Type: application/json

<code>
Body
{
"content_id": "the id of content",
"content_provider_id": "the id of the content provider",
"access_token": "access token as provisioned by the MNO portal"
}
</code>

### Response 200

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "a return code here"
}
</code>

### Response 400

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "a return code here"
}
</code>

### Response 500

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "Unknown"
}
</code>

> Example code for Getting Content Status:

```php
<?php
$curl = curl_init();

curl_setopt_array($curl, array(
CURLOPT_URL => "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/getActivityStatusById",
CURLOPT_RETURNTRANSFER => true,
CURLOPT_CUSTOMREQUEST => "POST",
CURLOPT_POSTFIELDS => "{\"content_provider_id\": \"akamai_internal\",\"access_token\": \"B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE\",\"content_id\": \"346708\"}",
CURLOPT_HTTPHEADER => array(
"content-type: application/json"
)
));

$response = curl_exec($curl);
?>
```

```shell
curl -X POST -H "Content-Type: application/json" -d '{
"content_provider_id": "akamai_internal",
"access_token": "B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE",
"content_id":346708
}' "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/getActivityStatusById"
```

### Result:

### Response200

HEADERS

Content-Type:application/json

<code>
Body:
{
"return_code": "Active"
}
</code>

## ACTIVATING CONTENT

This call enables you to set a given content status as ACTIVE.

Using the following command, you can set a given content status to ACTIVE:

<code> POST/ingest/v1/setContentActiveById </code>

URI Parameters: content_provider_id,access_token,content_id

Parameter | Required | Description
--------- | -------- | -----------
access_token | true | Access token associated with each provider
content_provider_id | true | ID associated with each provider
content_id | true | ID of the content created by the provider

### RETURN CODES

The following table provides a list of possible return codes:

HTTP Status | Error Code | Description
----------- | ---------- | -----------
200 | Active | Success. Request completed.
400 | Missing data | The request JSON is missing essential data to complete the request.
401 | Access denied | The access token validation failed.
403 | Bad content id | The content ID is not associated with any content.
403 | Already active | The content is already marked as active.
403 | Content is purged from PVOC | The content does not reside on the PCD application.
500 | Unknown | An unknown error occurred while processing the request.


### Request Set content as Active

Headers

Content-Type: application/json

<code>
Body
{
"content_id": "the id of content",
"content_provider_id": "the id of the content provider",
"access_token": "access token as provisioned by the MNO portal"
}
</code>

### Response 200

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "a return code here"
}
</code>

### Response 400

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "a return code here"
}
</code>

### Response 500

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "Unknown"
}
</code>

> Example code for Activating Content:

```php
<?php
$curl = curl_init();

curl_setopt_array($curl, array(
CURLOPT_URL => "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/setContentActiveById",
CURLOPT_RETURNTRANSFER => true,
CURLOPT_CUSTOMREQUEST => "POST",
CURLOPT_POSTFIELDS => "{\"content_provider_id\": \"akamai_internal\",\"access_token\": \"B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE\",\"content_id\": \"346708\"}",
CURLOPT_HTTPHEADER => array(
"content-type: application/json"
)
));

$response = curl_exec($curl);
?>
```

```shell
curl -X POST -H "Content-Type: application/json" -d '{
"content_provider_id": "akamai_internal",
"access_token": "B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE",
"content_id":346708
}' "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/setContentActiveById"
```

### Result:

### Response 200

HEADERS

Content-Type:application/json

<code>
Body:
{
"return_code": "Success"
}
</code>

## SETTING CONTENT AS INACTIVE

This call enables you to set a specific content status as INACTIVE.

Using the following command, set a given content status to INACTIVE:

<code> POST/ingest/v1/setContentInactiveById </code>

URI Parameters: content_provider_id,access_token,content_id

Parameter | Required | Description
--------- | -------- | ------------
access_token | true | Access token associated with each provider
content_provider_id | true | ID associated with each provider
content_id | true | ID of the content created by the provider

### RETURN CODES

The following table provides a list of possible return codes:

HTTP Status | Error Code | Description
----------- | ---------- | -----------
200 | Active | Success. Request completed.
400 | Missing data | The request JSON is missing essential data to complete the request.
401 | Access denied | The access token validation failed.
403 | Bad content id | Content ID is not associated with any content.
403 | Aready inactive | The content is already marked as active.
403 | Content is purged from PVOC | The content does not reside on the PCD application.
500 | Unknown | An unknown error occurred while processing the request.

### Request Set content as Inactive

Headers

Content-Type: application/json

<code>
Body
{
"content_id": "the id of content",
"content_provider_id": "the id of the content provider",
"access_token": "access token as provisioned by the MNO portal"
}
</code>

### Response 200

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "a return code here"
}
</code>

### Response 500

Headers

Content-Type: application/json

<code>
Body
{
"return_code": "Unknown"
}
</code>

> Example Code Setting Content As Inactive:

```php
<?php
$curl = curl_init();

curl_setopt_array($curl, array(
CURLOPT_URL => "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/setContentInactiveById",
CURLOPT_RETURNTRANSFER => true,
CURLOPT_CUSTOMREQUEST => "POST",
CURLOPT_POSTFIELDS => "{\"content_provider_id\": \"akamai_internal\",\"access_token\": \"B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE\",\"content_id\": \"346708\",}",
CURLOPT_HTTPHEADER => array(
"content-type: application/json"
)
));

$response = curl_exec($curl);
?>
```

```shell
curl -X POST -H "Content-Type: application/json" -d '{
"content_provider_id": "akamai_internal",
"access_token": "B5672A98CB9A989755C5D419296237F5E00F9D759F4533B5E6E2541C88798ABE",
"content_id":346708
}' "https://ingest-sdktrial2.pvoc-anaina.com/ingest/v1/setContentInactiveById"
```

### Result:

### Response 200

HEADERS

Content-Type:application/json

<code>
Body:
{
"return_code": "Success"
}
</code>
