# SYNC.md — 本仓库的同步维护说明

本仓库是 **VIKINGYFY/packages** 的 fork，作为 OpenWrt/ImmortalWrt 的第三方 feed 使用
（ponwrt 中通过 `src-git farsky https://github.com/Farsky327/packages.git` 挂载）。

仓库里有两类包：

| 来源 | 目录 | 维护方式 |
|------|------|----------|
| VIKINGYFY 上游 | axonhub / gecoosac / luci-app-axonhub / luci-app-gecoosac / luci-app-homeproxy / luci-app-wolultra / sing-box | 跟随上游，**不改动** |
| Ponwrt 定制 | luci-theme-vantage / patches / SYNC.md | 本仓库自有，见下文 |

## 原则：只增不改（附加式 fork）

对 VIKINGYFY 上游已有的文件和目录**保持零改动**，定制内容一律放在新目录里。
这样与上游同步时永远是 fast-forward / 干净 rebase，不会产生冲突。

## 同步线 1：VIKINGYFY/packages（fork 父仓库）

```bash
git remote add upstream git@github.com:VIKINGYFY/packages.git   # 首次
git fetch upstream
git rebase upstream/main
git push origin main
```

我们的定制目录（luci-theme-vantage、patches、SYNC.md）上游不存在，rebase 不会碰它们。

## 同步线 2：anothaDev/luci-theme-vantage（主题上游）

品牌定制只改 3 个文件，改动已提取为
`patches/luci-theme-vantage-ponwrt-brand.patch`：

- `ucode/template/themes/vantage/header.ut` — 主界面侧栏+顶栏品牌（V → P）
- `ucode/template/themes/vantage/sysauth.ut` — 登录页品牌（V → P）
- `htdocs/luci-static/vantage/logo.svg` — 站点图标（V → P）

上游发新版本时的同步步骤：

```bash
git remote add vantage git@github.com:anothaDev/luci-theme-vantage.git   # 首次
git fetch vantage

# 1. 用新版本整包替换（保留 .git，只换工作区文件）
rm -rf luci-theme-vantage
git checkout vantage/main -- luci-theme-vantage

# 2. 重打品牌 patch
patch -p1 < patches/luci-theme-vantage-ponwrt-brand.patch

# 3. 核对：只有 patch 里的 3 个文件有差异，Makefile 版本号应为上游新版
git diff --stat
git diff luci-theme-vantage/Makefile    # 应只看到 PKG_VERSION 变化

# 4. 若上游重构了上述 3 个文件导致 patch 打不上：
#    手动把品牌 SVG（P 形 path + Ponwrt 字样）移植过去，
#    然后重新生成 patch：
#    git diff --no-index <上游包副本> luci-theme-vantage \
#      | sed 's|<上游副本路径>|luci-theme-vantage|g' \
#      > patches/luci-theme-vantage-ponwrt-brand.patch

# 5. 提交
git add -A && git commit -m "theme: sync luci-theme-vantage to <新版本>，重打 Ponwrt 品牌 patch"
```

## ponwrt 侧配套

- `feeds.conf.default`：`src-git farsky https://github.com/Farsky327/packages.git`
- `configs/release.config`：`CONFIG_PACKAGE_luci-theme-vantage=y`（只装主题，不带 luci-app-vantage 仪表盘）
- 品牌已完全由本 feed 提供，ponwrt 仓库的 `files/` overlay 不再需要放主题模板
