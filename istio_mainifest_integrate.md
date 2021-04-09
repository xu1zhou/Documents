# Profile
templates for different type of deployment
- enable/disable component
- customized part of values for components
# Charts
- components definition(bash ,gateway, istio-cni, istio-control)
## compiled charts into istioctl
2. make gen-charts and rebuild istioctl
```sh
root@master:/vagrant/istio-1source# make gen-charts
useradd: UID 0 is not unique
+ set -e
++ mktemp -d -t istio-charts.XXXXXXXXXX
+ OUT_DIR=/tmp/istio-charts.WPv2GtX6cu
+++ dirname operator/scripts/create_assets_gen.sh
++ cd operator/scripts
++ pwd
+ SCRIPTPATH=/work/operator/scripts
+ ROOTDIR=/work/operator/scripts/../..
+ OPERATOR_DIR=/work/operator/scripts/../../operator
+ /work/operator/scripts/create_release_charts.sh -o /tmp/istio-charts.WPv2GtX6cu
+++ dirname /work/operator/scripts/create_release_charts.sh
++ cd /work/operator/scripts
++ pwd
+ SCRIPT_DIR=/work/operator/scripts
+ INSTALLER_DIR=/work/operator/scripts/../../manifests
+ mkdir -p /tmp/istio-charts.WPv2GtX6cu
+ cp -R /work/operator/scripts/../../manifests/charts /tmp/istio-charts.WPv2GtX6cu
+ cp -R /work/operator/scripts/../../manifests/profiles /tmp/istio-charts.WPv2GtX6cu
+ cd /tmp/istio-charts.WPv2GtX6cu
+ go-bindata --nocompress --nometadata --pkg vfs -o /work/operator/scripts/../../operator/pkg/vfs/assets.gen.go ./...
+ rm -Rf /tmp/istio-charts.WPv2GtX6cu
```
1. check internal charts with `istioctl profile dump default|demo|....`

## find a way to generate all configuration of default profile, thus we could check the 
```
use command `out/linux_amd64/istioctl manifest generate` check validation
make gen-charts ;make build;./out/linux_amd64/istioctl manifest generate  | less 
# find deamonset configuration
```

## check render from profile value
make gen-charts ;make build
our/linux_amd64/istioctl install --set profile=default  -f manifests/profiles/default.yaml

error
```log
Istiod encountered an error: failed to update resource with server-side apply for obj DaemonSet/kube-system/bypass-tcpip: DaemonSet.apps "bypass-tcpip" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"install.operator.istio.io/owning-resource":"unknown", "istio.io/rev":"default", "name":"bypass-tcpip", "operator.istio.io/component":"Pilot"}: `selector` does not match template `labels`
```


## build image for daemonset
`docker build -t istio/bpf_bypass_tcpip:_1source_customized_ip_03_18_1540  -f Dockerfile .`