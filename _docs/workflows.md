---
title: Workflows
permalink: /docs/workflows/
---

## Setup and Call

The current version of kodou.io supports the calling of functions after a Setup call to provision the necessary resources. In future versions, the resources will be dynamically allocated.

kodou.io has a web UI for user account management and to search for code. The user types in keywords and phrases to identify the functions they want to find and use. The search page returns a list of results is indexed with unique urls. Once the user selects one of the functions it is stored in their account for convenience. The url serves as the reference for the function used in kodou.io API calls. 

Assuming the endpoint is `api.kodou.io`, the user will make calls against this endpoint using their API id stored in the HTTP header named X-Consumer-Custom-ID. This header must be used in all kodou API calls to identifiy the account.

### Setup call

1. Send a request to the `POST` https://api.kodou.io/library/setup endpoint.

The user will first call the kodou `library/setup` API against the endpoint. 
This call, if successful, will return a token that can be used for any calls to that function. 

Using a url the user selected from a code search on the kodou.io web UI, the following is a Setup call using `cURL`.

```
curl --request POST \
 --header 'X-Consumer-Custom-ID: {your API ID}' \
 --header "Content-Type: application/json" \
 --data '{"url":"https://github.com/antirez/redis.git|91685eeeb1462edfc12da2e079e76bdbeec0eddb|redis/src/sds.c|910|hex_digit_to_int"}' \
 https://api.kodou.io
```

The response will be a Json object with a `sessionid` field.
```
{
	"error": false,
    "sessionid": "eyJ0eXAiO..."
}
```

### Function call

2. Send a request to the `POST` https://api.kodou.io/library/call endpoint.

The function calls will require the X-Consumer-Custom-ID header and a Json payload with the `sessionid` field, a `timeout` field, and an `args` field, a sub-object with the function arguments as fields.

The `sessionid` value serves as a proxy for the function url in the Setup phase. Call this function using the `library/call` API. The call includes the 'X-Consumer-Custom-ID' header, and a Json payload with the `sessionid`, a `timeout` value, and an object named `args` of the function arguments. In the example below, `...hex_digit_to_int` accetps one argument named `c` and returns an integer.

```
curl --request POST \
 --header 'X-Consumer-Custom-ID: {your API ID}' \
 --header "Content-Type: application/json" \
 --data '{"sessionid": {from the Setup call}, "timeout":"20000", "args":{"c": "A"}}'
 https://api.kodou.io
```

The http response is a Json object with the function return value like the following:
```
{
    "return": {
    	"error": false,
        "value": "15"
    }
}
```


