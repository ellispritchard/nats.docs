# Basic NATS and NATS Streaming Setup

## Minimal NATS and NATS Streaming Setup

To try NATS with the minimal number of components, you can start with the following:

```bash
# Single server NATS
kubectl apply -f https://raw.githubusercontent.com/nats-io/k8s/master/nats-server/single-server-nats.yml

kubectl apply -f https://raw.githubusercontent.com/nats-io/k8s/master/nats-streaming-server/single-server-stan.yml
```

This will setup:

* A statefulset with a single NATS server \(no auth nor TLS\)
* A single NATS Streaming node using file store with persistence
* A `nats` headless service to which you can connect
* A NATS Streaming Server that uses the cluster name `stan`

Note that the only service your applications have to connect to is the `nats` service, the NATS Streaming server will be available by using NATS as a transport.

Next, try using `nats-box` to connect to the `nats` service to confirm that you have set both NATS and NATS Streaming correctly.

```bash
kubectl run -i --rm --tty nats-box --image=synadia/nats-box --restart=Never

# Send message to NATS
nats-box:~# nats sub -s nats hello &
Listening on [hello]

nats-box:~# nats pub -s nats hello world
# Send/Receive message to STAN
nats-box:~# stan-pub -s nats -c stan hello world
Published [hello] : 'world'

nats-box:~# stan-sub -s nats -c stan hello
Connected to nats clusterID: [stan] clientID: [stan-sub]
Listening on [hello], clientID=[stan-sub], qgroup=[] durable=[]
[#1] Received: sequence:1 subject:"hello" data:"world" timestamp:1579544643374163630
```

## HA Setup Using StatefulSets

In order to have higher availability you can setup NATS and NATS Streaming \(STAN\) to run in clustering mode. The following commands will setup a 3 node NATS cluster as well as a 3 node NATS Streaming cluster that has an attached volume for persistence. Note, you will need more than one node available in your Kubernetes cluster in order for this to work, so in case of deploying onto minikube or docker desktop, please try the single node installer instead.

```bash
# Create NATS cluster
kubectl apply -f https://raw.githubusercontent.com/nats-io/k8s/master/nats-server/simple-nats.yml

# Create STAN cluster
kubectl apply -f https://raw.githubusercontent.com/nats-io/k8s/master/nats-streaming-server/simple-stan.yml
```

For NATS Streaming, it is actually recommended to use the Fault Tolerance mode as that would show better performance than clustering mode and better failover. You can follow this guide to setup [NATS Streaming with Fault Tolerance.](stan-ft-k8s-aws.md)

## Using Helm Charts

Both NATS and NATS Streaming have officially supported Helm charts as well:

* [NATS Helm Chart](https://github.com/nats-io/k8s/tree/master/helm/charts/nats)
* [NATS Streaming Helm Chart](https://github.com/nats-io/k8s/tree/master/helm/charts/stan)

