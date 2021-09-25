
# EdgeXFoundary-on-k3s

## Setting up the VMs for K3s
We will configure k3s-server and k3s-agent differently on VirtualBox VMs. For this we will use Ubuntu-20.04 OS. Once the VMs are created, proceed with the following steps to deploy k3s.

Note that, it will be better to configure static IPs on both VMs.

### k3s server/master

To configure k3s server, we will use the following steps

```
export K3S_NODE_NAME=${HOSTNAME//_/-}
export K3S_EXTERNAL_IP=<host-ip>
curl -sfL https://docs.rancher.cn/k3s/k3s-install.sh |  sh -
```

Copy the node token generated
```
cat /var/lib/rancher/k3s/server/node-token
```
Check whether the k3s server is up and running

```
systemctl status k3s
```

### k3s agent

To configure k3s agent, we will use the following steps

```
export K3S_TOKEN=<node-token of k3s server>
export K3S_URL=https://<ip of k3s server>:6443
export INSTALL_K3S_EXEC="--docker --token $K3S_TOKEN --server $K3S_URL"
export K3S_NODE_NAME=${HOSTNAME//_/-}
curl -sfL https://docs.rancher.cn/k3s/k3s-install.sh | sh -
```

Check whether the k3s server is up and running

```
systemctl status k3s-agent
```

### EdgeXFoundary on k3s

Once k3s is is up and running, clone this repository for deploying EdgeXFoundary.

First, we will start with Consul. Consul is used and registry by EdgeXFoundary.

```
sudo helm --kubeconfig /etc/rancher/k3s/k3s.yaml upgrade --install consul ./consul-helm
```

Once Consul is in running state, you can visit the dashboard http://[k3s server ip]:[NodePort of service]

Note that if you visit the Key-Value store, it will be empty. We need to provide the respective configs for the EdgeXFoundary services. For this we will import the Key-Values and store it in Consul.

```
consul kv import --http-addr=http://[k3s server ip] @edgex-kv.json
```

Now you will see the configs in Key-Value store.

Deploy the EdgeXFoundary application services

```
kubectl apply -f /k3s/.
```

Notice that all the application services are up and running.

```sh
$ sudo kubectl get svc
NAME                                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                   AGE
kubernetes                             ClusterIP   10.43.0.1       <none>        443/TCP                                                                   4d1h
edgex-redis                            ClusterIP   10.43.56.89     <none>        6379/TCP                                                                  3d23h
edgex-core-consul                      ClusterIP   None            <none>        8500/TCP,8301/TCP,8301/UDP,8302/TCP,8302/UDP,8300/TCP,8600/TCP,8600/UDP   25h
consul                                 NodePort    10.43.193.246   <none>        80:32688/TCP                                                              25h
edgex-app-service-configurable-mqtt    NodePort    10.43.102.126   <none>        48101:32294/TCP                                                           4h26m
edgex-app-service-configurable-rules   NodePort    10.43.138.76    <none>        48100:30136/TCP                                                           4h26m
edgex-core-command                     NodePort    10.43.52.70     <none>        48082:32400/TCP                                                           4h26m
edgex-device-rest                      NodePort    10.43.167.127   <none>        49986:32536/TCP                                                           4h26m
edgex-core-metadata                    NodePort    10.43.132.29    <none>        48081:30220/TCP                                                           4h26m
edgex-support-notifications            NodePort    10.43.39.183    <none>        48060:32680/TCP                                                           4h26m
edgex-kuiper                           NodePort    10.43.231.24    <none>        48075:30082/TCP,20498:31868/TCP                                           4h26m
edgex-support-scheduler                NodePort    10.43.6.49      <none>        48085:31497/TCP                                                           4h26m
edgex-sys-mgmt-agent                   NodePort    10.43.250.114   <none>        48090:31736/TCP                                                           4h25m
edgex-core-data                        NodePort    10.43.251.191   <none>        5563:32220/TCP,48080:31931/TCP
```
```sh
$ sudo kubectl get pods
NAME                                                    READY   STATUS    RESTARTS   AGE
edgex-redis-54fb576f64-bdv9x                            1/1     Running   1          3d12h
consul-0                                                1/1     Running   0          25h
edgex-core-metadata-5bd45879cf-h9tbs                    1/1     Running   0          4h25m
edgex-kuiper-bbc6cf47-trkdl                             1/1     Running   0          4h25m
edgex-sys-mgmt-agent-7fb78c6fc5-qmqc2                   1/1     Running   0          4h25m
edgex-support-notifications-7b45446cbc-bqhvb            1/1     Running   0          4h25m
edgex-app-service-configurable-mqtt-59b5c7b6c8-8zhfc    1/1     Running   1          4h25m
edgex-app-service-configurable-rules-58c6846d54-s29hp   1/1     Running   1          4h25m
edgex-core-command-78b5ff9864-hb6wr                     1/1     Running   0          4h25m
edgex-core-data-86f5864db6-cvbl7                        1/1     Running   0          4h25m
edgex-support-scheduler-755b5779dc-br2d8                1/1     Running   0          4h25m
edgex-device-rest-599c579bf5-zbrg8                      1/1     Running   0          4h25m

```

### WIP - Adding Ingress LB to K3s