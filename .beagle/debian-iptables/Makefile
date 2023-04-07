# Copyright 2016 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

.PHONY:	build push all all-build all-push-images all-push push-manifest

REGISTRY?=registry.cn-qingdao.aliyuncs.com/wod
IMAGE=$(REGISTRY)/debian-iptables

TAG ?= $(shell git describe --tags --always --dirty)
IMAGE_VERSION ?= v1.5.1
CONFIG ?= bullseye
DEBIAN_BASE_VERSION ?= v1.4.2
GORUNNER_VERSION ?= v2.3.1

ARCH?=amd64
ALL_ARCH = amd64 arm64 ppc64le mips64le loong64

BASE_REGISTRY?=registry.cn-qingdao.aliyuncs.com/wod

# Build args
BASEIMAGE?=$(BASE_REGISTRY)/debian-base:$(DEBIAN_BASE_VERSION)-$(ARCH)
GORUNNERIMAGE?=$(BASE_REGISTRY)/kubernetes-go-runner:$(GORUNNER_VERSION)-$(ARCH)

QEMUVERSION=6.1.0-8

# This option is for running docker manifest command
export DOCKER_CLI_EXPERIMENTAL := enabled

SUDO=$(if $(filter 0,$(shell id -u)),,sudo)

build:
	# Fix possible issues with the local umask
	umask 0022

	# Enable execution of multi-architecture containers
	# docker run --rm --privileged multiarch/qemu-user-static:$(QEMUVERSION) --reset -p yes
	# docker buildx version
	# BUILDER=$(shell docker buildx create --use)
	docker build \
		--pull \
		-t $(IMAGE):$(IMAGE_VERSION)-$(ARCH) \
		-t $(IMAGE):$(TAG)-$(ARCH) \
		-t $(IMAGE):latest-$(ARCH) \
		--build-arg=BASEIMAGE=$(BASEIMAGE) \
		--build-arg=GORUNNERARCH=$(GORUNNERARCH) \
		$(CONFIG)
	# docker buildx rm $$BUILDER

push: build
	docker push $(IMAGE):$(IMAGE_VERSION)-$(ARCH)
	docker push $(IMAGE):$(TAG)-$(ARCH)
	docker push $(IMAGE):latest-$(ARCH)

sub-build-%:
	$(MAKE) ARCH=$* build

all-build: $(addprefix sub-build-,$(ALL_ARCH))

sub-push-image-%:
	$(MAKE) ARCH=$* push

all-push-images: $(addprefix sub-push-image-,$(ALL_ARCH))

all-push: all-push-images push-manifest

push-manifest:
	docker manifest create --amend $(IMAGE):$(IMAGE_VERSION) $(shell echo $(ALL_ARCH) | sed -e "s~[^ ]*~$(IMAGE):$(IMAGE_VERSION)\-&~g")
	@for arch in $(ALL_ARCH); do docker manifest annotate --arch $${arch} ${IMAGE}:${IMAGE_VERSION} ${IMAGE}:${IMAGE_VERSION}-$${arch}; done
	docker manifest push --purge ${IMAGE}:${IMAGE_VERSION}

all: all-push
