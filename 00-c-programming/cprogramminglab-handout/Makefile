#
# Makefile for cprogramminglab
#

# Windows GNU Make cannot run this lab: clang is not installed, and the
# test harness needs POSIX headers (sys/select.h, alarm). Rebuild in WSL.
ifeq ($(OS),Windows_NT)

.PHONY: wsl-make
wsl-make:
	wsl --cd "$(CURDIR)" make $(MAKECMDGOALS)

# An empty recipe stops Make from rebuilding these files with the
# catch-all rule below.
Makefile helper.mk: ;

.DEFAULT_GOAL := wsl-make

# Explicit goals such as "test" and "clean" are forwarded above.
% : wsl-make
	@:

else

# Path to LLVM when running on autograder and shark cluster
ifneq (,$(wildcard /usr/lib/llvm-7/bin/))
  LLVM_PATH = /usr/lib/llvm-7/bin/
else
  ifneq (,$(wildcard /usr/local/depot/llvm-7.0/bin/))
    LLVM_PATH = /usr/local/depot/llvm-7.0/bin/
  endif
endif

# Path to ASan runtime when running on autograder and shark cluster
SHARK_LLVM_RBASE = /afs/cs.cmu.edu/academic/class/15213/lib/clang
ifneq (,$(LLVM_PATH))
  ifeq (,$(wildcard $(LLVM_PATH)../lib/clang/7.*/lib/linux/libclang_rt.asan*.a))
    ifneq (,$(wildcard $(SHARK_LLVM_RBASE)/7.*/lib/linux/libclang_rt.asan*.a))
      LLVM_RSRC_DIR = -resource-dir $(wildcard $(SHARK_LLVM_RBASE)/7.*)
    else
      ifneq (,$(wildcard ./clang/lib/linux/libclang_rt.asan*.a))
        LLVM_RSRC_DIR = -resource-dir $(abspath ./clang)
      endif
    endif
  endif
endif

# The course machines provide clang. A local Linux install may only have gcc;
# both accept the warning and sanitizer flags used below.
ifeq ($(LLVM_PATH),)
  CC := $(shell command -v clang 2>/dev/null || command -v gcc)
  CLANG_FORMAT := $(shell command -v clang-format 2>/dev/null || echo $(HOME)/.local/bin/clang-format)
else
  CC := $(LLVM_PATH)clang
  CLANG_FORMAT := $(LLVM_PATH)clang-format
endif
CFLAGS = -std=c99 -Og -g -Wall -Wextra -Wpedantic -Wconversion
CFLAGS += -Wstrict-prototypes -Wwrite-strings -Werror

# Targets to compile
HANDIN_TAR = cprogramminglab-handin.tar
FILES = qtest $(HANDIN_TAR)

.PHONY: all
all: $(FILES)

# List header dependencies
qtest.o: harness.h queue.h report.h console.h
harness.o: harness.h
console.o: report.h console.h
report.o: report.h
queue.o: harness.h queue.h

# Compile object files
%.o: %.c
	$(CC) $(CFLAGS) -o $@ -c $<

# Compile qtest binaries
qtest: qtest.o report.o console.o harness.o queue.o
	$(CC) $(LDFLAGS) -o $@ $^ $(LDLIBS)

# Compile certain targets with ASan and UBSan
ASAN_TARGETS = qtest
$(ASAN_TARGETS): CFLAGS += -fsanitize=address,undefined
$(ASAN_TARGETS): LDFLAGS += -fsanitize=address,undefined $(LLVM_RSRC_DIR)

.PHONY: test
test: qtest driver.py
	chmod +x driver.py
	./driver.py

.PHONY: clean
clean:
	rm -rf $(FILES)
	rm -rf *.o *~ *.pyc *.dSYM
	(cd traces; rm -f *~)

# Include rules for submit, format, etc
FORMAT_FILES = queue.c queue.h
HANDIN_FILES = queue.c queue.h .clang-format
include helper.mk

# Add check-format dependencies
submit: | check-format
$(FILES): | check-format

endif
