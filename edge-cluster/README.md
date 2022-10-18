# Horizon Operator Example For Edge Cluster

This is a simple example of installing and using Edge Cluster (K3S) with Open Horizon.

- [Preparing Edge Cluster](#preparation)
- [Installing Agent on Edge Cluster](#installation)
- [Deploying services to your edge cluster](#deploy-service)
- [Uninstalling Agent and cleaning Edge Cluster](#uninstall)

## <a id=preparation></a> Preparing Edge Cluster

Prequisites for Installing Agent on the Cluster:

1. Either login as root or elevate to root with `sudo -i`

2. The full hostname of your machine must contain at least 2 dots. Check the full `hostname`

    - If the full hostname of your machine contains less than 2 dots, change the hostname:
    ```bash
    hostnamectl set-hostname <your-new-hostname-with-2-dots>
    ```

3. Install `k3s`:
```bash
curl -sfL https://get.k3s.io | sh -
```

### Create the image registry service:

4. Copy followig files to edge cluster machine.
    - `k3s-persistent-claim.yml`
    - `k3s-registry-deployment.yml`

5. Create the persistent volume claim:
```bash
kubectl apply -f k3s-persistent-claim.yml
```

6. Verify that the persistent volume claim was created and it is in "Pending" status
```bash
kubectl get pvc
```

7. Create the registry deployment and service:
```bash
kubectl apply -f k3s-registry-deployment.yml
```

8. Verify that the service was created:
```bash
kubectl get deployment
kubectl get service
```

9. Define the registry endpoint:
```bash
export REGISTRY_ENDPOINT=$(kubectl get service docker-registry-service | grep docker-registry-service | awk '{print $3;}'):5000
cat << EOF >> /etc/rancher/k3s/registries.yaml
mirrors:
  "$REGISTRY_ENDPOINT":
    endpoint:
      - "http://$REGISTRY_ENDPOINT"
EOF
```

10. Restart k3s to pick up the change to /etc/rancher/k3s/registries.yaml:
```bash
systemctl restart k3s
```

### Define this registry to docker as an insecure registry:

11. Create or add to /etc/docker/daemon.json (replacing <registry-endpoint> with the value of the $REGISTRY_ENDPOINT environment variable you obtained in a previous step).
```bash
{
  "insecure-registries": [ "<registry-endpoint>" ]
}
```

12. Restart docker to pick up the change:
```bash
systemctl restart docker
```

## <a id=installation></a> Installing Agent on Edge Cluster

Steps to install agent on cluster :

1. Login to your edge cluster as root.

2. Obtain the agentInstall script:
```bash
curl https://raw.githubusercontent.com/open-horizon/anax/master/agent-install/agent-install.sh --output agent-install.sh
```

3. Export your Horizon exchange user credentials:
```bash
export HZN_EXCHANGE_USER_AUTH="iamapikey:<api-key>"
```

4. The agent-install.sh script will store the IEAM agent in the edge cluster image registry. Set the full image path (minus the tag) that should be used. For example:
```bash
export IMAGE_ON_EDGE_CLUSTER_REGISTRY=$REGISTRY_ENDPOINT/openhorizon-agent/amd64_anax_k8s
```

5. Instruct agent-install.sh to use the default storage class:

    ```bash
     export EDGE_CLUSTER_STORAGE_CLASS=local-path
    ```

6. Run the agent-install.sh command to install and configure the Horizon agent and register your edge cluster with policy:

```bash
./agent-install.sh -D cluster -i https://github.com/open-horizon/anax/releases -d <node-id>
```

7. Verify the agent pod is running:
```bash
kubectl get namespaces
kubectl -n openhorizon-agent get pods
```

## <a id=deploy-service></a> Deploying services to your edge cluster

1. Set some aliases to make it more convenient to run the hzn command. (The hzn command is inside the agent container, but these aliases make it possible to run hzn from this host.)
```bash
cat << 'END_ALIASES' >> ~/.bash_aliases
alias getagentpod='kubectl -n openhorizon-agent get pods --selector=app=agent -o jsonpath={.items[*].metadata.name}'
alias hzn='kubectl -n openhorizon-agent exec -i $(getagentpod) -- hzn'
END_ALIASES
source ~/.bash_aliases
```

2. Verify that your edge node is configured (registered with the IEAM management hub):
```bash
hzn node list
```

3. To test your edge cluster agent, set your node policy with a property that will result in example helloworld operator and service being deployed to this edge node:
```bash
cat << 'EOF' > operator-example-node.policy.json
{
  "properties": [
    { "name": "openhorizon.example", "value": "nginx-operator" }
  ],
  "constraints": [
  ]
}
EOF

cat operator-example-node.policy.json | hzn policy update -f-
hzn policy list
```

4. After a minute, check for an agreement and the running edge operator and service containers:
```bash
hzn agreement list
kubectl -n openhorizon-agent get pods
```

## <a id=uninstall></a> Uninstalling Agent and cleaning Edge Cluster

1. Run following command to uninstall the Agent:
```bash
export HZN_EXCHANGE_USER_AUTH=iamapikey:<api-key>
./agent-uninstall.sh -u $HZN_EXCHANGE_USER_AUTH -d
```

2. Uninstalling K3s:
```bash
/usr/local/bin/k3s-uninstall.sh
```
