# Vault on Konvoy

## Reference Links

https://www.vaultproject.io/docs/platform/k8s/run.html

## Prerequisites
### 1. Connect to Kubernetes

Konvoy already has Helm installed so use D2iQ's Kubernetes installer Konvoy.

Tested Kubernetes versions: 1.15.3, 1.15.4

```
konvoy apply kubeconfig
```

Should look like this...
```
K8S $ kubectl get nodes
NAME                                         STATUS   ROLES    AGE     VERSION
ip-10-0-128-43.us-west-2.compute.internal    Ready    <none>   3h50m   v1.15.3
ip-10-0-128-69.us-west-2.compute.internal    Ready    <none>   3h50m   v1.15.3
ip-10-0-130-176.us-west-2.compute.internal   Ready    <none>   3h50m   v1.15.3
ip-10-0-131-15.us-west-2.compute.internal    Ready    <none>   3h50m   v1.15.3
ip-10-0-131-160.us-west-2.compute.internal   Ready    <none>   3h50m   v1.15.3
ip-10-0-131-94.us-west-2.compute.internal    Ready    <none>   3h50m   v1.15.3
ip-10-0-194-165.us-west-2.compute.internal   Ready    master   3h52m   v1.15.3
ip-10-0-195-2.us-west-2.compute.internal     Ready    master   3h52m   v1.15.3
ip-10-0-195-78.us-west-2.compute.internal    Ready    master   3h50m   v1.15.3
```

### 2. Install Helm

https://helm.sh/docs/using_helm/#installing-helm

Mac:
```
brew install helm
```

Should look like this...
```
K8S $ helm version
Client: &version.Version{SemVer:"v2.11.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
```

## Install Vault


```
# Clone the chart repo
git clone https://github.com/hashicorp/vault-helm.git
cd vault-helm

# Checkout a tagged version
git checkout v0.1.2

# Run Helm
helm install --name vault ./
```

Should look like this...
```
K8S PWD:vault-helm $ helm install --name vault ./
NAME:   vault
LAST DEPLOYED: Fri Oct  4 15:45:44 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME          DATA  AGE
vault-config  1     1s

==> v1/Pod(related)
NAME     READY  STATUS   RESTARTS  AGE
vault-0  0/1    Pending  0         1s

==> v1/Service
NAME   TYPE       CLUSTER-IP   EXTERNAL-IP  PORT(S)            AGE
vault  ClusterIP  10.0.11.174  <none>       8200/TCP,8201/TCP  1s

==> v1/ServiceAccount
NAME   SECRETS  AGE
vault  1        1s

==> v1/StatefulSet
NAME   READY  AGE
vault  0/1    1s


NOTES:

Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get vault
```

"That's it. The Helm chart does everything to setup a Vault-on-Kubernetes deployment."
                                                            -HashiCorp

Pod list should look like...
```
K8S vault-helm $ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
vault-0   0/1     Running   0          26m
```

You will see in the logs that it immediately needs to be initialized...
```
K8S vault-helm $ kubectl logs vault-0
==> Vault server configuration:

             Api Address: http://192.168.95.206:8200
                     Cgo: disabled
         Cluster Address: https://192.168.95.206:8201
              Listener 1: tcp (addr: "[::]:8200", cluster address: "[::]:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
               Log Level: info
                   Mlock: supported: true, enabled: true
                 Storage: file
                 Version: Vault v1.2.2

==> Vault server started! Log data will stream in below:

2019-10-09T02:15:26.718Z [INFO]  core: seal configuration missing, not initialized
...
```

## Access Vault Server

(Do this in a dummy terminal)
Should setup a service to expose this... TBD-1
```
kubectl port-forward vault-0 8200:8200
```

Should look like...
```
K8S-dummyterm ~ $ kubectl port-forward vault-0 8200:8200
Forwarding from 127.0.0.1:8200 -> 8200
Forwarding from [::1]:8200 -> 8200
```

Install Vault Server
- If you have a problem with this intallation skip to ALT-1
```
brew install vault
```

To avoid the TLS connection error, set this environment variable
```
export VAULT_ADDR=http://127.0.0.1:8200
```

ALT-1: Alternatively execute commands directly within the container
```
K8S vault-helm $ kubectl exec vault-0 -- vault version
Vault v1.2.2
```

## Initialize Vault

```
vault operator init
```

This is a Vault so they keep security as a high priority so you need to store these unseal keys and the root token.  If you lose these then you lose access to the Vault.

**HEY!!!!  KEEP THE 5 UNSEAL KEYS!!!!**

**HEY!!!!  KEEP THE INITIAL ROOT TOKEN!!!!**

Should look like...
```
K8S vault-helm $ vault operator init
Unseal Key 1: 6IRtCLLDkLJqBH4JyROqke8nIDMOB2YbpiGmuFr6G8fa
Unseal Key 2: ctOyIJx0LjYdypARO9ynkRIKv0DplLVTEFcVXJrk7OIf
Unseal Key 3: Ec66GNennT5/zy6aX61NdJkOAIYmDEF2d+UV3B2JlPce
Unseal Key 4: ckf/fonXhWnF1fhzvQH3gHunJmcF2/oi3yHACBXpXf4O
Unseal Key 5: s2FAn6RZxeB6zbdwznZ5hmV/bJfWaMMzYBCkLmZ5+zMw

Initial Root Token: s.fJ3esKKQ6C3izZqIhKltmgfG

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 key to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```

Check initialization (Alternative path to running commands locally)
```
kubectl exec vault-0 -- vault status -address=http://127.0.0.1:8200
```

Should look like...
```
K8S vault-helm $ kubectl exec vault-0 -- vault status -address=http://127.0.0.1:8200
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    0/3
Unseal Nonce       n/a
Version            1.2.2
HA Enabled         false
command terminated with exit code 2
```

## Access Vault UI

Open browser page
```
open http://127.0.0.1:8200
```

Should look like...
![IMG V1-UI1-UnsealVault](https://github.com/jdyver/konvoy-vault/blob/master/IMAGES/V1-UI1-UnsealVault.png)

Unseal Vault by copying 3 of the unseal keys (from the init above) into the prompt

Should look like...
![IMG V1-UI2-InputUnsealKeys](https://github.com/jdyver/konvoy-vault/blob/master/IMAGES/V1-UI2-InputUnsealKeys.png)

Now, login to Vault using the initial root token (from the init above) into the Token field

Should look like...
![IMG V1-UI3-SigningWithRootToken](https://github.com/jdyver/konvoy-vault/blob/master/IMAGES/V1-UI3-SigninWithRootToken.png)

Now we are in the Vault UI!
![IMG V1-UI4-Success](https://github.com/jdyver/konvoy-vault/blob/master/IMAGES/V1-UI4-Success.png)


You will also see that the Vault pod now has passed its health checks so fully up.
```
K8S vault-helm $ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
vault-0   1/1     Running   0          3m15s
```

## Setup Test







## Notes

TBD-1 NOTE - May need to use LB in values.yaml of the helm chart...
```
global:
  enabled: true
  image: "vault:1.2.2"

server:
  standalone:
    enabled: true
    config: |
      ui = true

      listener "tcp" {
        tls_disable = 1
        address = "[::]:8200"
        cluster_address = "[::]:8201"
      }
      storage "file" {
        path = "/vault/data"
      }

  service:
    enabled: true

  dataStorage:
    enabled: true
    size: 10Gi
    storageClass: null
    accessMode: ReadWriteOnce

ui:
  enabled: true
  serviceType: LoadBalancer
```