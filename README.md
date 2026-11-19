# FongMi TV APK Builder

基于上游开源项目 [FongMi/TV](https://github.com/FongMi/TV)（GPL-3.0）的源码同步 + GitHub Actions APK 构建仓库。

目标：保留与上游一致的完整 Android 源码，支持一键更新，并在 GitHub 上直接构建可安装的 APK。

## 仓库结构

- `app-source/`：从 FongMi/TV `fongmi` 分支同步进来的完整源码（由工作流自动写入，首次为空）
- `.github/workflows/sync-upstream.yml`：手动触发，把上游最新源码同步进 `app-source/` 并提交
- `.github/workflows/build-apk.yml`：手动或定时触发，设置 Android 构建环境并输出 APK

## 使用方法

### 1. 同步源码

进入 Actions → **Sync upstream source** → Run workflow。

完成后 `app-source/` 就是上游 `fongmi` 分支的完整源码。以后想更新，再点一次即可，提交信息会带上上游 commit SHA。

也可以本地手动同步：

```bash
git clone --depth 1 --branch fongmi https://github.com/FongMi/TV.git app-source
rm -rf app-source/.git
git add app-source && git commit -m "Sync upstream FongMi/TV"
git push
```

### 2. 构建 APK

进入 Actions → **Build Android APK** → Run workflow，选择要打的 flavor。

| Flavor | 说明 |
| --- | --- |
| `mobileArm64_v8a` | 手机 64 位 |
| `mobileArmeabi_v7a` | 手机 32 位 |
| `leanbackArm64_v8a` | 电视/盒子 64 位 |
| `leanbackArmeabi_v7a` | 电视/盒子 32 位 |

构建完成后在对应 Run 页面底部 **Artifacts** 下载 APK。勾选 `create_release` 时会同时创建 GitHub Release。

### 3. 本地构建（可选）

需要 JDK 21、Android SDK Platform 37 / Build Tools 37.0.0、Python 3.10：

```bash
cd app-source
printf 'sdk.dir=%s\n' "$ANDROID_HOME" > local.properties
bash gradlew :app:assembleMobileArm64_v8aDebug
```

## 关于“一模一样”

- 源码直接来自 FongMi/TV 上游 `fongmi` 分支，不做代码改写，因此与上游版本一致。
- 你提供的 APK 版本号为 `5.8.9`；上游没有 tag，构建时请核对 `app-source/app/build.gradle` 中的 `versionName`。若不一致，可切到上游对应历史 commit 后再同步。
- 本仓库不内置任何影视资源、接口或爬虫配置，仅提供源码与构建能力。使用前请阅读上游 LICENSE（GPL-3.0）。
