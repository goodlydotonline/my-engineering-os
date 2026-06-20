# macOS 磁盘清理最佳实践

> 适用场景：开发环境长期运行后，系统可用空间快速下降，需要定位并安全清理大文件与缓存。

---

## 1. 核心原则

1. **先定位，再删除**：不要盲删 `~/Library`，先用 `du` 或 `ncdu` 找出真正的大户。
2. **先备份重要数据**：聊天记录、笔记、项目源码、Docker 数据卷等在清理前应确认价值。
3. **优先清理缓存**：`Caches` 目录里的内容通常可安全删除，应用会按需重建。
4. **慎删 Containers 和 Application Support**：这两处可能包含用户数据、聊天记录、离线文件。
5. **用应用自带清理工具**：微信、Chrome、Docker 等都提供安全的内置清理方式，优先使用。

---

## 2. 快速定位磁盘大户

### 2.1 查看用户目录顶层占用

```bash
du -sh ~/* ~/.[^.]* 2>/dev/null | sort -hr | head -20
```

### 2.2 查看 `~/Library` 子目录占用

```bash
du -sh ~/Library/* 2>/dev/null | sort -hr | head -20
```

### 2.3 查看常见开发缓存目录

```bash
du -sh \
  ~/Library/Developer \
  ~/Library/Caches \
  ~/Library/Containers \
  ~/Library/Application\ Support \
  ~/Library/Logs \
  ~/.gradle \
  ~/.pub-cache \
  ~/.cocoapods \
  ~/.npm \
  ~/.yarn \
  ~/.docker \
  ~/.conda \
  ~/.pyenv \
  ~/.nvm \
  ~/.m2 \
  ~/.vscode \
  2>/dev/null | sort -hr | head -30
```

---

## 3. macOS 常见磁盘占用大户

| 目录类型 | 典型位置 | 说明 |
|---|---|---|
| App 沙盒数据 | `~/Library/Containers` | 微信、QQ、Docker、WPS 等 App 的数据 |
| App 支持文件 | `~/Library/Application Support` | Chrome、Notion、JetBrains、VS Code 等 |
| 缓存 | `~/Library/Caches` | 浏览器、IDE、包管理器、系统缓存 |
| Android SDK | `~/Library/Android/sdk` | SDK、模拟器系统镜像 |
| Gradle 缓存 | `~/.gradle/caches` | 构建缓存与依赖 |
| Flutter/Dart 缓存 | `~/.pub-cache` | Pub 包缓存 |
| npm 缓存 | `~/.npm/_cacache` | npm 下载的包缓存 |
| VS Code 扩展 | `~/.vscode/extensions` | 安装的扩展插件 |
| Maven 缓存 | `~/.m2` | Java Maven 依赖 |

---

## 4. 安全清理清单

### 4.1 高度安全：缓存类

这些目录删除后，应用会按需重建，通常不会丢失用户数据。

```bash
# 系统与应用缓存
rm -rf ~/Library/Caches/*

# Chrome 浏览器缓存（保留登录态与历史记录）
rm -rf ~/Library/Caches/Google

# JetBrains IDE 索引与编译缓存
rm -rf ~/Library/Caches/JetBrains

# VS Code 自动更新安装包
rm -rf ~/Library/Caches/com.microsoft.VSCode.ShipIt

# 微信缓存（非聊天记录）
rm -rf ~/Library/Caches/com.tencent.xinWeChat
```

### 4.2 包管理器缓存

```bash
# Homebrew
brew cleanup

# npm
npm cache clean --force

# Yarn
yarn cache clean

# pnpm
pnpm store prune

# pip
pip cache purge

# Maven
rm -rf ~/.m2/repository
```

### 4.3 Docker 清理

Docker 是常见的大户，通常占用 **5–15 GB**。优先使用 Docker 自带命令：

```bash
# 清理停止的容器、未使用的镜像、网络、构建缓存、未使用的卷
docker system prune -a --volumes
```

如需查看详情：

```bash
docker images
docker ps -a
docker volume ls
docker system df
```

> ⚠️ 不要直接删除 `~/Library/Containers/com.docker.docker`，可能导致 Docker Desktop 损坏。应先退出 Docker Desktop，再考虑删除。

### 4.4 IDE 与编辑器

```bash
# VS Code 不用的扩展
code --list-extensions          # 列出扩展
ncdu ~/.vscode/extensions       # 可视化查看并按需删除

# JetBrains 缓存（配置目录建议保留）
rm -rf ~/Library/Caches/JetBrains
```

### 4.5 移动开发相关

```bash
# Flutter 当前项目构建产物
flutter clean

# Gradle 缓存
rm -rf ~/.gradle/caches

# Android 模拟器数据（谨慎，会删除模拟器内数据）
rm -rf ~/.android/avd
```

---

## 5. 使用 ncdu 进行可视化清理

`ncdu` 是一款交互式磁盘分析工具，适合逐层深入查看空间占用。

### 5.1 安装

```bash
brew install ncdu
```

### 5.2 常用扫描命令

```bash
ncdu ~
ncdu ~/Library/Containers
ncdu ~/Library/Application\ Support
ncdu ~/Library/Caches
ncdu ~/.vscode/extensions
```

### 5.3 交互快捷键

| 按键 | 功能 |
|---|---|
| `↑` / `↓` | 移动选择 |
| `Enter` | 进入目录 |
| `q` | 退出 |
| `d` | 删除当前选中项（会二次确认） |
| `?` | 帮助 |

### 5.4 使用建议

1. 先用 `ncdu` 扫描 `~/Library/Containers`、`~/Library/Application Support`、`~/Library/Caches` 这三大目录。
2. 优先清理占比高、且明确是缓存的目录。
3. 遇到不确定的目录，先不要按 `d` 删除，搜索该目录名称确认作用。
4. 对于 Docker、微信、Notion 等应用，优先使用应用自带清理功能。

---

## 6. 谨慎删除的目录

以下目录可能包含重要用户数据，删除前应确认备份或无价值。

| 目录 | 内容 | 风险 |
|---|---|---|
| `~/Library/Containers/com.tencent.xinWeChat` | 微信聊天记录、图片、视频 | 删除后聊天记录丢失 |
| `~/Library/Application Support/Notion` | Notion 离线笔记与附件 | 离线内容会丢失，需重新同步 |
| `~/Library/Application Support/Google` | Chrome 用户数据、登录态、历史记录 | 会退出所有账号，丢失历史记录 |
| `~/Library/Application Support/JetBrains` | IDE 配置、插件、项目历史 | 配置和插件状态会重置 |
| `~/Library/Containers/com.tencent.QQMusicMac` | QQ 音乐下载的歌曲与缓存 | 下载歌曲会被删除 |
| `~/.android/avd` | Android 模拟器用户数据 | 模拟器内数据丢失 |
| `~/Library/Containers/com.docker.docker` | Docker 容器与镜像数据 | 应使用 `docker system prune` |

---

## 7. 推荐清理流程

### 第一步：快速扫描

```bash
du -sh ~/Library/* 2>/dev/null | sort -hr | head -20
du -sh ~/* ~/.[^.]* 2>/dev/null | sort -hr | head -20
```

### 第二步：执行安全清理

```bash
# Docker
docker system prune -a --volumes

# 包管理器缓存
brew cleanup
npm cache clean --force
yarn cache clean
pnpm store prune
pip cache purge

# IDE / 浏览器 / 系统缓存
rm -rf ~/Library/Caches/JetBrains
rm -rf ~/Library/Caches/Google
rm -rf ~/Library/Caches/com.microsoft.VSCode.ShipIt
rm -rf ~/Library/Caches/*

# Flutter 项目构建产物
flutter clean
```

### 第三步：用 ncdu 深入排查

```bash
ncdu ~/Library/Containers
ncdu ~/Library/Application\ Support
ncdu ~/Library/Caches
ncdu ~/.vscode/extensions
```

### 第四步：处理应用数据

- 微信：使用微信自带「设置 → 通用 → 存储空间」清理。
- Chrome：使用「设置 → 隐私和安全 → 清除浏览数据」。
- Notion：确认离线笔记已同步后再考虑清理缓存。

---

## 8. 日常维护习惯

1. **定期 `flutter clean`**：每个 Flutter 项目开发结束后执行。
2. **定期清理包管理器缓存**：每月执行一次 `brew cleanup`、`npm cache clean`、`pnpm store prune`。
3. **定期清理 Docker**：每月执行 `docker system prune -a --volumes`。
4. **慎用自动更新缓存**：及时清理 `~/Library/Caches/com.microsoft.VSCode.ShipIt` 等更新包。
5. **控制模拟器数量**：只保留必要的 Android/iOS 模拟器镜像。
6. **不用的扩展及时卸载**：特别是 AI 助手类、语言服务器类扩展。

---

## 9. 常见问题

### Q: 清理缓存会影响开发环境吗？

通常不会。缓存删除后，首次构建或打开应用时会重新生成，耗时可能稍长，但不会影响功能。

### Q: 磁盘突然爆满，最常见的原因是什么？

对于开发者，常见原因是：
- Docker 镜像和构建缓存累积
- Chrome/JetBrains/VS Code 缓存膨胀
- 微信聊天记录和文件缓存
- Homebrew、npm、pnpm 等包管理器缓存
- Android 模拟器系统镜像

### Q: 可以直接删除整个 `~/Library/Caches` 吗？

可以，但部分应用正在运行时可能无法删除某些文件。建议先退出占用大的应用（Chrome、Docker、IDE），再执行清理。

---

## 10. 参考命令速查

```bash
# 安装 ncdu
brew install ncdu

# 扫描三大目录
ncdu ~/Library/Containers
ncdu ~/Library/Application\ Support
ncdu ~/Library/Caches

# 安全清理组合
docker system prune -a --volumes
brew cleanup
npm cache clean --force
yarn cache clean
pnpm store prune
pip cache purge
rm -rf ~/Library/Caches/*
rm -rf ~/Library/Caches/JetBrains
rm -rf ~/Library/Caches/Google
rm -rf ~/Library/Caches/com.microsoft.VSCode.ShipIt
flutter clean
```
