include $(TOPDIR)/rules.mk

PKG_NAME:=sing-box-extended
PKG_VERSION:=1.14.3-extended-2.7.3
PKG_RELEASE:=4

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
  CONFLICTS:=sing-box
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

	config SING_BOX_EXTENDED_GEORULES
		bool "Install CN rule sets (.srs) into the package"
		default y
		help
		  sing-box 1.12 起已删除 geoip.db/geosite.db，分流改用 .srs 规则集。
		  客户端常规是开机从 GitHub 拉这些文件，国内网络拉不动就没分流，
		  所以规则随包装在 files/geo/ 下，安装到 /usr/share/sing-box/geo/。
		  构建阶段不联网；更新规则见 Makefile 里 GEO_FILES 上方的说明。

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
	CONFIG_SING_BOX_EXTENDED_GEORULES \
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

# ---------------------------------------------------------------------------
# with_manager 在 mips 上编译不过，原因不是本包写错，而是它的存储层
# modernc.org/sqlite （纯 Go sqlite）根本没有 32 位 mips 的实现：
#
#   modernc.org/sqlite/lib 只有 sqlite_linux_{386,amd64,arm,arm64,
#   loong64,ppc64le,riscv64,s390x}.go 和 darwin/freebsd/netbsd/openbsd/
#   windows 的文件；依赖的 modernc.org/libc 也只有 mips64le。
#   GOARCH=mipsle（mt7621 即 mipsel）时这些包没有可编译的源文件，
#   go build 会直接报 build constraints exclude all Go files in
#   .../modernc.org/libc，或者 undefined: sqlite3.X... 之类的错。
#
# 上游 .goreleaser.yaml 也是这么处理的：main 构建带 with_manager，而
# id: mips / id: openwrt-mips 两份 tag 列表里没有 with_manager、
# with_admin_panel。这里跟随同样的做法：mips/mips64 上自动关掉这两项，
# 其他架构（x86_64、aarch64、armv7、riscv64…）照旧可用。
# ---------------------------------------------------------------------------
SING_BOX_EXTENDED_MIPS:=$(strip $(filter mips% mips64%,$(ARCH)))

ifneq ($(SING_BOX_EXTENDED_MIPS),)
  ifneq ($(filter y,$(CONFIG_SING_BOX_EXTENDED_MANAGER) $(CONFIG_SING_BOX_EXTENDED_ADMIN_PANEL)),)
    $(warning sing-box-extended: ARCH=$(ARCH) 上 modernc.org/sqlite 无实现，已自动关闭 with_manager / with_admin_panel)
  endif
endif

# 解析后的开关：mips 上一律为空
SING_BOX_EXTENDED_MANAGER:=$(if $(SING_BOX_EXTENDED_MIPS),,$(CONFIG_SING_BOX_EXTENDED_MANAGER))
SING_BOX_EXTENDED_PANEL:=$(if $(SING_BOX_EXTENDED_MIPS),,$(CONFIG_SING_BOX_EXTENDED_ADMIN_PANEL))

# 管理面板要用主机上的 npm 现场编（//go:embed dist，源码树里没有 dist）。
# Build/Prepare 的 PATH 会被 include/package.mk 换成 TARGET_PATH_PKG，所以
# npm 路径在解析阶段就取好。
NPM:=$(shell command -v npm 2>/dev/null)

# 没有 npm 就自动把面板关掉，而不是让整个包崩掉：
# CONFIG_ALL=y 的通用 CI（openwrt/actions-shared-workflows 的
# "Build all core packages"）里 ADMIN_PANEL 走默认值，而它的 toolchain
# 容器里没有 node/npm。面板只是塞进 /usr/bin/sing-box 的静态资源，
# 关掉不影响任何协议功能；要面板的机器装好 Node.js >= 20 + npm 重新
# 编译即可（Kconfig 里 ADMIN_PANEL 默认仍是 y）。
ifeq ($(SING_BOX_EXTENDED_PANEL),y)
  ifeq ($(NPM),)
    $(warning sing-box-extended: 找不到 npm，自动关闭 with_admin_panel（管理面板不编入）)
    SING_BOX_EXTENDED_PANEL:=
  endif
endif

# 扩展分支独有的协议始终启用；关闭它们就换用官方 sing-box 了
SING_BOX_EXTENDED_TAGS:= \
	with_dhcp \
	with_mtproxy \
	with_trusttunnel \
	with_call \
	with_sudoku \
	with_ccm \
	with_ocm \
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
	$(if $(SING_BOX_EXTENDED_MANAGER),with_manager) \
	$(if $(SING_BOX_EXTENDED_PANEL),with_admin_panel) \
	badlinkname \
	tfogo_checklinkname0

GO_PKG_TAGS:=$(subst $(space),$(comma),$(strip $(SING_BOX_EXTENDED_TAGS)))

# 分流规则集：sing-box 1.12 起 geoip.db/geosite.db 被彻底删除（route/rule 里会
# 报 "removed in sing-box 1.12.0"），只剩 .srs。客户端常规做法是开机从 GitHub
# 拉规则，国内拉不到就等于没有分流，所以构建阶段就把规则装进
# /usr/share/sing-box/geo/。
#
# 规则文件直接入库在 files/geo/ 下（geoip-cn 34KB + geosite-cn 58KB）：
# 构建不联网、不依赖 dl/ 里的临时文件，install 用显式文件名拷贝，缺文件当场
# 报错，不会出现「规则没装上但构建成功」或者「install 阶段找不到 *.srs」。
# 更新规则（rule-set 分支是滚动更新的）：
#   curl -sSfL -o files/geo/geoip-cn.srs \
#     https://raw.githubusercontent.com/SagerNet/sing-geoip/rule-set/geoip-cn.srs
#   curl -sSfL -o files/geo/geosite-cn.srs \
#     https://raw.githubusercontent.com/SagerNet/sing-geosite/rule-set/geosite-cn.srs
# 然后 PKG_RELEASE 加一。
GEO_FILES:=geoip-cn.srs geosite-cn.srs

ifeq ($(CONFIG_SING_BOX_EXTENDED_GEORULES),y)
define Install/GeoRules
	$(INSTALL_DIR) $(1)/usr/share/sing-box/geo
	$(INSTALL_DATA) ./files/geo/geoip-cn.srs $(1)/usr/share/sing-box/geo/geoip-cn.srs
	$(INSTALL_DATA) ./files/geo/geosite-cn.srs $(1)/usr/share/sing-box/geo/geosite-cn.srs
endef
endif

ifeq ($(SING_BOX_EXTENDED_PANEL),y)
define Install/AdminPanelExample
	$(INSTALL_DATA) ./files/config.admin-panel.example.json $(1)/etc/sing-box/config.admin-panel.example.json
endef

define Build/AdminPanel
	cd $(PKG_BUILD_DIR)/service/admin_panel/web && \
		"$(NPM)" install --no-audit --no-fund && \
		"$(NPM)" run build
endef
endif

# 有 manager 的架构才装带 manager/manager-api 服务的配置；
# 面板另有一份 config.admin-panel.example.json，只在面板真的编进去时安装。
SING_BOX_EXTENDED_CONFIG_JSON:=$(if $(SING_BOX_EXTENDED_MANAGER),./files/config.manager.json,./files/config.json)

define Build/Prepare
	$(call Build/Prepare/Default)
	@echo "sing-box-extended: ARCH=$(ARCH) MANAGER=$(if $(SING_BOX_EXTENDED_MANAGER),on,off) ADMIN_PANEL=$(if $(SING_BOX_EXTENDED_PANEL),on,off)"
	@echo "sing-box-extended: GO_PKG_TAGS=$(GO_PKG_TAGS)"
	@ls -l ./files/geo 2>/dev/null || true
	$(Build/AdminPanel)
endef

define Package/sing-box-extended/conffiles
/etc/config/sing-box
/etc/sing-box/config.json
endef

define Package/sing-box-extended/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(GO_PKG_BUILD_BIN_DIR)/sing-box $(1)/usr/bin/sing-box

	$(INSTALL_DIR) $(1)/etc/sing-box
	$(INSTALL_DATA) $(SING_BOX_EXTENDED_CONFIG_JSON) $(1)/etc/sing-box/config.json
	$(INSTALL_DATA) ./files/config.tun.example.json $(1)/etc/sing-box/config.tun.example.json
	$(call Install/AdminPanelExample,$(1))

	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) ./files/sing-box.conf $(1)/etc/config/sing-box

	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/sing-box.init $(1)/etc/init.d/sing-box

	$(INSTALL_DIR) $(1)/lib/upgrade/keep.d
	$(INSTALL_DATA) ./files/sing-box.keep $(1)/lib/upgrade/keep.d/sing-box

	$(INSTALL_DIR) $(1)/usr/share/sing-box
	$(call Install/GeoRules,$(1))
endef

$(eval $(call BuildPackage,sing-box-extended))

# CI/排错用：make -C package/fcm/sing-box-extended TOPDIR=$PWD show-tags
show-tags:
	@echo "ARCH=$(ARCH)"
	@echo "MANAGER=$(if $(SING_BOX_EXTENDED_MANAGER),on,off)"
	@echo "ADMIN_PANEL=$(if $(SING_BOX_EXTENDED_PANEL),on,off)"
	@echo "GEORULES=$(if $(CONFIG_SING_BOX_EXTENDED_GEORULES),on,off)"
	@echo "GEO_FILES=$(GEO_FILES)"
	@echo "GO_PKG_TAGS=$(GO_PKG_TAGS)"
