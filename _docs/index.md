---
title: Get started
permalink: /docs/home/
redirect_from: /docs/index.html
---

The code you want to write today has probably been written 100X already somewhere in opensource or your company's code base. Instead of beginning the software design process and development cycles, why not use the best existing code that has been road-tested, improved over time by many people, and meets your needs. This enables speed of software prototyping, development, and deployment, while using high quality code. 

kodou.io is a Software-as-a-Service platform where the software is high-quality code from existing source code repositories. Whether open source or enterprise code bases, kodou.io decomposes code repositories into the functions
that comprise them. The functions are available for API calls whenever needed by a user.

**No Compiler, IDE, Deployment Process, Dependency Management or Dependency Conflicts**

kodou.io isolates all functions into independent artifacts, no more dependency conflicts. All that is required is a reference to the desired function.

Use the kodou.io web page to search for code using keywords and phrases that describe the code you would like. The results are a list of functions that may have been buried in a repository but are extracted by kodou.io. The choices come from many repositories and across various programming languages. Once a function is selected it is available for API calls. 

Similarly, kodou.io provides a dependency-level service called an Environment. This enables function calls on a requested set of library dependencies. The functions exposed by the dependencies are available for calls. Again, each Environment is independent, so a client could set up two Environments that normally combined would have dependency conflicts, but are effectively combined (piece-wise) from the client's perspective.

Here are some usage examples

* You want to use the best encryption code but you don't know where to find it or the best is written in another programming language. kodou.io enables you find and call the most highly regarded encryption code even it is in a
programming language you are not familiar with.
* You are a software architect and there is an idea you've wanted to explore for a while. A fast way to do
so is using kodou.io to leverage existing code that matches your expectations. You can put together most of the
code components and begin testing them out without having to first lay down software tooling.
* There is a series of functions and libraries you want to use together but they require deep dependency handling to combine into an application. kodou.io functions and Environments allow you to use any combination of code bases or libraries since they are all kept independent.

kodou.io uses HTTPS protocols `POST` with Json or Protocol Buffers payloads for requests. Responses are returned in the same formats. 

To access kodou.io, you'll need to sign up using the web UI. The sign up process will generate a custom id that can be used for API calls. 

The base URL is api.kodou.io unless otherwise specified. Each user is rate limited depending on the account. If you exceed the rate limit, the system will throttle your requests.

## Authentication

To make API calls you use the kodou.io web UI to set up your account to receive a key (a JWT token). Copy the key/token displayed in the UI and apply it to API https requests as a parameter named `api_id` or a header `Authorization: Bearer XXXXX`. Please keep this key/token private as it represent your account without need for further evidence. Don't share it with other users, and make sure to remove it from any code samples.

All API requests must have either the `api_id` parameter key or the header `Authorization: Bearer XXXX`. The API ID expiration is account dependent. 

For each account the API key and the API Endpoint Address are displayed like the image below. Copy these values for use in your API calls. Look further in the documentation for examples on how to use the JWT key/token for authentication. 

Usually the API Endpoint Address is https://kodou.io. 
![Example API tokens page](/img/screencapture-kodou-io-api.png "API tokens")

Some kodou.io API calls produce a session ID, a JWT token, to be used in subsequent calls. This token encodes
the state needed the configuration specified in the original call. The session ID is sent in a `POST` JSON payload.

