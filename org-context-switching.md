## **[DEPRECATED] Organization Context Switching - Front end overview**


Last edited: 11 August 2018
Status: Draft

### SUMMARY

Outline the journey an enterprise user takes as they switch between parent orgs.
Clarify objectives. 
Provide an overview of approach on the front end to satisfy objectives.

### TASK AT HAND

Give enterprise users the ability to change their organization context. 

### RECAP - A High level example:

* Consider an enterprise user who belongs to Kroenke Rams, Kroenke Avalanche, and Kroenke Arsenal
* Suppose this user authenticates into the EC of Kroenke Rams. 
* This user should have access to a tool that allows them to see all other organizations they are a member of. 
* They should have the ability to select an org from this list, and authenticate into the EC of that org.
* Before they are granted access, they must complete the authentication flow that was provisioned for them by the selected org.
* If the user has already authenticated with an org, they should be able to return to that context without re-auth, provided that context has not expired.
* When the user explicitly logs out, all session information for all orgs is destroyed. The user must now repeat the process of authenticating with their parent orgs.

### Objectives

**Security**

* ***Always authenticate first***. On first log in to a particular org, ensure the user is required to authenticate under the authentication flow specified by that org.
* Clean up on log out by ***destroying all session information*** across all orgs.
* Ensure that locally cached entities from previous contexts ***do not pollute the local cache*** of future contexts.

**Ease of use**

*  If the user hasn’t explicitly logged out, ***persist their session information***. Do not require re-auth if this information is available.
* Before log out, ***persist authentication details****,** *so this information does not have to be re-fetched at the login screen. This will make re-auth a one-step process.

**Best practice**

* Our front end architecture is rapidly evolving. Javascript components are being migrated to Typescript. An approach is being developed that will deprecate Redux, including the use of sagas. Our core data flow paradigm is being lifted out of react-redux, and into a set of loosely couple libraries which communicate changes asynchronously with components that register interest.
* Try to make use of the above ideas. Migrate new files to Typescript. Iterate on existing shared libraries in order to abstract away persistent session context. 

### recap - Authentication Flows

Organizations specify the authentication flow a user must complete before gaining access to their EC. 

A user can be either:

**Managed**: This is traditional username / password registration.

**Federated**: The user will be redirected externally, out of the EC, and into a hosted log in page. This page is configured by the organization. On authentication success, the user receives a redirect back to EC with a code. The code is verified by our system, and if valid, the user will be granted a session for that org in the EC.


### Overview of APPROACH - front end implementation

**Application bootstrap**

On application bootstrap we will prefetch the authentication options available to a user. This can happen in our StartupService. These options will live in local storage alongside other entities.

**Authentication Details**

Each option holds the authentication details needed to initiate the auth flow for a particular org. 

Here's the shape of those details.

```
AuthDetails {
  isManaged: true | false, 
  organizationId: string, 
  
  providers?: {
    providerName: string,
    clientId: string
    authenticationUrl: string
    redirectUri: string
  }[]
}
```


The options will be rendered to the user in a modal.

When an option is selected, one of two paths is taken.

*  Managed users are shown simple username / password input. They will use this to authenticate with user service.

*  Federated users will receive a redirect. The parameters needed to initiate this redirect are available on the **providers** attribute of the authentication details of their selection. 


**SDK AuthenticationProvider**

In both cases we will finish the auth flow through the AuthenticationProvider of our SDK. These operations will occur at the saga layer. In the managed case we will re-use our existing `authenticateUser()` function, and in the federated case we will re-use our existing `authenticateFederatedUser()` function.

The SDK AuthenticationProvider will be refactored in order to capture the notion of persistent session information. 

*  It will bundle this information into a utility class named UserContext (previously named AuthenticationToken) 
* The UserContext will hold four things: (user, authentication token, permissions, scope)
* Helpers will be added for swapping out user contexts, existence checking, and retrieval of session information. 


**Caching contexts**

On authentication success: 

* AuthenticationProvider will hold onto the UserContext of that session. Api consumers must confer with this context before api requests are made.
* We must preserve pre-existing contexts, and destroy pre-existing entity state. These will be an additional operations at the saga layer. This will *only* occur for requests when a context switch is detected.

Due to re-use of authentication methods in our sagas, we will need a way of knowing if the call is happening within the context of an organization switch. 

*  In the managed case:
    * This can be specified as an additional property on an action. 
*  In the federated (redirect) case:
    * We will lose any information that isn’t persisted between page reload. 
    * In order to preserve the knowledge that an organization context switch in progress, we will create a new “mode” in redux state named “appSwitchOrganizationMode”. This is a flag that will be set to true before a redirect occurs. When a user is redirected back to our app, we check this flag, and if true, we will invalidate previous entity state, and cache the previous context.

**Replacing Contexts**

If the user is returning to an organization for which we have a cached context, we will avoid sending them through the authentication method of our saga, and instead issue an action that will restore the cached context. This will also happen at at saga layer, and will be issued through the AuthenticationProvider replaceUserContext() method. Just like in the previous example, the previous context will remain cached, and the previous local entity state will be invalidated.

**Re-initiating application bootstrap**

In all cases, the final step is to repeat the bootstrapApp process under the new context. 

* All initialization procedures will repeat under this context. 
* Any data fetch procedures will be made against the organization of the context, using the authentication token of the context.

* Redux connect will force a re-render on all components that are both connected and mounted.

* The user will see a refreshed view of entity data corresponding to the new context.

**Caching Authentication Details**

In order to make subsequent log ins easier, we will cache the users current authentication details before log out. This will allow us to initiate the authentication flow for their last active session without an additional fetch. This information will live in redux state. 


