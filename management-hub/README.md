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
  If all went right, we are good to go and test if all components of Open Horizon Management Hub are working correctly.

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
