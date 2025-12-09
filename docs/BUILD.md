# Music Scraper 打包指南

## 一、环境要求

- Docker Desktop（已启用 buildx）
- macOS / Linux / Windows

## 二、构建命令

### 2.1 构建多架构镜像

```bash
# 进入项目目录
cd /path/to/music4docker

# 构建 amd64 版本（适配 x86 NAS，如绿联云 NAS）
docker buildx build --platform linux/amd64 -t music-scraper:amd64 . --load

# 构建 arm64 版本（适配 ARM NAS，如群晖 DS923+、树莓派）
docker buildx build --platform linux/arm64 -t music-scraper:arm64 . --load
```

### 2.2 导出镜像文件

```bash
# 创建输出目录
mkdir -p dist

# 导出 amd64 版本
docker save -o dist/music-scraper-amd64.tar music-scraper:amd64

# 导出 arm64 版本
docker save -o dist/music-scraper-arm64.tar music-scraper:arm64
```

### 2.3 一键打包脚本

```bash
#!/bin/bash
# build.sh - 一键打包脚本

set -e

echo "=== 构建 amd64 版本 ==="
docker buildx build --platform linux/amd64 -t music-scraper:amd64 . --load

echo "=== 构建 arm64 版本 ==="
docker buildx build --platform linux/arm64 -t music-scraper:arm64 . --load

echo "=== 导出镜像文件 ==="
mkdir -p dist
docker save -o dist/music-scraper-amd64.tar music-scraper:amd64
docker save -o dist/music-scraper-arm64.tar music-scraper:arm64

echo "=== 打包完成 ==="
ls -lh dist/music-scraper-*.tar
```

## 三、部署到 NAS

### 3.1 上传镜像

将对应架构的 `.tar` 文件上传到 NAS：
- x86 架构 NAS（绿联云等）→ `music-scraper-amd64.tar`
- ARM 架构 NAS（群晖 DS923+ 等）→ `music-scraper-arm64.tar`

### 3.2 导入镜像

```bash
# SSH 登录 NAS 后执行
docker load -i music-scraper-amd64.tar
# 或
docker load -i music-scraper-arm64.tar
```

### 3.3 创建并运行容器

```bash
docker run -d \
  --name music-scraper \
  --restart unless-stopped \
  -p 7301:7301 \
  -v /volume1/docker/music-scraper/data:/app/data \
  -v /volume1/music:/app/music \
  music-scraper:amd64
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `--name music-scraper` | 容器名称 |
| `--restart unless-stopped` | 自动重启策略 |
| `-p 7301:7301` | 端口映射（宿主机:容器） |
| `-v .../data:/app/data` | 数据持久化（数据库、授权信息） |
| `-v .../music:/app/music` | 音乐文件目录 |

### 3.4 自定义端口

如需使用其他端口，通过环境变量 `PORT` 指定：

```bash
docker run -d \
  --name music-scraper \
  -p 8080:8080 \
  -e PORT=8080 \
  -v /path/to/data:/app/data \
  -v /path/to/music:/app/music \
  music-scraper:amd64
```

## 四、访问应用

浏览器访问：`http://NAS_IP:7301`

## 五、版本信息

| 架构 | 文件名 | 适用设备 |
|------|--------|----------|
| amd64 | `music-scraper-amd64.tar` | x86 NAS、普通 PC |
| arm64 | `music-scraper-arm64.tar` | ARM NAS、树莓派、M1/M2 Mac |

## 六、注意事项

1. **数据持久化**：务必挂载 `/app/data` 目录，否则重启后数据丢失
2. **音乐目录**：挂载包含音乐文件的目录到 `/app/music`
3. **端口冲突**：如 7301 被占用，可自定义其他端口
4. **首次运行**：自动进入 30 天试用期

