# Copyright 2017 The Go Authors. All rights reserved.
# Use of this source code is governed by a BSD-style
# license that can be found in the LICENSE file.

MUTABLE_VERSION ?= latest
VERSION ?= $(shell git rev-parse --short HEAD)

IMAGE_STAGING := gcr.io/go-dashboard-dev/gerritbot
IMAGE_PROD := gcr.io/symbolic-datum-552/gerritbot

DOCKER_IMAGE_build0=build0/gerritbot:latest
DOCKER_CTR_build0=gerritbot-build0

build0: *.go Dockerfile.0
	docker build --force-rm -f Dockerfile.0 --tag=$(DOCKER_IMAGE_build0) ../..

gerritbot: build0
	docker create --name $(DOCKER_CTR_build0) $(DOCKER_IMAGE_build0)
	docker cp $(DOCKER_CTR_build0):/go/bin/$@ $@
	docker rm $(DOCKER_CTR_build0)

update-deps:
	go install golang.org/x/build/cmd/gitlock
	gitlock --update=Dockerfile.0 golang.org/x/build/cmd/gerritbot

docker-prod: Dockerfile gerritbot
	docker build --force-rm --tag=$(IMAGE_PROD):$(VERSION) .
	docker tag $(IMAGE_PROD):$(VERSION) $(IMAGE_PROD):$(MUTABLE_VERSION)
docker-staging: Dockerfile gerritbot
	docker build --force-rm --tag=$(IMAGE_STAGING):$(VERSION) .
	docker tag $(IMAGE_STAGING):$(VERSION) $(IMAGE_STAGING):$(MUTABLE_VERSION)

push-prod: docker-prod
	gcloud docker -- push $(IMAGE_PROD):$(VERSION)
push-staging: docker-staging
	gcloud docker -- push $(IMAGE_STAGING):$(VERSION)

deploy-prod: push-prod
	kubectl set image deployment/gerritbot-deployment gerritbot=$(IMAGE_PROD):$(VERSION)
deploy-staging: push-staging
	kubectl set image deployment/gerritbot-deployment gerritbot=$(IMAGE_STAGING):$(VERSION)

.PHONY: clean
clean:
	$(RM) gerritbot
