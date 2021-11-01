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

#### Trip states
Each trip can be in only one the following state at any given time

Name | Description
---- | -----------
assign_queue       | Trip is created but no driver has been assigned to it yet
pickup             | Trip is assigned to a courier and the courier is on his way to pickup packages from the source
dropoff            | Trip is picked up by the courier and he is on his way to deliver of the packages
delivered          | All of the packages of the trip are delivered (and if trip was a round trip, courier has returned to the source)
canceled_by_miare  | Trip is canceled by our support staff. This only happens either with aggrement of the source or due to a violation of terms of service
canceled_by_client | Trip is canceled by client (either from the web panel or Third Party API)

### Course
A destination and a package to be delivered at that destination. Each trip can have one or more courses. 
As an example, if a **trip** has two **course**s, our courier will drive to the source, pick up two packages, 
and deliver them to two separate clients in two different destinations.

### Area
Part of the city defined by a set of latitude-longitudes forming a polygon that Miare operates in it.
If **source** of a trip is inside an area, that trip is considered part of trips of that specific area.

### Shift
A period of time in each day defined by a start and an end time. If deadline of a trip is within a shift, that trip is considered part of trips of that specific shift.

<aside class="success">
The current state of the shifts can be considered constant, and any change will be announced by the sales team.
</aside>

Start and end time of shifts area as follows:

name | start | end
---- | ----- | ---
morning  | 08:15 | 11:00
noon     | 11:00 | 17:30
night    | 17:30 | 23:30
midnight | 23:30 | 03:00

### Concurrency
The maximum number of **active** trips in each **area** during each **shift** for a client.
If the concurrency limit for a client is reached that is if the total number of its active trips is equal to its concurrency limit, the next attempts to create new trips will fail.
Canceling a trip (by client) or ending it (will free up a concurrency limit (by moving the trip out of the active trips list).

<aside class="success">
Only trips in the <code>assign_queue</code>, <code>pickup</code>, <code>dropoff</code>, and <code>delivered</code> states are considered active and contribute to the concurrency limit of the client.
</aside>

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

<aside class="success">
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


# Errors
All application level error of our API are in the following format:

<code>
{"code": "constant_error_code", "detail": "Variable human readable details"}
</code>

The `code` value is the main indicator of what went wrong. You can use it in a `switch-case` statement to determine the type of error.

The `detail` value is human readable details of what went wrong which is suitable for developers (**not** users).

<aside class="warning">
You should <b>not</b> rely on the value of the <code>detail</code> for automatic error detection. It is both variable and subject to change.
</aside>

<aside class="success">
You can find possible error codes of each endpoints in this document at the end of each endpoint's related section.
</aside>


# Delivery
Services related to creation and management of trips and courses.

## Create Trip

Creates a new trip and puts it in the assign queue.

<aside class="warning">
Trip creation will fail if you don't have enough concurrency limit in the area of the pickup location of the given trip in the deadline shift.
</aside>

> Request example:

```shell
curl --location --request POST "$BASE_URL/trips/" \
--header 'Authorization: Token <Your Token>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "pickup": {
    "name": "رستوران بزرگمهر",
    "phone_number": "09123456789",
    "address": "تهران، صادقیه، بلوار آیت الله کاشانی",
    "image": "https://example.com/restaurants/bm_logo.png",
    "location": {
      "latitude": 35.737004,
      "longitude": 51.413569
    },
    "deadline": "2021-11-01T17:12:00+0000"
  },
  "courses": [
    {
      "bill_number": "DEL-119",
      "name": "علی علوی",
      "phone_number": "09123456789",
      "address": "تهران، خیابان استاد معین، پلاک ۱۲",
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
    "phone_number": "09123456789",
    "address": "تهران، صادقیه، بلوار آیت الله کاشانی",
    "image": "https://example.com/restaurants/bm_logo.png",
    "location": {
      "latitude": 35.737004,
      "longitude": 51.413569
    },
    "deadline": "2021-11-01T17:12:00+0000"
  },
  "courses": [
    {
      "bill_number": "DEL-119",
      "name": "علی علوی",
      "phone_number": "09123456789",
      "address": "تهران، خیابان استاد معین، پلاک ۱۲",
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
  base_url + "/trips/",
  headers={"Authorization": "Token <Your Token>"},
  data = data,
)
```

> If you are getting the a `Deadline: (value_in_past)` error, make sure to update the `pickup.deadline` to a time in the future.


### HTTP Request

`POST /trips/`


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


### Response

> Response example:

```json
{
  "id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
  "created_at": "2021-11-01T16:59:00+0330",
  "assigned_at": "2021-11-01T17:00:00+0330",
  "picked_up_at": "2021-11-01T17:04:00+0330",
  "state": "assign_queue",
  "pickup": {
    "address": "تهران، صادقیه، بلوار آیت الله کاشانی",
    "deadline": "2021-11-01T20:42:00+0330",
    "image": "https://example.com/restaurants/bm_logo.png",
    "location": {
      "latitude": 35.737004,
      "longitude": 51.413569
    },
    "name": "رستوران بزرگمهر",
    "phone_number": "09123456789"
  },
  "area": {
    "id": "3",
    "name": "یوسف آباد"
  },
  "courier": null,
  "courses": [
    {
      "id": "7484f530-5e3e-491d-8a4a-9432f6db01d6",
      "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
      "bill_number": "DEL-119",
      "name": "علی علوی",
      "address": "تهران، خیابان استاد معین، پلاک ۱۲",
      "dropped_off_at": null,
      "phone_number": "09123456789",
      "location": {
        "latitude": 35.737004,
        "longitude": 51.413569
      },
      "manifest_items": [
        {
          "name": "پیتزا پپرونی خانواده",
          "quantity": 2
        }
      ],
      "payment": {
        "payment_type": "cash",
        "price": 0
      }
    }
  ],
  "tracking_url": "https://www.staging.mia.re/p/trip_watching/#!/7484f5305e",
}
```

Value | Type | Description
----- | ---- | -----------
**id** | string | Universally unique identifier of this trip
**created_at** | string [date-time] | The exact time this trip was created on our servers
**assigned_at** | string [date-time] **(nullable)** | The assign datetime of the time this trip was assigned to its courier. Will be **null** if trip is not assigned to a courier yet
**picked_up_at** | string [date-time] **(nullable)** | The datetime that the courier of the trip picked up its content from the source. Will be **null** if courier is not assigned or is not picked up packages just yet
**state** | string | The current state of the trip. Is one of the following values: "assign_queue" "pickup" "dropoff" "delivered" "canceled_by_miare" "canceled_by_client". You can find a description about each of these states [here](/#trip)
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
---- | -----------
not_authenticated    | Token is missing or invalid
parse_error          | The request body is not a valid JSON string (has syntax error)
invalid_request_body | Request body does not follow the valid format
concurrency_limit    | Client does not have enough concurrency in the area of the pickup location in the shift specified by deadline
service_level_order  | Based on the current enviromental state, your service level is too low to receive services from Miare. Service limitation error mostly happens due to whether conditions or special occasions. In order to increase your service level or if you need more details about this limitation please contant sales team.


## Cancel Trip

Cancels one of your trips indicated by its ID.

<aside class="warning">
Note that a trip **cannot** be canceled when it has passed 30 seconds since we’ve assigned it to a courier or the courier has already arrived at the source. In special cases that you still need to cancel the trip, please contact our support.
</aside>

> Request example:

```shell
TRIP_ID="<Trip ID>"

curl --location --request POST "$BASE_URL/trips/$TRIP_ID/cancel/" \
--header 'Authorization: Token <Your Token>' \
--header 'Content-Type: application/json' \
```

```python
import requests

trip_id = '<Trip ID>'

requests.post(
  base_url + f"/trips/${trip_id}/cancel/",
  headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`POST /trips/{trip_id}/cancel/`


### Path parameters


Name | Type | Description
----- | ---- | -----------
trip_id | string | The ID of the trip to cancel

<aside class="success">
You can find ID of your trip in the response body of [Create Trip](/#create-a-trip) request
</aside>

<aside class="warning">
The given trip ID most belong to your client.
</aside>

### Response

> Response example:

```json
{
  "created_at": "2021-11-01T16:59:00+0330",
  "id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
  "state": "canceled_by_client",
  "picked_up_at": null,
  "assigned_at": null,
  "area": {
    "id": "3",
    "name": "یوسف آباد"
  },
  "courier": null,
    "pickup": {
    "address": "تهران، صادقیه، بلوار آیت الله کاشانی",
    "deadline": "2021-11-01T20:42:00+0330",
    "image": "https://example.com/restaurants/bm_logo.png",
    "location": {
      "latitude": 35.737004,
      "longitude": 51.413569
    },
    "name": "رستوران بزرگمهر",
    "phone_number": "09123456789"
  },
  "courses": [
    {
      "address": "تهران، خیابان استاد معین، پلاک ۱۲",
      "bill_number": "DEL-119",
      "dropped_off_at": null,
      "id": "7484f530-5e3e-491d-8a4a-9432f6db01d6",
      "location": {
        "latitude": 35.737004,
        "longitude": 51.413569
      },
      "manifest_items": [
        {
          "name": "پیتزا پپرونی خانواده",
          "quantity": 2
        }
      ],
      "name": "test name",
      "payment": {
        "payment_type": "cash",
        "price": 0
      },
      "phone_number": "09123456789",
      "tracking_url": "https://www.staging.mia.re/p/trip_watching/#!/7484f5305e",
      "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7"
    }
  ]
}
```

Response body is a serialized trip. For a detailed version of it take a look at the response body of [Create Trip](/#create-a-trip) request.

<aside class="notice">
Succesful calls to this endpoint will update the state of the trip to <code>canceled_by_client</code>.
</aside>

### Errors

Code | Description
---- | -----------
not_authenticated        | Token is missing or invalid
parse_error              | The request body is not a valid JSON string (has syntax error)
invalid_request_body     | Request body does not follow the valid format
canceling_after_deadline | The trip has been in assigned state for more than 30 seconds
canceling_arrived_trip   | The trip's courier is already at the source location
canceling_ended_trip     | The trip is already ended (delivered)
invalid_state_change     | Changing the trip's state from its current state to canceled is not possible

