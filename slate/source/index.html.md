---
title: Miare API

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - python

toc_footers:
  - <a href='fa/'>نسخه فارسی</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Miare API
---


# Introduction

This is Miare's Third-Party API, designed to enable registered Miare users to integrate our delivery services into their applications.

This document walks you through the steps required to use Miare's services and endpoints.
If you need any further information about how Miare works or you are still not sure whether or not our services meet your needs, 
feel free to check out our website <a href='https://mia.re'>mia.re</a> or contact us at <a href='mailto: tech@miare.ir'>tech@miare.ir</a>.

# Registration
To use our services, you need to first have an API account. An API account is a specific type of account that 
can access our Third-Party endpoints and receive our event callbacks.

In order to create an account and discuss the basic terms and financial details of our services, you can reach our sales team at 
<a href=tel:02191009282>02191009282</a> Ext. 3, they will provide us with the required information to create your account.
After registration, you'll be given a string called **Token**. Your token is used to authenticate you, so keep it safe. 

<aside class="notice">
Authenticate process is further discussed in the <a href="#authentication">Authentication</a>
</aside>


# Definitions

### Client
A registered user who can use Miare's delivery service.

### Courier
A biker from our pool of bikers who can deliver packages from sources to destinations.
In Miare, bikers don't pick and choose what packages they deliver. Instead, each order is automatically is assigned to one of the available couriers based on their distance from the source and many other parameters.

### Trip
An order given by one of our clients for one or more packages to be delivered. A trip can have one and only one source
but multiple destinations (and multiple packages to be delivered at those destinations).
Our clients may also indicate that they wish the courier to return to the source at the end of the trip.

A trip is known as an active trip if its order is placed by the client but it's still not ended 
(delivered by the courier) or canceled.

### Course
A destination and a package to be delivered at that destination. Each trip can have one or more courses. 
As an example, if a **trip** has two **course**s, our courier will drive to the source, pick up two packages, 
and deliver them to two separate clients in two different destinations.

### Area
Part of the city defined by a set of latitude-longitudes forming a polygon that Miare operates in it.
If **source** of a trip is inside an area, that trip is considered part of trips of that specific area.

### Concurrency
The maximum number of **active** trips in each **area** for a client.
If the concurrency limit for a client is reached that is if the total number of its active trips is equal to its concurrency limit, the next attempts to create new trips will fail.
Canceling a trip (by client) or ending it (will free up a concurrency limit (by moving the trip out of the active trips list).

# Servers
Miare has two sets of servers.

## Staging
Sandbox servers -usually referred to as **Staging**- are the playground for development teams to test integrations of their application with ours.
In staging servers, your account will be given 10 concurrencies in each of our areas and all trips are free of charge.

Note that there is no active courier on the staging so your trips will never be assigned to a courier, delivered, or ended so you have to cancel your trips to avoid reaching the concurrency limit.

> Base URL of staging server:

```
https://staging.ws.mia.re/trip-management/third-party-api/v2/
```

## Production
Main servers -usually referred to as **Production**- are the main product's servers for real-world use.
In production servers, your concurrency should be purchased through the sales team and trips have normal rates applied to them.

> Base URL of production server:

```
https://ws.mia.re/trip-management/third-party-api/v2/
```


# Authentication
Each client has a specific API key called **Token**. 



> To authorize, use this code:

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -X DELETE \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

