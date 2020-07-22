# Understanding customer profiles


Last edited: 17 July 2018
Status: Draft

## **Goals**

Clarify what a customer profile is, and describe the various contexts in which it is created.

## Philosophical Aside

customer profiles are an important feature to get right. "Knowing all of your customers" is technically equivalent to establishing accounts for every person who attends your events, and linking back to those accounts on all subsequent visits. To support use cases like anonymous product purchases at the window, we will need to make concessions on this value proposition. Frustrating or inflexible registration processes can be a huge bottleneck to growing our user base. No matter how beautiful or performant our app is, if our users are too bothered by our sign up flow, we lose them right there, possibly to another vendor.

## **Intro**

At a high level, the customer profile is an organization's view of a customer. It also serves as a parent through which entitlements are assigned. The customer profile should provide isolation between orgs, in that an organization is restricted from viewing the customer profiles and the entitlements of other orgs. Looking at things from the customer's perspective, an event goer will accumulate customer profiles as they transact with organizations. By extension, a user's transaction history spans their customer profiles.

## Contexts

There are 4 contexts in which a customer profile is created.

1. White labeled clients
    1. The customer profile will be created during the user registration process. We will brand [company] as the identity provider of these apps - so users will register by creating their [company] user account. We will associate the customer profile as the child of the [company] user, and scope it to the org of the white label app. It will serve as the container for the users organization-specific standard attributes. For example, a user may have an org specific profile image, or first name, that is editable in this client. Updates to this information does not affect their [company] user, despite the [company] user being their current auth context.
2. General consumer clients
    1. The customer profile will be created dynamically, once the user has transacted with an organization. In these clients, the [company] user's existence is a pre-requisite before a transaction can be made, so we can associate it to the authenticated [company] user, as well as the organization hosting the event. This is a one time process. Any subsequent purchases with the organization will be tied to this profile.
3. Point of sale (Enterprise client) 
    1. In this case we create the customer profile at the window. We will take in some amount of details from the user in order to generate a customer profile. For example, we might check their ID, take a first and last name and append that to a customer profile. Ideally the org will collect identifiers that could ultimately result in a [company] user (email, phone number). We will need to think through how strongly we suggest that they do this, as this introduces friction, but is vital to knowing their customers. If we end up with any identifiers that can be used to activate a [company] user, we may want to create an unregistered [company] user at the moment, which will be dormant until the event goer signs up through our general client. In the case we have an anonymous cash purchase, the customer profile will be orphaned, however it will still serve as the parent to any entitlements purchased. In the case of an anonymous carded purchase, we may end up with identifiers that can result in a [company] account.
4. Third party integrations? 
    1. In the future we will likely integrate with sites like StubHub. These sites will get a view of our available inventory through our api, and sell products to users who are logged into their site. We will desire there be a vector for the attendee to create [company] user - ideally as an option through the notification they receive to claim their products. The customer profile itself will be created on our backend.


