---
tags: [delivery]
---

# States of Trips

Each trip at any time has exactly one of these states. You can think of them like phases of fulfulling a trip, in a sense that a trip advances in these states to be completed.

These states are:  

 - **`assign_queue`:** The initial state of a trip is `assign_queue`. It means that your request for a trip has been processed successfully and it is currently placed in the assign queue, awaiting for a suitable courier to be assigned to it.
 - **`pickup`:** The courier has been dispatched to the source location, and it has yet to pickup the package. A trip that has reached this state may return to the previous state if the courier gets changed for any reason.
 - **`dropoff`:** The courier has picked up the package, and is currently going to dropoff the package or packages. A trip that has reached this state will not return to any previous state.
 - **`delivered`:** All of the packages in the trip has been delivered and the trip is fulfilled.
 - **`canceled_by_miare`:** This trip has been canceled by Miare's staff, either by client's request, after concluding that this trip violates Miare's terms and conditions, or reaching the conclusion that Miare is unable to fulfill this trip. When a trip reaches this state it means that it was not and will not be fulfilled.
 - **`canceled_by_client`:** This trip has been canceled by client. When a trip reaches this state it means that it was not and will not be fulfilled.