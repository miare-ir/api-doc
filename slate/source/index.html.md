---
title: Miare API

language_tabs: # must be one of https://git.io/vQNgJ

  - shell
  - python

toc_footers:

  - <a href='Miare - Third Party API.postman_collection.json' download>Postman Collection</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

search: true

code_clipboard: true

meta:

  - name: description
    content: Documentation for the Miare API

---

# Introduction

This is Miare's Third-Party API, designed to enable registered Miare users to integrate our
delivery services into their
applications.

This document walks you through the steps required to use Miare's services and endpoints.
If you need any further information about how Miare works or you are still not sure whether or not
our services meet
your needs,
feel free to check out our website <a href='https://miare.ir'>miare.ir</a> or contact us
at <a href='mailto: tech@miare.ir'>
tech@miare.ir</a>.

# Registration

To use our services, you need to first have an API account. An API account is a specific type of
account that
can access our Third-Party endpoints and receive our event callbacks.

In order to create an account and discuss the basic terms and financial details of our services,
you can reach our sales
team at
<a href=tel:02191009282>02191009282</a> Ext. 3, they will provide us with the required information
to create your
account.
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
In Miare, bikers don't pick and choose what packages they deliver. Instead, each order is
automatically assigned to one
of the available couriers based on their distance from the source and many other parameters.

### Trip

An order given by one of our clients for one or more packages to be delivered. A trip can have one
and only one source
but multiple destinations (and multiple packages to be delivered at those destinations).
Our clients may also indicate that they wish the courier to return to the source at the end of the
trip.

#### Trip States

Each trip can be in only one the following states at any given time

| Name               | Description                                                                                                                |
|--------------------|----------------------------------------------------------------------------------------------------------------------------|
| assign_queue       | Trip is created but no courier has been assigned to it yet                                                                 |
| pickup             | Trip is assigned to a courier and the courier is on his way to pickup packages from the source                             |
| dropoff            | Trip is picked up by the courier and he is on his way to deliver the packages                                              |
| delivered          | All of the packages of the trip are delivered (and if trip was a round trip, courier has returned to the source)           |
| canceled_by_miare  | Trip is canceled by our support staff. This only happens with source's aggrement or due to a violation of terms of service |
| returning          | When the trip is a round trip and it is returning to the source.                                                           |
| canceled_by_delay  | Trip has been cancelled by Miare due to fulfillment delay from your side                                                   |
| canceled_by_client | Trip is canceled by client (either from the web panel or Third Party API)                                                  |

### Course

A destination and a package to be delivered at that destination. Each trip can have one or more
courses.
As an example, if a **trip** has two **course**s, our courier will drive to the source, pick up two
packages,
and deliver them to two separate customers in two different destinations.

### Area

Part of the city defined by a set of latitude-longitudes forming a polygon that Miare operates in
it.
If **source** of a trip is inside an area, that trip is considered part of trips of that specific
area.

### Shift

A period of time in each day defined by a start and an end time. If deadline of a trip is within a
shift, that trip is
considered part of trips of that specific shift.

<aside class="success">
The current state of the shifts can be considered constant, and any change will be announced by the sales team.
</aside>

Start and end time of shifts area as follows:

| name     | start | end   |
|----------|-------|-------|
| morning  | 08:15 | 11:00 |
| noon     | 11:00 | 17:30 |
| night    | 17:30 | 23:30 |
| midnight | 23:30 | 03:00 |

### Concurrency

The maximum number of **active** trips in each **area** during each **shift** for a client.
If the concurrency limit for a client is reached that is if the total number of its active trips is
equal to its
concurrency limit, the next attempts to create new trips will fail.
Canceling a trip (by client) or ending it will free up a concurrency limit (by moving the trip out
of the active trips
list).

<aside class="success">
Only trips in the <code>assign_queue</code>, <code>pickup</code>, <code>dropoff</code>, and <code>delivered</code> states are considered active and contribute to the concurrency limit of the client.
</aside>

### Issue

An operational problem that is reported or detected on a trip.

#### Issue States

Each issue can be in only one the following states at any given time

| Name     | Description                                                  |
|----------|--------------------------------------------------------------|
| reported | Issue is created but no staff has picked the issue to follow |
| picked   | Issue is picked by a staff and is investigating it           |
| resolved | Issue is resolved by one of the user types                   |

# Servers

Miare has two sets of servers.

## Staging

Sandbox servers -usually referred to as **Staging**- are the playground for development teams to
test integration of
their application with ours.
In the staging servers, your account will be given 10 concurrencies in each of our areas and all
trips are free of
charge.

<aside class="notice">
There is no active courier on the staging so your trips will never be assigned to a courier, delivered, or ended so you have to cancel your trips to avoid reaching the concurrency limit.
</aside>

<aside class="success">
The main url part for staging server is <code>staging.miare.ir</code> according to the production servers which is <code>miare.ir</code>.
</aside>

## Production

Main servers -usually referred to as **Production**- are the main product's servers for real-world
use.
In production servers, your concurrency should be purchased through the sales team and trips have
normal rates applied
to them.

<aside class="notice">
You should only start using our production server after all technical and non-technical major integration issues are resolved.
</aside>

# Authentication

Each client has a specific API key called **Token** for each of our servers. The token is the sole
representative of an
account on a server as well as its trips, courses, and financial transactions.
The sales team will provide you with the staging token after the **registration** process and the
production token after
the **test** process.

<aside class="success">
At this moment, there is no automatic process for resetting your token, but you can ask the sales team to reset it for you at any time.
</aside>

<aside class="notice">
Note that staging and production tokens are in no way related to each other and their accounts are two separate accounts on two sets of servers.
</aside>

> To authorize, pass the authorization token among headers and prefix it with `Token `:

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

Miare API expects for the token to be included in **ALL** API requests to the server in
an `Authorization` header that
looks like the following:

`Authorization: Token <Your Token>`

<aside class="notice">
You must replace <code>&#60;Your Token&#62;</code> with your personal API key.
</aside>

# Errors

All application level error of our API are in the following format:

<code>
{"code": "constant_error_code", "detail": "Variable human readable details"}
</code>

The `code` value is the main indicator of what went wrong. You can use it in a `switch-case`
statement to determine the
type of error.

The `detail` value is human readable details of what went wrong which is suitable for developers (*
*not** users).

<aside class="warning">
You should <b>not</b> rely on the value of the <code>detail</code> for automatic error detection. It is both variable and subject to change.
</aside>

<aside class="success">
You can find possible error codes of each endpoints in this document at the end of each endpoint's related section.
</aside>

# Postman Collection

All of the services included in this document can be downloaded as
a [Postman Collection](https://www.postman.com/collection/)
from <a href="Miare - Third Party API.postman_collection.json" download>here</a>.

In order for the collection to be fully functional, you need to set
three [Postman Variables](https://learning.postman.com/docs/sending-requests/variables/) in your
environment.

Name | Description
----- | ---- | -----------
**staging_prefix** | Set it to `staging.` for connection to staging servers or leave it empty for
production servers
**token** | Your token. Make sure it is token for the right server set
**callback_address** | Address of your http server waiting for Miare webhook calls to come in

# Delivery

> Base URL of staging servers for delivery services:

```shell
BASE_URL="https://ws.staging.miare.ir/trip-management/third-party-api/v2"
```

```python
base_url = "https://ws.staging.miare.ir/trip-management/third-party-api/v2"
```

> Base URL of production servers for delivery services:

```shell
BASE_URL="https://ws.miare.ir/trip-management/third-party-api/v2"
```

```python
base_url = "https://ws.miare.ir/trip-management/third-party-api/v2"
```

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
    "deadline": "2021-11-01T17:12:00+0330"
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
    data=data,
)
```

> If you are getting the a `Deadline: (value_in_past)` error, make sure to update
> the `pickup.deadline` to a time in the
> future.

### HTTP Request

`POST /trips/`

### Body

| Value                               | Type                 | Description                                                                                                                                                                                                                                   |
|-------------------------------------|----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **pickup**                          | object               | The source of the trip                                                                                                                                                                                                                        |
| pickup.**name**                     | string               | The human readable name of the pickup                                                                                                                                                                                                         |
| pickup.**phone_number**             | string               | The phone number associated with the source which will be used by courier and support staffs in order to contact to pickup if necessary                                                                                                       |
| pickup.**address**                  | string               | The human readable address of the source, preferably down to every necessary detail for a human to find the source quickly                                                                                                                    |
| pickup.**image**                    | string [uri]         | A valid URL which points to an image file which should be the logo of the pickup. This image will be used in both support panel, and courier’s application. Make sure that the URL is both reachable and is configured to allow CORS requests |
| pickup.**location**                 | object               | The exact location of the pickup                                                                                                                                                                                                              |
| pickup.location.**latitude**        | number [double]      | The latitude of the pickup location                                                                                                                                                                                                           |
| pickup.location.**longitude**       | number [double]      | The longitude of the pickup location                                                                                                                                                                                                          |
| pickup.**deadline**                 | string [date-time]   | The time that you expect the courier to arrive at the pickup, so optimally it should be the time package content is ready and packaged. **Can't be in the past**                                                                              |
| **courses**                         | array                | List of destinations of the trips                                                                                                                                                                                                             |
| courses.**bill_number**             | string               | An string field left for you to store sort of a human readable bill number in it which will be used as a reference point among our support team, you and the pickup staffs                                                                    |
| courses.**name**                    | string               | Name of the dropp-off                                                                                                                                                                                                                         |
| courses.**phone_number**            | string               | The phone number associated with the drop-off which will be used by courier and support staffs in order to contact to them if necessary                                                                                                       |
| coureses.**address**                | string               | The human readable drop-off address, preferably down to every necessary detail for a human to find it quickly                                                                                                                                 |
| courses.location                    | object               | The exact location of the drop-off.                                                                                                                                                                                                           |
| courses.location.**latitude**       | number [double]      | The latitude of the drop-off location                                                                                                                                                                                                         |
| courses.location.**longitude**      | number [double]      | The longitude of the drop-off location                                                                                                                                                                                                        |
| courses.manifest_items              | array **(Optional)** | The contents of the package to be delivered to the drop-off                                                                                                                                                                                   |
| courses.manifest_items.**name**     | string               | Human readable name of the content which will be verified by courier                                                                                                                                                                          |
| courses.manifest_items.**quantity** | string               | The quantity of the item                                                                                                                                                                                                                      |

### Response

> Response example:

```json
{
  "id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
  "is_round_trip": false,
  "created_at": "2021-11-01T16:59:00+0330",
  "assigned_at": "2021-11-01T17:00:00+0330",
  "arrived_at": "2021-11-01T17:02:00+0330",
  "picked_up_at": "2021-11-01T17:04:00+0330",
  "departed_at": "2021-11-01T17:06:00+0330",
  "state": "dropoff",
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
  "devlivery_cost": null,
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
      ,
      "old_trip_id": "00000000-0000-0000-0000-000000000000",
      "bill_number": "DEL-119",
      "name": "علی علوی",
      "address": "تهران، خیابان استاد معین، پلاک ۱۲",
      "dropped_off_at": null,
      "phone_number": "09123456789",
      "tracking_url": "https://www.staging.miare.ir/p/trip_watching/#!/7484f5305e",
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
  ]
}
```

| Value                               | Type                              | Description                                                                                                                                                                                                                                                         |
|-------------------------------------|-----------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **id**                              | string                            | Universally unique identifier of this trip                                                                                                                                                                                                                          |
| **is_round_trip**                   | boolean                           | The is_round_trip determines that a trip is a round trip or not                                                                                                                                                                                                     |
| **created_at**                      | string [date-time]                | The exact time this trip was created on our servers                                                                                                                                                                                                                 |
| **assigned_at**                     | string [date-time] **(nullable)** | The assign datetime of the time this trip was assigned to its courier. Will be **null** if trip is not assigned to a courier yet                                                                                                                                    |
| **arrived_at**                      | string [date-time] **(nullable)** | The datetime that the courier of the trip arrived to the source. Will be **null** if courier has not assigned or has not arrived to the pickup location                                                                                                             |
| **picked_up_at**                    | string [date-time] **(nullable)** | The datetime that the courier of the trip picked up its content from the source. Will be **null** if courier is not assigned or is not picked up packages yet                                                                                                       |
| **departed_at**                     | string [date-time] **(nullable)** | The datetime that the courier of the trip has left the pickup location. Will be **null** if courier is not assigned or has not left the pickup location yet                                                                                                         |
| **state**                           | string                            | The current state of the trip. Is one of the following values: "assign_queue" "pickup" "dropoff" "delivered" "canceled_by_miare" "returning" "canceled_by_delay" "canceled_by_client"  . You can find a description about each of these states [here](#trip) |
| **pickup**                          | object                            | The source of the trip                                                                                                                                                                                                                                              |
| pickup.**name**                     | string                            | The human readable name of the pickup                                                                                                                                                                                                                               |
| pickup.**phone_number**             | string                            | The phone number associated with the source which will be used by courier and support staffs in order to contact to pickup if necessary                                                                                                                             |
| pickup.**address**                  | string                            | The human readable address of the source, preferably down to every necessary detail for a human to find the source quickly                                                                                                                                          |
| pickup.**image**                    | string [uri]                      | A valid URL which points to an image file which should be the logo of the pickup. This image will be used in both support panel, and courier’s application. Make sure that the URL is both reachable and is configured to allow CORS requests                       |
| pickup.**location**                 | object                            | The exact location of the pickup                                                                                                                                                                                                                                    |
| pickup.location.**latitude**        | number [double]                   | The latitude of the pickup location                                                                                                                                                                                                                                 |
| pickup.location.**longitude**       | number [double]                   | The longitude of the pickup location                                                                                                                                                                                                                                |
| pickup.**deadline**                 | string [date-time]                | The time that you expect the courier to arrive at the pickup, so optimally it should be the time package content is ready and packaged. **Can't be in the past**                                                                                                    |
| **courses**                         | array                             | List of destinations of the trips                                                                                                                                                                                                                                   |
| courses.**id**                      | string                            | Universally unique identifier of this course                                                                                                                                                                                                                        |
| course.**trip_id**                  | string                            | Universally unique identifier of the trip that this course belongs to it                                                                                                                                                                                            |
| course.**old_trip_id**              | string                            | The trip id of the trip before batching. Will be zero value if batching has not been occurred                                                                                                                                                                       |
| courses.**bill_number**             | string                            | An string field left for you to store sort of a human readable bill number in it which will be used as a reference point among our support team, you and the pickup staffs                                                                                          |
| courses.**name**                    | string                            | Name of the dropp-off                                                                                                                                                                                                                                               |
| courses.**phone_number**            | string                            | The phone number associated with the drop-off which will be used by courier and support staffs in order to contact to them if necessary                                                                                                                             |
| coureses.**address**                | string                            | The human readable drop-off address, preferably down to every necessary detail for a human to find it quickly                                                                                                                                                       |
| courses.location                    | object **(nullable)**             | The exact location of the pickup. If there is no provided drop-off location, the courier will find the location based on the address and accounting calculations will be based on that location                                                                     |
| courses.location.**latitude**       | number [double]                   | The latitude of the drop-off location                                                                                                                                                                                                                               |
| courses.location.**longitude**      | number [double]                   | The longitude of the drop-off location                                                                                                                                                                                                                              |
| courses.manifest_items              | array **(nullable)**              | The contents of the package to be delivered to the drop-off                                                                                                                                                                                                         |
| courses.manifest_items.**name**     | string                            | Human readable name of the content which will be verified by courier                                                                                                                                                                                                |
| courses.manifest_items.**quantity** | string                            | The quanitiy of the item                                                                                                                                                                                                                                            |
| course.**tracking_url**             | string [uri]                      | The URL of a webpage in which the end customer can track the exact state and location of his/her package while it's being delivered                                                                                                                                 |
| course.**dropped_off_at**           | string [date-time] **(nullable)** | The exact time this course we delivered to the customer. It will be **null** if the course is not delivered yet                                                                                                                                                     |
| course.**payment**                  | object                            | The payment information of the course                                                                                                                                                                                                                               |
| course.payment.**payment_type**     | string                            | They selected method for this course's payment. At this moment the only available method for API users is `cash`                                                                                                                                                    |
| course.payment.**price**            | string                            | The price of the package content (**not** to be confused with delivery cost). At this moment the only available value for API users is 0                                                                                                                            |
| **area**                            | object                            | The detected area of the trip (based on pickup.location)                                                                                                                                                                                                            |
| area.**id**                         | string                            | The identifier of this trip's area                                                                                                                                                                                                                                  |
| area.**name**                       | string                            | Human readable name of this trip's area                                                                                                                                                                                                                             |
| **delivery_cost**                   | integer **(nullable)**            | The final cost of a Trip Delivery. It will be **null** until trip is not delivered                                                                                                                                                                                  |
| courier                             | object **(nullable)**             | Courier of this trip. Will be **null** if trip is not assigned to a courier yet                                                                                                                                                                                     |
| courier.**name**                    | string                            | Name of the courier                                                                                                                                                                                                                                                 |
| courier.**phone_number**            | string                            | Phone number of the courier                                                                                                                                                                                                                                         |
| courier.**image**                   | string [uri]                      | The URL of courier's profile picture                                                                                                                                                                                                                                |
| courier.**location**                | object                            | Last known location of the courier                                                                                                                                                                                                                                  |
| courier.location.**latitude**       | number [double]                   | Latitude of the last known location of the courier                                                                                                                                                                                                                  |
| courier.location.**longitude**      | number [double]                   | Longitude of the last known location of the courier                                                                                                                                                                                                                 |
| courier.**location_updated_at**     | string [date-time]                | Datetime of the last location of the courier                                                                                                                                                                                                                        |

### Errors

| Code                   | Description                                                                                                                                                                                                                                                                                                          |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| not_authenticated      | Token is missing or invalid                                                                                                                                                                                                                                                                                          |
| parse_error            | The request body is not a valid JSON string (has syntax error)                                                                                                                                                                                                                                                       |
| invalid_request_body   | Request body does not follow the valid format                                                                                                                                                                                                                                                                        |
| concurrency_limit      | Client does not have enough concurrency in the area of the pickup location in the shift specified by deadline                                                                                                                                                                                                        |
| service_level_order    | Based on the current environmental state, your service level is too low to receive services from Miare. Service limitation error mostly happens due to whether conditions or special occasions. In order to increase your service level or if you need more details about this limitation please contant sales team. |
| duplicated_bill_number | There is already a course related to trip with states of (`assign_queue`, `pickup`, `dropoff`) with this bill number.                                                                                                                                                                                                |

## Cancel Trip

Cancels one of your trips indicated by its ID.

<aside class="warning">
Note that a trip **cannot** be canceled when it has passed 30 seconds since we’ve assigned it to a courier or the courier has already arrived at the source. In special cases that you still need to cancel the trip, please contact our support.
</aside>

> Request example:

```shell
TRIP_ID="<Trip ID>"

curl --location --request POST "$BASE_URL/trips/$TRIP_ID/cancel/" \
--header 'Authorization: Token <Your Token>'
```

```python
import requests

trip_id = "<Trip ID>"

requests.post(
    f"{base_url}/trips/{trip_id}/cancel/",
    headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`POST /trips/{trip_id}/cancel/`

### Path parameters

| Name    | Type   | Description                  |
|---------|--------|------------------------------|
| trip_id | string | The ID of the trip to cancel |

<aside class="success">
You can find ID of your trip in the response body of <a href="#create-trip">Create Trip</a> request
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
  "is_round_trip": false,
  "state": "canceled_by_client",
  "picked_up_at": null,
  "assigned_at": null,
  "arrived_at": null,
  "departed_at": null,
  "area": {
    "id": "3",
    "name": "یوسف آباد"
  },
  "delivery_cost": null,
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
      "tracking_url": "https://www.staging.miare.ir/p/trip_watching/#!/7484f5305e",
      "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
      "old_trip_id": "00000000-0000-0000-0000-000000000000"
    }
  ]
}
```

Response body is a serialized trip. For a detailed version of it take a look at the response body
of [Create Trip](#create-trip) request.

<aside class="notice">
Successful calls to this endpoint will update the state of the trip to <code>canceled_by_client</code>.
</aside>

### Errors

| Code                     | Description                                                                     |
|--------------------------|---------------------------------------------------------------------------------|
| not_authenticated        | Token is missing or invalid                                                     |
| parse_error              | The request body is not a valid JSON string (has syntax error)                  |
| invalid_request_body     | Request body does not follow the valid format                                   |
| record_not_found         | A trip with the given ID does not belong to your client or doesn't exist at all |
| canceling_after_deadline | The trip has been in assigned state for more than 30 seconds                    |
| canceling_arrived_trip   | The trip's courier is already at the source location                            |
| canceling_ended_trip     | The trip is already ended (delivered)                                           |
| invalid_state_change     | Changing the trip's state from its current state to canceled is not possible    |

## Round Trip

Change state of trip to "returning" indicated by its ID.

<aside class="warning">
Note that only trips with <code>assign_queue</code>, <code>pickup</code>, <code>dropoff</code> 
state can be returned and other states cannot be changed.
</aside>

> Request example:

```shell
TRIP_ID="<Trip ID>"

curl --location --request POST "$BASE_URL/trips/$TRIP_ID/round-trip/" \
--header 'Authorization: Token <Your Token>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "is_round_trip": "true",
}'
```

```python
import requests

trip_id = "<Trip ID>"

data = {
    "is_round_trip": True
}

requests.post(
    f"{base_url}/trips/{trip_id}/round-trip/",
    headers={"Authorization": "Token <Your Token>"},
    data=data,
)
```

### HTTP Request

`POST /trips/{trip_id}/round-trip/`

### Path parameters

| Name    | Type   | Description                 |
|---------|--------|-----------------------------|
| trip_id | string | The ID of the trip to round |

<aside class="success">
You can find ID of your trip in the response body of <a href="#create-trip">Create Trip</a> request
</aside>

<aside class="warning">
The given trip ID most belong to your client.
</aside>

### Body

| Value         | Type    | Description                                                              |
|---------------|---------|--------------------------------------------------------------------------|
| is_round_trip | boolean | for convert trip to round trip set it to true. otherwise set it to false |

### Response

> Response example:

```json
{
  "created_at": "2021-11-01T16:59:00+0330",
  "id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
  "state": "returning",
  "is_round_trip": true,
  "picked_up_at": null,
  "assigned_at": null,
  "arrived_at": null,
  "departed_at": null,
  "area": {
    "id": "3",
    "name": "یوسف آباد"
  },
  "delivery_cost": null,
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
      "tracking_url": "https://www.staging.miare.ir/p/trip_watching/#!/7484f5305e",
      "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
      "old_trip_id": "00000000-0000-0000-0000-000000000000"
    }
  ]
}
```

Response body is a serialized trip. For a detailed version of it take a look at the response body
of [Create Trip](#create-trip) request.

<aside class="notice">
Successful calls to this endpoint will update the trip state, is_round_trip to 
<code>returning</code>,<code>true</code>
</aside>

### Errors

| Code                 | Description                                                                      |
|----------------------|----------------------------------------------------------------------------------|
| not_authenticated    | Token is missing or invalid                                                      |
| parse_error          | The request body is not a valid JSON string (has syntax error)                   |
| invalid_request_body | Request body does not follow the valid format                                    |
| record_not_found     | A trip with the given ID does not belong to your client or doesn't exist at all  |
| invalid_state_change | Changing the trip's state from its current state to "returning" is not possible. |

## Update Trip

Updates an existing trip and it's courses.

<aside class="warning">
You can only update trip in the <code>assign_queue</code> or <code>pickup</code> states.
</aside>

> Request example:

```shell
TRIP_ID="<Trip ID>"

curl --location --request PUT "$BASE_URL/trips/$TRIP_ID/ \
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

trip_id = "<Trip ID>"

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

requests.put(
    base_url + "/trips/{trip_id}/",
    headers={"Authorization": "Token <Your Token>"},
    data=data,
)
```

> If you are getting the a `Deadline: (value_in_past)` error, make sure to update
> the `pickup.deadline` to a time in the
> future.

### HTTP Request

`PUT /trips/{trip_id}/`

### Path parameters

| Name    | Type   | Description                  |
|---------|--------|------------------------------|
| trip_id | string | The ID of the trip to update |

<aside class="success">
You can find ID of your trip in the response body of <a href="#create-trip">Create Trip</a> request
</aside>

<aside class="warning">
The given trip ID most belong to your client.
</aside>

### Body

| Value                               | Type                 | Description                                                                                                                                                                                                                                   |
|-------------------------------------|----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **pickup**                          | object               | The source of the trip                                                                                                                                                                                                                        |
| pickup.**name**                     | string               | The human readable name of the pickup                                                                                                                                                                                                         |
| pickup.**phone_number**             | string               | The phone number associated with the source which will be used by courier and support staffs in order to contact to pickup if necessary                                                                                                       |
| pickup.**address**                  | string               | The human readable address of the source, preferably down to every necessary detail for a human to find the source quickly                                                                                                                    |
| pickup.**image**                    | string [uri]         | A valid URL which points to an image file which should be the logo of the pickup. This image will be used in both support panel, and courier’s application. Make sure that the URL is both reachable and is configured to allow CORS requests |
| pickup.**location**                 | object               | The exact location of the pickup                                                                                                                                                                                                              |
| pickup.location.**latitude**        | number [double]      | The latitude of the pickup location                                                                                                                                                                                                           |
| pickup.location.**longitude**       | number [double]      | The longitude of the pickup location                                                                                                                                                                                                          |
| pickup.**deadline**                 | string [date-time]   | The time that you expect the courier to arrive at the pickup, so optimally it should be the time package content is ready and packaged. **Can't be in the past**                                                                              |
| **courses**                         | array                | List of destinations of the trips                                                                                                                                                                                                             |
| courses.**bill_number**             | string               | An string field left for you to store sort of a human readable bill number in it which will be used as a reference point among our support team, you and the pickup staffs                                                                    |
| courses.**name**                    | string               | Name of the dropp-off                                                                                                                                                                                                                         |
| courses.**phone_number**            | string               | The phone number associated with the drop-off which will be used by courier and support staffs in order to contact to them if necessary                                                                                                       |
| coureses.**address**                | string               | The human readable drop-off address, preferably down to every necessary detail for a human to find it quickly                                                                                                                                 |
| courses.location                    | object               | The exact location of the drop-off.                                                                                                                                                                                                           |
| courses.location.**latitude**       | number [double]      | The latitude of the drop-off location                                                                                                                                                                                                         |
| courses.location.**longitude**      | number [double]      | The longitude of the drop-off location                                                                                                                                                                                                        |
| courses.manifest_items              | array **(Optional)** | The contents of the package to be delivered to the drop-off                                                                                                                                                                                   |
| courses.manifest_items.**name**     | string               | Human readable name of the content which will be verified by courier                                                                                                                                                                          |
| courses.manifest_items.**quantity** | string               | The quantity of the item                                                                                                                                                                                                                      |

### Response

> Response example:

```json
{
  "id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
  "is_round_trip": false,
  "created_at": "2021-11-01T16:59:00+0330",
  "assigned_at": "2021-11-01T17:00:00+0330",
  "arrived_at": "2021-11-01T17:02:00+0330",
  "picked_up_at": "2021-11-01T17:04:00+0330",
  "departed_at": "2021-11-01T17:06:00+0330",
  "state": "dropoff",
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
  "devlivery_cost": null,
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
      ,
      "old_trip_id": "00000000-0000-0000-0000-000000000000",
      "bill_number": "DEL-119",
      "name": "علی علوی",
      "address": "تهران، خیابان استاد معین، پلاک ۱۲",
      "dropped_off_at": null,
      "phone_number": "09123456789",
      "tracking_url": "https://www.staging.miare.ir/p/trip_watching/#!/7484f5305e",
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
  ]
}
```

Response body is a serialized trip. For a detailed version of it take a look at the response body
of [Create Trip](#create-trip) request.

### Errors

| Code                   | Description                                                                                                          |
|------------------------|----------------------------------------------------------------------------------------------------------------------|
| not_authenticated      | Token is missing or invalid                                                                                          |
| parse_error            | The request body is not a valid JSON string (has syntax error)                                                       |
| invalid_request_body   | Request body does not follow the valid format                                                                        |
| record_not_found       | A trip with the given ID does not belong to your client or doesn't exist at all                                      |
| too_late               | Trip's state is later than `pickup`                                                                                  |
| duplicated_bill_number | There is already a course related to trip with states of (`assign_queue`, `pickup`, `dropoff`) with this bill number |

## Add Course

Adds a new course to the courses of a trip.

<aside class="warning">
You can only add a course to a trip in the <code>assign_queue</code> or <code>pickup</code> states.
</aside>

> Request example:

```python
import requests

trip_id = "<Trip ID>"

requests.post(
    f"{base_url}/trips/{trip_id}/cancel/",
    headers={"Authorization": "Token <Your Token>"},
)
```

```shell
TRIP_ID="<Trip ID>"

curl --location --request PATCH "$BASE_URL/trips/$TRIP_ID/courses" \
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
    f"{base_url}/trips/{trip_id}/courses/",
    headers={"Authorization": "Token <Your Token>"},
    data=data,
)
```

### HTTP Request

`PATCH /trips/{trip_id}/courses/`

### Path parameters

| Name    | Type   | Description                           |
|---------|--------|---------------------------------------|
| trip_id | string | The ID of the trip to add a course to |

<aside class="success">
You can find ID of your trip in the response body of <a href="#create-trip">Create Trip</a> request
</aside>

### Body

| Value                       | Type                  | Description                                                                                                                                                                                     |
|-----------------------------|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **bill_number**             | string                | An string field left for you to store sort of a human readable bill number in it which will be used as a reference point among our support team, you and the pickup staffs                      |
| **name**                    | string                | Name of the dropp-off                                                                                                                                                                           |
| **phone_number**            | string                | The phone number associated with the drop-off which will be used by courier and support staffs in order to contact to them if necessary                                                         |
| **address**                 | string                | The human readable drop-off address, preferably down to every necessary detail for a human to find it quickly                                                                                   |
| location                    | object **(Optional)** | The exact location of the pickup. If there is no provided drop-off location, the courier will find the location based on the address and accounting calculations will be based on that location |
| location.**latitude**       | number [double]       | The latitude of the drop-off location                                                                                                                                                           |
| location.**longitude**      | number [double]       | The longitude of the drop-off location                                                                                                                                                          |
| manifest_items              | array **(Optional)**  | The contents of the package to be delivered to the drop-off                                                                                                                                     |
| manifest_items.**name**     | string                | Human readable name of the content which will be verified by courier                                                                                                                            |
| manifest_items.**quantity** | string                | The quantity of the item                                                                                                                                                                        |

### Response

> Response example:

```json
{
  "id": "8e209688-19c9-4982-a755-517408b6a29f",
  "created_at": "2021-11-01T18:44:19+0330",
  "picked_up_at": null,
  "state": "assign_queue",
  "assigned_at": null,
  "arrived_at": null,
  "departed_at": null,
  "delivery_cost": null,
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
      "tracking_url": "https://www.staging.miare.ir/p/trip_watching/#!/c57cb6bce1",
      "trip_id": "8e209688-19c9-4982-a755-517408b6a29f",
      "old_trip_id": "00000000-0000-0000-0000-000000000000"
    }
  ]
}
```

Response body is a serialized trip. For a detailed version of it take a look at the response body
of [Create Trip](#create-trip) request.

### Errors

| Code                   | Description                                                                                                          |
|------------------------|----------------------------------------------------------------------------------------------------------------------|
| not_authenticated      | Token is missing or invalid                                                                                          |
| parse_error            | The request body is not a valid JSON string (has syntax error)                                                       |
| invalid_request_body   | Request body does not follow the valid format                                                                        |
| record_not_found       | A trip with the given ID does not belong to your client or doesn't exist at all                                      |
| too_late               | Trip's state is later than `pickup`                                                                                  |
| duplicated_bill_number | There is already a course related to trip with states of (`assign_queue`, `pickup`, `dropoff`) with this bill number |

## Remove Course

Removes the course indicated by the given ID from the courses of its trip.

<aside class="warning">
You can only remove courses from a trip in the <code>assign_queue</code> or <code>pickup</code> states which has at least <b>two</b> courses.
</aside>

> Request example:

```shell
COURSE_ID="<Course ID>"

curl --location --request DELETE "$BASE_URL/courses/$COURSE_ID/" \
--header 'Authorization: Token <Your Token>'
```

```python
import requests

course_id = '<Course ID>'

requests.delete(
    f"{base_url}/courses/{course_id}/",
    headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`DELETE /courses/{course_id}/`

### Path parameters

| Name      | Type   | Description                    |
|-----------|--------|--------------------------------|
| course_id | string | The ID of the course to delete |

<aside class="success">
You can find ID of your course in the response body of <a href="#create-trip">Create Trip</a> or <a href="#add-course">Add Course</a> requests.
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
  "arrived_at": null,
  "departed_at": null,
  "area": {
    "id": "3",
    "name": "یوسف آباد"
  },
  "delivery_cost": null,
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
      "tracking_url": "https://www.staging.miare.ir/p/trip_watching/#!/7484f5305e",
      "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
      "old_trip_id": "00000000-0000-0000-0000-000000000000"
    }
  ]
}
```

Response body is a serialized trip. For a detailed version of it take a look at the response body
of [Create Trip](#create-trip) request.

### Errors

| Code                 | Description                                                                       |
|----------------------|-----------------------------------------------------------------------------------|
| not_authenticated    | Token is missing or invalid                                                       |
| parse_error          | The request body is not a valid JSON string (has syntax error)                    |
| invalid_request_body | Request body does not follow the valid format                                     |
| record_not_found     | A course with the given ID does not belong to your client or doesn't exist at all |
| single_course        | The course belongs to a trip that has only one course                             |

## Get Trip

Returns information about one of your trips indicated by its ID.

> Request example:

```shell
TRIP_ID="<Trip ID>"

curl --location --request GET "$BASE_URL/trips/$TRIP_ID/" \
--header 'Authorization: Token <Your Token>'
```

```python
import requests

trip_id = "<Trip ID>"

requests.get(
    f"{base_url}/trips/{trip_id}/",
    headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`GET /trips/{trip_id}/`

### Path parameters

| Name    | Type   | Description        |
|---------|--------|--------------------|
| trip_id | string | The ID of the trip |

<aside class="success">
You can find ID of your trip in the response body of <a href="#create-trip">Create Trip</a> request
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
  "is_round_trip": false,
  "picked_up_at": null,
  "assigned_at": null,
  "arrived_at": null,
  "departed_at": null,
  "area": {
    "id": "3",
    "name": "یوسف آباد"
  },
  "delivery_cost": null,
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
      "tracking_url": "https://www.staging.miare.ir/p/trip_watching/#!/7484f5305e",
      "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
      "old_trip_id": "00000000-0000-0000-0000-000000000000"
    }
  ]
}
```

Response body is a serialized trip. For a detailed version of it take a look at the response body
of [Create Trip](#create-trip) request.

### Errors

| Code                 | Description                                                                     |
|----------------------|---------------------------------------------------------------------------------|
| not_authenticated    | Token is missing or invalid                                                     |
| parse_error          | The request body is not a valid JSON string (has syntax error)                  |
| invalid_request_body | Request body does not follow the valid format                                   |
| record_not_found     | A trip with the given ID does not belong to your client or doesn't exist at all |

## List Trips

Returns list of your trips matching the given conditions.

> Request example:

```shell
curl --location --request GET "$BASE_URL/trips/?area_id=2&state=pickup&bill_number=b1&bill_number=b3&from_datetime=2021-11-02T14:48:18+0330&to_datetime=2021-11-02T14:48:18+0330&offset=0&limit=10" \
--header "Authorization: Token <Your Token>"
```

```python
import requests

params = [
    ('area_id', 2),
    ('state', 'pickup'),
    ('bill_number', 'b1'),
    ('bill_number', 'b3'),
    ('is_round_trip', False),
    ('from_datetime', '2021-11-02T14:48:18+0330'),
    ('to_datetime', '2021-11-02T14:48:18+0330'),
    ('offset', 0),
    ('limit', 10),
]
query = '&'.join([f'{k}={v}' for k, v in params])

requests.get(
    f"{base_url}/trips/?{query}",
    headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`GET /trips/`

### Path parameters

| Name          | Type               | Description                                                                                                                           |
|---------------|--------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| area_id       | number [integer]   | **Filter:** The ID of the area of the pickup location. You can find list of available areas in the [List Areas](#list-areas) endpoint |
| state         | string             | **Filter:** The state of the trip. Should be one of the states specified in [Trip](#trip) definition                                  |
| is_round_trip | boolean            | **Filter:** Base on is_round_trip of trip                                                                                             |
| bill_number   | string             | **Filter:** The `bill_number` of the courses in the trips.                                                                            |
| from_datetime | string [date-time] | **Filter:** Minimum acceptable value for trip's `created_at` field. This filter is **inclusive**                                      |
| to_datetime   | string [date-time] | **Filter:** Maximum acceptable value for trip's `created_at` field. This filter is **inclusive**                                      |
| offset        | number [integer]   | Give results excluding the first **offset** number of objects                                                                         |
| limit         | number [integer]   | Give *at most* **limit** number of results. The default value is 100                                                                  |

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
      "arrived_at": null,
      "departed_at": null,
      "delivery_cost": null,
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
          "tracking_url": "https://www.staging.miare.ir/p/trip_watching/#!/3b0e8576f8",
          "trip_id": "05fd0397-5a78-4852-98e7-9fbc310685c0",
          "old_trip_id": "00000000-0000-0000-0000-000000000000"
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
      "state": "canceled_by_miare",
      "is_round_trip": false
    }
  ],
  "next": "https://ws.staging.miare.ir/trip-management/third-party-api/v2/trips/?offset=1&limit=1&from_datetime=2020-11-02T14:48:18Z&to_datetime=2021-12-02T14:48:18Z",
  "previous": "",
  "total_count": 35
}
```

| Value           | Type                        | Description                                                                                                                                                                                   |
|-----------------|-----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **data**        | array                       | List of trips matching the given query. Each object in this array is a serialized trip. For a detailed version of it take a look at the response body of [Create Trip](#create-trip) request. |
| **next**        | string [uri] **(nullable)** | The valid url to the next set of result with the same predicates. Will be **null** if there are no more objects left matching the given filters                                               |
| **previous**    | string [uri] **(nullable)** | The valid url to the previous set of result with the same predicates. Will be **null** if there was no objects matching the given filters before this page                                    |
| **total_count** | number [integer]            | The **total** number of objects matching the given filters (not only the items in this page)                                                                                                  |

### Errors

| Code              | Description                 |
|-------------------|-----------------------------|
| not_authenticated | Token is missing or invalid |

# Supports Integration

> Base URL of staging servers for support services:

```shell
BASE_URL="https://ws.staging.miare.ir/support/third-party-api/v2"
```

```python
base_url = "https://ws.staging.miare.ir/support/third-party-api/v2"
```

> Base URL of production servers for delivery services:

```shell
BASE_URL="https://ws.miare.ir/support/third-party-api/v2"
```

```python
base_url = "https://ws.miare.ir/support/third-party-api/v2"
```

Services related to reporting, responding, and getting information about issues.

## List Problems

Returns list of Miare problems. Each `issue` is categorized based on its `problem`. API clients
should store a mapping
of Miare's problems and theirs.

> Request example:

```shell
curl --location --request GET "$BASE_URL/problems/" \
--header "Authorization: Token <Your Token>"
```

```python
import requests

requests.get(
    f"{base_url}/problems/",
    headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`GET /problems/`

### Response

> Response example:

```json
[
  {
    "id": "728cfd38-3267-4bd0-bcec-c4fc904cebda",
    "title": "مجموعه سفارش را به کوریر تحویل نداده است",
    "reporter_type": "miare",
    "description_required": false
  },
  {
    "id": "728cfd38-3267-4bd0-bcec-c4fc904cebda",
    "title": "مشتری سفارش را تحویل نگرفته است",
    "reporter_type": "client",
    "description_required": true
  }
]
```

The response is an array of all the problems you may face in your webhook calls.
The details about properties of each object is as follows:

| Value                    | Type    | Description                                                                                                                                              |
|--------------------------|---------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| **id**                   | string  | Universally unique identifier of this problem                                                                                                            |
| **title**                | string  | The human readable title of the problem                                                                                                                  |
| **for_reporter_type**    | string  | The reporter user type.  Is one of the following values: "miare" "client". API clients are only allowed to report issues with problems of their own type |
| **description_required** | boolean | Mentions if the issues reported for this problem are required to have `description` field                                                                |

### Errors

| Code              | Description                 |
|-------------------|-----------------------------|
| not_authenticated | Token is missing or invalid |

## Report Issue

Creates an issue with given data.

> Request example:

```shell
curl --location --request POST "$BASE_URL/issue/" \
--header 'Authorization: Token <Your Token>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
  "problem_id": "728cfd38-3267-4bd0-bcec-c4fc904cebda"
}'
```

```python
import requests

data = {
    "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
    "problem_id": "728cfd38-3267-4bd0-bcec-c4fc904cebda",
    "description": None
}

requests.post(
    base_url + "/issues/",
    headers={"Authorization": "Token <Your Token>"},
    data=data,
)
```

### HTTP Request

`POST /issues/`

### Body

| Value           | Type                  | Description                                                                                                                                                                          |
|-----------------|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **trip_id**     | string **(nullable)** | The id of the trip in Miare services. could be null if course_id is provided                                                                                                         |
| **course_id**   | string **(nullable)** | The id of the course in Miare services. could be null if trip_id is provided                                                                                                         |
| **problem_id**  | string                | The id of the problem for `client` reporter type in Miare services                                                                                                                   |
| **description** | string **(nullable)** | The extra description in addition to problem, the nullability of this field depends on the `description_required` value of the problem. Max length for this field is 256 characters. |

**note:** both **trip_id** and **course_id** can **NOT** be null at the same time. if both are
provided, **trip_id** will be considered and **course_id** will be ignored.

### Response

> Response example:

```json
{
  "id": "ccba8f45-6ef6-409f-a1ee-453219aaa04f",
  "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
  "problem_id": "728cfd38-3267-miare4bd0-bcec-c4fc904cebda",
  "reported_at": "2021-11-01T18:44:19+0330",
  "reporter_type": "client",
  "resolved_at": null,
  "resolver_type": null,
  "resolve_description": null,
  "picked_at": null,
  "picker_type": null,
  "state": "reported",
  "messages": [
    {
      "id": "9ad24b31-8f1e-4c5e-b488-b715b2825792",
      "created_at": "2021-11-02T14:48:18+0330",
      "sender_type": "miare",
      "type": "text",
      "message": "سلام! پشتیبان میاره هستم. "
    }
  ]
}
```

The success response is the serialized created issue.
The details about properties of each object is as follows:

| Value                    | Type                              | Description                                                                                                                                                          |
|--------------------------|-----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **id**                   | string                            | Universally unique identifier of this issue                                                                                                                          |
| **trip_id**              | string                            | Universally unique identifier of the trip that the issue is reported for                                                                                             |
| **problem_id**           | string                            | Universally unique identifier of the problem that the issue is categorized for                                                                                       |
| **reported_at**          | string [date-time]                | The datetime that the issue is reported at                                                                                                                           |
| **reporter_type**        | string                            | The type of the reporter of the issue. Is one of the following values: "miare" "client".                                                                             |
| **resolved_at**          | string [date-time] **(nullable)** | The datetime that the issue is resolved at. It is null for unresolved issues.                                                                                        |
| **resolve_description**  | string **(nullable)**             | The description that is submitted about resolution of the issue. Max length for this field is 256 characters. It is null for unresolved issues.                      |
| **resolver_type**        | string **(nullable)**             | The type of the reporter of the issue. Is one of the following values: "miare" "client" null. The value is null for unresolved issues.                               |
| **picked_at**            | string [date-time] **(nullable)** | The datetime that the issue has been picked by a staff.                                                                                                              |
| **picker_type**          | string **(nullable)**             | The type of the picker of the issue. Is one of the following values: "miare" "client" null. The value is null for issues that are waiting for pick.                  |
| **state**                | string                            | The current state of the issue. Is one of the following values: "reported" "picked" "resolved". You can find a description about each of these states [here](#issue) |
| **messages**             | array                             | List of messages relating to the issue                                                                                                                               |
| **messages.id**          | string                            | Universally unique identifier of this message                                                                                                                        |
| **messages.type**        | string                            | The type of the message. Is one of the following values: "text" (Other message types will be supported in further versions)                                          |
| **messages.created_at**  | string [date-time]                | The datetime that the message is created in Miare system                                                                                                             |
| **messages.sender_type** | string                            | The type of the sender of the message. It is one of the following values: "miare" "client"                                                                           |
| **messages.message**     | string                            | The body of this message. Max length for this field is 256 characters.                                                                                               |

### Errors

| Code                 | Description                                                                                                                                         |
|----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| not_authenticated    | Token is missing or invalid                                                                                                                         |
| trip_not_ongoing     | The trip is not in a situation to be able to accept new issues. Currently the trips requested before 3 hours from now are considered as not ongoing |
| forbidden_problem    | The relative problem of `problem_id` is not `for_reporter_type` of client                                                                           |
| description_required | The relative problem of `problem_id` is forcing `description` field on it's issues                                                                  |
| long_description     | The description value of the request is longer than the defined limit                                                                               |

## List Issues

Returns list of issues of your trips matching the given conditions.

> Request example:

```shell
curl --location --request GET "$BASE_URL/issues/?reporter_type=miare&picker_type=client&reported_from_datetime=2021-11-02T14:48:18+0330&reported_to_datetime=2021-11-02T14:48:18+0330&resolved=1&state=resolved&picked=0&trip_id=bf125e0a-840c-4f6f-80d0-f71db7406558&trip_id=0df33447-128c-41b3-bcbd-7eb36cf5e938&offset=0&limit=10" \
--header "Authorization: Token <Your Token>"
```

```python
import requests

params = [
    ('reporter_type', 'miare'),
    ('picker_type', 'client'),
    ('reported_from_datetime', '2021-11-02T14:48:18+0330'),
    ('reported_to_datetime', '2021-11-02T14:48:18+0330'),
    ('resolved', 1),
    ('state', 'resolved'),
    ('picked', 0),
    ('trip_id', 'bf125e0a-840c-4f6f-80d0-f71db7406558'),
    ('trip_id', '0df33447-128c-41b3-bcbd-7eb36cf5e938'),
    ('offset', 0),
    ('limit', 10),
]
query = '&'.join([f'{k}={v}' for k, v in params])

requests.get(
    f"{base_url}/issues/?{query}",
    headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`GET /issues/`

### Path parameters

| Name                   | Type               | Description                                                                                        |
|------------------------|--------------------|----------------------------------------------------------------------------------------------------|
| reporter_type          | string             | **Filter:** The value of the user type that has reported the issue                                 |
| picker_type            | string             | **Filter:** The value of the user type that has picked the issue                                   |
| reported_from_datetime | string [date-time] | **Filter:** Minimum acceptable value for issue's `reported_at` field. This filter is **inclusive** |
| reported_to_datetime   | string [date-time] | **Filter:** Maximum acceptable value for issue's `reported_at` field. This filter is **inclusive** |
| resolved               | boolean            | **Filter:** The value of issues's `resolved_at` comparing to `null`                                |
| state                  | string             | **Filter:** The value of issues's `state`. Acceptable values of state are available [here](#issue) |
| picked                 | boolean            | **Filter:** The value of issues's `picked_at` comparing to `null`                                  |
| trip_id                | string             | **Filter:** The value of the trip that the issue is reported for                                   |
| offset                 | number [integer]   | Give results excluding the first **offset** number of objects                                      |
| limit                  | number [integer]   | Give *at most* **limit** number of results. The default value is 100                               |

<aside class="notice">
Server might not have or decide not to send you as many result items as <code>limit</code> but it never sends you more than that.
</aside>

### Response

> Response example:

```json
{
  "data": [
    {
      "id": "ccba8f45-6ef6-409f-a1ee-453219aaa04f",
      "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
      "problem_id": "728cfd38-3267-4bd0-bcec-c4fc904cebda",
      "reported_at": "2020-11-02T14:48:18Z",
      "reporter_type": "client",
      "resolved_at": null,
      "resolver_type": null,
      "resolve_description": null,
      "picked_at": null,
      "picker_type": null,
      "state": "reported",
      "messages": [
        {
          "id": "9ad24b31-8f1e-4c5e-b488-b715b2825792",
          "created_at": "2020-11-02T14:58:18Z",
          "sender_type": "miare",
          "type": "text",
          "message": "سلام! پشتیبان میاره هستم. "
        }
      ]
    }
  ],
  "next": "https://ws.staging.miare.ir/support/third-party-api/v2/issues/?offset=1&limit=1&reported_from_datetime=2020-11-02T14:48:18Z&reported_to_datetime=2021-12-02T14:48:18Z",
  "previous": "",
  "total_count": 35
}
```

| Value           | Type                        | Description                                                                                                                                                                                       |
|-----------------|-----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **data**        | array                       | List of issues matching the given query. Each object in this array is a serialized issue. For a detailed version of it take a look at the response body of [Report Issue](#report-issue) request. |
| **next**        | string [uri] **(nullable)** | The valid url to the next set of result with the same predicates. Will be **null** if there are no more objects left matching the given filters                                                   |
| **previous**    | string [uri] **(nullable)** | The valid url to the previous set of result with the same predicates. Will be **null** if there was no objects matching the given filters before this page                                        |
| **total_count** | number [integer]            | The **total** number of objects matching the given filters (not only the items in this page)                                                                                                      |

### Errors

| Code              | Description                 |
|-------------------|-----------------------------|
| not_authenticated | Token is missing or invalid |

## Resolve Issue

Resolves an unresolved issue.

> Request example:

```shell
curl --location --request POST "$BASE_URL/issue/{issue_id}/resolve/" \
--header 'Authorization: Token <Your Token>' \
--header 'Content-Type: application/json'
```

```python
import requests

data = {
    "resolve_description": "سفارش تحویل کوریر داده شد. "
}

requests.post(
    base_url + "/issues/{issue_id}/resolve/",
    headers={"Authorization": "Token <Your Token>"},
    data=data,
)
```

### HTTP Request

`PATCH /issues/{issue_id}/resolve/`

### Path parameters

| Name     | Type   | Description                    |
|----------|--------|--------------------------------|
| issue_id | string | The ID of the issue to resolve |

### Body

| Value                   | Type   | Description                                                                                    |
|-------------------------|--------|------------------------------------------------------------------------------------------------|
| **resolve_description** | string | The description required when resolving an issue. Max length for this field is 256 characters. |

### Response

> Response example:

```json
{
  "id": "ccba8f45-6ef6-409f-a1ee-453219aaa04f",
  "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
  "problem_id": "728cfd38-3267-4bd0-bcec-c4fc904cebda",
  "reported_at": "2021-11-01T18:44:19+0330",
  "reporter_type": "client",
  "resolved_at": null,
  "resolver_type": null,
  "resolve_description": null,
  "picked_at": null,
  "picker_type": null,
  "state": "reported",
  "messages": []
}
```

The success response is the serialized updated issue. For a detailed version of it take a look at
the response body
of [Report Issue](#report-issue) request.

### Errors

| Code                     | Description                                                                   |
|--------------------------|-------------------------------------------------------------------------------|
| not_authenticated        | Token is missing or invalid                                                   |
| resolved_issue           | The issue is already resolved                                                 |
| long_resolve_description | Teh resolve description value in the request is longer than the defined limit |
| not_picked               | The issue is reported by `miare` but not picked yet                           |

## Pick Issue

Picks an issue that means the issue is being checked by the client.

> Request example:

```shell
curl --location --request POST "$BASE_URL/issue/{issue_id}/pick/" \
--header 'Authorization: Token <Your Token>' \
--header 'Content-Type: application/json'
```

```python
import requests

requests.post(
    base_url + "/issues/{issue_id}/pick/",
    headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`PATCH /issues/{issue_id}/pick/`

### Path parameters

| Name     | Type   | Description                 |
|----------|--------|-----------------------------|
| issue_id | string | The ID of the issue to pick |

### Response

> Response example:

```json
{
  "id": "ccba8f45-6ef6-409f-a1ee-453219aaa04f",
  "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
  "problem_id": "728cfd38-3267-4bd0-bcec-c4fc904cebda",
  "reported_at": "2021-11-01T18:44:19+0330",
  "reporter_type": "client",
  "resolved_at": null,
  "resolver_type": null,
  "resolve_description": null,
  "picked_at": null,
  "picker_type": null,
  "state": "reported",
  "messages": []
}
```

The success response is the serialized updated issue. For a detailed version of it take a look at
the response body
of [Report Issue](#report-issue) request.

### Errors

| Code              | Description                   |
|-------------------|-------------------------------|
| not_authenticated | Token is missing or invalid   |
| resolved_issue    | The issue is already resolved |
| picked_issue      | The issue is already picked   |

## Append Message

Appends a new message to an unresolved issue.

<aside class="notice">
Currently only messages with text type are supported.
</aside>
> Request example:

```shell
curl --location --request PATCH "$BASE_URL/issue/{issue_id}/messages/" \
--header 'Authorization: Token <Your Token>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "message": "با مشتری نهایی تماس گرفته شد و گفتن که به لابی تحویل بدین"
}'
```

```python
import requests

data = {
    "message": "با مشتری نهایی تماس گرفته شد و گفتن که به لابی تحویل بدین"
}

requests.patch(
    base_url + "/issues/{issue_id}/messages/",
    headers={"Authorization": "Token <Your Token>"},
    data=data,
)
```

### HTTP Request

`PATCH /issues/{issue_id}/messages/`

### Path parameters

| Name     | Type   | Description                                  |
|----------|--------|----------------------------------------------|
| issue_id | string | The ID of the issue to append the message to |

### Body

| Value       | Type   | Description                                                                     |
|-------------|--------|---------------------------------------------------------------------------------|
| **message** | string | The body of the message to be sent.Max length for this field is 256 characters. |

### Response

> Response example:

```json
{
  "id": "ccba8f45-6ef6-409f-a1ee-453219aaa04f",
  "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
  "problem_id": "728cfd38-3267-4bd0-bcec-c4fc904cebda",
  "reported_at": "2021-11-01T18:44:19+0330",
  "reporter_type": "client",
  "resolved_at": null,
  "resolver_type": null,
  "resolve_description": null,
  "picked_at": null,
  "picker_type": null,
  "state": "reported",
  "messages": [
    {
      "id": "9c212393-8ac3-4917-ac7a-844f76c361f2",
      "created_at": "2021-11-01T18:46:19+0330",
      "type": "text",
      "message": "با مشتری نهایی تماس گرفته شد و گفتن که به لابی تحویل بدین",
      "sender_type": "client"
    }
  ]
}
```

The success response is the serialized created issue. For a detailed version of it take a look at
the response body
of [Report Issue](#report-issue) request.

### Errors

| Code              | Description                                                       |
|-------------------|-------------------------------------------------------------------|
| not_authenticated | Token is missing or invalid                                       |
| resolved_issue    | The issue is already resolved                                     |
| long_message      | The message value of the request is longer than the defined limit |

# Areas

> Base URL of staging servers for area services:

```shell
BASE_URL="https://ws.staging.miare.ir/area/third-party-api/v2"
```

```python
base_url = "https://ws.staging.miare.ir/area/third-party-api/v2"
```

> Base URL of production servers for delivery services:

```shell
BASE_URL="https://ws.miare.ir/area/third-party-api/v2"
```

```python
base_url = "https://ws.miare.ir/area/third-party-api/v2"
```

Services related to getting information about working areas.

## List Areas

Returns list of Miare areas and your usage of concurrency in each area.

> Request example:

```shell
curl --location --request GET "$BASE_URL/areas" \
--header "Authorization: Token <Your Token>"
```

```python
import requests

requests.get(
    f"{base_url}/areas/",
    headers={"Authorization": "Token <Your Token>"},
)
```

### HTTP Request

`GET /areas/`

### Response

> Response example:

```json
[
  {
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
  }
]
```

The response is an array of all of the active areas in the Miare whether you have any trips or
concurrency in it or not.
The details about properties of each objects is as follows:

| Value                   | Type             | Description                                                           |
|-------------------------|------------------|-----------------------------------------------------------------------|
| **id**                  | number [integer] | The constant identifier of the area                                   |
| **name**                | string           | The human readable name of the area                                   |
| **ongoing_trips**       | number [integer] | The number of trips currently [**active**](#trip) trips in this area. |
| **max_ongoing_trips**   | number [integer] | Your [**concurrency**](#concurrency) limit in this area at this shift |
| **polygon**             | object           | A valid [Polygon](https://geojson.org/geojson-spec.html#id4) object   |
| polygon.**type**        | string           | GeoJson feature type. Its value is always "Polygon"                   |
| polygon.**coordinates** | array            | Coordinates of the points forming the exterior ring of the polygon.   |

<aside class="notice">
Based on <a href="https://geojson.org/geojson-spec.html#positions">GeoJson.org</a>'s convention, the order of coordinates in the coordinates inner arrays is <b>longitude</b>, <b>latitude</b>.
</aside>

### Errors

| Code              | Description                 |
|-------------------|-----------------------------|
| not_authenticated | Token is missing or invalid |

# Accounting

> Base URL of staging servers for accounting services:

```shell
BASE_URL="https://www.staging.miare.ir/api/accounting"
```

```python
base_url = "https://www.staging.miare.ir/api/accounting"
```

> Base URL of production servers for accounting services:

```shell
BASE_URL="https://www.miare.ir/api/accounting"
```

```python
base_url = "https://www.miare.ir/api/accounting"
```

Accounting related services.

## Estimate Price

> Request example:

```shell
curl --location --request GET "$BASE_URL/estimate/price/?source=35.764090,51.331956&destination=35.767832,51.335135" \
--header "Authorization: Token <Your Token>"
```

```python
import requests

params = {
    'source': '35.764090,51.331956',
    'destination': '35.767832,51.335135',
}
query = '&'.join([f'{k}={v}' for k, v in params.items()])

requests.get(
    f"{base_url}/estimate/price/?{query}",
    headers={"Authorization": "Token <Your Token>"},
)
```

Estimates the delivery cost of the course based on the distance between the given source and
destination.

<aside class="notice">
The output price is not accurate; it might be affected by the applied conditions at the request time.
</aside>

<aside class="warning">
The output price is only the <b>course</b>'s cost. The final cost includes a <b>trip</b> cost as well. For more information contant the sales team.
</aside>

### HTTP Request

`GET /estimate/price/`

### Path parameters

| Name        | Type   | Description                                                                    |
|-------------|--------|--------------------------------------------------------------------------------|
| source      | string | Latitude and Longitude of the source of the course. The **order** matters      |
| destination | string | Latitude and Longitude of the destination of the course. The **order** matters |

### Response

> Response example:

```json
{
  "price": 5000,
  "status": "ok",
  "area_coverage": true
}
```

| Value             | Type             | Description                                                    |
|-------------------|------------------|----------------------------------------------------------------|
| **price**         | number [integer] | The estimated cost of the course's delivery                    |
| **status**        | string           | The human readable status of the request                       |
| **area_coverage** | boolean          | Check if the source location is within the Miare areas or not. |

### Errors

| Code              | Description                          |
|-------------------|--------------------------------------|
| not_authenticated | Token is missing or invalid          |
| invalid           | The given query params are not valid |

# Webhook

Other than services above which are exposed endpoints by Miare servers, we support a callback
system to push events to
your severs as they occur.

In order to use this service, you need to provide us with a valid URL, pointing to your servers,
waiting for HTTP
requests.

## Authentication

> Authentication header:

```
authorization: Token 8357c002a28512f778cab11a425ff91a7d3483d1
```

All requests made from our servers to yours, carry the `authorization` header. The value of this
header
is [your token](#authentication) prefixed with a `Token ` string.

<aside class="warning">
You should discard any requests made to your webhook servers without a valid <code>authorization</code> header.
</aside>

## Versioning

> Version header:

```
x-miare-api-version: 2
```

At this moment, we only support the latest version of API (v2). But there is a version included in
all of requests made
by our servers with the key `x-miare-api-version` and value of `2`.

## Retry

We ignore the response body of the request made to your servers but expect a `2XX` response code.
In case of a `5XX`
error from your servers we'll retry the request up to 5 times. There will be a 5 seconds cooldown
between each pair of
retries.

<aside class="notice">
You should not rely solely on our webhook requests. In case of a network failures or a long service outage you should be able to recover to the latest state using a <a href="#list-trips">List Trips</a> request.
</aside>

## Trip Events

### Request Body

> Request example:

```json
{
  "event": "added",
  "trip": {
    "area": {
      "id": "3",
      "name": "یوسف آباد"
    },
    "assigned_at": null,
    "arrived_at": null,
    "departed_at": null,
    "courier": null,
    "courses": [
      {
        "address": "تهران، خیابان استاد معین، پلاک ۱۲",
        "bill_number": "DEL-119",
        "dropped_off_at": null,
        "id": "0f662083-284e-4729-b34d-2479bd7b558a",
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
        "tracking_url": "https://www.staging.miare.ir/p/trip_watching/#!/0f66208328",
        "trip_id": "4f3d252a-ebbc-4147-b6e8-aba9b64200f3",
        "old_trip_id": "00000000-0000-0000-0000-000000000000"
      }
    ],
    "created_at": "2021-11-10T17:35:14+0330",
    "id": "4f3d252a-ebbc-4147-b6e8-aba9b64200f3",
    "picked_up_at": null,
    "pickup": {
      "address": "تهران، صادقیه، بلوار آیت الله کاشانی",
      "deadline": "2021-11-10T21:06:00+0330",
      "image": "https://example.com/restaurants/bm_logo.png",
      "location": {
        "latitude": 35.737004,
        "longitude": 51.413569
      },
      "name": "رستوران بزرگمهر",
      "phone_number": "09123456789"
    },
    "state": "assign_queue",
    "is_round_trip": false
  }
}
```

| Value     | Type   | Description                                                                                                                                         |
|-----------|--------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| **event** | string | The constant identifier of the event                                                                                                                |
| **trip**  | object | The affected trip. This is a serialized trip. For a detailed version of it take a look at the response body of [Create Trip](#create-trip) request. |

### Events

There are 10 trip events each of which make a request to your server with one the following strings
as the event and a
serialized trip.

| Event                  | Description                                                                            |
|------------------------|----------------------------------------------------------------------------------------|
| **added**              | Trip has been successfully created                                                     |
| **assigned**           | Trip has been assigned to a courier                                                    |
| **course_added**       | A new course has been added to the trip                                                |
| **course_deleted**     | A course has been deleted from the trip                                                |
| **picked_up**          | Courier picked up the packages from the source of the trip                             |
| **course_dropped_off** | Courier dropped off a package at one of the destinations of one of courses of the trip |
| **delivered**          | Trip has been ended successfully                                                       |
| **canceled_by_client** | Trip has been cancelled by client (you)                                                |
| **canceled_by_miare**  | Trip has been cancelled by Miare (our support team)                                    |
| **returning**          | Trip state changed to returning                                                        |
| **canceled_by_delay**  | Trip has been cancelled by Miare due to fulfillment delay from your side               |

## Course Events

### Request Body

> Request example:

```json
{
  "event": "course_added__course",
  "course": {
    "address": "تهران، خیابان استاد معین، پلاک ۱۲",
    "bill_number": "DEL-119",
    "dropped_off_at": null,
    "id": "0f662083-284e-4729-b34d-2479bd7b558a",
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
    "tracking_url": "https://www.staging.miare.ir/p/trip_watching/#!/0f66208328",
    "trip_id": "4f3d252a-ebbc-4147-b6e8-aba9b64200f3",
    "old_trip_id": "00000000-0000-0000-0000-000000000000"
  }
}
```

| Value      | Type   | Description                                                                                                                                             |
|------------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **event**  | string | The constant identifier of the event                                                                                                                    |
| **course** | object | The affected course. This is a serialized course. For a detailed version of it take a look at the response body of [Create Trip](#create-trip) request. |

### Events

There is one course event that makes a request to your server with one the following string as the
event and a
serialized course.

| Event                    | Description                     |
|--------------------------|---------------------------------|
| **course_added__course** | Course has been added to a trip |

## Issue Events

### Request Body

> Request example:

```json
{
  "event": "issue_added",
  "issue": {
    "id": "ccba8f45-6ef6-409f-a1ee-453219aaa04f",
    "trip_id": "b3951922-4f3e-43dc-a051-a9b765b2cbe7",
    "problem_id": "728cfd38-3267-miare4bd0-bcec-c4fc904cebda",
    "reported_at": "2021-11-01T18:44:19+0330",
    "reporter_type": "client",
    "resolved_at": null,
    "resolver_type": null,
    "resolve_description": null,
    "picked_at": null,
    "picker_type": null,
    "state": "reported",
    "messages": [
      {
        "id": "9ad24b31-8f1e-4c5e-b488-b715b2825792",
        "created_at": "2021-11-02T14:48:18+0330",
        "sender_type": "miare",
        "type": "text",
        "message": "سلام! پشتیبان میاره هستم"
      }
    ]
  }
}
```

| Value     | Type   | Description                                                                                                                                             |
|-----------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **event** | string | The constant identifier of the event.                                                                                                                   |
| **issue** | object | The affected issue. This is a serialized issue. For a detailed version of it take a look at the response body of [Report Issue](#report-issue) request. |

### Events

There are some issue events each of which make a request to your server with one the following
strings as the event and
a serialized issue.

| Event                   | Description                               |
|-------------------------|-------------------------------------------|
| **issue_added**         | Issue has been created                    |
| **issue_picked**        | Issue has been picked                     |
| **issue_message_added** | A new message has been added to the issue |
| **issue_resolved**      | The issue has been resolved               |
