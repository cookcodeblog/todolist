[TOC]

# Exercise 2

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







