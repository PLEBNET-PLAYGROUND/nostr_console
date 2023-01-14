SHELL := /bin/bash

PWD 									?= pwd_unknown

THIS_FILE								:= $(lastword $(MAKEFILE_LIST))
export THIS_FILE
TIME									:= $(shell date +%s)
export TIME

ARCH                                    :=$(shell uname -m)
export ARCH
ifeq ($(ARCH),x86_64)
TRIPLET                                 :=x86_64-linux-gnu
export TRIPLET
endif
ifeq ($(ARCH),arm64)
TRIPLET                                 :=aarch64-linux-gnu
export TRIPLET
endif
#available services:
#	bitcoind
#	lnd
#	tor
#	thunderhub
#	rtl
#	notebook
#	dashboard
#	lndg
#	docs
ifeq ($(services),)
services                                :=bitcoind,lnd,cln,rtl,thunderhub,docs
else
services                                :=$(services)
endif
export services
ifeq ($(user),)
HOST_USER								:= root
HOST_UID								:= $(strip $(if $(uid),$(uid),0))
else
HOST_USER								:=  $(strip $(if $(user),$(user),nodummy))
HOST_UID								:=  $(strip $(if $(shell id -u),$(shell id -u),4000))
endif
export HOST_USER
export HOST_UID

ifeq ($(target),)
SERVICE_TARGET							?= shell
else
SERVICE_TARGET							:= $(target)
endif
export SERVICE_TARGET

ifeq ($(docker),)
DOCKER							        := $(shell which docker)
else
DOCKER   							    := $(docker)
endif
export DOCKER

ifeq ($(compose),)
DOCKER_COMPOSE						    := $(shell which docker-compose)
else
DOCKER_COMPOSE							:= $(compose)
endif
export DOCKER_COMPOSE
ifeq ($(reset),true)
RESET:=true
else
RESET:=false
endif
export RESET

PYTHON                                  := $(shell which python)
export PYTHON
PYTHON2                                 := $(shell which python2)
export PYTHON2
PYTHON3                                 := $(shell which python3)
export PYTHON3

PIP                                     := $(shell which pip)
export PIP
PIP2                                    := $(shell which pip2)
export PIP2
PIP3                                    := $(shell which pip3)
export PIP3

python_version_full := $(wordlist 2,4,$(subst ., ,$(shell python3 --version 2>&1)))
python_version_major := $(word 1,${python_version_full})
python_version_minor := $(word 2,${python_version_full})
python_version_patch := $(word 3,${python_version_full})

my_cmd.python.3 := $(PYTHON3) some_script.py3
my_cmd := ${my_cmd.python.${python_version_major}}

PYTHON_VERSION                         := ${python_version_major}.${python_version_minor}.${python_version_patch}
PYTHON_VERSION_MAJOR                   := ${python_version_major}
PYTHON_VERSION_MINOR                   := ${python_version_minor}

export python_version_major
export python_version_minor
export python_version_patch
export PYTHON_VERSION

#PROJECT_NAME defaults to name of the current directory.
ifeq ($(project),)
PROJECT_NAME							:= $(notdir $(PWD))
else
PROJECT_NAME							:= $(project)
endif
export PROJECT_NAME

#GIT CONFIG
GIT_USER_NAME							:= $(shell git config user.name)
export GIT_USER_NAME
GIT_USER_EMAIL							:= $(shell git config user.email)
export GIT_USER_EMAIL
GIT_SERVER								:= https://github.com
export GIT_SERVER

GIT_REPO_NAME							:= $(PROJECT_NAME)
export GIT_REPO_NAME

#Usage
#make package-all profile=rsafier
#make package-all profile=asherp
#note on GH_TOKEN.txt file below
ifeq ($(profile),)
GIT_PROFILE								:= $(GIT_USER_NAME)
ifeq ($(GIT_REPO_ORIGIN),git@github.com:PLEBNET_PLAYGROUND/plebnet-playground-docker.dev.git)
GIT_PROFILE								:= PLEBNET-PLAYGROUND
endif
ifeq ($(GIT_REPO_ORIGIN),https://github.com/PLEBNET_PLAYGROUND/plebnet-playground-docker.dev.git)
GIT_PROFILE								:= PLEBNET-PLAYGROUND
endif
else
GIT_PROFILE								:= $(profile)
endif
export GIT_PROFILE

GIT_BRANCH								:= $(shell git rev-parse --abbrev-ref HEAD)
export GIT_BRANCH
GIT_HASH								:= $(shell git rev-parse --short HEAD)
export GIT_HASH
GIT_PREVIOUS_HASH						:= $(shell git rev-parse --short HEAD^1)
export GIT_PREVIOUS_HASH
GIT_REPO_ORIGIN							:= $(shell git remote get-url origin)
export GIT_REPO_ORIGIN
GIT_REPO_PATH							:= $(HOME)/$(GIT_REPO_NAME)
export GIT_REPO_PATH

ifneq ($(bitcoin-datadir),)
BITCOIN_DATA_DIR						:= $(bitcoin-datadir)
else
BITCOIN_DATA_DIR						:= $(HOME)/.bitcoin
endif
export BITCOIN_DATA_DIR

ifeq ($(nocache),true)
NOCACHE					     			:= --no-cache
#Force parallel build when --no-cache to speed up build
PARALLEL                                := --parallel
else
NOCACHE						    		:=
PARALLEL                                :=
endif
ifeq ($(parallel),true)
PARALLEL                                := --parallel
endif
ifeq ($(para),true)
PARALLEL                                := --parallel
endif
export NOCACHE
export PARALLEL

ifeq ($(verbose),true)
VERBOSE									:= --verbose
else
VERBOSE									:=
endif
export VERBOSE

#TODO more umbrel config testing
ifeq ($(port),)
PUBLIC_PORT								:= 80
else
PUBLIC_PORT								:= $(port)
endif
export PUBLIC_PORT

ifeq ($(nodeport),)
NODE_PORT								:= 8333
else
NODE_PORT								:= $(nodeport)
endif
export NODE_PORT

ifneq ($(passwd),)
PASSWORD								:= $(passwd)
else
PASSWORD								:= changeme
endif
export PASSWORD

ifeq ($(cmd),)
CMD_ARGUMENTS							:=
else
CMD_ARGUMENTS							:= $(cmd)
endif
export CMD_ARGUMENTS

#ifeq ($(umbrel),true)
##comply with umbrel conventions
#PWD=/home/umbrel/umbrel/apps/$(PROJECT_NAME)
#UMBREL=true
#else
#pwd ?= pwd_unknown
#UMBREL=false
#endif
#export PWD
#export UMBREL
########################

PACKAGE_PREFIX                          := ghcr.io
export PACKAGE_PREFIX
.PHONY: - all
-:
	#NOTE: 2 hashes are detected as 1st column output with color
	@awk 'BEGIN {FS = ":.*?## "} /^[a-zA-Z_-]+:.*?##/ {printf "\033[36m%-15s\033[0m %s\n", $$1, $$2}' $(MAKEFILE_LIST)

.PHONY: help
help:## 	print verbose help
	@echo 'make [COMMAND] [EXTRA_ARGUMENTS]	'
	@echo ''
	#@echo ''
	@echo 'make '
	@echo '	 make all                        install and run playground and cluster'
	@echo '	 make help                       print help'
	@echo '	 make report                     print environment variables'
	@echo '	 make init                       initialize basic dependencies'
	@echo '	 make build'
	@echo '	 make build para=true            parallelized build'
	@echo '	 make install'
	@echo ''
	@echo '	[NOSTR SERVICES]:	'
	@echo ''
	@echo ''
	@echo '	[DEV ENVIRONMENT]:	'
	@echo ''
#	@echo '	 make shell            compiling environment on host machine'
	@echo '	 make signin profile=gh-user     ~/GH_TOKEN.txt required from github.com'
#	@echo '	 make header package-header'
	@echo '	 make build'
	@echo ''
	@sed -n 's/^# //p' ${MAKEFILE_LIST} | column -t -s ':' |  sed -e 's/^/# /'
	@sed -n 's/^## //p' ${MAKEFILE_LIST} | column -t -s ':' |  sed -e 's/^/## /'
	@sed -n 's/^### //p' ${MAKEFILE_LIST} | column -t -s ':' |  sed -e 's/^/### /'

.PHONY: report
report:## 	print environment arguments
	@echo ''
	@echo '	[ARGUMENTS]	'
	@echo '      args:'
	@echo '        - PROJECT_NAME=${PROJECT_NAME}'
	@echo '        - HOME=${HOME}'
	@echo '        - PWD=${PWD}'
	@echo '        - PYTHON=${PYTHON}'
	# @echo '        - PYTHON2=${PYTHON2}'
	@echo '        - PYTHON3=${PYTHON3}'
	@echo '        - PYTHON_VERSION=${PYTHON_VERSION}'
	@echo '        - PYTHON_VERSION_MAJOR=${PYTHON_VERSION_MAJOR}'
	@echo '        - PYTHON_VERSION_MINOR=${PYTHON_VERSION_MINOR}'
	@echo '        - PIP=${PIP}'
	# @echo '        - PIP2=${PIP2}'
	@echo '        - PIP3=${PIP3}'
	# @echo '        - UMBREL=${UMBREL}'
	# @echo '        - THIS_FILE=${THIS_FILE}'
	@echo '        - TIME=${TIME}'
	@echo '        - PACKAGE_PREFIX=${PACKAGE_PREFIX}'
	@echo '        - ARCH=${ARCH}'
	@echo '        - TRIPLET=${TRIPLET}'
	@echo '        - services=${services}'
	@echo '        - HOST_USER=${HOST_USER}'
	@echo '        - HOST_UID=${HOST_UID}'
	@echo '        - PUBLIC_PORT=${PUBLIC_PORT}'
	@echo '        - NODE_PORT=${NODE_PORT}'
	# @echo '        - SERVICE_TARGET=${SERVICE_TARGET}'
	@echo '        - DOCKER_COMPOSE=${DOCKER_COMPOSE}'
	@echo '        - GIT_USER_NAME=${GIT_USER_NAME}'
	@echo '        - GIT_USER_EMAIL=${GIT_USER_EMAIL}'
	@echo '        - GIT_SERVER=${GIT_SERVER}'
	@echo '        - GIT_PROFILE=${GIT_PROFILE}'
	@echo '        - GIT_BRANCH=${GIT_BRANCH}'
	@echo '        - GIT_HASH=${GIT_HASH}'
	@echo '        - GIT_PREVIOUS_HASH=${GIT_PREVIOUS_HASH}'
	@echo '        - GIT_REPO_ORIGIN=${GIT_REPO_ORIGIN}'
	@echo '        - GIT_REPO_NAME=${GIT_REPO_NAME}'
	@echo '        - GIT_REPO_PATH=${GIT_REPO_PATH}'
	@echo '        - NOCACHE=${NOCACHE}'
	# @echo '        - VERBOSE=${VERBOSE}'
	# @echo '        - PASSWORD=${PASSWORD}'
	# @echo '        - CMD_ARGUMENTS=${CMD_ARGUMENTS}'

#######################

ORIGIN_DIR:=$(PWD)
MACOS_TARGET_DIR:=/var/root/$(PROJECT_NAME)
LINUX_TARGET_DIR:=/root/$(PROJECT_NAME)
export ORIGIN_DIR
export TARGET_DIR

all: install## 	all
.PHONY: init
.SILENT:
init:## 	basic setup

ifneq ($(shell id -u),0)
	@echo
	@echo $(shell id -u -n) 'not root'
	@echo
endif

	git config --global --add safe.directory $(PWD)
	mkdir -p volumes
	mkdir -p cluster/volumes
	chown -R $(shell id -u) *                 || echo

	install -v -m=o+rwx $(PWD)/scripts/*  /usr/local/bin/
	install -v -m=o+rwx $(PWD)/getcoins.py  /usr/local/bin/play-getcoins

	$(PYTHON3) -m pip install --upgrade pip 2>/dev/null
	$(PYTHON3) -m pip install -q omegaconf 2>/dev/null
	$(PYTHON3) -m pip install -q -r requirements.txt 2>/dev/null
	pushd docs 2>/dev/null && $(PYTHON3) -m pip install -q -r requirements.txt && popd  2>/dev/null
	$(PYTHON3) plebnet_generate.py TRIPLET=$(TRIPLET) services=$(SERVICES)

	pushd scripts > /dev/null; for string in *; do sudo chmod -R o+rwx /usr/local/bin/$$string; done; popd  2>/dev/null || echo


.PHONY: install install-cluster
.SILENT:
install: venv## 	create docker-compose.yml and run playground
	bash -c './install.sh $(TRIPLET)'
.PHONY: uninstall
uninstall: 	run uninstall.sh script
	bash -c './uninstall.sh $(TRIPLET)'
#######################
.PHONY: run
run: init## 	...
	@echo "run: init"
	$(DOCKER_COMPOSE) $(VERBOSE) $(NOCACHE) up -d
#######################
.PHONY: build
build: init
	@echo "build..."

SIGNIN=$(GIT_PROFILE)
export SIGNIN

.PHONY: signin
signin:## 	signin
#Place a file named GH_TOKEN.txt in your $HOME - create in https://github.com/settings/tokens (Personal access tokens)
	bash -c 'cat ~/GH_TOKEN.txt | docker login $(PACKAGE_PREFIX)/v1 -u $(GIT_PROFILE) --password-stdin'
#######################
package-plebnet: signin

	@docker compose build

	bash -c 'docker tag  $(PROJECT_NAME)_thunderhub   $(PACKAGE_PREFIX)/$(GIT_PROFILE)/$(PROJECT_NAME)/thunderhub-$(TRIPLET)/$(HOST_USER):$(TIME) || echo skip thunderhub'
	bash -c 'docker push                              $(PACKAGE_PREFIX)/$(GIT_PROFILE)/$(PROJECT_NAME)/thunderhub-$(TRIPLET)/$(HOST_USER):$(TIME) || echo skip thunderhub'
