# FongMi TV APK Builder

基于上游开源项目 [FongMi/TV](https://github.com/FongMi/TV)（GPL-3.0）的源码同步 + GitHub Actions APK 构建仓库。

目标：保留与上游一致的完整 Android 源码，支持一键更新，并在 GitHub 上直接构建可安装的 APK。

## 与你的 APK 的关系

你提供的 APK 是 `com.fongmi.android.tv` **v5.8.9**（`versionCode 589`，arm64-v8a）。上游 `FongMi/TV` 的 `fongmi` 分支当前 `versionName` 正好是 `5.8.9`、`versionCode 589`，因此本仓库同步的源码与该 APK 版本一致。

> 复刻方式说明：这不是把 APK 反编译成源码（反编译产物不可维护、无法持续更新）。这个 APK 本身就来自 FongMi/TV 开源项目，所以直接采用其官方源码是最可靠、可更新、且能在 GitHub 构建出等价 APK 的方式。

## 仓库结构

- `app-source/`：从 FongMi/TV `fongmi` 分支同步进来的完整源码（由工作流自动写入，首次为空）
- `.github/workflows/sync-upstream.yml`：手动触发，把上游最新源码同步进 `app-source/` 并提交
- `.github/workflows/build-apk.yml`：手动或定时触发，设置 Android 构建环境并输出 APK

## 使用方法

### 1. 同步源码

进入 Actions → **Sync upstream source** → Run workflow。默认拉取 `fongmi` 分支。

完成后 `app-source/` 就是上游完整源码。以后想更新，再点一次即可，提交信息会带上上游 commit SHA。也可以用 `upstream_ref` 参数指定某个历史 commit/tag，用来锁定到特定版本。

也可本地手动同步：

```bash
git clone --depth 1 --branch fongmi https://github.com/FongMi/TV.git app-source
rm -rf app-source/.git
git add app-source && git commit -m "Sync upstream FongMi/TV"
git push
```

### 2. 构建 APK

进入 Actions → **Build Android APK** → Run workflow，选择 flavor 和构建类型：

| 参数 | 选项 | 说明 |
| --- | --- | --- |
| `flavor` | `mobile`（默认）/ `leanback` | 手机/平板 / 电视盒子 |
| `build_type` | `Debug`（默认）/ `Release` | 调试版 / 发布版 |
| `create_release` | 勾选框 | 同时创建 GitHub Release |

上游用 `splits.abi` 一次构建会产出 `arm64-v8a` 和 `armeabi-v7a` 两个 APK。你要的 `mobile-arm64_v8a` 对应：flavor=`mobile`，产物中的 `app-mobile-arm64-v8a-*.apk`。

构建完成后在对应 Run 页面底部 **Artifacts** 下载 APK。

### 3. 本地构建（可选）

需要 JDK 21、Android SDK Platform 37 / Build Tools 37.0.0、Python 3.10：

```bash
cd app-source
printf 'sdk.dir=%s\n' "$ANDROID_HOME" > local.properties
bash gradlew :app:assembleMobileDebug
# 产物在 app/build/outputs/apk/mobile/debug/
```

## 关于“一模一样”

- 源码直接来自 FongMi/TV 上游 `fongmi` 分支，不做代码改写，因此与上游版本一致，`versionName 5.8.9` 与你的 APK 匹配。
- 本仓库不内置任何影视资源、接口或爬虫配置，仅提供源码与构建能力。
- 使用前请阅读上游 LICENSE（GPL-3.0）：二次分发需遵守 GPL 开源义务。
