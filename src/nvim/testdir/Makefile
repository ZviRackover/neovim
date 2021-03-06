# vim: noet ts=8
# Makefile to run all tests for Vim
#

ifeq ($(OS),Windows_NT)
  NVIM_PRG ?= ../../../build/bin/nvim.exe
else
  NVIM_PRG ?= ../../../build/bin/nvim
endif
ROOT := ../../..

export SHELL := sh
export NVIM_PRG := $(NVIM_PRG)
export TMPDIR := $(abspath ../../../Xtest-tmpdir)

SCRIPTS_DEFAULT = \
	test14.out             \
	test24.out             \
	test37.out             \
	test42.out             \
	test48.out             \
	test52.out             \
	test64.out             \

ifneq ($(OS),Windows_NT)
  SCRIPTS_DEFAULTS := $(SCRIPTS_DEFAULT) \
	test17.out             \
	test49.out             \

endif

SCRIPTS ?= $(SCRIPTS_DEFAULT)

# Tests using runtest.vim.
NEW_TESTS_ALOT := test_alot_utf8 test_alot
NEW_TESTS_IN_ALOT := $(shell sed '/^source/ s/^source //;s/\.vim$$//' test_alot*.vim)
# Ignored tests.
# test_alot_latin1: Nvim does not allow setting encoding.
# test_arglist: ported to Lua, but kept for easier merging.
# test_autochdir: ported to Lua, but kept for easier merging.
# test_eval_func: used as include in old-style test (test_eval.in).
# test_listlbr: Nvim does not allow setting encoding.
# test_largefile: uses too much resources to run on CI.
NEW_TESTS_IGNORE := $(NEW_TESTS_IN_ALOT) $(NEW_TESTS_ALOT) \
  test_alot_latin \
  test_arglist \
  test_autochdir \
  test_eval_func \
  test_listlbr \
  test_largefile \

NEW_TESTS = $(addsuffix .res,$(sort $(filter-out $(NEW_TESTS_IGNORE),$(basename $(notdir $(wildcard test_*.vim))))) $(NEW_TESTS_ALOT))

SCRIPTS_GUI := test16.out


ifdef VALGRIND_GDB
	VGDB := --vgdb=yes     \
	        --vgdb-error=0
endif

ifdef USE_VALGRIND
	VALGRIND_TOOL := --tool=memcheck     \
	                 --leak-check=yes    \
	                 --track-origins=yes
#        VALGRIND_TOOL := exp-sgcheck
	TOOL := valgrind -q                                     \
	                 -q                                     \
	                 $(VALGRIND_TOOL)                       \
	                 --suppressions=../../.valgrind.supp \
	                 --error-exitcode=123                   \
	                 --log-file=valgrind-\%p.$*             \
	                 $(VGDB)                                \
	                 --trace-children=yes
else
	ifdef USE_GDB
		TOOL = gdb --args
	endif
endif

ifdef TESTNUM
	SCRIPTS := test$(TESTNUM).out
endif

nongui: nolog $(SCRIPTS) newtests report

gui:    nolog $(SCRIPTS) $(SCRIPTS_GUI) newtests report

.gdbinit:
	@echo "[OLDTEST-PREP] Setting up .gdbinit"
	@echo 'set $$_exitcode = -1\nrun\nif $$_exitcode != -1\n  quit\nend' > .gdbinit

report:
	@echo
	@echo 'Test results:'
	@/bin/sh -c "if test -f test.log; then \
	                 cat test.log;         \
	                 echo TEST FAILURE;    \
	                 exit 1;               \
	             else                      \
	                 echo ALL DONE;        \
	             fi"

test1.out: $(NVIM_PRG)

$(SCRIPTS) $(SCRIPTS_GUI): $(NVIM_PRG) test1.out

RM_ON_RUN   := test.out X* viminfo
RM_ON_START := test.ok
RUN_VIM     := $(TOOL) $(NVIM_PRG) -u unix.vim -U NONE -i viminfo --headless --noplugin -s dotest.in

clean:
	-rm -rf *.out          \
	        *.failed       \
	        *.res          \
	        *.rej          \
	        *.orig         \
	        *.tlog         \
	        test.log       \
	        messages       \
	        $(RM_ON_RUN)   \
	        $(RM_ON_START) \
	        valgrind.*     \
	        .*.swp         \
	        .*.swo         \
	        .gdbinit       \
	        $(TMPDIR)      \
	        del

test1.out: .gdbinit test1.in
	@echo "[OLDTEST-PREP] Running test1"
	@rm -rf $*.failed $(RM_ON_RUN) $(RM_ON_START) wrongtermsize
	@mkdir -p $(TMPDIR)
	@/bin/sh runnvim.sh $(ROOT) $(NVIM_PRG) $* $(RUN_VIM) $*.in
	@rm -f wrongtermsize
	@rm -rf X* viminfo

%.out: %.in .gdbinit
	@echo "[OLDESTTEST] Running" $*
	@rm -rf $*.failed test.ok $(RM_ON_RUN)
	@mkdir -p $(TMPDIR)
	@cp $*.ok test.ok
	@/bin/sh runnvim.sh --oldesttest $(ROOT) $(NVIM_PRG) $* $(RUN_VIM) $*.in
	@rm -rf X* test.ok viminfo

# Explicit dependencies.
test49.out: test49.vim

nolog:
	@echo "[OLDTEST-PREP] Removing test.log and messages"
	@rm -f test.log messages


# New style of tests uses Vim script with assert calls.  These are easier
# to write and a lot easier to read and debug.
# Limitation: Only works with the +eval feature.
RUN_VIMTEST = $(TOOL) $(NVIM_PRG) -u unix.vim -U NONE --headless --noplugin

newtests: newtestssilent
	@/bin/sh -c "if test -f messages && grep -q 'FAILED' messages; then \
	                 cat messages && cat test.log;                      \
	             fi"

newtestssilent: $(NEW_TESTS)

%.res: %.vim .gdbinit
	@echo "[OLDTEST] Running" $*
	@mkdir -p $(TMPDIR)
	@/bin/sh runnvim.sh $(ROOT) $(NVIM_PRG) $* $(RUN_VIMTEST) -u NONE -S runtest.vim $*.vim
