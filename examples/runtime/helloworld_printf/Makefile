#
#MIT License 
#
#Copyright © 2016 Advanced Micro Devices, Inc.  
#
#Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software
#without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit
#persons to whom the Software is furnished to do so, subject to the following conditions:
#
#The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
#
#THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR
#PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR
#OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#
ATMI_RUNTIME_PATH ?= /opt/rocm/libatmi
ATMI_INC=${ATMI_RUNTIME_PATH}/include

#Set HSA Environment variables
HSA_RUNTIME_PATH ?= /opt/rocm

#Set LC Environment variables
AMDLLVM ?= /opt/amd/llvm

#MCPU
MCPU ?= $(shell mymcpu)

# Kernel format
BRIG ?= 0
ifeq ($(BRIG),1)
	BRIGCLOPTS=-brig
	BRIGCPPOPTS=-DUSE_BRIG
endif

# Kernel compiler
CLC ?= 1

# Temp files
SAVETEMP ?= 0

# CLOC
CLOC_PATH ?= /usr/bin
#CLOCOPTS = -vv
#CLOCOPTS += -clopts "-v"

# ROCm-Device-lib
BITCODE_LIB ?= ${ROCMLIB}/dist/lib

# GPU compiler
CLCC=$(AMDLLVM)/bin/clang

LLK=$(AMDLLVM)/bin/llvm-link
LLC=$(AMDLLVM)/bin/llc
LLD=$(AMDLLVM)/bin/lld

CLCFLAGS = -x cl -Xclang -cl-std=CL2.0 -Xclang -finclude-default-header
CLCFLAGS += -target amdgcn--amdhsa

ifeq ($(SAVETEMP),0)
CLCFLAGS += -mcpu=$(MCPU)

CLCFLAGS += -Xclang -mlink-bitcode-file -Xclang $(BITCODE_LIB)/opencl.amdgcn.bc
CLCFLAGS += -Xclang -mlink-bitcode-file -Xclang $(BITCODE_LIB)/ockl.amdgcn.bc
CLCFLAGS += -Xclang -mlink-bitcode-file -Xclang $(BITCODE_LIB)/ocml.amdgcn.bc
CLCFLAGS += -Xclang -mlink-bitcode-file -Xclang $(BITCODE_LIB)/irif.amdgcn.bc
else
CLCFLAGS += -emit-llvm -c

#LLFLAGS = -suppress-warnings
LLFLAGS += $(BITCODE_LIB)/opencl.amdgcn.bc
LLFLAGS += $(BITCODE_LIB)/ockl.amdgcn.bc
LLFLAGS += $(BITCODE_LIB)/ocml.amdgcn.bc
LLFLAGS += $(BITCODE_LIB)/irif.amdgcn.bc

LCFLAGS = -O2
LCFLAGS += -filetype=obj
LCFLAGS += -mtriple amdgcn--amdhsa
LCFLAGS += -mcpu=$(MCPU)

LDFLAGS = -flavor gnu -shared
endif


# CPU compiler
CC=$(AMDLLVM)/bin/clang
CCFLAGS = -g
#CCFLAGS += -v

# Host compiler
CXX = g++
CXXFLAGS = -g -std=c++11
#CXXFLAGS += -v

INC_FLAGS=-I${ATMI_INC} -I${HSA_RUNTIME_PATH}/include -I.

LIBS=-latmi_runtime
LIB_FLAGS=-L${ATMI_RUNTIME_PATH}/lib -L${HSA_RUNTIME_PATH}/lib

OBJS = hello

.PHONY: clean all

all: $(OBJS)

%.hsaco: %.cl $(INC_FILES)
ifeq ($(CLC),1)
	$(CLOC_PATH)/cloc.sh ${CLOCOPTS} ${BRIGCLOPTS} -opt 2 -o $@ $<
else
ifeq ($(SAVETEMP),0)
	$(CLCC) ${CLCFLAGS} -o $@ $<
else
	$(CLCC) ${CLCFLAGS} -o $*.bc $<
	$(LLK) -o $*.linked.bc $*.bc ${LLFLAGS}
	$(LLC) ${LCFLAGS} -o $*.o $*.linked.bc
	$(LLD) ${LDFLAGS} -o $@ $*.o
endif
endif
	@echo

%.o: %.cpp $(INC_FILES)
	$(CXX) $(CXXFLAGS) $(INC_FLAGS) -o $@ -c $<
	@echo

%.o: %.c $(INC_FILES)
	$(CC) $(CCFLAGS) $(INC_FLAGS) -o $@ -c $<
	@echo

hello: hw_gpu.hsaco hw_cpu.o hw_host.o
	$(CXX) -o $@ hw_host.o hw_cpu.o $(LIBS) $(LIB_FLAGS)

clean:
	rm -rf *.o *.brig *.i *.bc *.hsaco $(OBJS)

test:
	./hello
