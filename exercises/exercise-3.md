[TOC]

# Exercise-3

## Tools and Frameworks

* Jest - Zero configuration testing platform Jest is used by Facebook to test all JavaScript code including React applications. One of Jest's philosophies is to provide an integrated "zero-configuration" experience. We observed that when engineers are provided with ready-to-use tools, they end up writing more tests, which in turn results in more stable and healthy code bases.

* Vue Test Utils - Vue Test Utils is the official unit testing utility library for Vue.js.
* Nightwatch.js - Nightwatch.js is an easy to use Node.js based End-to-End (E2E) testing solution for browser based apps and websites. It uses the powerful W3C WebDriver API to perform commands and assertions on DOM elements.
* Mocha - Mocha is a feature-rich JavaScript test framework running on Node.js and in the browser, making asynchronous testing simple and fun. Mocha tests run serially, allowing for flexible and accurate reporting, while mapping uncaught exceptions to the correct test cases. Hosted on GitHub.
* Sinon - Standalone test spies, stubs and mocks for JavaScript. Works with any unit testing framework.



## Test in local


Test in local:
```bash
"test:client": "vue-cli-service test",
"test:server": "node_modules/.bin/nyc node_modules/.bin/mocha 'server/**/*.spec.js' --exit",
   
```



## Test in pipeline

Test in pipeline:

```bash
"test:all:ci": "npm run test:client && npm run test:server:ci",
"test:server:ci": "export MOCHA_FILE='reports/server/mocha/test-results.xml' && export NODE_ENV=ci && node_modules/.bin/nyc node_modules/.bin/mocha 'server/**/*.spec.js' -R mocha-junit-reporter --exit",
    
```



Check Test Results in Jenkins pipeline build result.



Run many times test, you can see the test trend.



## End to End test in local



```bash
 "serve:all": "npm-run-all -p -r serve dev:server"
 
  "selenium": "nohup java -Dwebdriver.chrome.driver=\"$(pwd)/node_modules/chromedriver/lib/chromedriver/chromedriver\" -jar $(pwd)/node_modules/selenium-server/lib/runner/selenium-server-standalone-3.11.0.jar -port 8084 0<&- &>/dev/null &",
   
   
   "e2e": "export SELENIUM_PORT=443; export SELENIUM_SSL=1; export SELENIUM_HOST=$(oc get routes/$(oc get routes | grep 8084 | cut -d ' ' -f 1) --template='{{.spec.host}}') && export E2E_TEST_ROUTE=$(oc get routes/$(oc get routes | grep 8080 | cut -d ' ' -f 1)  --template='{{.spec.host}}') 
   && echo \"testing app on https://${E2E_TEST_ROUTE} \n with selenium address ${SELENIUM_HOST}\"
   && vue-cli-service e2e --env jenkins --url https://${E2E_TEST_ROUTE}",
   
   
   "e2e:ide": "vue-cli-service e2e --url=http://localhost:8080 --env jenkins",
   
```



No log after run `npm run selenium` ？？



Add a new `specs` folder in new test report, which is e2e test result.



https://jenkins-williamnew-ci-cd.apps.hkha-1.apac-1.rht-labs.com/job/todolist-ci/job/develop/4/testReport/specs/test/e2e_test___default_e2e_tests/



## Feature TDD



Check if `feature/important-flag` branch is auto triggered in Jenkins multi branch pipeline.



Using [Mocha](https://mochajs.org/) as our test runner; we will now write some tests for backend functionality to persist our important-flag. The changes required to the backend are minimal but we will use TDD to create our test first, then implement the functionality.



Using [Jest](https://facebook.github.io/jest/) as our test runner and the `vue-test-utils` library for managing our vue components; we will now write some tests for front end functionality to persist our important-flag. The changes required to the front end are quite large but we will use TDD to create our test first, then implement the functionality.



*Using [Nightwatch.js](http://nightwatchjs.org/) We will write a reasonably simple e2e test to test the functionality of the feature we just implemented.*





## OpenShift jenkinsfile build strategy



https://docs.openshift.com/container-platform/4.6/builds/build-strategies.html



The Pipeline build strategy is deprecated in OpenShift Container Platform 4. Equivalent and improved functionality is present in the OpenShift Container Platform Pipelines based on Tekton.



Jenkins images on OpenShift Container Platform are fully supported and users should follow Jenkins user documentation for defining their `jenkinsfile` in a job or store it in a Source Control Management system.





