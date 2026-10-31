<div align="center">

# Nexio课程表（Fork：xiaoxun007）

一款基于 Jetpack Compose 的 Android 课程表应用，支持自定义课表外观、教务系统导入、多格式课表导入、WebDAV 同步、桌面小组件等功能。

本仓库是 [HaoZai000/NexioSchedule](https://github.com/HaoZai000/NexioSchedule) 的个人维护分支（fork），为满足个人使用习惯做了针对性修改，并接入**自动同步上游 + 自动构建发布**流水线。

[![Stars](https://img.shields.io/github/stars/xiaoxun007/NexioSchedule?style=flat-square&color=yellow)](https://github.com/xiaoxun007/NexioSchedule/stargazers)
[![Downloads](https://img.shields.io/github/downloads/xiaoxun007/NexioSchedule/total?style=flat-square&color=orange)](https://github.com/xiaoxun007/NexioSchedule/releases)
[![Latest Release](https://img.shields.io/github/v/release/xiaoxun007/NexioSchedule?style=flat-square&color=blue)](https://github.com/xiaoxun007/NexioSchedule/releases/latest)
[![Build](https://img.shields.io/github/actions/workflow/status/xiaoxun007/NexioSchedule/sync-upstream-build-release.yml?style=flat-square&color=green)](https://github.com/xiaoxun007/NexioSchedule/actions)

</div>

---

# ⚠️ 本 Fork 与上游的差异（详细说明）

## 1. 功能差异：移除了「下课自动恢复」（自动关闭静音/勿扰）

这是本 fork **唯一的代码功能改动**。

| 行为 | 上游原版 | 本 fork |
|------|----------|---------|
| 上课 | 自动开启勿扰 / 静音 | 自动开启勿扰 / 静音（相同） |
| 下课 | **自动恢复**上课前的系统状态 | **保持勿扰 / 静音，不自动恢复** |
| 恢复方式 | 自动 | 手动：点击通知栏 / 超级岛「上课勿扰」按钮还原 |

**改动内容**（仅 1 个文件，+2 / -8 行）：
- 删除 `ClassDndHelper.applyCurrentState()` 非课堂分支中的自动恢复调用（`handBack(requireConsistency = true)`），下课后保持当前静音/勿扰状态
- 不再注册「下课」闹钟（真实课与测试课均只保留上课闹钟），减少无效的闹钟调度
- **保留**的主动行为：上课自动开启、通知栏/超级岛按钮手动开启与手动还原、在设置中关闭课程提醒总开关时的状态还原

补丁文件位于 `patches/class-dnd-auto-end.patch`，可在干净的上游代码上直接应用。

## 2. 自动同步上游 + 自动构建发布

本 fork 的 `master` 分支由 GitHub Actions 工作流全权管理（`.github/workflows/sync-upstream-build-release.yml`）：

1. **每小时**检查上游 `HaoZai000/NexioSchedule` 是否有新提交
2. 有更新时自动执行：
   - 同步上游最新代码（fork `master` 强制对齐上游）
   - 应用本 fork 的功能补丁（`patches/class-dnd-auto-end.patch`）
   - R8 优化编译 Release APK（`./gradlew :app:assembleRelease`）
   - 使用 Android 调试密钥签名（`signing/debug.keystore.b64`），可直接安装
   - 发布 GitHub Release
3. 同步过程会保护本 fork 特有文件：`patches/`、`signing/`、`.github/workflows/`、`README.md`，不会被上游覆盖

**已知边界**：若上游发生大版本重构（如重写核心文件），补丁可能无法自动应用，构建会失败并通过 GitHub 邮件通知，届时需要人工重新适配补丁。

## 3. 版本命名规则

Release 版本号与上游主版本对齐，附加构建计数后缀，**版本名称（versionName）与版本代码（versionCode）同步变更**：

```
Release tag：v{上游主版本}-gh{n}
APK versionName：{上游主版本}-gh{n}
APK versionCode：{上游versionCode × 100 + n}
```

- 例：上游 `versionCode 156`、版本 `1.5.6-0924` → 本 fork 依次发布 `v1.5.6-gh1`（versionCode **15601**）、`v1.5.6-gh2`（**15602**）…
- 每次上游升版（主版本变化，如 `1.6.0`）后，`gh` 计数从 1 重新开始 → `v1.6.0-gh1`（versionCode `16001`）
- versionCode 恒大于上游，且 fork 内部单调递增：**同一 fork 内可正常覆盖升级，且永远不会与上游原版的 versionCode 冲突**（不会出现"改了版本名却因版本代码没变而无法区分新旧"的问题）

## 4. 签名与安装说明

- APK 使用 **Android 调试密钥**（debug keystore）签名，可直接侧载安装（无需 root）
- 签名与上游原版**不同**：在已安装上游原版的设备上安装本 fork 版本，会因签名不一致而**覆盖安装失败**，需先卸载原版（注意备份数据）
- 每次构建生成的 APK 命名与上游一致：`Nexio.v{主版本}-gh{n}.apk`（如 `Nexio.v1.5.6-gh5.apk`；上游为 `Nexio.v1.5.5-0921.apk` 风格）

## 5. 应用内更新检测注意（重要）

- 应用内置的更新检测逻辑**仍指向上游仓库**（默认 Gitee `hyper_schedule`，可切换到 GitHub 上游 `HaoZai000/NexioSchedule`）
- **不会检测本 fork 发布的 Release**
- 若上游发布了更高版本，应用可能提示更新——**注意辨别来源，避免下载上游原版覆盖本 fork 的定制功能**（上游原版会恢复「下课自动关闭」行为）

## 6. 下载

所有构建产物发布在 [Releases 页面](https://github.com/xiaoxun007/NexioSchedule/releases)，最新的 `v{主版本}-gh{n}` 即为最新构建。

---

# 上游项目简介

## 功能特性

**课表视图**
- 周视图课程表，支持多周快速切换与跳转
- 今日课程页面，展示当天课程列表及当前/下一节课信息
- 课程详情弹窗，查看完整课程信息
- 周末显示设置（不显示 / 仅周六 / 仅周日 / 周六与周日）

**课程管理**
- 添加、编辑、删除课程
- 支持自定义课程颜色
- 支持全周、单周、双周及自定义周次
- 课表节数设置（上午/下午/晚上节数自定义）
- 课程时间设置（每节课起止时间自定义）

**多课表管理**
- 多课表创建与切换
- 课表重命名、删除、分享
- 开启新学期（复用当前课表设置，创建空课程新课表）

**导入导出**
- 教务系统一键导入（WebView + JavaScript 脚本适配）
- AI 文本导入（自然语言格式自动解析）
- 课表文件导入（JSON / ICS / 拾光课程表格式）
- 课表导出（JSON 格式完整数据导出）

**排班课表**
- 排班模式：多课表对比查看排班情况

**课程提醒**
- 课前提醒通知
- 次日课程提醒
- 超级岛 / 灵动岛通知展示（需 Shizuku 特权）

**桌面小组件**
- 课程预览小组件
- 今日课程小组件

**数据同步**
- WebDAV 云端备份与恢复

**个性化**
- 壁纸搭配自定义（卡片模糊、透明度、高度、圆角）
- 深色模式适配
- 应用偏好设置（主题模式、首页默认页、应用风格）

**UI 特色**
- HyperOS4 风格 UI（基于 MiUiX 组件库）
- 液态玻璃（LiquidGlass）效果
- 连续曲率圆角（Squircle）裁剪
- 渐进模糊、边缘光效与平滑过渡动画

## 预览界面

| 主课程表 | 课表外观 | 添加课程 |
|----------|----------|----------|
| ![主课程表](docs/picture/主课程表.png) | ![课表外观](docs/picture/课表外观.png) | ![添加课程](docs/picture/添加课程.png) |
| 教务导入 | 课程提醒 | 桌面小部件 |
|----------|----------|----------|
| ![教务导入](docs/picture/教务导入.png) | ![课程提醒](docs/picture/课程提醒.png) | ![桌面小部件](docs/picture/桌面小部件.png) |

---

## 致谢上游

本 fork 基于 [HaoZai000/NexioSchedule](https://github.com/HaoZai000/NexioSchedule) 维护，感谢原作者与上游贡献者。
