SHELL       := /usr/bin/env bash
.SHELLFLAGS := -euo pipefail -c

# CONFIG is the name of the make target someone
# would invoke to update the main config file (config.yml).
CONFIG ?= ci-config
# VERIFY is the name of the make target someone
# would invoke to verify the config file.
VERIFY ?= ci-verify

CIRCLECI := circleci --skip-update-check

CCI_INSTALL_LINK := https://circleci.com/docs/2.0/local-cli/\#installation
CCI_INSTALL_MSG := Please install CircleCI CLI. See $(CCI_INSTALL_LINK)
CCI_VERSION := $(shell $(CIRCLECI) version 2> /dev/null)
ifeq ($(CCI_VERSION),)
# Attempting to use the CLI fails with installation instructions.
CIRCLECI := echo '$(CCI_INSTALL_MSG)'; exit 1; \#
endif

SOURCE_DIR     := config
SOURCE_YML     := $(shell [ ! -d $(SOURCE_DIR) ] || find $(SOURCE_DIR) -name '*.yml')
CONFIG_SOURCE  := Makefile $(SOURCE_YML) | $(SOURCE_DIR)
OUT            := config.yml
TMP            := .tmp/config-processed
CONFIG_PACKED  := .tmp/config-packed

default: help

help:
	@echo "Usage:"
	@echo "  make $(CONFIG): recompile config.yml from $(SOURCE_DIR)/"
	@echo "  make $(VERIFY): verify that config.yml is a true mapping from $(SOURCE_DIR)/"
	@echo
	@echo "Diagnostics:"
	@[ -z "$(CCI_VERSION)" ] || echo "  circleci-cli version $(CCI_VERSION)"
	@[ -n "$(CCI_VERSION)" ] || echo "  $(CCI_INSTALL_MSG)"

$(SOURCE_DIR):
	@echo Source directory $(SOURCE_DIR)/ not found.; exit 1

# Make sure our .tmp dir exists.
$(shell [ -d .tmp ] || mkdir .tmp)

.PHONY: $(CONFIG)
$(CONFIG): $(OUT)

.PHONY: $(VERIFY)
$(VERIFY): config-up-to-date
	@$(CIRCLECI) config validate $(OUT)

GENERATED_FILE_HEADER := \#\#\# Generated by 'make $(CONFIG)' do not manually edit this file. 
define GEN_CONFIG
	@$(CIRCLECI) config pack $(SOURCE_DIR) > $(CONFIG_PACKED)
	@echo "$(GENERATED_FILE_HEADER)" > $@
	@$(CIRCLECI) config process $(CONFIG_PACKED) >> $@
endef

$(OUT): $(CONFIG_SOURCE) 
	$(GEN_CONFIG)
	@echo "$@ updated"

$(TMP): $(CONFIG_SOURCE) 
	$(GEN_CONFIG)

.PHONY: config-up-to-date
config-up-to-date: $(TMP) # Note this must not depend on $(OUT)!
	@if diff config.yml $<; then \
		echo "Generated $(OUT) is up to date!"; \
	else \
		echo "Generated $(OUT) is out of date, run make $(CONFIG) to update."; \
		exit 1; \
	fi
