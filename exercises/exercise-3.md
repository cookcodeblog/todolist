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

Test in pipeline:
```bash
"test:all:ci": "npm run test:client && npm run test:server:ci",
"test:server:ci": "export MOCHA_FILE='reports/server/mocha/test-results.xml' && export NODE_ENV=ci && node_modules/.bin/nyc node_modules/.bin/mocha 'server/**/*.spec.js' -R mocha-junit-reporter --exit",
    
```