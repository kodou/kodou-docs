---
title: Code samples
permalink: /docs/code_samples/
---

# kodou.io API Usage Examples

kodou.io is designed to be simple for clients. The following are a few examples in different languages and approaches of calling the same APIs.

## Generic Example

The most basic kodou.io sequence of operations is to setup the function you want to call, receiving a session id token in response, and using the session id when making calls with function parameters. 

Every function in kodou.io is referenced with a url. For example, the opensource Redis repository has a function named hex_to_digit_to_int, with a kodou.io url:"https://github.com/antirez/redis.git|91685eeeb1462edfc12da2e079e76bdbeec0eddb|redis/src/sds.c|910|hex_digit_to_int"
 
Currently, we require a Setup call before a kodou function is called. In the future, this requirement will be relaxed. For all calls the default endpoint, api.kodoi.io, is assumed. You need to provide your API id, supplied by kodou.io, as a custom field, X-Consumer-Custom-ID. The examples in this section can be used with PostMan.
```Setup Call
POST X-Consumer-Custom-ID:XXXXXXXXXX {
	"url": "https://github.com/antirez/redis.git|91685eeeb1462edfc12da2e079e76bdbeec0eddb|redis/src/sds.c|910|hex_digit_to_int"
}
```

The call, if successful, will return a Json object with a "sessionid" token field. (i.e.,"sessionid": "eyJ0eXXXXXXXXXX")
```The Call w/ parameters
POST {
    "sessionid": "eyJ0eXXXXXXXXXX",
    "timeout":"20000",
    "args": {
    	"c":"A"
    }
}
```

## cURL Example


## JavaScript


## Python

## Java


