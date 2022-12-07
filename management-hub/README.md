# Horizon Management Hub Installation (All-in-One Script)

## System Requirements

- ### Minimum Resources
  - Memory: 4Gi
  - Storage: 20Gi
- ### OS Support
  - `Ubuntu 18.x` and `Ubuntu 20.x` are ***preferred***
  - `macOS` considered to be experimental due to docker issue
- ### Docker
  - Docker installed with `snap` is not supported by the script.
  - If docker has already been installed via Snap, remove the existing docker snap and allow the script to reinstall the latest version of docker.
  - Or can be installed using official package manager like `apt`.

## Installation
If all set we can run All-in-One script as `root`.

- ### Download and run All-in-One script
  Download script
  ```bash
  curl -sSL https://raw.githubusercontent.com/open-horizon/devops/master/mgmt-hub/deploy-mgmt-hub.sh
  ```
  Run the script
  ```bash
  ./deploy-mgmt-hub.sh
  ```
  If all went right, note down the credentials (like HZN_ORG_ID, HZN_EXCHANGE_USER_AUTH) from output summary which are required to access open-horizon commands and APIs. And test if all components of Open Horizon Management Hub are working correctly.
  
  Sample output summary:
  ```bash
  Summary of what was done:
  1. Started Horizon management hub services: agbot, exchange, postgres DB, CSS, mongo DB, vault
  2. Created exchange resources: system org (IBM) admin user, user org (myhorizonorg) and admin user, and agbot
    Automatically generated these passwords/tokens:
      EXCHANGE_ROOT_PW=UGBOHoXEbK8AdIHF3zGefSIGqVhmAsw6
      EXCHANGE_HUB_ADMIN_PW=jPgFb4zjQEY9lLHphW24V0VfAewXkf73
      EXCHANGE_SYSTEM_ADMIN_PW=wERtR8WADRBdkGgn45ztraY3PjtuQaER
      AGBOT_TOKEN=xGScOn0ae92LnLZBGuF2aMcp1UslV3UK
      EXCHANGE_USER_ADMIN_PW=CHFpT88En5Ps87WCtskBF8pK6erA8Uk1
      HZN_DEVICE_TOKEN=Lt0uprK68QLIFauT6aU3lEIhRD0Po9
    Important: save these generated passwords/tokens in a safe place. You will not be able to query them from Horizon.
  3. Installed and configured the Horizon agent and CLI (hzn)
  4. Created a Horizon developer key pair
  5. Installed the Horizon examples
  6. Created and registered an edge node to run the helloworld example edge service
  7. Created a vault instance: http://127.0.0.1:8200/ui/vault/auth?with=token
    Automatically generated this key/token:
      VAULT_UNSEAL_KEY=tKnQQZZkf1ljVTctp0ZqCBhZKQ1B90ebQZQPnQpc7qs=
      VAULT_ROOT_TOKEN=s.UNeFlFKG8qxAYFnACilNQBm2
    Important: save this generated key/token in a safe place. You will not be able to query them from Horizon.
  8. Added the hzn auto-completion file to ~/.bashrc (but you need to source that again for it to take effect in this shell session)
  For what to do next, see: https://github.com/open-horizon/devops/blob/master/mgmt-hub/README.md#all-in-1-what-next
  Before running the commands in the What To Do Next section, copy/paste/run these commands in your terminal:
   export HZN_ORG_ID=myhorizonorg
   export HZN_EXCHANGE_USER_AUTH=admin:CHFpT88En5Ps87WCtskBF8pK6erA8Uk1
  ```
  
  
  Otherwise refer following steps to fix the known issues.

- ### (Optional Fix) Error: http code 400 from: vaultUnseal 
  Stop and purge existing Management Hub services and then re-run the script.
  ```bash
  ./deploy-mgmt-hub.sh -PS
  ./deploy-mgmt-hub.sh
  ```

- ### (Optional Fix) Error: Encountered HTTP error: Put 
  Debug logs from pod `openhorizon/amd64_cloud-sync-service`:
  ```txt
  CSS: 2022/09/26 13:41:30 INFO: Horizon Authenticator starting with exchange authenticated identity
  * CSS: 2022/09/26 13:41:30 INFO: Connecting to mongo...
  * CSS: 2022/09/26 13:41:52 ERROR: Failed to dial mgo. Error: no reachable servers
  * CSS: 2022/09/26 13:41:52 ERROR: Retrying to connect to mongo
    ```
  This error caused due to unsupported `mongo` docker image version, you need to override this by setting an environment varible as follows.
  ```bash
  ./deploy-mgmt-hub.sh -PS
  export MONGO_IMAGE_TAG=4.0.6
  ./deploy-mgmt-hub.sh
  ``` 
