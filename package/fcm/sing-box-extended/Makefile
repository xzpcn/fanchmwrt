include $(TOPDIR)/rules.mk

PKG_NAME:=sing-box-extended
PKG_VERSION:=1.14.3-extended-2.7.3
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/shtorm-7/sing-box-extended/tar.gz/refs/tags/v$(PKG_VERSION)?
PKG_HASH:=9b9b4015b7ed9ff632a97b1ed5d024de8828e4d8e8afbebae6a10780bc555511

PKG_LICENSE:=GPL-3.0-or-later
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=fanchmwrt

PKG_BUILD_DEPENDS:=golang/host
PKG_BUILD_PARALLEL:=1
PKG_BUILD_FLAGS:=no-mips16

GO_PKG:=github.com/sagernet/sing-box
GO_PKG_BUILD_PKG:=$(GO_PKG)/cmd/sing-box
GO_PKG_LDFLAGS:=-checklinkname=0 -X runtime.godebugDefault=multipathtcp=0,tlssha1=1
GO_PKG_LDFLAGS_X:=$(GO_PKG)/constant.Version=$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk
# 由 ./scripts/feeds update packages 提供，必须在 make 之前执行
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

define Package/sing-box-extended
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  TITLE:=The universal proxy platform (extended)
  URL:=https://github.com/shtorm-7/sing-box-extended
  DEPENDS:=$(GO_ARCH_DEPENDS) +ca-bundle +kmod-inet-diag +kmod-tun +kmod-nft-queue
  USERID:=sing-box=5566:sing-box=5566
endef

define Package/sing-box-extended/description
  shtorm-7's sing-box extended fork: MASQUE, MTProxy, TrustTunnel, Sudoku,
  Call, VPN, Bond/Fallback/Failover, limiters, providers/subscriptions and a
  Manager API with a built-in web admin panel.
endef

define Package/sing-box-extended/config
	menu "sing-box-extended build options"
		depends on PACKAGE_sing-box-extended

	config SING_BOX_EXTENDED_MANAGER
		bool "Build Manager API (users/nodes/limiters backend)"
		default y

	config SING_BOX_EXTENDED_ADMIN_PANEL
		bool "Embed the web admin panel (needs host nodejs + npm)"
		depends on SING_BOX_EXTENDED_MANAGER
		default y

	config SING_BOX_EXTENDED_CLASH_API
		bool "Build Clash API support"
		default y

	config SING_BOX_EXTENDED_GVISOR
		bool "Build gVisor tunnel (tun inbound) support"
		default y

	config SING_BOX_EXTENDED_QUIC
		bool "Build QUIC / Hysteria / MASQUE support"
		default y

	config SING_BOX_EXTENDED_UTLS
		bool "Build uTLS support (VLESS Vision, Reality)"
		default y

	config SING_BOX_EXTENDED_WIREGUARD
		bool "Build WireGuard support"
		default y

	config SING_BOX_EXTENDED_ACME
		bool "Build ACME TLS certificate issuer support"
		default n

	config SING_BOX_EXTENDED_TAILSCALE
		bool "Build Tailscale support"
		default n

	config SING_BOX_EXTENDED_OPENVPN
		bool "Build OpenVPN client and server support"
		default n

	config SING_BOX_EXTENDED_OPENCONNECT
		bool "Build OpenConnect client support"
		default n

	config SING_BOX_EXTENDED_CLOUDFLARED
		bool "Build Cloudflare Tunnel inbound support"
		default n

	config SING_BOX_EXTENDED_GRPC
		bool "Build standard gRPC transport support"
		default n

	config SING_BOX_EXTENDED_V2RAY_API
		bool "Build V2Ray API support"
		default n
	endmenu
endef

PKG_CONFIG_DEPENDS:= \
	CONFIG_SING_BOX_EXTENDED_MANAGER \
	CONFIG_SING_BOX_EXTENDED_ADMIN_PANEL \
	CONFIG_SING_BOX_EXTENDED_CLASH_API \
	CONFIG_SING_BOX_EXTENDED_GVISOR \
	CONFIG_SING_BOX_EXTENDED_QUIC \
	CONFIG_SING_BOX_EXTENDED_UTLS \
	CONFIG_SING_BOX_EXTENDED_WIREGUARD \
	CONFIG_SING_BOX_EXTENDED_ACME \
	CONFIG_SING_BOX_EXTENDED_TAILSCALE \
	CONFIG_SING_BOX_EXTENDED_OPENVPN \
	CONFIG_SING_BOX_EXTENDED_OPENCONNECT \
	CONFIG_SING_BOX_EXTENDED_CLOUDFLARED \
	CONFIG_SING_BOX_EXTENDED_GRPC \
	CONFIG_SING_BOX_EXTENDED_V2RAY_API

# 扩展分支独有的协议始终启用；关闭它们就换用官方 sing-box 了
SING_BOX_EXTENDED_TAGS:= \
	with_dhcp \
	with_mtproxy \
	with_trusttunnel \
	with_call \
	with_sudoku \
	with_ccm \
	with_ocm \
	$(if $(CONFIG_SING_BOX_EXTENDED_MANAGER),with_manager) \
	$(if $(CONFIG_SING_BOX_EXTENDED_ADMIN_PANEL),with_admin_panel) \
	$(if $(CONFIG_SING_BOX_EXTENDED_CLASH_API),with_clash_api) \
	$(if $(CONFIG_SING_BOX_EXTENDED_GVISOR),with_gvisor) \
	$(if $(CONFIG_SING_BOX_EXTENDED_QUIC),with_quic with_masque) \
	$(if $(CONFIG_SING_BOX_EXTENDED_UTLS),with_utls) \
	$(if $(CONFIG_SING_BOX_EXTENDED_WIREGUARD),with_wireguard) \
	$(if $(CONFIG_SING_BOX_EXTENDED_ACME),with_acme) \
	$(if $(CONFIG_SING_BOX_EXTENDED_TAILSCALE),with_tailscale) \
	$(if $(CONFIG_SING_BOX_EXTENDED_OPENVPN),with_openvpn) \
	$(if $(CONFIG_SING_BOX_EXTENDED_OPENCONNECT),with_openconnect) \
	$(if $(CONFIG_SING_BOX_EXTENDED_CLOUDFLARED),with_cloudflared) \
	$(if $(CONFIG_SING_BOX_EXTENDED_GRPC),with_grpc) \
	$(if $(CONFIG_SING_BOX_EXTENDED_V2RAY_API),with_v2ray_api) \
	badlinkname \
	tfogo_checklinkname0

GO_PKG_TAGS:=$(subst $(space),$(comma),$(strip $(SING_BOX_EXTENDED_TAGS)))

# 上游 README 声称 service/admin_panel/dist 已入库，实际被 .gitignore 的
# "dist" 规则排除；//go:embed dist 在没有该目录时编译失败，所以本地用
# vite 生成一次。admin_panel_pack 只产出 *.gz 并清理 *.woff，缺省时
# service.go 会回退到原始文件，故跳过。
# Build/Prepare 的 PATH 会被 include/package.mk 换成 TARGET_PATH_PKG，
# 所以 npm 路径在解析阶段就取好。
NPM:=$(shell command -v npm 2>/dev/null)

define Build/Prepare
	$(call Build/Prepare/Default)
ifneq ($(CONFIG_SING_BOX_EXTENDED_ADMIN_PANEL),)
	[ -n "$(NPM)" ] || { \
		echo "sing-box-extended: with_admin_panel 需要主机安装 Node.js >= 20 和 npm" >&2; \
		exit 1; \
	}
	cd $(PKG_BUILD_DIR)/service/admin_panel/web && \
		"$(NPM)" install --no-audit --no-fund && \
		"$(NPM)" run build
endif
endef

define Package/sing-box-extended/conffiles
/etc/config/sing-box
/etc/sing-box/config.json
endef

define Package/sing-box-extended/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(GO_PKG_BUILD_BIN_DIR)/sing-box $(1)/usr/bin/sing-box

	$(INSTALL_DIR) $(1)/etc/sing-box
	$(INSTALL_DATA) ./files/config.json $(1)/etc/sing-box/config.json
	$(INSTALL_DATA) ./files/config.tun.example.json $(1)/etc/sing-box/config.tun.example.json

	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) ./files/sing-box.conf $(1)/etc/config/sing-box

	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/sing-box.init $(1)/etc/init.d/sing-box

	$(INSTALL_DIR) $(1)/lib/upgrade/keep.d
	$(INSTALL_DATA) ./files/sing-box.keep $(1)/lib/upgrade/keep.d/sing-box

	$(INSTALL_DIR) $(1)/usr/share/sing-box
endef

$(eval $(call BuildPackage,sing-box-extended))
