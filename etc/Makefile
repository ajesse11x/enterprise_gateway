# Copyright (c) Juper Development Team.
# Distributed under the terms of the Modified BSD License.

.PHONY: help clean clean-images clean-enterprise-gateway clean-enterprise-gateway-demo clean-nb2kg clean-demo-base \
    clean-kernel-images clean-py clean-tf-py clean-tf-gpu-py clean-r clean-spark-r clean-scala toree-launcher \
    kernelspecs_all kernelspecs_yarn kernelspecs_conductor kernelspecs_kubernetes kernelspecs_docker 

SA:=source activate
ENV:=enterprise-gateway-dev
SHELL:=/bin/bash

# Docker attributes - hub organization and tag.  Modify accordingly
HUB_ORG:=elyra

NO_CACHE?=--no-cache

help:
# http://marmelab.com/blog/2016/02/29/auto-documented-makefile.html
	@grep -E '^[a-zA-Z0-9_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

clean: ## Make a clean source tree
	-rm -rf kernel-launchers/scala/lib

#
# Kernelspec build section *************************************************
#

KERNELSPECS := kernelspecs_all kernelspecs_yarn kernelspecs_conductor kernelspecs_kubernetes kernelspecs_docker
kernelspecs: $(KERNELSPECS)

FILE_kernelspecs_all:=../dist/jupyter_enterprise_gateway_kernelspecs-$(VERSION).tar.gz
FILE_kernelspecs_yarn:=../dist/jupyter_enterprise_gateway_kernelspecs_yarn-$(VERSION).tar.gz
FILE_kernelspecs_conductor:=../dist/jupyter_enterprise_gateway_kernelspecs_conductor-$(VERSION).tar.gz
FILE_kernelspecs_kubernetes:=../dist/jupyter_enterprise_gateway_kernelspecs_kubernetes-$(VERSION).tar.gz
FILE_kernelspecs_docker:=../dist/jupyter_enterprise_gateway_kernelspecs_docker-$(VERSION).tar.gz

FILES_kernelspecs_all:=$(shell find kernel-launchers kernelspecs -type f -name '*')

TOREE_LAUNCHER_FILES:=$(shell find kernel-launchers/scala/toree-launcher/src -type f -name '*')

../build/kernelspecs: kernel-launchers/scala/lib  $(FILES_kernelspecs_all) 
	@mkdir -p ../build/kernelspecs
    # Seed the build tree with initial files
	cp -r kernelspecs ../build
    # Distribute language and config-sensitive files.
	@echo ../build/kernelspecs/{spark_,}python_* | xargs -t -n 1 cp -r kernel-launchers/python/scripts
	@echo ../build/kernelspecs/{spark_,}R_* | xargs -t -n 1 cp -r kernel-launchers/R/scripts
	@echo ../build/kernelspecs/{spark_,}scala_* | xargs -t -n 1 cp -r kernel-launchers/scala/lib
	@echo ../build/kernelspecs/{python,R,scala,python_tf,python_tf_gpu}_kubernetes | xargs -t -n 1 cp -r kernel-launchers/kubernetes/*
    # Fixme - deal with spark-based docker containers...
	@echo ../build/kernelspecs/{python,R,scala,python_tf,python_tf_gpu}_docker | xargs -t -n 1 cp -r kernel-launchers/docker/*
    # Perform the copy again to enable local, per-kernel, overrides
	cp -r kernelspecs ../build
	@(cd ../build/kernelspecs; find . -name 'kernel.json' -print0 | xargs -0 sed -i.bak "s/VERSION/$(TAG)/g"; find . -name *.bak -print0 | xargs -0 rm -f)
	@mkdir -p ../dist

PATTERN_kernelspecs_all := *
PATTERN_kernelspecs_yarn := *_yarn_*
PATTERN_kernelspecs_conductor := *_conductor_*
PATTERN_kernelspecs_kubernetes := *_kubernetes
PATTERN_kernelspecs_docker := *_docker

define BUILD_KERNELSPEC
$1: $$(FILE_$1)
$$(FILE_$1): ../build/kernelspecs
	rm -f $$(FILE_$1)
	@( cd ../build/kernelspecs; tar -pvczf "../$$(FILE_$1)" $$(PATTERN_$1) )
endef
$(foreach kernelspec,$(KERNELSPECS),$(eval $(call BUILD_KERNELSPEC,$(kernelspec))))

kernel-launchers/scala/lib: $(TOREE_LAUNCHER_FILES)
	-rm -rf kernel-launchers/scala/lib
	mkdir -p kernel-launchers/scala/lib
	@(cd kernel-launchers/scala/toree-launcher; sbt -Dversion=$(VERSION) package; cp target/scala-2.11/*.jar ../lib)
	curl -L https://repository.apache.org/content/repositories/releases/org/apache/toree/toree-assembly/0.3.0-incubating/toree-assembly-0.3.0-incubating.jar --output ./kernel-launchers/scala/lib/toree-assembly-0.3.0-incubating.jar
	
#
# Docker image build section ***********************************************
#

KERNEL_IMAGES := kernel-py kernel-spark-py kernel-r kernel-spark-r kernel-scala kernel-tf-py kernel-tf-gpu-py
DOCKER_IMAGES := demo-base enterprise-gateway-demo nb2kg enterprise-gateway $(KERNEL_IMAGES)
PUBLISHED_IMAGES := enterprise-gateway-demo nb2kg enterprise-gateway $(KERNEL_IMAGES)

docker-images: $(DOCKER_IMAGES)
kernel-images: $(KERNEL_IMAGES)

publish-images: publish-enterprise-gateway-demo publish-nb2kg publish-enterprise-gateway publish-kernel-py publish-kernel-spark-py publish-kernel-tf-py publish-kernel-tf-gpu-py publish-kernel-r publish-kernel-spark-r publish-kernel-scala

clean-images: clean-enterprise-gateway-demo clean-nb2kg clean-demo-base clean-enterprise-gateway clean-kernel-images
clean-kernel-images: clean-kernel-py clean-kernel-spark-py clean-kernel-tf-py clean-kernel-tf-gpu-py clean-kernel-r clean-kernel-spark-r clean-kernel-scala

# Extra dependencies for each docker image...
DEPENDS_nb2kg:
DEPENDS_demo-base:
DEPENDS_enterprise-gateway-demo: demo-base
DEPENDS_enterprise-gateway: $(FILE_kernelspecs_all)
DEPENDS_kernel-py DEPENDS_kernel-spark-py DEPENDS_kernel-r DEPENDS_kernel-spark-r DEPENDS_kernel-scala DEPENDS_kernel-tf-py DEPENDS_kernel-tf-gpu-py: $(FILE_kernelspecs_kubernetes) $(FILE_kernelspecs_docker)

# Extra targets for each docker image...
TARGETS_nb2kg:
TARGETS_demo-base:
TARGETS_enterprise-gateway TARGETS_enterprise-gateway-demo: kernelspecs
	@make -C .. bdist
TARGETS_kernel-py TARGETS_kernel-spark-py TARGETS_kernel-r TARGETS_kernel-spark-r TARGETS_kernel-scala TARGETS_kernel-tf-py TARGETS_kernel-tf-gpu-py: kernelspecs

# Extra files for each docker image...
FILES_nb2kg :=
FILES_demo-base :=
FILES_enterprise-gateway-demo := ../dist/jupyter_enterprise_gateway_kernelspecs-* ../dist/jupyter_enterprise_gateway*.whl
FILES_enterprise-gateway := ../dist/jupyter_enterprise_gateway_kernelspecs-* ../dist/jupyter_enterprise_gateway*.whl
FILES_kernel-py := kernel-launchers/python/bootstrap/*
FILES_kernel-spark-py :=
FILES_kernel-tf-py := kernel-launchers/python/bootstrap/*
FILES_kernel-tf-gpu-py := kernel-launchers/python/bootstrap/*
FILES_kernel-r := kernel-launchers/R/bootstrap/*
FILES_kernel-spark-r :=
FILES_kernel-scala := kernel-launchers/scala/bootstrap/*

# Trim out only the kernelspec files needed in each of the kernel images. Note that because kernel-based
# images only require the launcher (and supporting files) and launchers are consistent across kernel types,
# we can use the base kernelspecs.
LAUNCHER_FILES_nb2kg :=
LAUNCHER_FILES_demo-base :=
LAUNCHER_FILES_enterprise-gateway-demo :=
LAUNCHER_FILES_enterprise-gateway := kernel-launchers/scala
LAUNCHER_FILES_kernel-py := kernel-launchers/python
LAUNCHER_FILES_kernel-spark-py := kernel-launchers/python
LAUNCHER_FILES_kernel-tf-py := kernel-launchers/python
LAUNCHER_FILES_kernel-tf-gpu-py := kernel-launchers/python
LAUNCHER_FILES_kernel-r := kernel-launchers/R
LAUNCHER_FILES_kernel-spark-r := kernel-launchers/R
LAUNCHER_FILES_kernel-scala := kernel-launchers/scala

# Generate image creation targets for each entry in $(DOCKER_IMAGES).  Switch 'eval' to 'info' to see what is produced.
define BUILD_IMAGE
$1: ../.image-$1
../.image-$1: docker/$1/* DEPENDS_$1
	@make clean-$1 TARGETS_$1
	@mkdir -p ../build/docker/$1
	@cp -r docker/$1/* $$(FILES_$1) ../build/docker/$1
	@(if [[ ! -z "$$(LAUNCHER_FILES_$1)" ]]; then mkdir -p ../build/docker/$1/kernel-launchers; cp -r $$(LAUNCHER_FILES_$1) ../build/docker/$1/kernel-launchers; rm -rf ../build/docker/$1/kernel-launchers/*/bootstrap; fi;)
	@(cd ../build/docker/$1; docker build --build-arg HUB_ORG=${HUB_ORG} --build-arg TAG=${TAG} -t $(HUB_ORG)/$1:$(TAG) .)
	@touch ../.image-$1
	@-docker images $(HUB_ORG)/$1:$(TAG)
endef
$(foreach image,$(DOCKER_IMAGES),$(eval $(call BUILD_IMAGE,$(image))))

# Generate clean-xxx targets for each entry in $(DOCKER_IMAGES).  Switch 'eval' to 'info' to see what is produced.
define CLEAN_IMAGE
clean-$1:
	@rm -f ../.image-$1
	@-docker rmi -f $(HUB_ORG)/$1:$(TAG)
endef
$(foreach image,$(DOCKER_IMAGES),$(eval $(call CLEAN_IMAGE,$(image))))

# Publish each publish image on $(PUBLISHED_IMAGES) to DockerHub.  Switch 'eval' to 'info' to see what is produced.
define PUBLISH_IMAGE
publish-$1:
	@docker push $(HUB_ORG)/$1:$(TAG)
endef
$(foreach image,$(PUBLISHED_IMAGES),$(eval $(call PUBLISH_IMAGE,$(image))))

