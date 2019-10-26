# Makefile to generate working environment for Ops

# environment
SHELL := /bin/bash
UNAME := $(shell uname | tr '[:upper:]' '[:lower:]')

# venv settings
ACTIVATE_BIN := venv/bin/activate

# AWS authenticator
# see: https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html
#      https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
AWS_AUTH_BIN := venv/bin/aws-iam-authenticator
KUBECTL_BIN := venv/bin/kubectl
AMAZON_URL := https://amazon-eks.s3-us-west-2.amazonaws.com
AMAZON_VER := 1.11.9/2019-03-27

# helm management tool
# see: https://helm.sh/docs/using_helm/#installing-helm
HELM_BIN := venv/bin/helm
HELM_URL := https://storage.googleapis.com/kubernetes-helm
HELM_VER := v2.12.2

# SAML tool
# see: https://github.com/Versent/saml2aws/
SAML_BIN := venv/bin/saml2aws
SAML2AWS_URL := https://github.com/Versent/saml2aws/releases/download
SAML2AWS_VER := 2.14.0

# One password tool
# see: https://support.1password.com/command-line-getting-started/
1PW_BIN := venv/bin/op
1PW_URL := https://cache.agilebits.com/dist/1P/op/pkg
1PW_VER := v0.5.6
1PW_GPGKEY := 3FEF9748469ADBE15DA7CA80AC2D62742012EA22

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

all: $(ACTIVATE_BIN) pip_requirements $(AWS_AUTH_BIN) $(HELM_BIN) $(KUBECTL_BIN) $(SAML_BIN) $(1PW_BIN) $(JQ_BIN)

jq: $(JQ_BIN)

clean:
	$(RM) -r venv
	find . -name "*.pyc" -exec $(RM) -rf {} \;

$(ACTIVATE_BIN):
	virtualenv venv
	chmod +x $@

pip_requirements: $(ACTIVATE_BIN) requirements.txt
	. venv/bin/activate; PYTHONWARNINGS='ignore:DEPRECATION' pip install -r requirements.txt

gilt: venv
	venv/bin/gilt overlay

Makefile:

$(AWS_AUTH_BIN): Makefile venv
	curl -o /tmp/$(@F) $(AMAZON_URL)/$(AMAZON_VER)/bin/$(UNAME)/amd64/$(@F)
	curl -o /tmp/$(@F).sha256 $(AMAZON_URL)/$(AMAZON_VER)/bin/$(UNAME)/amd64/$@.sha256
	openssl sha1 -sha256 /tmp/$(@F)
	mv -f /tmp/$(@F) $@
	chmod +x $@
	touch $@
	$(RM) /tmp/$(@F).sha256

$(HELM_BIN): Makefile venv
	curl -o /tmp/$(@F).tar.gz $(HELM_URL)/helm-$(HELM_VER)-$(UNAME)-amd64.tar.gz
	curl -o /tmp/$(@F).tar.gz.sha256 $(HELM_URL)/helm-$(HELM_VER)-$(UNAME)-amd64.tar.gz.sha256
	openssl sha1 -sha256 /tmp/$(@F).tar.gz
	tar zxvf /tmp/$(@F).tar.gz -C /tmp
	mv -f /tmp/$(UNAME)-amd64/helm $@
	touch $@
	$(RM) -r /tmp/$(@F).tar.gz* /tmp/$(UNAME)-amd64

$(KUBECTL_BIN): Makefile venv
	curl -o /tmp/$(@F) $(AMAZON_URL)/$(AMAZON_VER)/bin/$(UNAME)/amd64/$(@F)
	curl -o /tmp/$(@F).sha256 $(AMAZON_URL)/$(AMAZON_VER)/bin/$(UNAME)/amd64/$(@F).sha256
	openssl sha1 -sha256 /tmp/$(@F)
	mv -f /tmp/$(@F) $@
	chmod +x $@
	touch $@
	$(RM) /tmp/$(@F).sha256

$(SAML_BIN): Makefile venv
	curl -Lo /tmp/$(@F).tar.gz $(SAML2AWS_URL)/v$(SAML2AWS_VER)/$(@F)_$(SAML2AWS_VER)_$(UNAME)_amd64.tar.gz
	curl -Lo /tmp/$(@F).tar.gz.sha512 $(SAML2AWS_URL)/v$(SAML2AWS_VER)/$(@F)_$(SAML2AWS_VER)_$(UNAME)_amd64.tar.gz.sha512
	openssl sha1 -sha512 /tmp/$(@F).tar.gz
	tar zxvf /tmp/$(@F).tar.gz -C /tmp
	mv -f /tmp/$(@F) $@
	chmod +x $@
	touch $@
	$(RM) -r /tmp/$(@F).tar.gz* $(UNAME)-amd64

# this is in blocks as make can't handle long in-line shell
# commands.
gpg:
	@if ! which gpg && ! [ '$(UNAME)' == 'darwin' -o '$(UNAME)' == 'linux']; then	\
		echo '$(UNAME) is an unsupported OS '; \
		echo ' - install the gnupg package from https://gnupg.org/download/index.html.'; \
		exit 1; \
	fi
	@if ! which gpg && [ '$(UNAME)' == 'darwin' ]; then \
		if [ -x /usr/local/bin/brew ]; then  \
			echo "installing gnupg via brew"; \
			brew install gpg; \
		else \
			echo "Please either install one of: "; \
			echo " - brew ( https://brew.sh ) "; \
			echo " - GPGSuite ( https://gpgtools.org/ ) "; \
			echo " and then rerun make."; \
			exit 1; \
		fi; \
	fi
	@if ! which gpg && [ '$(UNAME)' == 'linux' ]; then \
		if [ -x /usr/bin/apt-get ]; then \
			echo "installing gnupg via apt-get"; \
			sudo apt-get install gnupg; \
		elif [ -x /usr/bin/yum ]; then \
			echo "installing gnupg via yum"; \
			sudo yum install gnupg; \
		fi; \
	fi

# gnupg current source code handles both --recv-keys and --receive-keys
# so we're going with the older one for now .. but it's probably going
# to need to be fixed at some point
$(1PW_BIN): Makefile venv gpg
	curl -f -o /tmp/$(@F).zip $(1PW_URL)/$(1PW_VER)/op_$(UNAME)_amd64_$(1PW_VER).zip
	unzip -o /tmp/$(@F).zip -d /tmp
	gpg --recv-keys $(1PW_GPGKEY)
	gpg --verify /tmp/$(@F).sig /tmp/$(@F)
	mv -f /tmp/$(@F) $@
	chmod +x $@
	touch $@
	$(RM) -f /tmp/$(@F).*

# The Mac OSX recommended method to install is via Homebrew
# but we'll install via curl so it stays in the venv
$(JQ_BIN): Makefile venv
	curl -L -o /tmp/$(@F) $(JQ_URL)/jq-$(JQ_VER)/$(JQ_BIN_FILENAME)
	mv -f /tmp/$(@F) $@
	chmod +x $@
	touch $@
