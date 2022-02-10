# Rob OMB work

## How to build a custom docker image
```
$ export BENCHMARK_TARBALL=package/target/openmessaging-benchmark-0.0.1-SNAPSHOT-bin.tar.gz
$ docker build --build-arg BENCHMARK_TARBALL . -f docker/Dockerfile -t vectorized/openmessaging-benchmark 
```

## Dependencies

So far only non standard dependency the driver will need is yq to parse yaml from the command line from the output of a kubectl command
```
$ sudo pip3 install yq
```

## Starting kubernetes & redpanda with kind
```
$ cd src/go/k8s
$ kind create cluster --config.yaml # remove images in config file
$ kind export kubeconfig
$ make certmanager-install
$ kubectl apply -k https://github.com/vectorizedio/redpanda/src/go/k8s/config/default
$ kubectl apply -f https://raw.githubusercontent.com/vectorizedio/redpanda/dev/src/go/k8s/config/samples/one_node_cluster.yaml
```

TODO: From this step we must grab the IP addresses of the redpanda nodes
```
$ kubectl get cluster -A -oyaml | yq '.items[0].status.nodes.internal' 
```

## Loading a custom docker image into k8s (optional)
```
$ kind load docker-image vectorized/omb-rob:latest
```
## Starting omb pods via helm

The mininum number of workers you can initialize is 2
```
$ cd benchmark/deployment/kubernetes/helm
$ helm install -n default --set numWorkers=$NUM_WORKERS benchmark ./benchmark
```

At this point all workers should be deployed with their corresponding http servers up, listening on port 8080. The sole pod with the driver shouldn't be doing anything -- this is expected. 

## Starting the omb benchmark 

Issue the command to start the benchmark onto the driver node, locally this was performed by sshing into the driver container to issue the command:
```
$ kubectl exec -it benchmark-driver -- /bin/bash
$ bin/benchmark --drivers driver-kafka/kafka-throughput.yaml --workers $WORKERS --kafka-broker-overrides $REDPANDA_BROKERS workloads/1-topic-16-partitions-1kb.yaml
```

Pass the ip/port combination of the redpanda nodes to the benchmark program and the values will be distributed to all workers. 


