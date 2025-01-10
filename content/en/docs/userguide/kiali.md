---
draft: false
linktitle: Use Kiali to visualize traffic graph under Kmesh
menu:
  docs:
    parent: user guide
    weight: 19
title: Use Kiali to visualize traffic graph under Kmesh
toc: true
type: docs

---

### Preparation

1. Make default namespace managed by Kmesh.

2. Deploy bookinfo as sample application and sleep as curl client.

3. \[optional\] Install service granularity waypoint for service `reviews`.

*The above steps could refer to [Install Waypoint | Kmesh](https://istio.io/latest/docs/ambient/install/istioctl/). When installing Istio, we recommend installing Istio ambient mode instead of only installing Istiod, because Kiali currently depends on Istio components to work.*

4. Deploy prometheus that record Kmesh metrics as Istio standard metrics.

*This Prometheus addon leveages the Prometheus recording rules and relabeling configurations to convert Kmesh L4 metrics into Istio standard metrics, so that Kiali could visualize these metrics.*

```bash
kubectl apply -f https://raw.githubusercontent.com/kmesh-net/kmesh/main/samples/addons/prometheus_recording_istio.yaml
```

5. Deploy Kiali which reads metrics from the Prometheus.

```bash
kubectl apply -f https://raw.githubusercontent.com/kmesh-net/kmesh/main/samples/addons/kiali.yaml
```

### Generate some continuous traffic between applications in the mesh

```bash
kubectl exec deploy/sleep -- sh -c "while true; do curl -s http://productpage:9080/productpage | grep reviews-v.-; sleep 1; done"
```

### Use Kiali to visualize the traffic graph of services

1. Use the port-forward command to forward traffic to kiali.

```bash
kubectl port-forward --address 0.0.0.0 svc/kiali 20001:20001 -n kmesh-system
Forwarding from 0.0.0.0:20001 -> 20001
```

2. View the traffic graph in Kiali from browser.

Visit `Traffic Graph` panel. Select the `default` namespace at the top of left.

<div align="center">
<img src="/docs/userguide/kiali.png" width="1400" />
</div>

*In this traffic topology graph, the blue lines represent TCP traffic, which is proxied by Kmesh, while the green lines represent HTTP traffic, which is proxied by Waypoint. For more information about Kiali's traffic topology graph, please refer to [Kiali's documentation](https://kiali.io/docs/features/topology/).*

### Cleanup

1. Remove prometheus and grafana:

```bash
kubectl delete -f https://raw.githubusercontent.com/kmesh-net/kmesh/main/samples/addons/prometheus_recording_istio.yaml
kubectl delete -f https://raw.githubusercontent.com/kmesh-net/kmesh/main/samples/addons/kiali.yaml
```

2. If you are not planning to explore any follow-on tasks, refer to the [Install Waypoint/Cleanup](https://kmesh.net/en/docs/userguide/install_waypoint/#cleanup) instructions to remove waypoint and shutdown the application.