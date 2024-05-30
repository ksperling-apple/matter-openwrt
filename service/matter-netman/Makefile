# Copyright (c) 2023-2024 Project CHIP Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

include $(TOPDIR)/rules.mk

PKG_NAME:=matter-netman
PKG_RELEASE:=1
PKG_SOURCE_URL:=https://github.com/project-chip/connectedhomeip.git
PKG_SOURCE_PROTO:=git
PKG_SOURCE_MIRROR:=0 # don't try OpenWrt mirror
PKG_SOURCE_SUBMODULES:=\
  third_party/pigweed/repo \
  third_party/jsoncpp/repo \
  third_party/nlassert/repo \
  third_party/nlio/repo

# Hash can be regenerated with make package/matter-netman/check FIXUP=1
PKG_SOURCE_DATE:=2024-05-30
PKG_SOURCE_VERSION:=896f802e689fbb31744d50134a6a8b2a3d67fb52
PKG_MIRROR_HASH:=48b07ade631946e75f0872c6e9ace56cfb3c5e8fbb064399d426a262b33f7e4f

# Use local source dir for development
USE_SOURCE_DIR:=$(HOME)/workspace/connectedhomeip

PKG_LICENSE:=Apache-2.0
PKG_LICENSE_FILES:=LICENSE

PKG_BUILD_PARALLEL:=1
PKG_BUILD_DEPENDS:=gn/host

include $(INCLUDE_DIR)/package.mk

define Package/matter-netman
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Matter Network Infrastructure Manager Daemon
  URL:=https://github.com/project-chip/connectedhomeip
  DEPENDS:=+libstdcpp +libopenssl +libubus +libubox
  USERID:=matter:matter
endef

define Package/matter-netman/description
  Matter Network Infrastructure Manager Daemon
  Integrates a router / access point with the Matter IoT ecosystem.
endef

# https://github.com/project-chip/matter-openwrt/issues/2
# PKG_SOURCE_SUBMODULES support is in OpenWrt main but not 23.x releases.
ifeq ($(DUMP)$(filter SUBMODULES:=,$(Download/Defaults)),)
$(error PKG_SOURCE_SUBMODULES is not supported, ensure https://github.com/openwrt/openwrt/pull/13000 is included in your OpenWrt buildroot)
endif

# https://github.com/openwrt/openwrt/issues/13016
TARGET_CXXFLAGS += -Wno-format-nonliteral

# The build environment contains host tools that can be shared between targets
CHIP_BUILD_ENV_DIR:=$(STAGING_DIR_HOST)/share/chip-build-env
OUT_DIR:=$(PKG_BUILD_DIR)/out/openwrt

define Build/Configure
	mkdir -p $(OUT_DIR) && cd $(OUT_DIR) && \
		$(CONFIGURE_VARS) $(PKG_BUILD_DIR)/scripts/configure \
			--build-env-dir="$(CHIP_BUILD_ENV_DIR)" \
			--project=examples/network-manager-app/linux \
			--target=$(GNU_TARGET_NAME) \
			--enable-openthread=no \
			--enable-network-layer-ble=no \
			--enable-tracing-support=no \
			--enable-ubus
endef

define Build/Compile
	$(NINJA) -C $(OUT_DIR) matter-network-manager-app
endef

define Package/matter-netman/install
	$(INSTALL_DIR) $(1)/usr/sbin $(1)/etc/init.d $(1)/usr/share/acl.d
	$(INSTALL_BIN) $(OUT_DIR)/matter-network-manager-app $(1)/usr/sbin
	$(INSTALL_BIN) ./files/matter.init $(1)/etc/init.d/matter
	$(INSTALL_DATA) ./files/matter.acl.json $(1)/usr/share/acl.d/matter.json
endef

$(eval $(call BuildPackage,matter-netman))
