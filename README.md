# Moonbase Exchange WebSocket API documentation

## Contents

1. [Authentication](#authentication)  
2. [Rate limits](#limits)  
3. [Public channels](#public)  
4. [Authenticated channels](#private)
5. [Examples](#examples)

## Authentication
API key & secret can be generated in user dashboard.  

### Algorithm
* Build a map (associative array) of ```key``` and ```timestamp```, where:  
```key``` is your API key and ```timestamp``` is current UNIX timestamp (number of seconds since 1 January 1970).  
* Encode the map into query string (e.g. ```“key=XXX&timestamp=YYY”```).
* Build request string just as if you were signing a request to ```GET /users/verify``` (e.g. ```“GET\n/users/verify\n{{query_string}}”```).
* Calculate auth_signature via HMAC256 algorithm using API secret and request string.
* Add a ```signature``` parameter to you ```key``` and ```timestamp``` request parameters.
* Send request.

See also [REST API docs](https://moonbase.exchange/api/v1/swagger/index.html).
