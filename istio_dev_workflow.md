# Workflow

```sh
apt install make

# This defines the docker hub to use when running integration tests and building docker images
# eg: HUB="docker.io/istio", HUB="gcr.io/istio-testing"
export HUB="docker.io/xu1zhou"

# This defines the docker tag to use when running integration tests and
# building docker images to be your user id. You may also set this variable
# this to any other legitimate docker tag.
export TAG=$USER

# This defines a shortcut to change directories to $HOME/istio.io
export ISTIO=$HOME/istio.io

# 
complete -W "\`find . -iname \"?akefil*\" | xargs -I {} grep -hoE '^[a-zA-Z0-9_.-]+:([^=]|$)' {} | sed 's/[^a-zA-Z0-9_.-]*$//' | sort -u\`" make


```
## 编译
    make build
## 打镜像
```sh
#TAG=repo-feature-$(date +%m_%d_%H%M) 
TAG=_1source_test_$(date +%m_%d_%H%M) 
# docker repo: docker.io/xu1zhou
# local repo: istio
export HUB="istio"
export $TAG;make docker # 对各组件（istioctl、mixer、pilot、istio-auth等）进行二进制包编译、测试、镜像编译
```
## 替换部署
需要替换default 中的tag为最新

    sed -i  "s/tag:.*/tag: '$TAG'/g"  /vagrant/istio-1source/manifests/profiles/default.yaml

    istioctl install --set profile=default  -f manifests/profiles/default.yaml

```sh

make push # 推送镜像到dockerhub

# 其他指令
make   docker.pilot # 编译pilot组件和镜像
make app  docker.app # 编译app组件和镜像
make proxy  docker.proxy # 编译proxy组件和镜像
make proxy_init  docker.proxy_init # 编译proxy_init组件和镜像
make proxy_debug  docker.proxy_debug # 编译proxy_debug组件和镜像
make sidecar_injector  docker.sidecar_injector # 编译sidecar_injector组件和镜像
make proxyv2  docker.proxyv2 # 编译proxyv2组件和镜像

make push.docker.pilot # 推送pilot镜像到dockerhub，其他组件类似

其他脚本
cd $GOPATH/src/istio.io/istio

./bin/get_workspace_status # 查看当前工作目录状态，包括环境变量等
install/updateVersion.sh -a ${HUB},${TAG} # 使用当前环境变量生成Istio清单
samples/bookinfo/build_push_update_images.sh # 使用当前环境变量编译并推送bookinfo镜像

#bookinfo 安装
kubectl label namespace default istio-injection=enabled
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml
```