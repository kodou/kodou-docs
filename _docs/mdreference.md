---
title: Reference example
permalink: /docs/mdreference/
---

### Setup a kodou function by url reference


#### `POST https://api.kodou.io/library/setup` 

#### Headers

X-Consumer-Custom-ID: {your API ID}

#### Parameters

None

#### JSON payload
```
{
	"url": {kodou function url reference}
}
```

#### Example Request

```
curl --request POST \
 --header 'X-Consumer-Custom-ID: {your API ID}' \
 --header "Content-Type: application/json" \
 --data '{"url":"https://github.com/antirez/redis.git|91685eeeb1462edfc12da2e079e76bdbeec0eddb|redis/src/sds.c|910|hex_digit_to_int"}' \
 https://api.kodou.io
```

#### Example Response

```
{
    "sessionid": "eyJ0eXAiO..."
}
```

Element | Type | Description
------- | ---- | -----------
`sessionid` | String | sessionid for function calls. When the error field is true the sessionid is valid.

#### Error and Status Codes

Code | Message | Meaning
---- | ------- | -------
200 | OK | kodou function setup is completed. Return value in JSON payload with a sessionid field
400 | url format is incorrect. | Couldn't parse the kodou url
400 | Missing json fields. | Didn't find the kodou url in the JSON payload
500 | Internal error | The internal kodou.io system failed. Call Customer Support.

#### Call a kodou function w/ a session id

#### `POST https://api.kodou.io/library/call`

#### Headers

X-Consumer-Custom-ID: {your API ID}

#### Parameters

None

#### Example Request

```
curl --request POST \
 --header 'X-Consumer-Custom-ID: {your API ID}' \
 --header "Content-Type: application/json" \
 --data '{"sessionid": {from the Setup call}, "timeout":"20000", "args":{"c": "A"}}'
 https://api.kodou.io
```

#### Example Response

```
{
    "return": {
    	"error": false,
        "value": "15"
    }
}
```

Element | Type | Description
------- | ---- | -----------
`return` | JSON | Response from kodou function call parent field.
`error` | Boolean | If `error` is true then function call failed for some reason. 
`value` | String | If `error` is false then `value` is the return value from the function call.

#### Error and Status Codes

Code | Message | Meaning
---- | ------- | -------
200 | OK | Function call completed. Return value in JSON payload, an error field is false if call succeeded correctly.
400 | Internal error | The internal kodou.io system failed. Call Customer Support.
400 | Unable to complete request because session id doesn't match user id | session Token doesnt match user's session
500 | Internal error | The internal kodou.io system failed. Call Customer Support.

#### Call a kodou function w/ Protocol Buffers arguments w/ a session id 

#### `POST https://api.kodou.io/library/call`

#### Headers

X-Consumer-Custom-ID: {your API ID}

#### Parameters

None

#### Example Request

```
curl --request POST \
 --header 'X-Consumer-Custom-ID: {your API ID}' \
 --header "Content-Type: application/json" \
 --data '{"sessionid": {from the Setup call}, "timeout":"20000", "argspb":{Base64 encoded String Protobufs arguments}}}'
 https://api.kodou.io
```

#### Example Response

```
{
    "return": {
    	"error": false,
        "value": "XXXXX"
    }
}
```

Element | Type | Description
------- | ---- | -----------
`return` | JSON | Response from kodou function call parent field.
`error` | Boolean | If `error` is true then function call failed for some reason. 
`value` | Base64 Encoded String | If `error` is false then `value` is the return value from the function call Protocol Buffer encoded.

#### Error and Status Codes

Code | Message | Meaning
---- | ------- | -------
200 | OK | Function call completed. Return value in JSON payload, an error field is false if call succeeded correctly.
400 | Internal error | The internal kodou.io system failed. Call Customer Support.
400 | Unable to complete request because session id doesn't match user id | session Token doesnt match user's session
500 | Internal error | The internal kodou.io system failed. Call Customer Support.

