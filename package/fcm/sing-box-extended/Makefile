include $(TOPDIR)/rules.mk

PKG_NAME:=sing-box-extended
PKG_VERSION:=1.14.3-extended-2.7.3
PKG_RELEASE:=2

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
		bool "Download and install CN rule sets (.srs) into the package"
		default y
		help
		  sing-box 1.12 起已删除 geoip.db/geosite.db，分流改用 .srs 规则集。
		  客户端常规是开机从 GitHub 拉这些文件，国内网络拉不动，所以要在
		  构建阶段就取好装进 /usr/share/sing-box/geo/。下载结果缓存在源码树
		  的 dl/sing-box-geo 里，重复构建不联网；想要新规则时删掉该目录。

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

# 分流规则集。sing-box 1.12 起 geoip.db/geosite.db 被彻底删除（见
# route/rule/rule_default.go 报的 "removed in sing-box 1.12.0"），只剩 .srs。
# 客户端常规做法是开机从 GitHub 拉规则，国内拉不到就直接没有分流，所以在构建
# 阶段拉好、打进固件（geoip-cn 34KB + geosite-cn 58KB，共约 92KB）。
# rule-set 分支是滚动更新的，故意不校验 hash；下载结果缓存在 dl/sing-box-geo，
# 重建时想换新规则就删掉那个目录。要加集合往 GEO_RULES 里追加即可。
GEO_RULES:=geoip-cn.srs geosite-cn.srs
GEO_DL_DIR:=$(DL_DIR)/sing-box-geo
GEO_BASE:=https://raw.githubusercontent.com/SagerNet

# 上游 README 声称 service/admin_panel/dist 已入库，实际被 .gitignore 的
# "dist" 规则排除；//go:embed dist 在没有该目录时编译失败，所以本地用
# vite 生成一次。admin_panel_pack 只产出 *.gz 并清理 *.woff，缺省时
# service.go 会回退到原始文件，故跳过。
# Build/Prepare 的 PATH 会被 include/package.mk 换成 TARGET_PATH_PKG，
# 所以 npm 路径在解析阶段就取好。
NPM:=$(shell command -v npm 2>/dev/null)

# 有 manager 的架构才装带 services 的配置
SING_BOX_EXTENDED_CONFIG_JSON:=$(if $(SING_BOX_EXTENDED_MANAGER),./files/config.manager.json,./files/config.json)

define Build/Prepare
	$(call Build/Prepare/Default)
	@echo "sing-box-extended: ARCH=$(ARCH) MANAGER=$(if $(SING_BOX_EXTENDED_MANAGER),on,off) ADMIN_PANEL=$(if $(SING_BOX_EXTENDED_PANEL),on,off)"
	@echo "sing-box-extended: GO_PKG_TAGS=$(GO_PKG_TAGS)"
ifneq ($(CONFIG_SING_BOX_EXTENDED_GEORULES),)
	mkdir -p $(GEO_DL_DIR)
	for f in $(GEO_RULES); do \
		case "$$f" in geoip-*) r=sing-geoip;; *) r=sing-geosite;; esac; \
		u="$(GEO_BASE)/$$r/rule-set/$$f"; \
		[ -s "$(GEO_DL_DIR)/$$f" ] && continue; \
		echo "sing-box-extended: 拉取分流规则 $$u"; \
		if command -v curl >/dev/null 2>&1; then \
			curl -sSfL --connect-timeout 20 --retry 3 -o "$(GEO_DL_DIR)/$$f" "$$u"; \
		else \
			wget -q -T 20 -t 3 -O "$(GEO_DL_DIR)/$$f" "$$u"; \
		fi || { rm -f "$(GEO_DL_DIR)/$$f"; \
			echo "sing-box-extended: 下载 $$f 失败。可在能上网的机器上把它放进 $(GEO_DL_DIR)/，或关掉 CONFIG_SING_BOX_EXTENDED_GEORULES 并删掉 config.json 里的 rule_set 两条。" >&2; \
			exit 1; }; \
	done
endif
ifneq ($(SING_BOX_EXTENDED_PANEL),)
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
	$(INSTALL_DATA) $(SING_BOX_EXTENDED_CONFIG_JSON) $(1)/etc/sing-box/config.json
	$(INSTALL_DATA) ./files/config.tun.example.json $(1)/etc/sing-box/config.tun.example.json

	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) ./files/sing-box.conf $(1)/etc/config/sing-box

	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/sing-box.init $(1)/etc/init.d/sing-box

	$(INSTALL_DIR) $(1)/lib/upgrade/keep.d
	$(INSTALL_DATA) ./files/sing-box.keep $(1)/lib/upgrade/keep.d/sing-box

	$(INSTALL_DIR) $(1)/usr/share/sing-box
ifneq ($(CONFIG_SING_BOX_EXTENDED_GEORULES),)
	$(INSTALL_DIR) $(1)/usr/share/sing-box/geo
	$(INSTALL_DATA) $(GEO_DL_DIR)/*.srs $(1)/usr/share/sing-box/geo/
endif
endef

$(eval $(call BuildPackage,sing-box-extended))

# CI/排错用：make -C package/fcm/sing-box-extended TOPDIR=$PWD show-tags
show-tags:
	@echo "ARCH=$(ARCH)"
	@echo "MANAGER=$(if $(SING_BOX_EXTENDED_MANAGER),on,off)"
	@echo "ADMIN_PANEL=$(if $(SING_BOX_EXTENDED_PANEL),on,off)"
	@echo "GO_PKG_TAGS=$(GO_PKG_TAGS)"
