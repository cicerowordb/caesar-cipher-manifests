### Install Istio

Check lastest versions at [https://github.com/istio/istio/releases](https://github.com/istio/istio/releases)

```bash
# Download
export ISTIO_VERSION=1.22.2 TARGET_ARCH=x86_64
kubectl create ns istio-system
curl -sL https://istio.io/downloadIstio| sh - > /dev/null

# Install
istio-$ISTIO_VERSION/bin/istioctl install -y --set components.ingressGateways.[0].enabled=false --set components.ingressGateways.[0].name=istio-ingressgateway

# Add-ons
kubectl -n istio-system apply -f istio-$ISTIO_VERSION/samples/addons/prometheus.yaml
kubectl -n istio-system apply -f istio-$ISTIO_VERSION/samples/addons/grafana.yaml
kubectl -n istio-system apply -f istio-$ISTIO_VERSION/samples/addons/loki.yaml
kubectl -n istio-system apply -f istio-$ISTIO_VERSION/samples/addons/kiali.yaml
kubectl -n istio-system apply -f istio-$ISTIO_VERSION/samples/addons/jaeger.yaml
```
