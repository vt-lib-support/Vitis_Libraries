#
# Copyright 2019 Xilinx, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# -----------------------------------------------------------------------------
#                          project common settings

MK_PATH := $(abspath $(lastword $(MAKEFILE_LIST)))
CUR_DIR := $(patsubst %/,%,$(dir $(MK_PATH)))
XF_PROJ_ROOT ?= $(shell bash -c 'export MK_PATH=$(MK_PATH); echo $${MK_PATH%L2/tests/*}')

.SECONDEXPANSION:

# -----------------------------------------------------------------------------
#                            tool common setup

# MK_INC_BEGIN vitis_help.mk

.PHONY: help

help::
	@echo ""
	@echo "Makefile Usage:"
	@echo ""
	@echo "  make host xclbin TARGET=<sw_emu|hw_emu|hw> DEVICE=<FPGA platform>"
	@echo "      Command to generate the design for specified target and device."
	@echo ""
	@echo "      TARGET defaults to sw_emu."
	@echo ""
	@echo "      DEVICE is case-insensitive and support awk regex."
	@echo "      For example, \`make xclbin TARGET=hw DEVICE='u200.*qdma'\`"
	@echo "      It can also be an absolute path to platform file."
	@echo ""
	@echo "  make run TARGET=<sw_emu|hw_emu|hw> DEVICE=<FPGA platform>"
	@echo "      Command to run application in emulation."
	@echo ""
	@echo "  make clean "
	@echo "      Command to remove the generated non-hardware files."
	@echo ""
	@echo "  make cleanall"
	@echo "      Command to remove all the generated files."
	@echo ""

# MK_INC_END vitis_help.mk

# MK_INC_BEGIN vivado.mk


TOOL_VERSION ?= 2019.2

ifeq (,$(XILINX_VIVADO))
XILINX_VIVADO = /opt/xilinx/Vivado/$(TOOL_VERSION)
endif
export XILINX_VIVADO

.PHONY: check_vivado
check_vivado:
ifeq (,$(wildcard $(XILINX_VIVADO)/bin/vivado))
	@echo "Cannot locate Vivado installation. Please set XILINX_VIVADO variable." && false
endif

export PATH := $(XILINX_VIVADO)/bin:$(PATH)

# MK_INC_END vivado.mk

# MK_INC_BEGIN vitis.mk


TOOL_VERSION ?= 2019.2

ifeq (,$(XILINX_VITIS))
XILINX_VITIS = /opt/xilinx/Vitis/$(TOOL_VERSION)
endif
export XILINX_VITIS
.PHONY: check_vpp
check_vpp:
ifeq (,$(wildcard $(XILINX_VITIS)/bin/v++))
	@echo "Cannot locate Vitis installation. Please set XILINX_VITIS variable." && false
endif

ifeq (,$(XILINX_XRT))
XILINX_XRT = /opt/xilinx/xrt
endif
export XILINX_XRT
.PHONY: check_xrt
check_xrt:
ifeq (,$(wildcard $(XILINX_XRT)/lib/libxilinxopencl.so))
	@echo "Cannot locate XRT installation. Please set XILINX_XRT variable." && false
endif

export PATH := $(XILINX_VITIS)/bin:$(XILINX_XRT)/bin:$(PATH)

ifeq (,$(LD_LIBRARY_PATH))
LD_LIBRARY_PATH := $(XILINX_XRT)/lib
else
LD_LIBRARY_PATH := $(XILINX_XRT)/lib:$(LD_LIBRARY_PATH)
endif
ifneq (,$(wildcard $(XILINX_VITIS)/bin/ldlibpath.sh))
export LD_LIBRARY_PATH := $(shell $(XILINX_VITIS)/bin/ldlibpath.sh $(XILINX_VITIS)/lib/lnx64.o):$(LD_LIBRARY_PATH)
endif

# Target check
TARGET ?= sw_emu
ifeq ($(filter $(TARGET),sw_emu hw_emu hw),)
$(error TARGET is not sw_emu, hw_emu or hw)
endif

# MK_INC_END vitis.mk

# MK_INC_BEGIN vitis_set_platform.mk

ifneq (,$(wildcard $(DEVICE)))
# Use DEVICE as a file path
XPLATFORM := $(DEVICE)
else
# Use DEVICE as a file name pattern
DEVICE_L := $(shell echo $(DEVICE) | tr A-Z a-z)
# Match the name
ifneq (,$(PLATFORM_REPO_PATHS))
XPLATFORMS := $(foreach p, $(subst :, ,$(PLATFORM_REPO_PATHS)), $(wildcard $(p)/*/*.xpfm))
XPLATFORM := $(strip $(foreach p, $(XPLATFORMS), $(shell echo $(p) | awk '$$1 ~ /$(DEVICE_L)/')))
endif
ifeq (,$(XPLATFORM))
XPLATFORMS := $(wildcard $(XILINX_VITIS)/platforms/*/*.xpfm)
XPLATFORM := $(strip $(foreach p, $(XPLATFORMS), $(shell echo $(p) | awk '$$1 ~ /$(DEVICE_L)/')))
endif
ifeq (,$(XPLATFORM))
XPLATFORMS := $(wildcard /opt/xilinx/platforms/*/*.xpfm)
XPLATFORM := $(strip $(foreach p, $(XPLATFORMS), $(shell echo $(p) | awk '$$1 ~ /$(DEVICE_L)/')))
endif
endif

define MSG_PLATFORM
No platform matched pattern '$(DEVICE)'.
Available platforms are: $(XPLATFORMS)
To add more platform directories, set the PLATFORM_REPO_PATHS variable.
endef
export MSG_PLATFORM

define MSG_DEVICE
More than one platform matched: $(XPLATFORM)
Please set DEVICE variable more accurately to select only one platform file. For example: DEVICE='u200.*xdma'
endef
export MSG_DEVICE

.PHONY: check_platform
check_platform:
ifeq (,$(XPLATFORM))
	@echo "$${MSG_PLATFORM}" && false
endif
ifneq (,$(word 2,$(XPLATFORM)))
	@echo "$${MSG_DEVICE}" && false
endif

XDEVICE := $(basename $(notdir $(firstword $(XPLATFORM))))

# MK_INC_END vitis_set_platform.mk

# -----------------------------------------------------------------------------
# BEGIN_XF_MK_USER_SECTION
# -----------------------------------------------------------------------------
# TODO:                 data creation and other user targets

# a (typically hidden) file as stamp
DATA_STAMP :=
$(DATA_STAMP):
.PHONY: data
data: $(DATA_STAMP)

.PHONY: debug
debug:
	@echo "KERNELS are $(KERNELS)"
	@echo "> KERNEL_NAMES are $(KERNEL_NAMES)"
	@echo
	@echo "MCAE_k0_EXTRA_SRCS is $(MCAE_k0_EXTRA_SRCS)"
	@echo "MCAE_k0_EXTRA_HDRS is $(MCAE_k0_EXTRA_HDRS)"
	@echo "> MCAE_k0_SRCS is $(MCAE_k0_SRCS)"
	@echo "> MCAE_k0_HDRS is $(MCAE_k0_HDRS)"
	@echo
	@echo "MCAE_k1_EXTRA_SRCS is $(MCAE_k1_EXTRA_SRCS)"
	@echo "MCAE_k1_EXTRA_HDRS is $(MCAE_k1_EXTRA_HDRS)"
	@echo "> MCAE_k1_SRCS is $(MCAE_k1_SRCS)"
	@echo "> MCAE_k1_HDRS is $(MCAE_k1_HDRS)"
	@echo
	@echo "MCAE_k2_EXTRA_SRCS is $(MCAE_k2_EXTRA_SRCS)"
	@echo "MCAE_k2_EXTRA_HDRS is $(MCAE_k2_EXTRA_HDRS)"
	@echo "> MCAE_k2_SRCS is $(MCAE_k2_SRCS)"
	@echo "> MCAE_k2_HDRS is $(MCAE_k2_HDRS)"
	@echo
	@echo "MCAE_k3_EXTRA_SRCS is $(MCAE_k3_EXTRA_SRCS)"
	@echo "MCAE_k3_EXTRA_HDRS is $(MCAE_k3_EXTRA_HDRS)"
	@echo "> MCAE_k3_SRCS is $(MCAE_k3_SRCS)"
	@echo "> MCAE_k3_HDRS is $(MCAE_k3_HDRS)"
	@echo
	@echo "main_EXTRA_HDRS is $(main_EXTRA_HDRS)"
	@echo "> main_HDRS is $(main_HDRS)"

# -----------------------------------------------------------------------------
# TODO:                          kernel setup

XFLIB_DIR = $(XF_PROJ_ROOT)
KSRC_DIR = $(CUR_DIR)/kernel

HLS_DIR	= $(XF_PROJ_ROOT)/L2/include
HLS_DIR2 = $(XF_PROJ_ROOT)/L1/include

XCLBIN_NAME := MCAE_k
KERNEL = MCAE_k
KERNEL_IDS = 0 1 2

ifneq (,$(shell echo $(XPLATFORM) | awk '/u250/'))
   KERNEL_IDS = 0 1 2 3
endif
KERNELS = $(foreach id,$(KERNEL_IDS),$(KERNEL)$(id))

# must provide path
MCAE_k0_EXTRA_HDRS += $(KSRC_DIR)/MCAE_kernel.hpp $(wildcard $(HLS_DIR)/*.hpp) $(wildcard $(HLS_DIR2)/*.hpp)
MCAE_k1_EXTRA_HDRS += $(KSRC_DIR)/MCAE_kernel.hpp $(wildcard $(HLS_DIR)/*.hpp) $(wildcard $(HLS_DIR2)/*.hpp)
MCAE_k2_EXTRA_HDRS += $(KSRC_DIR)/MCAE_kernel.hpp $(wildcard $(HLS_DIR)/*.hpp) $(wildcard $(HLS_DIR2)/*.hpp)

VPP_CFLAGS += -I$(XFLIB_DIR)/L1/include -I$(XFLIB_DIR)/L2/include
MCAE_k0_VPP_CFLAGS += -D KERNEL0 -D KERNEL_NAME=MCAE_k0
MCAE_k1_VPP_CFLAGS += -D KERNEL1 -D KERNEL_NAME=MCAE_k1
MCAE_k2_VPP_CFLAGS += -D KERNEL2 -D KERNEL_NAME=MCAE_k2


ifneq ($(TARGET),sw_emu)
    CXXFLAGS += -DOUT_OF_ORDER
endif

DATATYPE ?= double
ifeq ($(DATATYPE),double)
    VPP_CFLAGS += -D DPRAGMA
endif

ifneq (,$(shell echo $(XPLATFORM) | awk '/u50/'))
    VPP_CFLAGS += --sp MCAE_k0.m_axi_gmem0:HBM[0]
    VPP_CFLAGS += --sp MCAE_k0.m_axi_gmem1:HBM[1]

    VPP_CFLAGS += --sp MCAE_k1.m_axi_gmem0:HBM[0]
    VPP_CFLAGS += --sp MCAE_k1.m_axi_gmem1:HBM[1]
    VPP_CFLAGS += --sp MCAE_k1.m_axi_gmem2:HBM[2]

    VPP_CFLAGS += --sp MCAE_k2.m_axi_gmem0:HBM[2]
    VPP_CFLAGS += --sp MCAE_k2.m_axi_gmem1:HBM[3]

    VPP_CFLAGS += --slr MCAE_k0:SLR0
    VPP_CFLAGS += --slr MCAE_k1:SLR0
    VPP_CFLAGS += --slr MCAE_k2:SLR1
# VCU1525
else ifneq (,$(shell echo $(XPLATFORM) | awk '/u250/'))
#  U250
    MCAE_k3_EXTRA_HDRS += $(KSRC_DIR)/MCAE_kernel.hpp $(wildcard $(HLS_DIR)/*.hpp) $(wildcard $(HLS_DIR2)/*.hpp)
    MCAE_k3_VPP_CFLAGS += -D KERNEL3 -D KERNEL_NAME=MCAE_k3

    VPP_CFLAGS += --sp MCAE_k0.m_axi_gmem0:bank0
    VPP_CFLAGS += --sp MCAE_k0.m_axi_gmem1:bank1

    VPP_CFLAGS += --sp MCAE_k1.m_axi_gmem0:bank0
    VPP_CFLAGS += --sp MCAE_k1.m_axi_gmem1:bank1
    VPP_CFLAGS += --sp MCAE_k1.m_axi_gmem2:bank2

    VPP_CFLAGS += --sp MCAE_k2.m_axi_gmem0:bank2
    VPP_CFLAGS += --sp MCAE_k2.m_axi_gmem1:bank3

    VPP_CFLAGS += --sp MCAE_k3.m_axi_gmem0:bank2
    VPP_CFLAGS += --sp MCAE_k3.m_axi_gmem1:bank3

    VPP_CFLAGS += --slr MCAE_k0:SLR0
    VPP_CFLAGS += --slr MCAE_k1:SLR1
    VPP_CFLAGS += --slr MCAE_k2:SLR2
    VPP_CFLAGS += --slr MCAE_k3:SLR3
else ifneq (,$(shell echo $(XPLATFORM) | awk '/u200/'))
    VPP_CFLAGS += --sp MCAE_k0.m_axi_gmem0:bank0
    VPP_CFLAGS += --sp MCAE_k0.m_axi_gmem1:bank1

    VPP_CFLAGS += --sp MCAE_k1.m_axi_gmem0:bank0
    VPP_CFLAGS += --sp MCAE_k1.m_axi_gmem1:bank1
    VPP_CFLAGS += --sp MCAE_k1.m_axi_gmem2:bank2

    VPP_CFLAGS += --sp MCAE_k2.m_axi_gmem0:bank2
    VPP_CFLAGS += --sp MCAE_k2.m_axi_gmem1:bank3

    VPP_CFLAGS += --slr MCAE_k0:SLR0
    VPP_CFLAGS += --slr MCAE_k1:SLR0
    VPP_CFLAGS += --slr MCAE_k2:SLR2
else
$(warning Unsupported platform $(XPLATFORM))
endif

VPP_LFLAGS += $(foreach k,$(KERNEL_NAMES), --nk $(k):1:$(k))

# -----------------------------------------------------------------------------
# TODO:                           host setup

SRC_DIR = $(CUR_DIR)/src

EXE_NAME = host
ifeq ($(TARGET),cpu)
    HOST_ARGS += -mode cpu
else
    HOST_ARGS = -mode fpga -xclbin $(XCLBIN_FILE)
endif

SRCS = main

main_EXTRA_HDRS += $(KSRC_DIR)/MCAE_kernel.hpp
main_CXXFLAGS += -I$(KSRC_DIR) -I$(EXT_DIR)/xcl2

CXXFLAGS += -D XDEVICE=$(XDEVICE) -I$(XFLIB_DIR)/L1/include/  -I$(XFLIB_DIR)/L2/include/

HOST_CCOPT = DBG
ifeq (${HOST_CCOPT},DBG)
    CXXFLAGS += -g
endif
ifeq (${HOST_CCOPT},OPT)
    CXXFLAGS += -O3
endif

ifneq (,$(shell echo $(XPLATFORM) | awk '/u50/'))
    CXXFLAGS += -DUSE_HBM
endif

ifneq (,$(shell echo $(XPLATFORM) | awk '/u50/ || /u200/'))
    CXXFLAGS += -DKN2=1
else ifneq (,$(shell echo $(XPLATFORM) | awk '/u250/'))
    CXXFLAGS += -DKN2=2
endif

# EXTRA_OBJS is cannot be compiled from SRC_DIR, user should provide the rule
EXTRA_OBJS += xcl2

EXT_DIR = $(XFLIB_DIR)/ext
xcl2_SRCS = $(EXT_DIR)/xcl2/xcl2.cpp
xcl2_HDRS = $(EXT_DIR)/xcl2/xcl2.hpp
xcl2_CXXFLAGS = -I $(EXT_DIR)/xcl2

# -----------------------------------------------------------------------------
# END_XF_MK_USER_SECTION
# -----------------------------------------------------------------------------

.PHONY: all
all: host xclbin

# MK_INC_BEGIN vitis_kernel_rules.mk

VPP_DIR_BASE ?= _x
XO_DIR_BASE ?= xo
XCLBIN_DIR_BASE ?= xclbin

XCLBIN_DIR_SUFFIX ?= _$(XDEVICE)_$(TARGET)

VPP_DIR = $(CUR_DIR)/$(VPP_DIR_BASE)$(XCLBIN_DIR_SUFFIX)
XO_DIR = $(CUR_DIR)/$(XO_DIR_BASE)$(XCLBIN_DIR_SUFFIX)
XCLBIN_DIR = $(CUR_DIR)/$(XCLBIN_DIR_BASE)$(XCLBIN_DIR_SUFFIX)

XFREQUENCY ?= 300

VPP = v++
VPP_CFLAGS += -I$(KSRC_DIR)
VPP_CFLAGS += --target $(TARGET) --platform $(XPLATFORM) --temp_dir $(VPP_DIR) --save-temps --debug
VPP_CFLAGS += --kernel_frequency $(XFREQUENCY) --report_level 2
VPP_LFLAGS += --optimize 2 --jobs 8

KERNEL_NAMES := $(foreach k,$(KERNELS),$(word 1, $(subst :, ,$(k))))
XO_FILES := $(foreach k,$(KERNEL_NAMES),$(XO_DIR)/$(k).xo)
XCLBIN_FILE ?= $(XCLBIN_DIR)/$(XCLBIN_NAME).xclbin

define kernel_src_dep
kernelname := $(word 1, $(subst :, ,$(1)))
kernelfile := $(if $(findstring :, $(1)),$(word 2, $(subst :, ,$(1))),$$(kernelname).cpp)
$$(kernelname)_SRCS := $(KSRC_DIR)/$$(kernelfile)
$$(kernelname)_SRCS += $$($$(kernelname)_EXTRA_SRCS)
endef

$(foreach k,$(KERNELS),$(eval $(call kernel_src_dep,$(k))))

define kernel_hdr_dep
kernelname := $(word 1, $(subst :, ,$(1)))
kernelfile := $(if $(findstring :, $(1)),$(basename $(word 2, $(subst :, ,$(1)))),$$(kernelname))
$$(kernelname)_HDRS := $$(wildcard $(KSRC_DIR)/$$(kernelfile).h $(KSRC_DIR)/$$(kernelfile).hpp)
$$(kernelname)_HDRS += $$($(1)_EXTRA_HDRS)
endef

$(foreach k,$(KERNELS),$(eval $(call kernel_hdr_dep,$(k))))

$(XO_DIR)/%.xo: VPP_CFLAGS += $($(*)_VPP_CFLAGS)
$(XO_DIR)/%.xo: $$($$(*)_SRCS) $$($$(*)_HDRS) | check_vpp
	@echo -e "----\nCompiling kernel $*..."
	mkdir -p $(XO_DIR)
	$(VPP) -o $@ --kernel $* --compile $(filter %.cpp,$^) \
		$(VPP_CFLAGS)

$(XCLBIN_FILE): $(XO_FILES) | check_vpp
	@echo -e "----\nCompiling xclbin..."
	mkdir -p $(XCLBIN_DIR)
	$(VPP) -o $@ --link $^ \
		$(VPP_CFLAGS) $(VPP_LFLAGS) \
		$(foreach k,$(KERNEL_NAMES),$($(k)_VPP_CFLAGS)) \
		$(foreach k,$(KERNEL_NAMES),$($(k)_VPP_LFLAGS))

.PHONY: xo xclbin

xo: $(XO_FILES) | check_vpp check_platform

xclbin: $(XCLBIN_FILE) | check_vpp check_platform

# MK_INC_END vitis_kernel_rules.mk

# MK_INC_BEGIN vitis_host_rules.mk

OBJ_DIR_BASE ?= obj
BIN_DIR_BASE ?= bin

BIN_DIR_SUFFIX ?= _$(XDEVICE)

OBJ_DIR = $(CUR_DIR)/$(OBJ_DIR_BASE)$(BIN_DIR_SUFFIX)
BIN_DIR = $(CUR_DIR)/$(BIN_DIR_BASE)$(BIN_DIR_SUFFIX)

CXX := xcpp
CC := gcc

CXXFLAGS += -std=c++14 -fPIC \
	-I$(SRC_DIR) -I$(XILINX_XRT)/include -I$(XILINX_VIVADO)/include \
	-Wall -Wno-unknown-pragmas -Wno-unused-label -pthread
CFLAGS +=
LDFLAGS += -pthread -L$(XILINX_XRT)/lib -lxilinxopencl
LDFLAGS += -L$(XILINX_VIVADO)/lnx64/tools/fpo_v7_0 -Wl,--as-needed -lgmp -lmpfr \
	   -lIp_floating_point_v7_0_bitacc_cmodel

OBJ_FILES = $(foreach s,$(SRCS),$(OBJ_DIR)/$(basename $(s)).o)

define host_hdr_dep
$(1)_HDRS := $$(wildcard $(SRC_DIR)/$(1).h $(SRC_DIR)/$(1).hpp)
$(1)_HDRS += $$($(1)_EXTRA_HDRS)
endef

$(foreach s,$(SRCS),$(eval $(call host_hdr_dep,$(basename $(s)))))

$(OBJ_DIR)/%.o: CXXFLAGS += $($(*)_CXXFLAGS)

$(OBJ_FILES): $(OBJ_DIR)/%.o: $(SRC_DIR)/%.cpp $$($$(*)_HDRS) | check_vpp check_xrt check_platform
	@echo -e "----\nCompiling object $*..."
	mkdir -p $(@D)
	$(CXX) -o $@ -c $< $(CXXFLAGS)

EXTRA_OBJ_FILES = $(foreach f,$(EXTRA_OBJS),$(OBJ_DIR)/$(f).o)

$(EXTRA_OBJ_FILES): $(OBJ_DIR)/%.o: $$($$(*)_SRCS) $$($$(*)_HDRS) | check_vpp check_xrt check_platform
	@echo -e "----\nCompiling extra object $@..."
	mkdir -p $(@D)
	$(CXX) -o $@ -c $< $(CXXFLAGS)

EXE_EXT ?= exe
EXE_FILE ?= $(BIN_DIR)/$(EXE_NAME)$(if $(EXE_EXT),.,)$(EXE_EXT)

$(EXE_FILE): $(OBJ_FILES) $(EXTRA_OBJ_FILES) | check_vpp check_xrt check_platform
	@echo -e "----\nCompiling host $(notdir $@)..."
	mkdir -p $(BIN_DIR)
	$(CXX) -o $@ $^ $(CXXFLAGS) $(LDFLAGS)

.PHONY: host
host: $(EXE_FILE) | check_vpp check_xrt check_platform

# MK_INC_END vitis_host_rules.mk

# MK_INC_BEGIN vitis_test_rules.mk

# -----------------------------------------------------------------------------
#                                clean up

clean:
ifneq (,$(OBJ_DIR_BASE))
	rm -rf $(CUR_DIR)/$(OBJ_DIR_BASE)*
endif
ifneq (,$(BIN_DIR_BASE))
	rm -rf $(CUR_DIR)/$(BIN_DIR_BASE)*
endif

cleanx:
ifneq (,$(VPP_DIR_BASE))
	rm -rf $(CUR_DIR)/$(VPP_DIR_BASE)*
endif
ifneq (,$(XO_DIR_BASE))
	rm -rf $(CUR_DIR)/$(XO_DIR_BASE)*
endif
ifneq (,$(XCLBIN_DIR_BASE))
	rm -rf $(CUR_DIR)/$(XCLBIN_DIR_BASE)*
endif
ifneq (,$(BIN_DIR_BASE))
	rm -rf $(CUR_DIR)/$(BIN_DIR_BASE)*/emconfig.json
endif

cleanall: clean cleanx
	rm -rf *.log plist $(DATA_STAMP)

# -----------------------------------------------------------------------------
#                                simulation run

$(BIN_DIR)/emconfig.json :
	emconfigutil --platform $(XPLATFORM) --od $(BIN_DIR)

ifeq ($(TARGET),sw_emu)
RUN_ENV += export XCL_EMULATION_MODE=sw_emu;
EMU_CONFIG = $(BIN_DIR)/emconfig.json
else ifeq ($(TARGET),hw_emu)
RUN_ENV += export XCL_EMULATION_MODE=hw_emu;
EMU_CONFIG = $(BIN_DIR)/emconfig.json
else ifeq ($(TARGET),hw)
RUN_ENV += echo "TARGET=hw";
EMU_CONFIG =
endif

.PHONY: run run_sw_emu run_hw_emu run_hw check

run_sw_emu:
	make TARGET=sw_emu run

run_hw_emu:
	make TARGET=hw_emu run

run_hw:
	make TARGET=hw run

run: host xclbin $(EMU_CONFIG) $(DATA_STAMP)
	$(RUN_ENV) \
	$(EXE_FILE) $(HOST_ARGS)

check: run

.PHONY: build
build: xclbin host

# MK_INC_END vitis_test_rules.mk

