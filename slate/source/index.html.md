---
title: Miare API

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - python

toc_footers:
  - <a href='fa/'>نسخه فارسی</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

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
After registration, you'll be given a string called **Token**.

<aside class="warning">
Your token is our only way to make sure requests are coming from you, so keep it safe. 
</aside>

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
Sandbox servers -usually referred to as **Staging**- are the playground for development teams to test integration of their application with ours.
In the staging servers, your account will be given 10 concurrencies in each of our areas and all trips are free of charge.

> Base URL of staging server:

```shell
BASE_URL="https://staging.ws.mia.re/trip-management/third-party-api/v2/"
```

```python
base_url = "https://staging.ws.mia.re/trip-management/third-party-api/v2/"
```

<aside class="notice">
There is no active courier on the staging so your trips will never be assigned to a courier, delivered, or ended so you have to cancel your trips to avoid reaching the concurrency limit.
</aside>


## Production
Main servers -usually referred to as **Production**- are the main product's servers for real-world use.
In production servers, your concurrency should be purchased through the sales team and trips have normal rates applied to them.

> Base URL of production server:

```shell
BASE_URL="https://ws.mia.re/trip-management/third-party-api/v2/"
```

```python
base_url = "https://ws.mia.re/trip-management/third-party-api/v2/"
```


<aside class="notice">
You should only start using our production server after all technical and non-technical major integration issues are resolved.
</aside>


# Authentication

Each client has a specific API key called **Token** for each of our servers. The token is the sole representative of an account on a server, its trips, courses, and financial transactions.
The sales team will provide you with the staging token after the **registration** process and the production token after the **test** process.

<aside class="notice">
At this moment, there is no automatic process for resetting your token, but you can ask the sales team to reset it for you at any time.
</aside>

<aside class="notice">
Note that staging and production tokens are in no way related to each other and their accounts are two separate accounts on two sets of servers.
</aside>

> To authorize, pass the authorization token among headers:


```shell
curl "api_endpoint_here" \
  --header "Authorization: Token <Your Token>"
```

```python
import requests

requests.post(
  "api_endpoint_here",
  headers={"Authorization": "Token <Your Token>"}
)
```


> Make sure to replace `<Your Token>` with your API Token.


Maire uses API Tokens to allow access to the API.

Miare API expects for the token to be included in **ALL** API requests to the server in a header that looks like the following:

`Authorization: Token <Your Token>`

<aside class="notice">
You must replace <code>&#60;Your Token&#62;</code> with your personal API key.
</aside>



# Delivery
Services related to creation and management of trips and courses.

## Create a Trip

Creates a new trip and puts it in the assign queue.

<aside class="warning">
Trip creation will fail if you don't have enough concurrency limit in the area of the pickup location of the given trip.
</aside>

> Request examples:

```shell
curl --location --request POST "$BASE_URL/trips/" \
--header 'Authorization: Token <Your Token>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "pickup": {
    "name": "رستوران بزرگمهر",
    "phone_number": "0212345678",
    "address": "تهران، صادقیه، بلوار آیت الله کاشانی",
    "image": "https://example.com/restaurants/bm_logo.png",
    "location": {
      "latitude": 35.737004,
      "longitude": 51.413569
    },
    "deadline": "2019-04-12T14:12:00+0000"
  },
  "courses": [
    {
      "bill_number": "DEL-119",
      "name": "علی علوی",
      "phone_number": "09123456789",
      "address": "هران، خیابان استاد معین، پلاک ۱۲",
      "location": {
        "latitude": 35.737004,
        "longitude": 51.413569
      },
      "manifest_items": [
        {
          "name": "پیتزا پپرونی خانواده",
          "quantity": 2
        }
      ]
    }
  ]
}'
```

```python
import requests

data = {
  "pickup": {
    "name": "رستوران بزرگمهر",
    "phone_number": "0212345678",
    "address": "تهران، صادقیه، بلوار آیت الله کاشانی",
    "image": "https://example.com/restaurants/bm_logo.png",
    "location": {
      "latitude": 35.737004,
      "longitude": 51.413569
    },
    "deadline": "2019-04-12T14:12:00+0000"
  },
  "courses": [
    {
      "bill_number": "DEL-119",
      "name": "علی علوی",
      "phone_number": "09123456789",
      "address": "هران، خیابان استاد معین، پلاک ۱۲",
      "location": {
        "latitude": 35.737004,
        "longitude": 51.413569
      },
      "manifest_items": [
        {
          "name": "پیتزا پپرونی خانواده",
          "quantity": 2
        }
      ]
    }
  ]
}

requests.post(
  base_url + "/trip/",
  headers={"Authorization": "Token <Your Token>"},
  data = data,
)
```

> If you are getting a 

### HTTP Request

`GET /trips/`


### Body


Value | Type | Description
----- | ---- | -----------
**pickup** | object | The source of the trip
pickup.**name** | string | The human readable name of the pickup
pickup.**phone_number** | string | The phone number associated with the source which will be used by courier and support staffs in order to contact to pickup if necessary
pickup.**address** | string | The human readable address of the source, preferably down to every necessary detail for a human to find the source quickly
pickup.**image** | string [uri] | A valid URL which points to an image file which should be the logo of the pickup. This image will be used in both support panel, and courier’s application. Make sure that the URL is both reachable and is configured to allow CORS requests
pickup.**location** | object | The exact location of the pickup
pickup.location.**latitude** | number [double] | The latitude of the pickup location
pickup.location.**longitude** | number [double] | The longitude of the pickup location
pickup.**deadline** | string [date-time] | The time that you expect the courier to arrive at the pickup, so optimally it should be the time package content is ready and packaged. **Can't be in the past**
**courses** | array | List of destinations of the trips
courses.**bill_number** | string | An string field left for you to store sort of a human readable bill number in it which will be used as a reference point among our support team, you and the pickup staffs
courses.**name** | string | Name of the dropp-off
courses.**phone_number** | string | The phone number associated with the drop-off which will be used by courier and support staffs in order to contact to them if necessary
coureses.**address** | string | The human readable drop-off address, preferably down to every necessary detail for a human to find it quickly
courses.location | object **(Optional)** | The exact location of the pickup. If there is no provided drop-off location, the courier will find the location based on the address and accounting calculations will be based on that location
courses.location.**latitude** | number [double] | The latitude of the drop-off location
courses.location.**longitude** | number [double] | The longitude of the drop-off location
courses.manifest_items | array **(Optional)** | The contents of the package to be delivered to the drop-off
courses.manifest_items.**name** | string | Human readable name of the content which will be verified by courier
courses.manifest_items.**quantity** | string | The quanitiy of the item


### Errors
Code | Description
----- | -----------
**pickup** | The source of the trip
