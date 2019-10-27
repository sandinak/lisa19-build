# Makefile to generate working environment for Ops

# environment
SHELL := /bin/bash
UNAME := $(shell uname | tr '[:upper:]' '[:lower:]')

# venv settings
ACTIVATE_BIN := venv/bin/activate

# JQ commandline JSON processor
JQ_BIN := venv/bin/jq
JQ_URL := https://github.com/stedolan/jq/releases/download
JQ_VER := 1.6
ifeq ($(UNAME),darwin)
  JQ_BIN_FILENAME := jq-osx-amd64
endif
ifeq ($(UNAME),linux)
  JQ_BIN_FILENAME := jq-linux64
endif

all: $(ACTIVATE_BIN) pip_requirements $(JQ_BIN)

jq: $(JQ_BIN)

clean:
	$(RM) -r venv
	find . -name "*.pyc" -exec $(RM) -rf {} \;

$(ACTIVATE_BIN):
	virtualenv venv
	chmod +x $@

pip_requirements: $(ACTIVATE_BIN) requirements.txt
	. venv/bin/activate; PYTHONWARNINGS='ignore:DEPRECATION' pip install -r requirements.txt


Makefile:


# The Mac OSX recommended method to install is via Homebrew
# but we'll install via curl so it stays in the venv
$(JQ_BIN): Makefile venv
	curl -L -o /tmp/$(@F) $(JQ_URL)/jq-$(JQ_VER)/$(JQ_BIN_FILENAME)
	mv -f /tmp/$(@F) $@
	chmod +x $@
	touch $@
