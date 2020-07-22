# **[DEPRECATED] Simplifying Enterprise User Registration**


Last edited: 7 August 2018
Status: Draft

## **Summary**

Make an argument that enterprise user registration is too complicated in its current state, and describe a couple approaches to simplify it.

## **Problem description**

Enterprise users belonging to N orgs must complete N registration processes. 

From a user experience perspective this can be bothersome for a couple reasons:

* A user must to respond to N registration emails, and configure N passwords. This has the largest effect on users who interface across many orgs. 
* Consider a sports league with N teams. 
    * A given team will compete against the other (N-1) teams. 
    * Before game day, an employee in team 1 may need to log into the EC of team 2 in order to manage reserved entitlements.
    * As the season progresses, the employee in team 1 might repeat this process before game day in the EC of team 3, team 4, up to team N-1. 
    * By the end of the season employee in team 1 would have set up (N-1) additional accounts, each with a custom password.
    * If each team has a designated employee with a similar role, we have a total of N*(N-1) additional accounts by season end.
* A user must understand they have N representations in the enterprise client, even if the email is exactly the same, which can lead to confusion.

There is additional confusion for enterprise users who are members of multiple subsidiary orgs. For example, a member of [customer] [team], [customer] [team], [customer] Arsenal, and Dallas Cowboys would need to maintain a username / password for each sub-org, in addition to any external orgs. The continual process of establishing unique credentials for subsidiary orgs could be abrasive.

## **Solution**

* Brand [company] as the singular identity provider for the enterprise client. From both the individual and organization perspective, [company] is a candidate identity provider.
* Establish a single “enterprise user account” for enterprise users who share identity—i.e., they are the same person. 
* Communicate to enterprise users that password registration is a one time process. Their credentials can be used to authenticate into the enterprise client for a specific organization, provided the organization has added them as a user from their enterprise client. 

## Approach 1

Collapse organization specific user pools into a single pool. 

**BENEFITS**

* A single pool can be configured in terraform, which reduces complexity in our service code, and simplifies areas like replication and resource provisioning.
* User management operations are trivial, since they only ever apply to a single pool.

### Challenges

A single user pool is limited to 25 identity providers.

* 25 identity providers should be more than enough to support a live launch with [customer], but as we on board new organizations we will eventually need to request an increase. It is still unknown how likely it is that AWS would reject this request.
* If AWS permanently restricts us from increasing this limit, we would need to develop a strategy for partitioning a single logical enterprise user pool into multiple partitioned user pools. We would add more identity providers to the partitions. This could get hairy, and might pigeon hole us into going with approach no. 2.

### Implications on THE DB

* The **user_pool** schema will lose the **organization_id** column
* **(organization** → **user_pool) **relation will go from (one → many) to (one → one)

### Implications on Registration

*  An enterprise user’s initial invitation will register them into this pool.
* Subsequent invitations from other organizations will skip adding them to the pool.
* Federated users don’t configure a username / password, and will still enter the pool in an unregistered state. They will remain this way until an organization marks them as managed. When this happens, they will be prompted to register a username / password with the [company] platform.

### Implications on Log In

* User service will still require organization context at log in. This is delivered via a hint, or a locally cached identifier from a previous successful login.
* User service will no longer need the organization context to identify a user pool, as there is only one. It will however need the organization context in order to permission the user.
* In both the managed and federated cases, on authentication success user service will pass back the (token + permission set + org scope) of the requesting user. All api requests made with this token will be filtered by our RBAC layer using the corresponding permission set.




## Approach 2


Maintain one user pool per organization, but abstract this reality from the enterprise user.

### Benefits

* Each pool can hold up to 25 identity providers, which should be more than enough on a per organization basis
* Migrating a large organization containing subsidiaries would be less of a headache, since each sub-org would get a pool, which has plenty of room for identity providers

### Challenges

User management operations become non-trivial.

* Operations like change password, forgot password would need to propagate across all the organization pools a user is a member of.

Spawning a pool per organization would preserve the existing complexity in our service code.

* Replication becomes more challenging since we cannot just target a Terraform module. We would need to replicate resources ad-hoc.

### Implications on THE DB

* No schema changes! Our existing setup is modeled with one enterprise user pool per organization

### Implications on Registration

*  An enterprise user’s initial invitation will register them into an organization specific pool. This can be considered the users’ primary pool.
* On subsequent invitations, the user is prompted to provide their original set of credentials. These credentials will be used against the primary pool, and on success our system will silently register them into the secondary pool using the same credentials.
* Federated users don’t configure a username / password, and will still enter their organization specific pool in an unregistered state. They will remain this way until an organization marks them as managed. When this happens, they will be prompted to register a username / password with the [company] platform. 

### Implications on Log In

* Not much would change here. The only real difference is that the user is using the same password across all of their organization user pools.

