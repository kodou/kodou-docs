---
title: Code samples
permalink: /docs/code_samples/
---

# kodou.io API

kodou.io is designed to be a minimal protocol for clients. The client either requests a specific function, known as an isolated code executable (ICE), or specifies a set of dependencies to be available in an "Environment" for a variety of function calls exported by the dependencies.

Thus, there are two kinds of API services: 
+ Environments
+ Functions (ICE)

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

You need to provide your API id (it looks like eyJhbGciOiJXXXXX....), supplied by kodou.io, as a field, `Authorization: Bearer eyJhbGciOiJXXXXX....`. Json payloads are used for the protocol's data. The examples in this section can be used with PostMan.

Below, set up a Python 3 environment with a set of Python packages available on pypi.org.
```
POST /environment/python/setup 
Authorization: Bearer eyJhbGciOiJXXXXX....
Content-Type: application/json; charset=utf-8  
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
A function reference is typically some sort of module or class import statement as a preamble to the name reference. This sequence of unique names, module name followed by class or function name is what we call a "namepath".  In JavaScript this is called a Cascade. 
"namepath" looks like a dot path, for example in Java, `new Integer("200).compareTo(100)`, instantiates the object with an argument and calls the `compareTo` function expecting a single argument. 

Our system supports function specification of both static and instance functions with a "namepath" array format where the dot is substituted by array commas (`[ , , ]`) and each class attribute is a string quoted name. This is a sequence of objects with names and optional arguments (`{ "name": xxxx, "args": [ , ,]}`). 
Notice the arguments are in an array, and a single argument is an array of one item.
For example, if we wanted to create an `Integer` object in Java and call the `compareTo` function with another argument we would make the following namepath.
```json
[ {"name": "Integer", "args": [ "200" ]}, {"name": "compareTo"} ]
```
The first namepath array element is an object with the class/constructor `"name"` attribute and the `"args"` attribute is an array of the single constructor string argument. The next object is the `"name"` attribute only since this is a function reference and not a function call.

Finally, we provide a shortcut for static method calls, i.e., no arguments in the namepath. Simply provide an array of string names. For example, the Java static function to convert a string to an `Integer` is `Integer.decode(String number)`. The namepath shortcut is :
```json
[ "Integer", "decode"]
```

### Environment Function Call
The Setup call, if successful, will return a Json object with a "sessionid" token field. To make a function call, use the "/environment/python/call" path, the "Authorization" header is also required. The function call is defined by the Json payload, including the "sessionid", function arguments, and other fields.

A function name is expressed explicitly as a "namepath" which represents the usual dot-separated class attribute expression found in code. All function "namepaths" must include the toplevel/module name followed by a class and/or sequence of function names. Optional arguments can be included in the case of object construction or intermediate function calls.
```json
{
 "namepath" : [
	{"moduleName":"moduleNameX"},
	{"name":"classX","args":["optionalX"]}, 
	{"name":"functionX or __init__","args":["optionalX"]}
	],
 "args": ["optionalX", ["optionalY"], {"optionalZ":{}}]
}
```
For example, below we want to sum an array of values with the desired function `numpy.array(xxxxx).sum()`:
```https
POST /environment/python/call HTTPS/1.1
Authorization: Bearer eyJhbGciOiJXXXXX....
Content-Type: application/json; charset=utf-8  

{
  "sessionid": "eyJ0eXAi.....",
  "timeout": "20000",
  "namepath": [
    	{"moduleName": "numpy"},
    	{"name": "array","args": [ "[2,4,6]" ]},
    	{"name": "sum"}	
    	],
  "args": []
}
```
The `"moduleName"` key is not required as the first object key, `"name"` can also be used. Python environments may find it helpful to use `"moduleName"`.

#### Note: Json "args" and "tuple" typed arguments
We showed how to reference a function in a Python environment, however, we also need to address the options for sending the arguments.

There are 3 ways to send arguments via the API. The differences are due to type specification scheme.

1. Protcol Buffer specification of arguments types in order.
2. "args" field, an array of Json values.  
3. "tuple" field, type specification of arguments or, equivalently, conversion functions.

A Protocol Buffer payload can be sent as arguments to a function call. 

Json has a weak type specification that translates automatically in some scripted languages (Python) but is
problematic in richly-typed languages (Java). The `"args"` field is an array of Json values `{"args": [ 1 2 3 ]}` in function argument order. Use `"args"` when Json types are sufficient.

Json values can be converted to language-specific typed arguments using the `"tuple"` specification. Each tuple specifies a raw value and either a language-specific type or a function that converts the value to a type, the function return type. A tuple is a set of typed arguments defined using an array of Json objects. We embed the specification in regular Json, that is the reason for the format.

Currently, tuple cannot be used in a namepath as an args alternative. This will change in the future.

Generally, tuple is useful when you want to set the arguments types separately from the composition of functions that make up your code. Otherwise, you would have to include type conversion in your function compositions, muddying the meaning of your algorithms.

There are two kinds of tuples, value conversion and array conversion. A tuple is an function arguments-ordered sequence of objects. Each object is a single key-value pair, specified with a `"value"`,`"type"` pair or an `"array"`,`"type"` pair. The value or array are valid Json values. The type is a namepath, however, without arguments, simply a sequence of strings which make up the unique identifier of a one parameter function or a language-specific type. 

```json
{"tuple": [ {"value": JsonValue, "type": ["namepathX", "language-specific type or class"]}, ...]}
{"tuple": [ {"array": [JsonValue,JsonValue,JsonValue], "type": ["namepathY", "array-element-to-type conversion function"]}]}
	or for convenience
{"tuple": [ {"string_argument1": ["namepathZ", "string-to-type function_name"] }, {"string_argument1": ["namepathA", "requested-type"] } ]}

{
	"sessionid": "eyJ0eXAi.....",
  	"timeout": "20000",
  	"namepath": [
    	{"moduleName": "numpy"},
    	{"name": "array","args": [ "[2,4,6]" ]},
    	{"name": "sum"}	
    	],
	"tuple": [ {"1": ["Long", "parseLong"]}, {"2": ["Long", "parseLong"]} ]
}
```
As a convenience, a compact form of tuple is allowed using a string value as the object key and the argument-free namepath. Obviously, the value cannot be the strings "value" or "array". The namepath can be interpreted as a one parameter function which takes a string and returns a typed value. Or, the namepath is the requested language-specific type conversion from the Json value. The default language-specific string-to-type operation will be requested, so this may fail at runtime.

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
	  	[
	  		{"moduleName":"moduleName"},
	  		{"name":"classX","args":["optionalX"]}, 
			{"name":"functionX or __init__","args":["optionalX"]}
		],
		[
			{"moduleName":"moduleName"},
	  		{"name":"classY","args":["optionalX"]}, 
			{"name":"functionY or __init__","args":["optionalX"]}
		],
		[
	  		{"moduleName":"moduleNameNext"},
	  		{"name":"classZ","args":["optionalX"]}, 
			{"name":"functionZ or __init__","args":["optionalX"]}
		]
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
	[
  		{"moduleName":"moduleNameFirst"},
  		{"name":"classX","args":["optionalX"]}, 
		{"name":"functionX or __init__","args":["optionalX"]}
	],
	[
  		{"moduleName":"moduleNameSecond"},
  		{"name":"classY","args":["optionalX"]}, 
		{"name":"functionY or __init__","args":["optionalX"]}
	],
	[
  	{"moduleName":"moduleNameNext"},
  	{"name":"classZ","args":["optionalX"]}, 
	{"name":"functionZ or __init__","args":["optionalX"]}
	]
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

id_header = {"Authorization":"Bearer XXXXXXXXXX"}

# setup a kodou.io Environment with numpy
setup_json = {"version":"3", dependencies:["numpy"]}

# call numpy.array( [1,2,3] ).sum()
call_json = 
{
"timeout":"20000",
"namepath":[
	{"moduleName":"numpy"},
   	{"name":"array","args": [ [2,4,6] ]},
   	{"name":"sum"}
   ],
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
Authorization: Bearer eyJhbGciOiJXXXXX....
{
	"url": "https://github.com/antirez/redis.git|91685eeeb1462edfc12da2e079e76bdbeec0eddb|redis/src/sds.c|910|hex_digit_to_int"
}
```

### Function Call w/ Arguments
The Setup call, if successful, will return a Json object with a "sessionid" token field. (i.e.,"sessionid": "eyJ0eXXXXXXXXXX"). The "Authorization" header is also required to use the "library/call" path. The "sessionid", function arguments, and other fields are put in the Json payload.

```
POST /library/call 
Authorization: Bearer eyJhbGciOiJXXXXX....
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
 --header 'Authorization: Bearer {your API ID eyJhbGciOiJXXXXX....}' \
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
	headers: {'Authorization': 'Bearer {your API ID eyJhbGciOiJXXXXX....}' },
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
		headers: {'Authorization': 'Bearer {your API ID eyJhbGciOiJXXXXX....}' },
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
headers = {'Authorization': 'Bearer {your API ID eyJhbGciOiJXXXXX....}' }
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
	headers = {'Authorization': 'Bearer {your API ID eyJhbGciOiJXXXXX....}'}
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


