# soh-flatpak

非官方 **Ship of Harkinian**（《塞尔达传说：时之笛》PC 原生移植）的 Flatpak 封装。

**构建方式**：直接解包上游官方 AppImage Release 并重新打包为 Flatpak，**不做源码编译**，因此构建快、产物与上游二进制完全一致。

- 上游源码：https://github.com/HarbourMasters/Shipwright
- 官网：https://shipofharkinian.com
- 安装指南：https://www.shipofharkinian.com/setup-guide

---

## 下载安装

从本仓库的 [Releases](../../releases) 页面下载 `com.harbourmasters.Shipwright.flatpak`：

```bash
flatpak install --user com.harbourmasters.Shipwright.flatpak
flatpak run com.harbourmasters.Shipwright
```

## ⚠️ 重要：需要自备 ROM

**本包不含任何版权游戏资源。** 你需要自己合法获得的《时之笛》ROM，首次启动时程序会提示导入。

上游项目采取零容忍政策，不提供、不分发任何 ROM 文件。

## 数据目录

```bash
~/.var/app/com.harbourmasters.Shipwright/data
```

配置（`shipofharkinian.json`）、存档、Mods、ROM 都放在这里。

---

## 版本对应

| 项目 | 值 |
|---|---|
| 应用 ID | `com.harbourmasters.Shipwright` |
| 上游版本 | Ship of Harkinian **9.3.1** (Dewey Bravo) |
| 运行时 | `org.freedesktop.Platform` 25.08 |
| 构建来源 | 上游官方 `SoH-Dewey-Bravo.appimage` |

升级上游版本需要修改两处：
1. `com.harbourmasters.Shipwright.yaml` 里 `sources[].url` 的版本号和 `sha256`
2. `.github/workflows/build.yml` 里的 `SOH_VERSION`

---

## 为什么这样做（可行性依据）

封装前对官方 AppImage 9.3.1 做了实际拆包分析：

| 检查项 | 结果 | 影响 |
|---|---|---|
| 二进制 RUNPATH | `$ORIGIN/../lib` | 装到 `/app/bin` + `/app/lib` 即可，无需 patchelf |
| 动态依赖 | 仅 14 个 NEEDED | 大部分随包自带，宿主只需 `libOpenGL.so.0`（runtime 提供） |
| glibc 需求 | 最高 `GLIBC_2.34` / `GLIBCXX_3.4.30` | Freedesktop 25.08 满足 |
| `/app` 硬编码路径 | 无 | 不会出现只读 `/app` 写入失败（对比闭源软件常见坑） |
| 配置路径 | 遵循 `XDG_DATA_HOME` | Flatpak 自动沙箱化到 `~/.var/app/` |
| ROM 是否内含 | 否 | `assets/*.yml` 仅为元数据 |

验证结论：清理编译残留（`include/`、`lib/*.a`、`cmake/`）后总大小 176M，启动能正确加载 `soh.o2r` 资源包与手柄映射。

---

## 本仓库结构

```
com.harbourmasters.Shipwright.yaml        # Flatpak 清单（核心）
soh-wrapper                               # 启动包装脚本（固定工作目录）
com.harbourmasters.Shipwright.desktop     # 桌面入口
com.harbourmasters.Shipwright.metainfo.xml# AppStream 元数据
icons/com.harbourmasters.Shipwright.png   # 图标
.github/workflows/build.yml               # GitHub Actions 构建
```

## 本地构建（可选）

需要 `flatpak-builder` + `org.freedesktop.Sdk//25.08`：

```bash
git clone <this repo>
cd soh-flatpak
flatpak-builder --user --install --force-clean build-dir \
  com.harbourmasters.Shipwright.yaml
flatpak run com.harbourmasters.Shipwright
```

## 许可

- Flatpak 清单/包装脚本：MIT
- Ship of Harkinian 本体：见 [上游](https://github.com/HarbourMasters/Shipwright)
- 本项目与 Nintendo / Harbour Masters 均无隶属关系
