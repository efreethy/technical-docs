# Working with Cypress




## Get registered with [cypress.io](http://cypress.io/)


If you aren’t already registered, request an invite to cypress in the #ride-help slack channel. Once registered, you get visibility into all the latest test runs - in addition to helpful debug assets like screenshots and video snapshots.

There is a corresponding jenkins pipeline ([cc-smoke-tests](https://jenkins.riv.al/job/Utilities/job/cron/job/cc-smoke-tests/)) where the test run output can be viewed, but it doesn’t provide additional insights like saved image or video snapshots.




## **Iterating on cypress tests locally**



### Cypress suite

 [consumer-client](https://github.com/10eTechnology/consumer-client/tree/master/cypress)


### Required env vars:

Cypress tests will log in using a special acceptance test user with credentials that are stored in env vars. Cypress env var values are stored [here](https://github.com/10eTechnology/consumer-client/blob/master/cypress.json). To run locally, you will need to add your own entries for `ENTERPRISE_USER` and `ENTERPRISE_USER_PASSWORD` under the “env” key. Unless specified otherwise the tests will run against stage, so you can temporarily fill in your own user credentials there.


### Spinning up local cypress

You can start a local cypress client with the following command:

```
yarn open cypress
```

This client is pretty versatile. It will allow you to run the whole suite, or a specific test. When this client is open, cypress test code is under change detection, and new changes will hot reload your test runs. This cypress client will also store screenshot and video artifacts from the test run under `cypress/screenshots/examples` and `cypress/videos/examples`



### Iterating on cypress test code

If you are adding/modifying a test (or just fixing a stale test) you can immediately just start making changes to the cypress test code. You can easily see if these changes work by simply running the the test suite through your local client.


### Iterating on consumer client code

In other cases, you may need to update consumer client code in response to a real issue that cypress is surfacing. 

In this case, you can* *do the following:

* Go ahead and make your local consumer client changes, and get them running on `localhost:3002`
* Point your local consumer client to the environment under test (by swapping out the contents of `web/src/config.json` with what you find behind <env-url/config.json (https://enterprise.riv.al/config.json for example)
* Make sure your cypress test is visiting your local consumer client, can usually do this by making sure in test setup that the test is visiting localhost:3002

* Run your test of interest through the local cypress client

Its worth noting that the `cc-smoke-test` pipeline always runs against the latest version of consumer-client code - so in order to keep tests passing you would need to merge your consumer-client changes first.



## Running through docker

If you want to more closely mimic the test runtime being used in jenkins - you can run the following

```
docker run --rm --env CYPRESS_[company]]_API_ENVIRONMENT=stage \
    --env CYPRESS_[company]]_CONSUMER_ENVIRONMENT=stage \
    --env CYPRESS_[company]]_ENTERPRISE_ENVIRONMENT=stage \
    --env [CYPRESS_ENTERPRISE_USER=acceptance-test-enterprise-user-stage@riv.al](mailto:CYPRESS_ENTERPRISE_USER=acceptance-test-enterprise-user-stage@riv.al) \
    --env CYPRESS_ENTERPRISE_USER_PASSWORD=<redacted> \
    --env NO_COLOR=1 -v "$(pwd)":/workdir -w /workdir [[company]]-docker.jfrog.io/cypress:3.6.0](http://[company]]-docker.jfrog.io/cypress:3.6.0) cypress run --browser chrome
```

