ARCH = ARCH=riscv64-nutshell
NANOS_HOME ?= $(AM_HOME)/../nanos-lite
SINGLETEST = ALL=recursion
RVTEST_TARGET = towers

CPU_TEST_RESULT = $(shell cat cpu.log | grep -E "different|stuck")
MB_RESULT = $(shell cat microbench.log | grep IPC)
MBT_RESULT = $(shell cat microbench-train.log | grep IPC)
CM_RESULT = $(shell cat coremark.log | grep IPC)

# ------------------------------------------------------------------
# bulid CPU and run dummy test
# ------------------------------------------------------------------

cpu:
	$(MAKE) -C $(AM_HOME)/tests/cputest $(ARCH) ALL=dummy run

# ------------------------------------------------------------------
# unit tests
# ------------------------------------------------------------------

cputest:
	$(MAKE) -C $(AM_HOME)/tests/cputest $(ARCH) run 2>&1 | tee > cpu.log
	# cat cpu.log | grep different
ifeq ($(CPU_TEST_RESULT),)
	@echo "simple CPU test passed"
else
	@echo $(CPU_TEST_RESULT)
	@echo "simple CPU test failed"
endif

bputest:
	$(MAKE) -C $(AM_HOME)/tests/bputest $(ARCH) ALL=forcall2 run 2>&1 | tee > bpu.log
	cat bpu.log | grep Mbp

unittest:
	$(MAKE) -C $(AM_HOME)/tests/cputest $(ARCH) $(SINGLETEST) run 2>&1 | tee > test.log
	cat test.log | grep -E "different|stuck|GOOD"
	cat test.log | grep ISU > isu.log

microbench:
	$(MAKE) -C $(AM_HOME)/apps/microbench $(ARCH) mainargs=test run 2>&1 | tee > microbench.log
	cat microbench.log | grep IPC
	@date >> history.log
	@git log --pretty=format:"%h %an %ad %s" -1 >> history.log
	@echo -e "" >> history.log
	@echo $(MB_RESULT) >> history.log
	@echo -e "" >> history.log

microbench-train:
	$(MAKE) -C $(AM_HOME)/apps/microbench $(ARCH) mainargs=train run 2>&1 | tee > microbench-train.log
	cat microbench-train.log | grep IPC

coremark:
	$(MAKE) -C $(AM_HOME)/apps/coremark $(ARCH) mainargs=test run 2>&1 | tee > coremark.log
	cat coremark.log | grep IPC
	@date >> cm_history.log
	@git log --pretty=format:"%h %an %ad %s" -1 >> cm_history.log
	@echo -e "" >> cm_history.log
	@echo $(CM_RESULT) >> cm_history.log
	@echo -e "" >> cm_history.log


dhrystone:
	$(MAKE) -C $(AM_HOME)/apps/dhrystone $(ARCH) mainargs=test run 2>&1 | tee > dhrystone.log
	cat dhrystone.log | grep IPC

rvtest:
	$(MAKE) -C $(AM_HOME)/tests/riscv-test/benchmarks/$(RVTEST_TARGET) $(ARCH) run
	# usage: make rvtest RVTEST_TARGET=dhrystone

xj:
	$(MAKE) -C $(NANOS_HOME) $(ARCH) run

xjnemu:
	$(MAKE) -C $(NANOS_HOME) ARCH=riscv64-nemu run

rttos:
	$(MAKE) -C $(RTTOS_HOME)/bsp/riscv64-nutshell run

rttos-debug:
	$(MAKE) -C $(RTTOS_HOME)/bsp/riscv64-nutshell run 2>&1 | tee > rttos.log

freertos:
	$(MAKE) -C $(FREERTOS_HOME)/Demo/riscv64-nutshell nutshell_run

xv6:
	$(MAKE) -C $(XV6_HOME) nutshell

xv6-debug:
	$(MAKE) -C $(XV6_HOME) nutshell 2>&1 | tee > xv6.log

linux:
	$(MAKE) -C $(BBL_LINUX_HOME) nutshell

linux-debug:
	$(MAKE) -C $(BBL_LINUX_HOME) nutshell 2>&1 | tee > linux.log

# ------------------------------------------------------------------
# test set
# ------------------------------------------------------------------

smoke-test: cpu cputest microbench
	@echo "Smoke Test Passed"

regression-test: smoke-test linux
	@echo "Regression Test Passed"

perf-test: smoke-test microbench-train coremark
	@echo "Perf Test Finished"

# ------------------------------------------------------------------
# get disassembled test src
# ------------------------------------------------------------------

disassemble-rttos:
	cp $(RTTOS_HOME)/bsp/riscv64-nutshell/build/code.txt ./d-rttos.log

disassemble-freertos:
	cp $(FREERTOS_HOME)/Demo/riscv64-nutshell/build/FreeRTOS-simple.elf.txt ./d-freertos.log

disassemble-xv6:
	cp $(XV6_HOME)/build/code.txt ./d-xv6.log

