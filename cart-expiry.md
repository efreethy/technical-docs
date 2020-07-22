# Cart Expiry Rules 

12/20/2020


* The Consumer User’s cart expiry timer and cart idleness timer should start when they first attempt to place inventory into the cart
* After the Consumer User’s cart exists (linked to their wait token) and they’ve attempted to cart entitlements we should preserve this cart and wait token combo until the User either successfully purchases inventory, or the expiry timer triggers invalidation of the cart and wait token pairing. We should not otherwise drop the cart without explicit intent from the Consumer User (eg. exiting the buyflow for the event via back button), or hold sales contention / EC operations against the user (eg. if carting or purchase fails on single seat restriction, or an EC User revoking entitlements)
* The Consumer User should not be able to reset the clock on the Cart Expiry timer once it has started. If it does expire, this should remove their cart, consumer their wait token, and so behavior should be to route the Consumer User back to wait / PD selection
* The Consumer User should be able to reset the Cart idleness timer an arbitrary amount of times by taking action within the Consumer app including adding / removing cart items, interacting with MoP and MoD details, and Account details such as sign-up, sign in.

### **Client approach to idle expiry**

**Web**
Send cart keep alive requests on keydown and mousemove interactions. Throttle these calls with a throttling period equal to 1/4 of the total cart idle expiry length. 

Example: If the idle expiry length is 4 minutes, we will allow at most one keep alive request every 1/4 interval of that period (one minute). If the user doesn’t perform an interaction in that interval - no keep alive request is sent in that interval.
