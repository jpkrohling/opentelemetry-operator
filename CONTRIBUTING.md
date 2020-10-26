# How to contribube to this project

Coming soon.

## Testing it with OLM on OpenShift

Coming soon, but for the moment:

Build the operator
```
BUNDLE_VERSION=0.13.0
make set-image-controller IMG=quay.io/jpkroehling/opentelemetry-operator
make bundle
make bundle-build BUNDLE_IMG=quay.io/jpkroehling/opentelemetry-operator-bundle:${BUNDLE_VERSION}
podman push quay.io/jpkroehling/opentelemetry-operator-bundle:${BUNDLE_VERSION}
opm index add --bundles quay.io/jpkroehling/opentelemetry-operator-bundle:${BUNDLE_VERSION} --tag quay.io/jpkroehling/opentelemetry-operator-index:${BUNDLE_VERSION}
podman push quay.io/jpkroehling/opentelemetry-operator-index:${BUNDLE_VERSION}
```

Setup OLM:
```
cd ${OLM_HOME}
kubectl create -f deploy/upstream/quickstart/crds.yaml
kubectl create -f deploy/upstream/quickstart/olm.yaml
kubectl wait --for=condition=available deployment packageserver -n olm
kubectl wait --for=condition=available deployment olm-operator -n olm
kubectl wait --for=condition=available deployment catalog-operator -n olm
```

Install the operator
```
kubectl apply -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: opentelemetry-operator-manifests
  namespace: olm
spec:
  sourceType: grpc
  image: quay.io/jpkroehling/opentelemetry-operator-index:${BUNDLE_VERSION}
EOF
kubectl wait --for=condition=ready pod -l olm.catalogSource=opentelemetry-operator-manifests -n olm

kubectl apply -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: opentelemetry-operator-subscription
  namespace: operators
spec:
  channel: "alpha"
  installPlanApproval: Automatic
  name: opentelemetry-operator
  source: opentelemetry-operator-manifests
  sourceNamespace: olm
EOF

```