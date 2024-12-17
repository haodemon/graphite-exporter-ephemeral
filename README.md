# Graphite Exporter Ephemeral

This repo is a fork of the [Prometheus Graphite Exporter](https://github.com/prometheus/graphite_exporter) with the following changes:
  * The ability to run the exporter in the "ephemeral" mode – the exporter will only serve the metrics once and then delete them.

Please find the original README in the repo above.

## Usage

```sh
git apply patches/patches/delete-after-scrape.patch
make
./graphite_exporter
```

Configure existing monitoring to send Graphite plaintext data to port 9109 on UDP or TCP.
As a simple demonstration:


```sh
# Send the metric
echo "test.tcp 10 $(date +%s)" | nc localhost 9109

# The metric is available
curl localhost:9108/metrics  | grep 'test_tcp 10' | wc -l
  1

# The metric is not available anymore
curl localhost:9108/metrics | grep 'test_tcp 10' | wc -l
  0
```

Metrics will be available on [http://localhost:9108/metrics](http://localhost:9108/metrics).

To avoid using unbounded memory, metrics will be garbage collected five minutes after
they are last pushed to. This is configurable with the `--graphite.sample-expiry` flag.


## Kubernetes Setup

Replace the image in `k8s/graphite-exporter-deployment.yaml` with your own image and run the following commands:
```sh
kubectl apply -f k8s/graphite-exporter-deployment.yaml
kubectl apply -f k8s/graphite-exporter-service.yaml
```

To get the IP of the service, run:
```sh
kubectl get pod -l app=graphite-exporter-ephemeral -o jsonpath='{.items[0].status.podIP}'
  10.252.1.256  # example output
```


## Docker Setup
```sh
docker build -t graphite-exporter-ephemeral .
docker run -d -p 9108:9108 -p 9109:9109 -p 9109:9109/udp graphite-exporter-ephemeral
```
