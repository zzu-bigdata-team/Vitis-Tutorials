# Copyright 2021 Xilinx Inc.
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

help::
	@echo "Makefile Usage:"
	@echo "  make all TARGET=<hw_emu|hw>"
	@echo "       Command runs make xsa"
	@echo "       Default TARGET=hw"
	@echo ""
	@echo "  make xsa TARGET=<hw_emu|hw>"
	@echo "       Command to create the XSA from libadf.a and HLS kernels *.xo"
	@echo "       Default TARGET=hw"
	@echo ""
	@echo "  make clean "
	@echo "      Command to remove the generated files."
	@echo ""

# =========================================================
# PROJECT DIRECTORIES
# =========================================================
#relative project directory
RELATIVE_PROJECT_DIR := ../

#absolute project directory
PROJECT_DIR	:= $(shell readlink -f $(RELATIVE_PROJECT_DIR))

#Design Directory 
DESIGN_DIR := $(PROJECT_DIR)

AIE_DIR :=  $(DESIGN_DIR)/Module_02_aie
PL_KERNELS_DIR := $(DESIGN_DIR)/Module_03_pl_kernels
XSA_DIR := $(DESIGN_DIR)/Module_04_full_system_design

# =========================================================
# TARGET SELECTION
# TARGET=hw (default)
# TARGET=hw_emu 
# =========================================================
TARGET ?=hw

# =========================================================
# PL KERNELS DIR
# =========================================================
TEMP_DIR := $(PL_KERNELS_DIR)/build.$(TARGET).$(PFM)/_x_temp

# =========================================================
# V++ COMPILER FLAGS
# =========================================================
VPP_FLAGS += -t $(TARGET) --platform $(PLATFORM) --save-temps -g --optimize 2
VPP_FLAGS += --hls.jobs 8
VPP_FLAGS += --config $(XSA_DIR)/conn.cfg
VPP_FLAGS += --clock.defaultFreqHz 150000000

# =========================================================
# V++ LINKER FLAGS
# =========================================================
VPP_LDFLAGS := --vivado.synth.jobs 8 --vivado.impl.jobs 8

# =========================================================
# BUILD DIRECTORIES
# =========================================================
BUILD_DIR := $(XSA_DIR)/build.$(TARGET).$(PFM)
BUILD_REPORT_DIR := $(BUILD_DIR)/reports/_x

# ################ TARGET: make pl_kernels ################
pl_kernels: 
	$(MAKE) pl_kernels -C $(PL_KERNELS_DIR)

# ################ TARGET: make all ################
all: pl_kernels xsa

# ################ TARGET: make xsa ################
xsa: pl_kernels $(BUILD_DIR)/hpc.xsa
$(BUILD_DIR)/hpc.xsa: $(TEMP_DIR)/packet_sender.xo $(TEMP_DIR)/m2s_x2.xo $(TEMP_DIR)/packet_receiver.xo $(TEMP_DIR)/s2m_x4.xo $(AIE_DIR)/build/libadf.a
	@echo "### ***** linking pl kernels into hpc.xsa ... *****"
	mkdir -p $(BUILD_DIR) && cd $(BUILD_DIR); \
	v++ -l $(VPP_FLAGS) \
	--temp_dir $(BUILD_DIR)/_x_temp \
	--report_dir $(BUILD_REPORT_DIR)/hpc \
	$(VPP_LDFLAGS) \
	-o '$@' $(+) | tee $(BUILD_DIR)/hpc_xsa.log
	@echo "### ***** hpc.xsa linking done! *****"

# ################ TARGET: make clean ################
clean:
	rm -rf $(BUILD_DIR)
	rm -rf *~
	rm -rf $(TEMP_DIR)/link
