#################################################################################
# Variables	                                                                    #
#################################################################################

THISDIR:=$(abspath $(dir $(lastword $(MAKEFILE_LIST))))
python = $(shell if ( python3 -c 'import sys; sys.exit(0)' 2> /dev/null ); then echo "python3" ; else echo "python"; fi )
virtualenv = $(shell if ( virtualenv-3 -h > /dev/null 2>&1 ); then echo "virtualenv-3" ; else echo "virtualenv" ; fi )
virtualenv_dir ?= $(THISDIR)/venv


################################################################################
# Fix fork problem in macOS                                                    #
################################################################################

UNAME_S := $(shell uname -s)
ifeq ($(UNAME_S),Darwin)
	export OBJC_DISABLE_INITIALIZE_FORK_SAFETY = YES
endif


#################################################################################
# Clean	                                                                        #
#################################################################################

clean:
	rm -rf .pytest_cache
	rm -rf vision.egg-info
	rm -rf target

#################################################################################
# Environment	                                                                #
#################################################################################

.PHONY: clear-venv ensure-venv

clear-venv:
	rm -rf $(virtualenv_dir)

ensure-venv:
	$(virtualenv) --python=$(python) $(virtualenv_dir)
	@echo "Install requirements"
	$(virtualenv_dir)/bin/pip install -r requirements.txt

compile-requirements:
	$(virtualenv_dir)/bin/pip-compile --output-file requirements.txt requirements.in

sync-venv:
	$(virtualenv_dir)/bin/pip-sync

#################################################################################
# Run test	                                                                    #
#################################################################################

test:
	. $(virtualenv_dir)/bin/activate; pytest -v --cov=pyragnarok/


#################################################################################
# Run Service	                                                               #
#################################################################################

run-service-local:
	CONFIG_FILE="config/app/dev/pyragnarok.ini" gunicorn -k aiohttp.worker.GunicornWebWorker -c pyragnarok/gunicorn_conf_aiohttp.py pyragnarok.app.aiohttp_app:init_func

run-flask-service-local:
	CONFIG_FILE="config/app/dev/pyragnarok.ini" gunicorn -k gthread --thread=2 -c pyragnarok/gunicorn_conf_flask.py pyragnarok.app.flask_app:app
#################################################################################
# Build pex                                                                     #
#################################################################################

build: clean
	python --version
	pip install pex
	mkdir -p target
	pex . -v --disable-cache -r requirements.txt -o target/pyragnarok_env.pex

#################################################################################
# Docker                                                                        #
#################################################################################

.PHONY: docker-build docker-push-tag

## Build vision docker image to batch registry
docker-build:
	@echo "Building docker image"
	docker build . -t ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA}
	@echo "Pushing docker image to local registry"
	docker push ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA}

docker-push-tag:
	@echo "Pulling from gitlab registry"
	docker pull ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA}
	@echo "Pushing to gitlab registry"
	docker tag ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA} ${CI_REGISTRY_IMAGE}:${DOCKER_TAG}
	docker push ${CI_REGISTRY_IMAGE}:${DOCKER_TAG}
	@echo "Pushing to ECR"
	docker tag ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHA} ${ECR_REGISTRY}:${DOCKER_TAG}
	docker push ${ECR_REGISTRY}:${DOCKER_TAG}

