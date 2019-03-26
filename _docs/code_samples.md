---
title: Code samples
permalink: /docs/code_samples/
---

# kodou.io API

kodou.io is designed to be a minimal protocol for clients. The client either requests a specific function, known as an isolated code executable (ICE), or specifies a set of dependencies to be available in an "Environment" for a variety of function calls exported by the dependencies.

Thus, there are two kinds of API services: 
+ Functions (ICE)
+ Environments

The following are a few examples in different languages and approaches. 

## Generic Environment Example

kodou.io Environments are an API based way to perform function calls on a set of dependencies.

For example, here is the usual way to call a function in a Python program. 

First, *the numpy package must be installed on the operating system*. In the source code you import the required dependencies, then call functions and/or build the necessary objects. 

```python
import numpy

# Sum up the integers in an array
args = [1,2,3]

# Create the numpy array, specifying ints as the array type. Then call the sum function on the object.
result = numpy.array(args).sum()

```

A client can perform the above dynamically using the kodou.io API. 
After an API request specifying the dependencies, any function (supported by the dependencies) can be call over the API. 

kodou.io API calls replace DevOps and compilers.

The basic kodou.io environment protocol sequence is to specify the dependencies/libraries for the environment, receiving a session id token in response, and using the session id when making function calls. 

Every function called must be exported by a library in the dependency set, aka. environment. The functions are identified with the explicit name sequence starting with the module name, and followed by each subsequent attribute name. This is called the name-path-array, a descriptive form of the usual dot-separated way we write object attribute references in code. 

### Environment Setup
For all calls the default endpoint, api.kodoi.io, is assumed.

You need to provide your API id, supplied by kodou.io, as a custom field, X-Consumer-Custom-ID. Json payloads are used for the protocol's data. The examples in this section can be used with PostMan.

Below, set up a Python 3 environment with a set of Python packages available on pypi.org.
```
POST /environment/python/setup 
X-Consumer-Custom-ID:XXXXXXXXXX 
{
	"version": "3",
	"dependencies": ["numpy", "requests", ...]
}
```

The response will be a Json object with a "sessionid" field, assuming no error.
```
{
	"sessionid": "eyJ0ex.............."
}
```

#### Note: namepath format
A function reference is typically some sort of module or class import statement as a preamble to the name reference. Object-oriented languages allow instance functions which require instantiation before function invocation.  In JavaScript this is called a Cascade. This
looks like a dot path, for example in Java, `new Integer("200).compareTo(100)`, instantiates the object with an argument and calls the `compareTo` function expecting a single argument. Our system supports function specification of both static and instance functions with a "namepath" array format where the dot is substituted by array commas and each class attribute is a String quoted name. This is a Json array (`[ , , ]`) of a sequence of objects with names and optional arguments (`{ "name": xxxx, "args": [ , ,]}`). 
Notice that arguments are always arrays, a single argument is an array of one item.
For example, if we wanted to create an `Integer` object in Java and call the `compareTo` function with another argument we would make the following namepath.
```json
[ {"name": "Integer", "args": [ "200" ]}, {"name": "compareTo"} ]
```
The first namepath array element is an object with the class/constructor `"name"` attribute and the `"args"` attribute is an array of the single constructor `String` argument. The next object is the `"name"` attribute only since this is a function reference and not a function call.
Finally, we provide a shortcut for static method calls, i.e., no arguments in the namepath. Simply provide an array of `String` names. For example, the Java static function to convert a `String` to an `Integer` is `Integer.decode(String number)`. The namepath shortcut is :
```json
[ "Integer", "decode"]
```

### Environment Function Call
The Setup call, if successful, will return a Json object with a "sessionid" token field. To make a function call, use the "/environment/python/call" path, the "X-Consumer-Custom-ID" header is also required. The function call is defined by the Json payload, including the "sessionid", function arguments, and other fields.

A function name is expressed explicitly as a "namepath" which represents the usual dot-separated class attribute expression found in code. All function "namepaths" must include the module name followed by a class and/or sequence of function names. Optional arguments can be included in the case of object construction or intermediate function calls.
```json
{
 "namepath" : {
	"moduleName":"moduleNameX", 
	"path": [
		 {"name":"classX","args":["optionalX"]}, 
		 {"name":"functionX or __init__","args":["optionalX"]}
		]
	},
 "args": ["optionalX", ["optionalY"], {"optionalZ":{}}]
}
```
For example, below we want to sum an array of values with the desired function numpy.array(xxxxx).sum():
```http
POST /environment/python/call HTTPS/1.1
X-Consumer-Custom-ID: XXXXXXXXXX
Content-Type: application/json; charset=utf-8  

{
  "sessionid": "eyJ0eXAi.....",
  "timeout": "20000",
  "namepath": {
    	"moduleName": "numpy",
    	"path": [
    		 {"name": "array","args": [ "[2,4,6]" ]},
    		 {"name": "sum"}
    		]
    	},
  "args": []
}
```

#### Note: Json "args" and "tuple" typed arguments
We showed how to reference a function in a Python environment, however, we also need to address how to send the arguments.

There are 3 ways to send arguments via the API. The differences are due to type specification scheme.

1. Protcol Buffer specification of arguments types in order.
2. "args" field is an array of Json values.  
3. "tuple" field is a set of string-to-type value functions.

A Protocol Buffer payload can be sent as arguments to a function call. 

Json has a weak type specification that translates directly in some scripted languages (Python) but is
problematic in richly-typed languages (Java). The `"args"` field is an array of Json values `{"args": [ 1 2 3 ]}` in function argument order. Use `"args"` when Json types are sufficient.

Json values can be converted to language-specific typed arguments using the `"tuple"` specification. A tuple is a set of Json objects, each object is a single key-value pair. The object key is the string representation of the argument and the corresponding value is a function "namepath".  

```json
{"tuple": [ {"argument1": ["namepathZ", "namepathY"] }, {"argument2": ["namepathX", "namepathY"] } ]}

{
	"sessionid": "eyJ0eXAi.....",
  	"timeout": "20000",
  	"namepath": {
    	"moduleName": "numpy",
    	"path": [
    		 {"name": "array","args": [ "[2,4,6]" ]},
    		 {"name": "sum"}
    		]
    	},
	"tuple": [ {"1": ["Long", "parseLong"]}, {"2": ["Long", "parseLong"]} ]
}
```

In the specific example above we created a 2 parameter tuple, each of type Long by specifying the argument value as a string key, and the type as a "namepath" of a function to convert the string to a Long. Again, a tuple is
an array of single-field objects in function argument order. 

The Json specification allows for duplicate field names in objects. So we can specify any string value, even duplicates as arguments.

### Environment Pipe Call
Like the pipe (|) mechanism found in Unix, piping the output from one function to the next is very useful. 

Pipes can be constructed for a single session, utilizing functions in an existing dependency set, or distributed across multiple sessions. There are two types of pipe:

1. Pipeline 
  * Functions in a single session, assumed to run on the same VM.
2. Pipes 
  * Functions in a multi-session construction, fully distributed messaging.

A pipes specification includes a sessionid for each function stage. It is a list order sequence of cuntion calls (or the reverse of a bash pipe command). The head of the list receives the "args" arguments.

```json
{
 "args": ["optionalX", {"optionalY":{}}],
 "pipes" : [
 	{"session": "sessionidFirst",
 	 "pipeline": [
		{
	  	"moduleName":"moduleName",
	  	"path": [
	  		 {"name":"classX","args":["optionalX"]}, 
			 {"name":"functionX or __init__","args":["optionalX"]}
			]
		},
		{
	  	"moduleName":"moduleName",
	  	"path": [
	  		 {"name":"classY","args":["optionalX"]}, 
			 {"name":"functionY or __init__","args":["optionalX"]}
			]
		},
		{
	  	"moduleName":"moduleNameNext",
	  	"path": [
	  		 {"name":"classZ","args":["optionalX"]}, 
			 {"name":"functionZ or __init__","args":["optionalX"]}
			]
		}
		]
 	},
 	{}
 ]
}
```

A pipeline is a list order sequence of function calls or the reverse of bash pipe order. The head of the list is the first function call and receives the "args" arguments.

```json
{
 "args": ["optionalX", {"optionalY":{}}],
 "pipeline" : [
	{
  	"moduleName":"moduleNameFirst",
  	"path": [
  		 {"name":"classX","args":["optionalX"]}, 
		 {"name":"functionX or __init__","args":["optionalX"]}
		]
	},
	{
  	"moduleName":"moduleNameSecond",
  	"path": [
  		 {"name":"classY","args":["optionalX"]}, 
		 {"name":"functionY or __init__","args":["optionalX"]}
		]
	},
	{
  	"moduleName":"moduleNameNext",
  	"path": [
  		 {"name":"classZ","args":["optionalX"]}, 
		 {"name":"functionZ or __init__","args":["optionalX"]}
		]
	}
	]
}
```

### Environment List Processing
List processing is a fundamental computing operation. In addition to defining a list, and an operation such as `map`
or `filter`, a function from the code environment is specified to be applied by the operation to each list item.

```json
{
	"listop": {
		"op": "operation",
		"function": [
			{"name":"classX", "args":["optionalX"]},
			{"name":"functionX"}
			],
		"list": [ , , , [ {"name":"classY", "args":["optionalY"]}, {"name":"functionY"}]]
	}
}
```

Json data types are well-defined in a language like Python. Json has a weakly-defined type system under typed language implementations. So a type must be applied to list elements with a constructor function. The constructor is found at the last position in the list (as seen in the above example). An additional restriction is that elements in a Json array with a constructor must all be the same type.
We interpret Json numeric literals as `int` or `float` for typed languages to help simplify constructor usage.
if the list items are arrays, a constructor is required as the last item (maybe use an identity function) to meet the expected format. In the following Java example, we construct `Integer` objects from the `int` default type of the list elements.

```json
{
	"listop": {
		"op": "map",
		"function": "Integer.bitCount",
		"list": [1,3,5,7]
}
```


### Programming Examples

Sometimes code examples are clearer or simpler to understand than protocol formats.

Here is kodou.io usage from Python:

```python
import requests

setup_url = 'https://api.kodou.io/environment/python/setup'
call_url = 'https://api.kodou.io/environment/python/call'

id_header = {"X-Consumer-Custom-ID":"XXXXXXXXXX"}

# setup a kodou.io Environment with numpy
setup_json = {"version":"3", dependencies:["numpy"]}

# call numpy.array( [1,2,3] ).sum()
call_json = 
{
"timeout":"20000",
"namepath":{ 
	"moduleName":"numpy",
	"path":[
   		{"name":"array","args": [ [2,4,6] ]},
   		{"name":"sum"}
   	]
   },
"args": [] # No arguments for the function
}
	
setup_resp = requests.post(setup_url, headers=id_header, json=setup_json)

if setup_resp.code == 200:
	sessionid = set_resp.json().sessionid

 	call_json["sessionid"] = sessionid

	call_resp = requests.post(call_url, headers=id_header, json=call_json)

	if call_resp.code == 200 and call_resp.json().error == False:
		result = call_resp.json().value

		print("Answer is " + result)
```

## Generic Function (ICE) Example

The basic kodou.io ICE protocol sequence is to setup the function you want to call, receiving a session id token in response, and using the session id when making calls with function arguments. 

Every function in kodou.io is referenced with a url. For example, the opensource Redis repository has a function named hex_to_digit_to_int. We can reference this function with a kodou.io setup call using the url,
`"url":"https://github.com/antirez/redis.git|91685eeeb1462edfc12da2e079e76bdbeec0eddb|redis/src/sds.c|910|hex_digit_to_int"`
 
### Function Setup
Currently, we require a Setup call before a kodou.io function is called. In the future, this requirement will be relaxed. For all calls the default endpoint, api.kodoi.io, is assumed. You need to provide your API id, supplied by kodou.io, as a custom field, X-Consumer-Custom-ID. The examples in this section can be used with PostMan.

```
POST /libray/setup 
X-Consumer-Custom-ID:XXXXXXXXXX 
{
	"url": "https://github.com/antirez/redis.git|91685eeeb1462edfc12da2e079e76bdbeec0eddb|redis/src/sds.c|910|hex_digit_to_int"
}
```

### Function Call w/ Arguments
The Setup call, if successful, will return a Json object with a "sessionid" token field. (i.e.,"sessionid": "eyJ0eXXXXXXXXXX"). The "X-Consumer-Custom-ID" header is also required to use the "library/call" path. The "sessionid", function arguments, and other fields are put in the Json payload.

```
POST /library/call 
X-Consumer-Custom-ID:XXXXXXXXXX 
{
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


