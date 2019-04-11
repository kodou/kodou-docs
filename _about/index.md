---
title: About kodou.io
permalink: /about/index/
---

kodou.io is a service for ondemand API calls to code in opensource or enterprise code bases. The service creates
a library of functions extracted from existing code, even functions buried in a repository. Software from a range of programming languages is available. For example, if you are a Javascript programmer you can make API calls to  code in other languages used in with highly regarded repositories. 

## Usage

The Web UI (https://kodou.io) enables clients to search for code using descriptive terms and phrases. The list of search results, ranked by quality, has a (kodou.io) reference url for each entry. The client selects one fo the results and that reference url is stored in the user's profile for future reference.

The API interface (https://api.kodou.io) allows programmable clients to setup and call functions by (kodou.io) url reference.

The clients make an API call to setup a function and then makes calls to the function with arguments. In the future the Setup stage may not be needed. 

## Workflow

The service requires no new skills or tooling from the client. The workflow is 
1. Search for code
2. Choose a function from the results
3. Function is ready to be called via API

If you already know what you want, simply send an API request for the code Environment, and the functions are ready to be called via API.

## Advantages

Notice there is no compiler, IDE, deployment process, dependency management, or dependency conflicts. This simplicity and power solves many some important problems in modern software engineering. 

### No Dependency Conflicts

Each function is independent so there are no dependency conflicts. You can compose as many functions as needed.

### Multi-language Platform

kodou.io provides code from many programming languages. Functions from various languages can be composed and called.