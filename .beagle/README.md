# kubernetes/release

<https://github.com/kubernetes/release>

## git

```bash
git remote add upstream git@github.com:kubernetes/release.git
git fetch upstream
git merge v0.15.0
```

## prepare

```bash
# github.com/open-beagle/kubernetes-release/images/build/go-runner
# go-runner:v2.3.1
docker run \
--rm \
-v $PWD/images/build/go-runner:/go/src/k8s.io/release/images/build/go-runner \
-w /go/src/k8s.io/release/images/build/go-runner \
-e PLUGIN_BINARY=go-runner \
-e PLUGIN_MAIN=. \
-e CI_WORKSPACE=/go/src/k8s.io/release/images/build/go-runner \
registry.cn-qingdao.aliyuncs.com/wod/devops-go-arch:1.19-alpine

mv images/build/go-runner/dist .beagle/debian-iptables/buster/
```

## build

```bash
# github.com/open-beagle/kubernetes-release/images/build/debian-base
# debian-base:v1.4.2-$ARCH
export CONFIG=buster && \
cd $PWD/.beagle/debian-base && \
make all && \
make all-push

# github.com/open-beagle/kubernetes-release/images/build/debian-iptables
# debian-iptables:v1.5.1-$ARCH
export CONFIG=buster && \
cd $PWD/.beagle/debian-iptables && \
make all && \
make all-push
```

## images

```bash
# kube-cross
docker pull gcr.io/google-containers/kube-cross:v1.13.6-1 && \
docker tag gcr.io/google-containers/kube-cross:v1.13.6-1 registry.cn-qingdao.aliyuncs.com/wod-k8s/kube-cross:v1.13.6-1 && \
docker push registry.cn-qingdao.aliyuncs.com/wod-k8s/kube-cross:v1.13.6-1

docker build -t registry.cn-qingdao.aliyuncs.com/wod/kube-cross-plus:v1.13.6-1 -f .beagle/cross/dockerfile .beagle/cross/ && \
docker push registry.cn-qingdao.aliyuncs.com/wod/kube-cross-plus:v1.13.6-1
```
