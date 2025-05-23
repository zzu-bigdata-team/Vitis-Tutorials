# © Copyright 2022 Xilinx, Inc.
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

# =========================================================
# DEVICE SELECTION
# =========================================================
DEVICE_NAME=xcvc1902-vsva2197-2MP-e-S

# =========================================================
# PLATFORM NAME
# =========================================================
PLATFORM_USE := vck190_v1_0

# =========================================================
# XSA NAME
# =========================================================
XSA = $(PLATFORM_USE}.xsa

# =========================================================
# SOURCE DIRECTORIES
# =========================================================
SRC_DIR := src
QOR_SCRIPTS_DIR := $(SRC_DIR)/qor_scripts

# =========================================================
# XSA CREATION SCRIPT
# =========================================================
CREATE_XSA_TCL := $(PLATFORM_USE)_xsa.tcl

# =========================================================
# DEPENDENCIES
# =========================================================
DEPS := $(CREATE_XSA_TCL)
DEPS += pfm_decls.tcl
DEPS += $(QOR_SCRIPTS_DIR)/post_place.tcl
DEPS += $(QOR_SCRIPTS_DIR)/pre_place.tcl
DEPS += $(QOR_SCRIPTS_DIR)/post_route.tcl
DEPS += $(QOR_SCRIPTS_DIR)/post_physopt.tcl
DEPS += $(QOR_SCRIPTS_DIR)/timing.xdc
DEPS += $(SRC_DIR)/post_link_pfm_config.tcl
DEPS += $(SRC_DIR)/dr.bd.tcl
DEPS += $(SRC_DIR)/ddr4_dmc_3200_triplet_1.xdc

# =========================================================
# BUILD PRODUCTS 
# ========================================================
BUILD_DIR := build
XSA := $(PLATFORM_USE).xsa

xsa: $(BUILD_DIR)/$(XSA)

$(BUILD_DIR)/$(XSA): $(DEPS)
	vivado -mode batch -source $(CREATE_XSA_TCL) -tclargs $(DEVICE_NAME)

clean:
	rm -rf $(BUILD_DIR)/$(XSA)
	rm -rf $(BUILD_DIR) vivado*.jou vivado*.log
	rm -rf .Xil
