---
title: Workflows
permalink: /docs/workflows/
---

## Setup and Call Workflow

The current version of kodou.io supports the calling of functions after a Setup call to provision the necessary resources. In furture versions, the resources will be dynamically allocated.

kodou.io has a web UI for user account management and to search for code. The user types in keywords and phrases to identify the functions they want to find and use. The search page returns a list of results is indexed with unique urls. Once the user selects one of the functions it is stored in their account for convenience. The url serves as the reference for the function used in kodou.io API calls. 

Assuming the endpoint is `api.kodou.io`, the user will make calls against this endpoint using their API id stored in the HTTP header named X-Consumer-Custom-ID. This header must be used in all kodou API calls to identifiy the account.

The user will first call the kodou `library\setup` API against the endpoint. This call, if successful, will return a token that can be used for any calls to that function. 

The function calls will require the X-Consumer-Custom-ID header and a Json payload with the `sessionid` field, a `timeout` field, and an `args` field, a sub-object with the function arguments as fields.

Lets look at an example of Setup and Call of one call.

## SetUp call

Using a url the user selected from a code search on the kodou.io web UI, the following is a Setup call using `cURL`.

```
curl --request POST \
 --header 'X-Consumer-Custom-ID: {your API ID}' \
 --header "Content-Type: application/json" \
 --data '{"url":"https://github.com/antirez/redis.git|91685eeeb1462edfc12da2e079e76bdbeec0eddb|redis/src/sds.c|910|hex_digit_to_int"}' \
 https://api.kodou.io
```

The response will be a Json object with a `sessionid` field.

## Function call

The `sessionid` value serves as a proxy for the function url in the Setup phase. Call this function using the `library\call` API. The call includes the 'X-Consumer-Custom-ID' header, and a Json payload with the `sessionid`, a `timeout` value, and an object named `args` of the function arguments. In the example below, `...hex_digit_to_int` accetps one argument named `c` and returns an integer.

```
curl --request POST \
 --header 'X-Consumer-Custom-ID: {your API ID}' \
 --header "Content-Type: application/json" \
 --data '{"sessionid": {from the Setup call}, "timeout":"20000", "args":{"c": "A"}}'
 https://api.kodou.io
```

The return is a Json object with a
[Describe the optimal or assumed workflows for a few things users can accomplish with your API. Think about the most common and highest-impact outcomes. Each workflow should walk users through a brief explanation, any prerequisites, and the steps involved in completing the outcome with your API, including the reasons users must take each step and the API call required to finish the step.

Here's an example workflow for an API that allows you to retrieve and update timesheet records.

### Add a description to explain overtime on employee's timesheet

You asked an employee, Meghan, to work 2 hours of overtime last Thursday. To make sure Payroll approves Meghan's timesheet with the extra 2 hours, you need to add a description.

To complete this outcome, you need Meghan's employee ID number and your authentication token for the API.

1. Retrieve the record for Meghan's timesheet for last Thursday. Send a request to the `GET` https://api.payrollrecord.com/timesheet endpoint. Your request must include Meghan's employee ID number and the date as query parameters. Here's the complete request in cURL:

	```
	curl --request GET \
	  --url 'https://api.payrollrecord.com/timesheet?employee={employee ID number}&date=2017-05-25' \
	  --header 'authorization: {your authentication token}'
	```

2. The JSON object in the response will include a unique ID for the timesheet for last Thursday, `timesheet_ID`. Note this ID---you will use it in the next request.

3. Send a request to the `PUT` https://api.payrollrecord.com/timesheet endpoint. Your request must include the `timesheet_ID` as a query parameter and the description you want to add in the request body. Here's the complete request in cURL:

	```
	curl --request PUT \
	  --url 'https://api.payrollrecord.com/timesheet?timesheet_id={timesheet ID number}' \
	  --header 'authorization: {your authentication token}' \
	  --header 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \
	  --form 'description=Employee worked 2 hours of overtime to complete data entry. Overtime approved by {your name}.'
	```

Now, if you send another request for Meghan's timesheet for May 25, 2017 (like you did in Step 1), you will see the description you added at the end of the JSON object in the response.]

