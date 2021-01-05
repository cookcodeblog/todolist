[TOC]

# Exercise 2

https://rht-labs.github.io/enablement-docs/#/2-attack-of-the-pipelines/README





## Add webhook in GitLab

GitLab:

https://gitlab-ce.apps.hkha-1.apac-1.rht-labs.com/lsirui/todolist.git



Jenkins:

https://jenkins-williamnew-ci-cd.apps.hkha-1.apac-1.rht-labs.com/





```bash
https://jenkins-williamnew-ci-cd.apps.hkha-1.apac-1.rht-labs.com/multibranch-webhook-trigger/invoke?token=todolist
```



## Prepare local development



Start mongo in terminal1:

```bash
npm run mongo
```



Check matched command in package.json

```bash
# npm run mongo
"mongo": "docker run -i -d --name mongo-local -p 27017:27017 mongo"

# npm run mongo:stop
"mongo:stop": "docker stop mongo-local"

# npm run mongo:start
"mongo:start": "docker start mongo-local",
```





Run mongo with docker container.

Check :

```bash
docker ps
```





Start application in terminal2:

```bash
npm run serve:all
```



Acess application:

http://localhost:8080



Click todo menu.

Click 'Todo' at the top of the home page to get to the above page.



```bash
curl -s localhost:9000/api/todos | jq
```



The server hosting live reloads; so if you make changes to your code base the app will live update.



On todo page.

Input a new todos on "Add a new todos here", and press Enter key.









## Prepare Nexus

https://nexus-williamnew-ci-cd.apps.hkha-1.apac-1.rht-labs.com/



Prepare Nexus:

```bash
export YOUR_NAME=williamnew
export NEXUS_SERVICE_HOST=$(oc get route nexus --template='{{.spec.host}}' -n ${YOUR_NAME}-ci-cd)
export NEXUS_SERVICE_PORT=443

```



```bash
npm run prepare-nexus
```





Prepare-nexus command in package.json

```bash
"prepare-nexus": "curl -X POST  -u admin:admin123 -H 'Content-Type: application/json' -H 'Accept: application/json' -d @nexus.json https://${NEXUS_SERVICE_HOST}:${NEXUS_SERVICE_PORT}/service/siesta/rest/v1/script && curl -X POST  -u admin:admin123 -H 'Content-Type: text/plain' -H 'Accept: application/json' https://${NEXUS_SERVICE_HOST}:${NEXUS_SERVICE_PORT}/service/siesta/rest/v1/script/zip/run",
    "publish": "curl -vvv -u admin:admin123 --upload-file package-contents.zip http://${NEXUS_SERVICE_HOST}:${NEXUS_SERVICE_PORT}/repository/zip/com/redhat/todolist/${JOB_NAME}.${BUILD_NUMBER}/package-contents.zip",
    "publish:get": "curl -vvv http://admin:admin123@${NEXUS_SERVICE_HOST}:${NEXUS_SERVICE_PORT}/repository/zip/com/redhat/todolist/${BUILD_TAG}/package-contents.zip -o package-contents.zip"

```





```bash
curl -X POST  -u admin:admin123 -H 'Content-Type: application/json' -H 'Accept: application/json' -d @nexus.json https://${NEXUS_SERVICE_HOST}:${NEXUS_SERVICE_PORT}/service/siesta/rest/v1/script && curl -X POST  -u admin:admin123 -H 'Content-Type: text/plain' -H 'Accept: application/json' https://${NEXUS_SERVICE_HOST}:${NEXUS_SERVICE_PORT}/service/siesta/rest/v1/script/zip/run
```



Success response:

```json
{
  "name" : "zip",
  "result" : "RepositoryImpl$$EnhancerByGuice$$c5f0822b{type=hosted, format=raw, name='zip'}"
}
```



## Part 2 - Add configs to cluster



install the `openshift-applier` dependency：

```bash
ansible-galaxy install -r .openshift-applier/requirements.yml --roles-path=.openshift-applier/roles

```



Run ansible playbook:

```bash
ansible-playbook .openshift-applier/apply.yml -i .openshift-applier/inventory/
```



Check projects:

- William-ci-cd
  - BuildConfig: todolist
- Williamnew-dev
  - DeploymentConfig: todolis, mongoldb
- williamnew-test
  - DeploymentConfig: todolis, mongoldb



## Part3 - Build > Bake > Deploy



### 3a - build



```bash
dev-todolist-build

# Discard old builds
Max # of builds to keep with artifacts to 1

# labelled Restrict where this project can be run
jenkins-agent-npm


# git
https://gitlab-ce.apps.hkha-1.apac-1.rht-labs.com/lsirui/todolist.git
*/develop

Color ANSI Console Output checkbox
xterm


# add build step: execute shell
set -o xtrace
npm install
npm run build:ci
npm run package
npm run publish

# post-build actions
archive the artifacts
**

# git tag after build
Git Publisher
Tick the box Push Only If Build Succeeds
Tag to push: ${JOB_NAME}.${BUILD_NUMBER}
Tag message: Automated commit by jenkins from ${JOB_NAME}.${BUILD_NUMBER}
Check "Create new tag"
Target remote name "origin"


# trigger parameterized build on other project
Set the project to build to be dev-todolist-bake-deploy
Set the condition to be Stable or unstable but not failed
Click Add Parameters dropdown and select Predefined parameters.
BUILD_TAG=${JOB_NAME}.${BUILD_NUMBER}

```



Jenkins agent pending.

Wait for moment for jenkins agent is scheduled and running.





Nexus zip repository:

https://nexus-williamnew-ci-cd.apps.hkha-1.apac-1.rht-labs.com/#browse/browse:zip





### 3b - bake & deploy



```bash
dev-todolist-bake-deploy


# This project is parameterized
Add string parameter type
set the Name to BUILD_TAG. This will be available to the job as an Enviroment Variable.
You can set dev-todolist-build. as the default value for ease when triggering manually.
The description is not required but a handy one for reference would be ${JOB_NAME}.${BUILD_NUMBER} of previous build e.g. dev-todolist-build.1232


Restrict where this project can be run label to master

# build environment
tick Delete workspace before build starts


```



build step1 (bake):

```bash
#!/bin/bash
set -o xtrace
echo "### START BAKE IMAGE ###"
curl -v -f \
 http://admin:admin123@${NEXUS_SERVICE_HOST}:${NEXUS_SERVICE_PORT}/repository/zip/com/redhat/todolist/${BUILD_TAG}/package-contents.zip \
 -o package-contents.zip
unzip package-contents.zip
oc project williamnew-ci-cd
NAME=todolist
oc patch bc ${NAME} -p "{\"spec\":{\"output\":{\"to\":{\"kind\":\"ImageStreamTag\",\"name\":\"${NAME}:${BUILD_TAG}\"}}}}"
oc start-build ${NAME} --from-dir=package-contents/ --follow
echo "### END BAKE IMAGE ###"
```



Build step2 (deploy):

```bash
#!/bin/bash
set -o xtrace
echo "### START DEPLOY IMAGE ###"
PIPELINES_NAMESPACE=williamnew-ci-cd
NAMESPACE=williamnew-dev
NAME=todolist
oc project ${NAMESPACE}
oc tag ${PIPELINES_NAMESPACE}/${NAME}:${BUILD_TAG} ${NAMESPACE}/${NAME}:${BUILD_TAG}
oc set env dc ${NAME} NODE_ENV=dev
oc set image dc/${NAME} ${NAME}=image-registry.openshift-image-registry.svc:5000/${NAMESPACE}/${NAME}:${BUILD_TAG}
oc rollout latest dc/${NAME}
echo "### END DEPLOY IMAGE ###"
```



Run dev-todolist-bake-build, to see if auto trigger dev-todolist-bake-deploy.



On OpenShift, open williamnew-dev project, check todolist deployment status, and open todolist route url.

https://todolist-williamnew-dev.apps.hkha-1.apac-1.rht-labs.com/





### 3c - build pipeline (old fashion)



```bash
# create view
dev-todolist-pipeline
Build Pipeline View

Initial job: dev-todolist-build

```





## Part4 - Jenkinsfile



Jenkinsfile (develop):

```groovy
// Global Vars
NAMESPACE_PREFIX="williamnew"
GITLAB_DOMAIN = "gitlab-ce.apps.hkha-1.apac-1.rht-labs.com"
GITLAB_USERNAME = "lsirui"

```



Jenkinsfile (master):

```bash

```





Jenkins job:

```bash
Multibranch pipeline

#Branch Sources
Git
https://gitlab-ce.apps.hkha-1.apac-1.rht-labs.com/lsirui/todolist.git

# Scan Multibranch Pipeline Triggers
Check Scan by webhook
Trigger token: todolist


```



