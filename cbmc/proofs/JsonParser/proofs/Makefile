SHELL=/bin/bash

default: report

LEN = 10
KEYLEN = 10

UNWINDING = \
	--unwind $(LEN) \

ROOT = $(abspath ..)

OBJS = \
	$(ENTRY)_harness.goto \
	stub.goto \
	$(ROOT)/jsonparser.goto \

DEF = -DCBMC -DLEN=$(LEN) -DKEYLEN=$(KEYLEN) \

ifeq ($(ENTRY), Hash)
DEF += \
	-DSKIP_COLON_STUB \
	-DSKIP_COMMA_STUB \
	-DSKIP_WHITESPACE_STUB \
	-DSKIP_STRING_STUB \
	-DSKIP_INTEGER_STUB \
	-DSKIP_LIST_STUB \
	-DSKIP_VALUE_STUB \

endif

ifeq ($(ENTRY), List)
DEF += \
	-DSKIP_COLON_STUB \
	-DSKIP_COMMA_STUB \
	-DSKIP_WHITESPACE_STUB \
	-DSKIP_STRING_STUB \
	-DSKIP_INTEGER_STUB \
	-DSKIP_HASH_STUB \
	-DSKIP_VALUE_STUB \

endif

ifeq ($(ENTRY), Value)
DEF += \
	-DSKIP_COLON_STUB \
	-DSKIP_COMMA_STUB \
	-DSKIP_WHITESPACE_STUB \
	-DSKIP_STRING_STUB \
	-DSKIP_INTEGER_STUB \
	-DSKIP_HASH_STUB \
	-DSKIP_LIST_STUB \

endif

ifeq ($(ENTRY), Lookup)
DEF += \
	-DSKIP_COLON_STUB \
	-DSKIP_COMMA_STUB \
	-DSKIP_WHITESPACE_STUB \
	-DSKIP_STRING_STUB \
	-DSKIP_INTEGER_STUB \
	-DSKIP_HASH_STUB \
	-DSKIP_LIST_STUB \
	-DSKIP_VALUE_STUB \

endif

INC = -I$(ROOT)

CFLAGS = $(INC) $(DEF)

################################################################
# Define source location and cbmc binaries

GOTO_CC ?= goto-cc
GOTO_INSTRUMENT ?= goto-instrument
GOTO_ANALYZER ?= goto-analyzer
BATCH ?= cbmc-batch
VIEWER ?= cbmc-viewer

%.goto : %.c
	$(GOTO_CC) -o $@ $(CFLAGS) $<

$(ENTRY).goto: $(OBJS)
	$(GOTO_CC) -o $@ $(OBJS)

################################################################
# Run cbmc and build html report

CBMCFLAGS += \
	$(UNWINDING) \
	--bounds-check \
	--pointer-check \
	--unwinding-assertions \
	\
	--nondet-static \
	--div-by-zero-check \
	--float-overflow-check \
	--nan-check \
	--pointer-overflow-check \
	--undefined-shift-check \
	--signed-overflow-check \
	--unsigned-overflow-check \

goto: $(ENTRY).goto

cbmc.txt: $(ENTRY).goto
	cbmc $(CBMCFLAGS) --trace $< 2>&1 | tee $@

property.xml: $(ENTRY).goto
	cbmc $(CBMCFLAGS) --show-properties --xml-ui $< 2>&1 > $@

coverage.xml: $(ENTRY).goto
	cbmc $(filter-out --unwinding-assertions,$(CBMCFLAGS)) \
		--cover location --xml-ui $< 2>&1 > $@

cbmc: cbmc.txt

property: property.xml

coverage: coverage.xml

report: cbmc.txt property.xml coverage.xml
	$(VIEWER) \
	--goto $(ENTRY).goto \
	--srcdir $(ROOT) \
	--blddir $(ROOT) \
	--htmldir html \
	--result cbmc.txt \
	--property property.xml \
	--block coverage.xml

clean:
	$(RM) $(OBJS) $(ENTRY).goto
	$(RM) $(ENTRY)[0-9].goto $(ENTRY)[0-9].txt
	$(RM) cbmc.txt property.xml coverage.xml TAGS
	$(RM) *~ \#*

veryclean: clean
	$(RM) -r html

.PHONY: cbmc property coverage report clean veryclean

