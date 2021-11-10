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
BASE_URL="https://staging.ws.mia.re/"
```

```python
base_url = "https://staging.ws.mia.re/"
```

<aside class="notice">
There is no active courier on the staging so your trips will never be assigned to a courier, delivered, or ended so you have to cancel your trips to avoid reaching the concurrency limit.
</aside>


## Production
Main servers -usually referred to as **Production**- are the main product's servers for real-world use.
In production servers, your concurrency should be purchased through the sales team and trips have normal rates applied to them.

> Base URL of production server:

```shell
BASE_URL="https://ws.mia.re/"
```

```python
base_url = "https://ws.mia.re/"
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
  base_url + "/trip-management/third-party-api/v2/trips/",
  headers={"Authorization": "Token <Your Token>"},
  data = data,
)
```

> If you are getting the a `Deadline: (value_in_past)` error, make sure to update the `pickup.deadline` to a time in the future.


### HTTP Request

`POST /trip-management/third-party-api/v2/trips/`


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
  "courier": {
      "image": "https://image.miare.ir/avatars/123456.jpeg",
      "location": {
          "latitude": 35.753691,
          "longitude": 51.332284
      },
      "location_updated_at": "2021-11-01T19:10:13+0330",
      "name": "علی لطفی",
      "phone_number": "09379187928"
  },
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
courses.**id** | string | Universally unique identifier of this course
course.**trip_id** | string | Universally unique identifier of the trip that this course belongs to it
courses.**bill_number** | string | An string field left for you to store sort of a human readable bill number in it which will be used as a reference point among our support team, you and the pickup staffs
courses.**name** | string | Name of the dropp-off
courses.**phone_number** | string | The phone number associated with the drop-off which will be used by courier and support staffs in order to contact to them if necessary
coureses.**address** | string | The human readable drop-off address, preferably down to every necessary detail for a human to find it quickly
courses.location | object **(nullable)** | The exact location of the pickup. If there is no provided drop-off location, the courier will find the location based on the address and accounting calculations will be based on that location
courses.location.**latitude** | number [double] | The latitude of the drop-off location
courses.location.**longitude** | number [double] | The longitude of the drop-off location
courses.manifest_items | array **(nullable)** | The contents of the package to be delivered to the drop-off
courses.manifest_items.**name** | string | Human readable name of the content which will be verified by courier
courses.manifest_items.**quantity** | string | The quanitiy of the item
course.**tracking_url** | string [uri] | The URL of a webpage in which the end customer can track the exact state and location of his/her package while it's being delivered
course.**dropped_off_at** | string [date-time] **(nullable)** | The exact time this course we delivered to the customer. It will be **null** if the course is not delivered yet
course.**payment** | object | The payment information of the course
course.payment.**payment_type** | string | They selected method for this course's payment. At this moment the only available method for API users is `cash`
course.payment.**price** | string | The price of the package content (**not** to be confused with delivery cost). At this moment the only available value for API users is 0
**area** | object | The detected area of the trip (based on pickup.location)
area.**id** | string | The identifier of this trip's area
area.**name** | string | Human readable name of this trip's area
courier | object **(nullable)** | Courier of this trip. Will be **null** if trip is not assigned to a courier yet
courier.**name** | string | Name of the courier
courier.**phone_number** | string | Phone number of the courier
courier.**image** | string [uri] | The URL of courier's profile picture
courier.**location** | object | Last known location of the courier
courier.location.**latitude** | number [double] | Latitude of the last known location of the courier
courier.location.**longitude** | number [double] | Longitude of the last known location of the courier
courier.**location_updated_at** | string [date-time] | Datetime of the last location of the courier


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

curl --location --request POST "$BASE_URL/trip-management/third-party-api/v2/trips/$TRIP_ID/cancel/" \
--header 'Authorization: Token <Your Token>'
```

```python
import requests

trip_id = '<Trip ID>'

requests.post(
  f"{base_url}/trip-management/third-party-api/v2/trips/{trip_id}/cancel/",
  headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`POST /trip-management/third-party-api/v2/trips/{trip_id}/cancel/`


### Path parameters


Name | Type | Description
----- | ---- | -----------
trip_id | string | The ID of the trip to cancel

<aside class="success">
You can find ID of your trip in the response body of <a href="/#create-trip">Create Trip</a> request
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

Response body is a serialized trip. For a detailed version of it take a look at the response body of [Create Trip](/#create-trip) request.

<aside class="notice">
Succesful calls to this endpoint will update the state of the trip to <code>canceled_by_client</code>.
</aside>

### Errors

Code | Description
---- | -----------
not_authenticated        | Token is missing or invalid
parse_error              | The request body is not a valid JSON string (has syntax error)
invalid_request_body     | Request body does not follow the valid format
record_not_found         | A trip with the given ID does not belong to your client or doesn't exist at all
canceling_after_deadline | The trip has been in assigned state for more than 30 seconds
canceling_arrived_trip   | The trip's courier is already at the source location
canceling_ended_trip     | The trip is already ended (delivered)
invalid_state_change     | Changing the trip's state from its current state to canceled is not possible


## Add Course

Adds a new course to the courses of a trip.

<aside class="warning">
You can only add a course to a trip in the <code>assign_queue</code> or <code>pickup</code> states.
</aside>

> Request example:

```python
import requests

trip_id = '<Trip ID>'

requests.post(
  f"{base_url}/trip-management/third-party-api/v2/trips/{trip_id}/cancel/",
  headers={"Authorization": "Token <Your Token>"},
)
```


```shell
TRIP_ID="<Trip ID>"

curl --location --request PATCH "$BASE_URL/trip-management/third-party-api/v2/trips/$TRIP_ID/courses" \
--header 'Authorization: Token <Your Token>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "bill_number": "DEL-120",
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
}'
```

```python
import requests

data = {
  "bill_number": "DEL-120",
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

requests.patch(
  f"{base_url}/trip-management/third-party-api/v2/trips/{trip_id}/courses/",
  headers={"Authorization": "Token <Your Token>"},
  data = data,
)
```

### HTTP Request

`PATCH /trip-management/third-party-api/v2/trips/{trip_id}/courses/`

### Path parameters


Name | Type | Description
----- | ---- | -----------
trip_id | string | The ID of the trip to add a course to

<aside class="success">
You can find ID of your trip in the response body of <a href="/#create-trip">Create Trip</a> request
</aside>

### Body


Value | Type | Description
----- | ---- | -----------
**bill_number** | string | An string field left for you to store sort of a human readable bill number in it which will be used as a reference point among our support team, you and the pickup staffs
**name** | string | Name of the dropp-off
**phone_number** | string | The phone number associated with the drop-off which will be used by courier and support staffs in order to contact to them if necessary
**address** | string | The human readable drop-off address, preferably down to every necessary detail for a human to find it quickly
location | object **(Optional)** | The exact location of the pickup. If there is no provided drop-off location, the courier will find the location based on the address and accounting calculations will be based on that location
location.**latitude** | number [double] | The latitude of the drop-off location
location.**longitude** | number [double] | The longitude of the drop-off location
manifest_items | array **(Optional)** | The contents of the package to be delivered to the drop-off
manifest_items.**name** | string | Human readable name of the content which will be verified by courier
manifest_items.**quantity** | string | The quanitiy of the item


### Response

> Response example:

```json
{
    "id": "8e209688-19c9-4982-a755-517408b6a29f",
    "created_at": "2021-11-01T18:44:19+0330",
    "picked_up_at": null,
    "state": "assign_queue",
    "assigned_at": null,
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
    "area": {
        "id": "3",
        "name": "یوسف آباد"
    },
    "courses": [
        {
            "address": "تهران، خیابان استاد معین، پلاک ۱۲",
            "bill_number": "DEL-120",
            "dropped_off_at": null,
            "id": "c57cb6bc-e1c2-404a-a0d2-a54881fe96ec",
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
            "name": "علی علوی",
            "payment": {
                "payment_type": "cash",
                "price": 0
            },
            "phone_number": "09123456789",
            "tracking_url": "https://www.staging.mia.re/p/trip_watching/#!/c57cb6bce1",
            "trip_id": "8e209688-19c9-4982-a755-517408b6a29f"
        }
    ]
}
```

Response body is a serialized trip. For a detailed version of it take a look at the response body of [Create Trip](/#create-trip) request.

### Errors

Code | Description
---- | -----------
not_authenticated    | Token is missing or invalid
parse_error          | The request body is not a valid JSON string (has syntax error)
invalid_request_body | Request body does not follow the valid format
record_not_found     | A trip with the given ID does not belong to your client or doesn't exist at all
too_late             | Trip's state is later than `pickup`



## Remove Course

Removes the course indicated by the given ID from the courses of its trip.

<aside class="warning">
You can only remove courses from a trip in the <code>assign_queue</code> or <code>pickup</code> states which has at least <b>two</b> courses.
</aside>

> Request example:

```shell
COURSE_ID="<Course ID>"

curl --location --request DELETE "$BASE_URL/trip-management/third-party-api/v2/courses/$COURSE_ID/" \
--header 'Authorization: Token <Your Token>'
```

```python
import requests

course_id = '<Course ID>'

requests.delete(
  f"{base_url}/trip-management/third-party-api/v2/courses/{course_id}/",
  headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`DELETE /trip-management/third-party-api/v2/courses/{course_id}/`


### Path parameters


Name | Type | Description
----- | ---- | -----------
course_id | string | The ID of the course to delete

<aside class="success">
You can find ID of your course in the response body of <a href="/#create-trip">Create Trip</a> or <a href="/#add-course">Add Course</a> requests.
</aside>

<aside class="warning">
The given course ID most belong to your client.
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

Response body is a serialized trip. For a detailed version of it take a look at the response body of [Create Trip](/#create-trip) request.

### Errors

Code | Description
---- | -----------
not_authenticated    | Token is missing or invalid
parse_error          | The request body is not a valid JSON string (has syntax error)
invalid_request_body | Request body does not follow the valid format
record_not_found     | A course with the given ID does not belong to your client or doesn't exist at all
single_course        | The course belongs to a trip that has only one course



## Get Trip

Returns information about one of your trips indicated by its ID.

> Request example:

```shell
TRIP_ID="<Trip ID>"

curl --location --request GET "$BASE_URL/trip-management/third-party-api/v2/trips/$TRIP_ID/" \
--header 'Authorization: Token <Your Token>'
```

```python
import requests

trip_id = '<Trip ID>'

requests.get(
  f"{base_url}/trip-management/third-party-api/v2/trips/{trip_id}/",
  headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`GET /trip-management/third-party-api/v2/trips/{trip_id}/`


### Path parameters


Name | Type | Description
----- | ---- | -----------
trip_id | string | The ID of the trip

<aside class="success">
You can find ID of your trip in the response body of <a href="/#create-trip">Create Trip</a> request
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

Response body is a serialized trip. For a detailed version of it take a look at the response body of [Create Trip](/#create-trip) request.

### Errors

Code | Description
---- | -----------
not_authenticated        | Token is missing or invalid
parse_error              | The request body is not a valid JSON string (has syntax error)
invalid_request_body     | Request body does not follow the valid format
record_not_found         | A trip with the given ID does not belong to your client or doesn't exist at all


## List Trips

Returns list of your trips matching the given conditions.

> Request example:

```shell
curl --location --request GET "$BASE_URL/trip-management/third-party-api/v2/trips/?area_id=2&state=pickup&from_datetime=2021-11-02T14:48:18+0330&to_datetime=2021-11-02T14:48:18+0330&offset=0&limit=10" \
--header "Authorization: Token <Your Token>"
```

```python
import requests

params = {
  'area_id': 2,
  'state': 'pickup',
  'from_datetime': '2021-11-02T14:48:18+0330',
  'to_datetime': '2021-11-02T14:48:18+0330',
  'offset': 0,
  'limit': 10,
}
query = '&'.join([f'{k}={v}' for k, v in params.items()])

requests.get(
  f"{base_url}/trip-management/third-party-api/v2/trips/?{query}",
  headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`GET /trip-management/third-party-api/v2/trips/`


### Path parameters

Name | Type | Description
---- | ---- | -----------
area_id       | number [integer] | **Filter:** The ID of the area of the pickup location. You can find list of available areas in the [List Areas](/#list-areas) endpoint
state         | string  | **Filter:** The state of the trip. Should be one of the states specified in [Trip](/#trip) definition
from_datetime | string [date-time] | **Filter:** Minimum acceptable value for trip's `created_at` field. This filter is **inclusive**
to_datetime   | string [date-time] | **Filter:** Maximum acceptable value for trip's `created_at` field. This filter is **inclusive**
offset        | number [integer] | Give results excluding the first **offset** number of objects
limit         | number [integer] | Give *at most* **limit** number of results. The default value is 100


<aside class="notice">
Server might not have or decide not to send you as many result items as <code>limit</code> but it never sends you more than that.
</aside>

### Response

> Response example:

```json
{
    "data": [
        {
            "area": {
                "id": "27",
                "name": "پونک"
            },
            "assigned_at": null,
            "courier": null,
            "courses": [
                {
                    "address": "هران، خیابان استاد معین، پلاک ۱۲",
                    "bill_number": "DEL-119",
                    "dropped_off_at": null,
                    "id": "3b0e8576-f802-44f5-8654-d65e11fd3035",
                    "location": {
                        "latitude": 35.753515,
                        "longitude": 51.332119
                    },
                    "manifest_items": [],
                    "name": "علی علوی",
                    "payment": {
                        "payment_type": "cash",
                        "price": 0
                    },
                    "phone_number": "09379187928",
                    "tracking_url": "https://www.staging.mia.re/p/trip_watching/#!/3b0e8576f8",
                    "trip_id": "05fd0397-5a78-4852-98e7-9fbc310685c0"
                }
            ],
            "created_at": "2021-09-29T17:07:27+0330",
            "id": "05fd0397-5a78-4852-98e7-9fbc310685c0",
            "picked_up_at": null,
            "pickup": {
                "address": "تهران، صادقیه، بلوار آیت الله کاشانی",
                "deadline": "2021-09-29T17:19:19+0330",
                "image": "https://example.com/restaurants/bm_logo.png",
                "location": {
                    "latitude": 35.753515,
                    "longitude": 51.332119
                },
                "name": "بزرگ‌ترین رستوران خاور میانه و حواشی آن تا اقیانوس اطلس",
                "phone_number": "09379187928"
            },
            "state": "canceled_by_miare"
        }
    ],
    "next": "https://staging.ws.mia.re/trip-management/third-party-api/v2/trips/?offset=1&limit=1&from_datetime=2020-11-02T14:48:18Z&to_datetime=2021-12-02T14:48:18Z",
    "previous": "",
    "total_count": 35
}
```

Value | Type | Description
----- | ---- | -----------
**data** | array | List of trips matching the given query. Each object in this array is a serialized trip. For a detailed version of it take a look at the response body of [Create Trip](/#create-trip) request.
**next** | string [uri] **(nullable)** | The valid url to the next set of result with the same predicates. Will be **null** if there are no more objects left matching the given filters
**previous** | string [uri] **(nullable)** | The valid url to the previous set of result with the same predicates. Will be **null** if there was no objects matching the given filters before this page
**total_count** | number [integer] | The **total** number of objects matching the given filters (not only the items in this page)


### Errors

Code | Description
---- | -----------
not_authenticated | Token is missing or invalid


# Areas

Services related to getting information about working areas.


## List Areas

Returns list of Miare areas and your usage of concurrency in each area.

> Request example:

```shell
curl --location --request GET "$BASE_URL/area/third-party-api/v2/areas" \
--header "Authorization: Token <Your Token>"
```

```python
import requests

requests.get(
  f"{base_url}/area/third-party-api/v2/areas/",
  headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`GET /area/third-party-api/v2/areas/`

### Response

> Response example:

```json
[{
	"id": 27,
	"max_ongoing_trips": 20,
	"name": "پونک",
	"ongoing_trips": 8,
	"polygon": {
		"type": "Polygon",
		"coordinates": [
			[
				51.3318602487183,
				35.7511077950424
			],
			[
				51.3329908968811,
				35.7402265889212
			],
			[
				51.3408319127502,
				35.7456830619354
			],
			[
				51.3318602487183,
				35.7511077950424
			]
		]
	}
}]
```

The response is an array of all of the active areas in the Miare whether you have any trips or concurrency in it or not.
The details about properties of each objects is as follows:

Value | Type | Description
----- | ---- | -----------
**id** | number [integer] | The constant identifier of the area
**name** | string | The human readable name of the area
**ongoing_trips** | number [integer] | The number of trips currently [**active**](/#trip) trips in this area.
**max_ongoing_trips** | number [integer] | Your [**concurrency**](/#concurrency) limit in this area at this shift
**polygon** | object | A valid [Polygon](https://geojson.org/geojson-spec.html#id4) object
polygon.**type** | string | GeoJson feature type. Its value is always "Polygon"
polygon.**coordinates** | array | Coordinates of the points forming the exterior ring of the polygon.

<aside class="notice">
Based on [GeoJson.org](https://geojson.org/geojson-spec.html#positions)'s convention, the order of coordinates in the coordinates inner arrays is <b>longitude</b>, <b>latitude</b>.
</aside>

### Errors

Code | Description
---- | -----------
not_authenticated | Token is missing or invalid

