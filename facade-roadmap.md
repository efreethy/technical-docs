# Facade Roadmap

* * *
**Goals:**

* Understand what the facade looks like right now
* Identify the most immediate areas that are lacking support, work that is unfinished
* Identify inconsistencies in the facade, and any areas of repeated friction
* Suggest possible courses of action to resolve those challenges


* * *

# Current state of affairs



## Listing flow

Obtaining **browsing options**, **add ons**, **delivery methods, restrictions** (PD restrictions), **purchase info** (route rules)

**Mobile Facade**: `mobile/purchase/event_purchase_config`
**Web**: `browse/listing`




## **Browse flow**

**Mobile Facade**:
Obtaining **pricing options**: `mobile/purchase/event_seating_config` 
Obtaining **seats **(purchase choices): `mobile/purchase/purchase_choice`
Obtaining latest **seat index** (seat availability): `mobile/purchase/seat_index`
**Web:**
`browse/seat_filter_config`
`browse/seating` (replaced by `purchase/purchase_choice`)
`browse/seat_index`
`browse/pricing_directive` 




## **Checkout flow**

**Creating a cart, creating a best available cart, adding add ons to a cart, clear cart, change mode, expire, update delivery info, add seats, remove seats, update a variant on a seat**:

* `mobile/purchase/cart`

Making a **payment** on a cart: `me/payment`

**Cart keep alive**: `browse/cart_keep_alive`


### Web implementations:

* `browse/cart` 
* `browse/seating`
* `browse/extras`
* `browse/delivery_method` 
* `browse/cart_keep_alive`


* * *

# Observations:

* No mobile endpoints for **pricing_directive, cart_keep_alive**
* Lots of backlogged facade work
* Inconsistency in web and mobile on data format. Web is using JSON api, and mobile clients are split between JSON api (me service) and non-JSON api (browse service)
* We have a useful mechanism to prefix non-mobile routes with `/consumer` but this isn’t  being leveraged. We are still going through the `/browse` prefix 
* There isn’t a lot of usage of  `rival-api-view` on new resources. This library lets us generate Open API / Swagger docs, which could reduce a lot of backend developer overhead and client developer frustration.
* Web is tightly coupled to complicated API contracts that were eliminated by the browse client facade (listings, product distribution, add on distributions, etc). This would make it a lot harder to move these implementations over to simplified API contracts, should we ever re-architect them.


* * *

# Proposals:

### Small effort, short-term value:

* Backlogged facade work:** **
    * **~1 point **[RIV-8360](https://rivalco.atlassian.net/browse/RIV-8360?atlOrigin=eyJpIjoiNTY0NTlhODIxZDcyNGUyYjgzMjdlN2NhZDFiNDI2MGEiLCJwIjoiamlyYS1zbGFjay1pbnQifQ) Add price_code_id to the seat entry in mobile/purchase/cart
    * **~1 point** [RIV-5866](https://rivalco.atlassian.net/browse/RIV-5866) Return client consumable error when trying to cart sold out add-ons
    * **~3 point** [RIV-6041](https://rivalco.atlassian.net/browse/RIV-6041) Need to create me/device_attestation facade endpoint. This allows [validating mobile clients](https://10e.quip.com/TCqaAaYauIui) aren’t bots.
    * **~1 point** [RIV-7738](https://rivalco.atlassian.net/browse/RIV-7738) Add ons returned in the event purchase config need concatenated ids
* **~1 point **Extend mobile implementations for **pricing_directive**, **cart_keep_alive**, so clients aren’t forced to go through `/browse`



### Small effort, *long-term* value:

* **~1 pt** For any resource defs already defined with `rival-api-view` (which is very small), auto-gen docs and get these docs in swagger

* **~0 pts **Start leveraging `rival-api-view` on every new resource definition. This gives us comprehensive API documentation with minimal manual effort.
* **~3-5 points (spike?) **Reorganize our facade-development process. Get client developers and facade developers in the same room (or same slack channel) before publishing facade API contracts. This is mainly to encourage consensus and clarity on the facade API contract before its published.



### Medium to High effort, long-term value:

**~13 points** Migrate all facade resource definitions to use `rival-api-view` . 

* Enables generating API docs for our existing resource definitions.
* This should be a great help to mobile/web developers that are getting slowed down by absent API docs, or API docs that have fallen out of sync, and require update by hand.
* We have about 35 resources across browse/me service, so it would be a fair amount of work to migrate the resource definitions. Fortunately, migrating a resource is relatively fast and simple, amounting to re-defining the resource definition using `create_typed_view`, and porting the legacy marshmallow request / response types as `attrs types` to be consumed by the view. Any proxied resource definitions would need to issue the proxy request in the resource definition using `ApiQueryBuilder`
* 

### Medium to High effort, long-term value:

Stop feature development on all existing `me`, `browse` and `mobile` endpoints and wrap all of their existing functionality in `consumer_client` endpoints.


* Allows us to continue using the existing endpoints while new ones are created
* We can be more deliberate about the purpose of a particular endpoint, having given the proper consideration to how it will serve mobile and/or web.
* We can document the reasoning behind the endpoint, and the contract between it and the clients, before starting work. (As an example [Consumer Client Cart API](https://10e.quip.com/SgMLAtE1vxs7))
* New endpoint may be 1-1 with an existing endpoint (e.g. `/seat_index`) or wrap multiple calls to existing endpoints. 
* When clients have fully moved over to the new `consumer_client` endpoints we can refactor/replace the existing `me`, `browse` and `mobile` endpoints as desired without affecting clients.



### Very high effort, long-term value:

**>20 points** Rearchitect all legacy facade API contracts. Simplify these contracts to mimic the patterns introduced in event purchase config, event seating config (simplified domain model, non JSON api, no relationship properties)

* This would give us the highest amount of simplicity and consistency across our facade
* This would be an incredibly large amount of work, with roughly ~35 resource API contracts to redefine.
*  This would break our clients and require a considerable refactor


Most recent discussions around the facade have set us along this path. We have an epic to capture all of it.  (Cart RPC API: https://rivalco.atlassian.net/browse/RIV-8441) This is about 34 points of work.
* * *

## **Emerging Work**


Its worth noting that the existing consumer cart API is undergoing a lot of re-thinking. There is a [proposal from Chris](https://10e.quip.com/vX82Abnyoa9g/Proposed-Changes-to-Mobile-Cart-Endpoint) to make some significant refactors to the endpoint, which tie in very closely with our original intentions behind a generic my cart endpoint. I created a [ticket to capture](https://rivalco.atlassian.net/browse/RIV-8366) that work, and conversations are on going as to what we want the Cart API to support, and how this fits into our road map.
