---
title: Code samples
permalink: /docs/code_samples/
---

# kodou.io API Usage Examples

kodou.io is designed to be simple for clients. The following are a few examples in different languages and approaches of calling the same APIs.

## Generic Example

The most basic kodou.io sequence of operations is to setup the function you want to call, receiving a session id token in response, and using the session id when making calls with function parameters. 

Every function in kodou.io is referenced with a url. For example, the opensource Redis repository has a function named hex_to_digit_to_int. We can reference this function with a kodou.io setup call using the url,
`url:"https://github.com/antirez/redis.git|91685eeeb1462edfc12da2e079e76bdbeec0eddb|redis/src/sds.c|910|hex_digit_to_int"`
 
### Function Setup
Currently, we require a Setup call before a kodou function is called. In the future, this requirement will be relaxed. For all calls the default endpoint, api.kodoi.io, is assumed. You need to provide your API id, supplied by kodou.io, as a custom field, X-Consumer-Custom-ID. The examples in this section can be used with PostMan.

```
POST X-Consumer-Custom-ID:XXXXXXXXXX {
	"url": "https://github.com/antirez/redis.git|91685eeeb1462edfc12da2e079e76bdbeec0eddb|redis/src/sds.c|910|hex_digit_to_int"
}
```

### Function Call w/ Arguments
The call, if successful, will return a Json object with a "sessionid" token field. (i.e.,"sessionid": "eyJ0eXXXXXXXXXX"). The "X-Consumer-Custom-ID" and "sessionid" headers are required to use the "library/call" path and supply arguments to the function.

```
POST {
    "sessionid": "eyJ0eXXXXXXXXXX",
    "timeout":"20000",
    "args": {
    	"c":"A"
    }
}
```

The response will be a Json object with the error state and function return value.

```
{
    "return": {
    	"error": false,
        "value": "15"
    }
}
```

## cURL Example

### Function Setup

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
    "sessionid": "eyJ0eXAiO..."
}
```

### Function Call w/ Arguments

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

## JavaScript

### Function Setup
```
const axios = require('axios');
...

var sessionid;
var callerror = false;

axios({
	method: 'post',
	url: 'https://api.kodou.io/library/setup',
	headers: {'X-Consumer-Custom-ID': 'your API ID' },
	data: {
		url: 'https://github.com/antirez/redis.git|91685eeeb1462edfc12da2e079e76bdbeec0eddb|redis/src/sds.c|910|hex_digit_to_int'
	}
	})
	.then(function (response) {
		sessionid = response.json().return.value;
	})
	.catch(function (error) {
		callerror = true;
	})
```

### Function Call w/ Arguments
```
function(sessionid,argumentDict,timeout) { # argumentDict is dictionary with parameter names as keys and arguments as values
	let payload = {'sessionid': sessionid, 'args': argumentDict, 'timeout':timeout}

	axios({
		method: 'post',
		url: 'https://api.kodou.io/library/setup',
		headers: {'X-Consumer-Custom-ID': 'your API ID' },
		data: payload
		}).then(function (response) {
			if (response.json().return.error)
				console.log('Error occurred!');
			else 
		  		console.log('Function returned ' + response.json().return.value);
		})
}
```

## Python

### Function Setup
```
import requests

url = 'https://api.kodou.io/library/setup'
headers = {'X-Consumer-Custom-ID': 'your API ID' }
payload = {url: 'https://github.com/antirez/redis.git|91685eeeb1462edfc12da2e079e76bdbeec0eddb|redis/src/sds.c|910|hex_digit_to_int'}

resp = requests.post(url, headers=headers, data=payload)

sessionid = 0
callerror = false

if (resp.status_code == 200):
	callerror = false
	sessionid = resp.json().return.value
else:
	callerror = true

```

### Function Call w/ Arguments
```
import requests

def kodou_call(api_id, sessionid, argumentDict): # argumentDict is dictionary with parameter names as keys and arguments as values
	url = 'https://api.kodou.io/library/call'
	headers = {'X-Consumer-Custom-ID': api_id }
	payload = {'sessionid':sessionid, 'args':argumentDict}

	resp = requests.post(url, headers=headers, data=payload)

	if (resp.status_code == 200):
		if (resp.json().return.error == false):
			callerror = false
			print('Function returned ' + resp.json().return.value)
		else
			print('Function call failed')
	else:
		print('Function call failed')

```


