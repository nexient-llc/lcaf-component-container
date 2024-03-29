SHELL := /bin/bash

# Binaries

GO ?= go
GOLANGCI_LINT ?= golangci-lint
TEE ?= tee
GREP ?= grep
FIND ?= find

# Variables

GO_TEST_DIRECTORIES ?= tests
GO_TEST_TIMEOUT ?= 2h
GOLANGCI_LINT_CONFIG ?= .golangci.yaml
DISABLE_MAKE_CHECK_LINT ?= false

# Functions

# Checks for Go files in the GO_TEST_DIRECTORIES. If they exist, runs the default configuration for golangci-lint
# https://golangci-lint.run/usage/quick-start/
define go_lint
	$(FIND) $(1)/ -name '*.go' | $(GREP) -q '\.go' || exit 0; $(GOLANGCI_LINT) run -c $(GOLANGCI_LINT_CONFIG) -v ./$(1)/...;

endef

# Check for Go files. If they exist, run tests.
define go_test
	$(FIND) $(1)/ -name '*.go' | $(GREP) -q '\.go' || exit 0; $(GO) test -v -timeout=$(GO_TEST_TIMEOUT) ./$(1)/...;

endef

# Tasks

.PHONY: go/lint
go/lint :
	$(foreach test_dir,$(GO_TEST_DIRECTORIES),$(call go_lint,$(test_dir)))

.PHONY: go/test
go/test :
	$(foreach test_dir,$(GO_TEST_DIRECTORIES),$(call go_test,$(test_dir)))


.PHONY: lint
lint::
ifeq ($(DISABLE_MAKE_CHECK_LINT),false)
	$(MAKE) go/lint
else
	$(info "make go/lint has been disabled!")
endif

.PHONY: test
test:: tfmodule/plan
	$(MAKE) go/test
