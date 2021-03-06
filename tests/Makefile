# Top-level directory containing ableC repository and extensions folder
ABLEC_BASE?=../../../ableC
EXTS_BASE?=../../../extensions

# The jar file to built
ABLEC_JAR=ableC.jar
# The artifact specification grammar to compile
ARTIFACT=artifact
# All directories containing grammars that may be included
GRAMMAR_DIRS=$(wildcard $(EXTS_BASE)/*/grammars) $(ABLEC_BASE)/grammars
# All silver files in included grammars, to be included as dependancies
GRAMMAR_SOURCES=$(shell find $(GRAMMAR_DIRS) -name *.sv -print0 | xargs -0)
# Flags passed to silver including the appropriate directories
override SVFLAGS+=$(addprefix -I , $(GRAMMAR_DIRS))

# All extended C files to test
XC_FILES=$(wildcard translate_error/*.xc) $(wildcard runtime_error/*.xc) $(wildcard positive/*.xc)
# All extended C files that should translate successfully
TRANS_XC_FILES=$(wildcard runtime_error/*.xc) $(wildcard positive/*.xc)
# All C files that should be generated
C_FILES=$(TRANS_XC_FILES:.xc=.c)
# All object files that should be generated
OBJECTS=$(TRANS_XC_FILES:.xc=.o)
# All executables that should be generated
EXECUTABLES=$(TRANS_XC_FILES:.xc=.out)
# All evaluation marker files that should be generated
TESTS=$(XC_FILES:.xc=.test)

# All directories contining extension header files that may be included
XC_INCLUDE_DIRS=$(wildcard $(EXTS_BASE)/*/include)
# Flags passed to ableC including the appropriate directories
override XCFLAGS+=$(addprefix -I,$(XC_INCLUDE_DIRS))
# Disable comment stripping in the C preprocessor, so we can test layout
override XCFLAGS+=-C
# Flags passed to Java when invoking ableC
override JAVAFLAGS+=-Xss6M

# Flags passed to the C compiler, e.g. to enable various compiler extensions
override CFLAGS+=

# All directories contining extension libraries that may be linked
LIB_DIRS=$(wildcard $(EXTS_BASE)/*/lib)
# Flags passed to the linker specifying the appropriate directories
override LDFLAGS+=$(addprefix -L,$(LIB_DIRS))
# Flags passed to the linker specifying libraries to link
LDLIBS=-Wl,-Bstatic -lsqlite -Wl,-Bdynamic -lpthread -ldl

# All directories contining extension library sources
SRC_DIRS=$(EXTS_BASE)/ableC-sqlite/src
# All extension library targets
LIBS=$(SRC_DIRS:src=libs)
# All C files used to build libraries, to be included as dependencies
SRC_SOURCES=$(foreach dir,$(SRC_DIRS),$(wildcard $(dir)/*.c))

all: $(ABLEC_JAR) $(C_FILES) $(OBJECTS) $(EXECUTABLES) $(TESTS)

libs: $(LIBS)

$(LIBS):
	cd $(@:libs=src) && $(MAKE)

$(ABLEC_JAR): $(GRAMMAR_SOURCES)
	touch $(wildcard $(ARTIFACT)/*.sv)
	silver -o $(ABLEC_JAR) $(SVFLAGS) $(ARTIFACT)

%.c: %.xc $(ABLEC_JAR)
	java $(JAVAFLAGS) -jar $(ABLEC_JAR) $< $(XCFLAGS)

%.out: %.o $(SRC_SOURCES) | libs
	$(CC) $(LDFLAGS) $< $(LOADLIBES) $(LDLIBS) -o $@

translate_error/%.test: translate_error/%.xc $(ABLEC_JAR)
	@echo "java $(JAVAFLAGS) -jar $(ABLEC_JAR) $< $(XCFLAGS)"
	@if java $(JAVAFLAGS) -jar $(ABLEC_JAR) $< $(XCFLAGS); then echo "Failed to error"; exit 1; fi
	touch $@

runtime_error/%.test: runtime_error/%.out
	@echo "./$<"
	@if ./$<; then echo "Failed to error"; exit 2; else echo "(exit $$?)"; fi
	touch $@

%.test: %.out
	./$<
	touch $@

clean:
	rm -f *~ build.xml */*.i */*.c */*.o */*.out */*.test *.jar *.copperdump.html build.xml

.PHONY: all libs $(LIBS) clean
