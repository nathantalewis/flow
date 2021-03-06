# Copyright (c) 2013-present, Facebook, Inc.
#
# This source code is licensed under the MIT license found in the
# LICENSE file in the root directory of this source tree.

DIR:=$(dir $(abspath $(lastword $(MAKEFILE_LIST))))
TOP=$(DIR)/../..
REL_DIR=src/parser

OCB = ocamlbuild -use-ocamlfind

NATIVE_OBJECT_FILES=\
	hack/utils/sys/files.o\
	hack/utils/sys/nproc.o\
	hack/utils/sys/realpath.o\
	hack/utils/sys/sysinfo.o\
	hack/utils/sys/processor_info.o\
	hack/utils/sys/priorities.o

RUNNER_DEPS=\
	hack/heap\
	hack/injection/default_injector\
	hack/utils\
	hack/utils/collections\
	hack/utils/disk\
	hack/utils/hh_json\
	hack/utils/sys\
	hack/third-party/core\
	src/common/utils\
	src/parser\
	src/parser_utils

OCAML_PATH=$(shell ocamlc -where)
OCAML_HEADERS=\
	$(sort $(patsubst $(OCAML_PATH)/%,dist/libflowparser/include/%, \
		$(filter $(OCAML_PATH)/%,$(shell \
			$(CC) -I $(OCAML_PATH) -MM -MT deps libflowparser.h \
	))))

all: build-parser

clean:
	$(OCB) -clean; \
	cd $(TOP); \
	$(OCB) -clean; \
	rm -f $(REL_DIR)/flow_parser.js; \
	rm -rf $(REL_DIR)/dist

build-parser:
	cd $(TOP); \
	$(OCB) -no-links $(REL_DIR)/parser_flow.cmxa

.PHONY: dist/libflowparser/lib/libflowparser.a
dist/libflowparser/lib/libflowparser.a:
	@mkdir -p "$(@D)"
	cp "$(OCAML_PATH)/libasmrun_pic.a" "$@"
	cd $(TOP) && $(OCB) -no-links -tag "runtime_variant(_pic)" \
		$(REL_DIR)/libflowparser.native.o
	ar rcs "$@" "$(TOP)/_build/$(REL_DIR)/libflowparser.native.o"

$(OCAML_HEADERS): dist/libflowparser/include/%: $(OCAML_PATH)/%
	@mkdir -p "$(@D)"
	cp "$<" "$@"

dist/libflowparser/include/flowparser/libflowparser.h: libflowparser.h
	@mkdir -p "$(@D)"
	cp "$<" "$@"

dist/libflowparser.zip: \
		$(OCAML_HEADERS) \
		dist/libflowparser/include/flowparser/libflowparser.h \
		dist/libflowparser/lib/libflowparser.a
	cd dist && zip -r $(@F) libflowparser

js:
	cd $(TOP); \
	$(OCB) -pkgs js_of_ocaml $(REL_DIR)/flow_parser_dot_js.byte; \
	js_of_ocaml --opt 3 \
		-o $(REL_DIR)/flow_parser.js \
		_build/$(REL_DIR)/flow_parser_dot_js.byte; \
	rm flow_parser_dot_js.byte

test-js: js
	cd $(TOP)/packages/flow-parser; npm test

../../_build/$(REL_DIR)/test/run_esprima_tests.native: build-parser
	cd $(TOP); \
	$(OCB) \
		-ocamlc "ocamlopt" \
		$(NATIVE_OBJECT_FILES); \
	$(OCB) \
		$(foreach dir,$(RUNNER_DEPS),-I $(dir)) \
	 	-lib unix -lib str \
	 	-lflags "$(NATIVE_OBJECT_FILES)" \
		$(REL_DIR)/test/run_esprima_tests.native; \
	rm run_esprima_tests.native

../../_build/$(REL_DIR)/test/run_hardcoded_tests.native: build-parser
	cd $(TOP); \
	$(OCB) \
		-ocamlc "ocamlopt -ccopt -DNO_SQLITE3" \
		$(NATIVE_OBJECT_FILES); \
	$(OCB) \
		$(foreach dir,$(RUNNER_DEPS),-I $(dir)) \
	 	-lib unix -lib str \
	 	-lflags "$(NATIVE_OBJECT_FILES)" \
		-tag debug \
		$(REL_DIR)/test/run_hardcoded_tests.native; \
	rm run_hardcoded_tests.native

test-esprima-ocaml: ../../_build/$(REL_DIR)/test/run_esprima_tests.native
	cd $(TOP); \
	_build/$(REL_DIR)/test/run_esprima_tests.native $(REL_DIR)/test/esprima/

test-hardcoded-ocaml: ../../_build/$(REL_DIR)/test/run_esprima_tests.native
	cd $(TOP); \
	_build/$(REL_DIR)/test/run_esprima_tests.native $(REL_DIR)/test/flow/

test-ocaml: test-esprima-ocaml test-hardcoded-ocaml

test: test-js test-ocaml

ocamlfind-install:
	cd $(TOP); \
	$(OCB) $(REL_DIR)/parser_flow.cma $(REL_DIR)/parser_flow.cmxa; \
	ocamlfind install flow_parser $(REL_DIR)/META \
		_build/$(REL_DIR)/parser_flow.a \
		_build/$(REL_DIR)/parser_flow.cma \
		_build/$(REL_DIR)/parser_flow.cmxa \
		_build/$(REL_DIR)/*.cmi

lexer.native: lexer.ml
	$(OCB) $@
