---
title: Workflows
permalink: /docs/workflows/
---

## Setup and Call

The current version of kodou.io supports the calling of functions after a Setup call to provision the necessary resources. In future versions, the resources will be dynamically allocated.

kodou.io has a web UI for user account management and to search for code. The user types in keywords and phrases to identify the functions they want to find and use. The search page returns a list of results indexed with unique urls. These functions are called Isolated Code Executables "ICE". Once the user selects one of the functions it is stored in their account for convenience. The url serves as the reference for the function used in kodou.io API calls. 

kodou.io is also used to call functions in a set of dependencies. This is called an "Environment". The user requests a list of desired dependencies (Maven, Pypi, etc.) and kodou.io provisions resources (an environment) to support function calls to those dependencies.

Assuming the endpoint is `api.kodou.io`, the user will make calls against this endpoint using their API id stored in the HTTP header named Authorization. This header must be used in all kodou API calls to identifiy the account.

### Setup call (ICE)

1. Send a request to the `POST` https://api.kodou.io/library/setup endpoint.

The user will first call the kodou `library/setup` API against the endpoint. 
This call, if successful, will return a token that can be used for any calls to that function. 

Using a url the user selected from a code search on the kodou.io web UI, the following is a Setup call using `cURL`.

```
curl --request POST \
 --header 'Authorization: Bearer {your API ID eyJhbGciOiJXXXXX....}' \
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

### Function call (ICE)

2. Send a request to the `POST` https://api.kodou.io/library/call endpoint.

The function calls will require the Authorization header and a Json payload with the `sessionid` field, a `timeout` field, and an `args` field, a sub-object with the function arguments as fields. 

Protocol Buffers (PB) can be used for arguments as well with the https://api.kodou.io/library/callpb endpoint.

The `sessionid` value serves as a proxy for the function url in the Setup phase. Call this function using the `library/call` API. The request includes the 'Authorization' header, and a Json payload with the `sessionid`, a `timeout` value, and an object named `args` of the function arguments. In the example below, `...hex_digit_to_int` accepts one argument named `c` and returns an integer.

```
curl --request POST \
 --header 'Authorization: Bearer {your API ID eyJhbGciOiJXXXXX....}' \
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

### Setup call (Environment)

1. Send a request to the `POST` https://api.kodou.io/environment/java/setup endpoint.

The user will first call the kodou `environment/java/setup` API against the endpoint and supply the set of dependencies requested.

This request, if successful, will return a token that can be used for calls against those dependencies. 

The following is a Setup call using `cURL` where the Google Guava dependency is desired.

```
curl --request POST \
 --header 'Authorization: Bearer {your API ID eyJhbGciOiJXXXXX....}' \
 --header "Content-Type: application/json" \
 --data '{ "dependencies": [{"groupId":"com.google.guava", "artifactId":"guava","version":"25.0-jre"}] }' \
 https://api.kodou.io
```

The response will be a Json object with a `sessionid` field.
```
{
	"error": false,
    "sessionid": "eyJ0eXAiO..."
}
```

### Function call (Environment)

2. Send a request to the `POST` https://api.kodou.io/environment/kompose endpoint.

`kompose` is one of the commands of kodou.io. It supports composing function calls using constructs such as Pipes.

The function calls will require the Authorization header and a Json payload with the `sessionid` field, a `timeout` field, and other fields (see the other documentation). 

The `sessionid` value serves as a proxy for the resources in the Setup phase. The request includes the 'Authorization' header, and a Json payload with the `sessionid`, a `timeout` value, and other objects, in this case named `pipeline`, of function composition. In the example below, `Double.valueOf( "1001.1" )` is Piped into another function `Double("101.1").compareTo`, returning the value -1 (int). Note, to simplify the example we used well-known Java functions. 

```
curl --request POST \
 --header 'Authorization: Bearer {your API ID eyJhbGciOiJXXXXX....}' \
 --header "Content-Type: application/json" \
 --data '{"sessionid": {from the Setup call}, "timeout":"20000", "pipeline": [
        [
            {	"name": "Double"
            },
            {
                "name": "valueOf",
                "args": [
                    "1001.1"
                ]
            }
        ],
        [
            {
            	"name": "Double",
            	"args": [
            		"101.1"
            	]
            },
            {
            	"name": "compareTo"
            }
        ]
    ]}'
 https://api.kodou.io
```

The http response is a Json object with the function return value like the following:
```
{
    "return": {
    	"error": false,
        "value": -1
    }
}
```
