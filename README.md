# MusicOn 🎵

<div align="center">
  <h3>面向云盘音乐库管理的现代个人音乐中心</h3>
  <p>支持 AList 挂载、FastAPI 极速扫描、Subsonic 协议接入以及 Vue 3 现代化管理后台</p>

  <a href="https://hub.docker.com/r/geelonn/musicon">
    <img src="https://img.shields.io/docker/pulls/geelonn/musicon.svg" alt="Docker Pulls">
  </a>
  <a href="https://hub.docker.com/r/geelonn/musicon">
    <img src="https://img.shields.io/docker/image-size/geelonn/musicon/latest" alt="Docker Image Size">
  </a>
  <a href="https://github.com/gefl24/MusicOn/actions">
    <img src="https://github.com/gefl24/MusicOn/actions/workflows/docker-publish.yml/badge.svg" alt="Build Status">
  </a>
  <a href="https://github.com/gefl24/MusicOn/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License">
  </a>
</div>

## 🌟 核心特性

- **🚀 异步极速扫描**：采用 Python 异步 I/O (AsyncIO) 机制构建强大的 AList 扫描器，无缝解析网盘海量云端音乐。扫描任务运行于后台，不阻塞主应用响应。
- **📱 完美兼容 Subsonic**：内置兼容 Subsonic API (v1.16.1)，你可以使用任意支持 Subsonic 协议的客户端（如音流、Navidrome、DSub、PlaySub）全平台无缝串流。
- **🌐 零流量云端直链播放**：创新架构设计！播放音频时，后端会智能重定向 (302) 到云盘直链，听无损音乐**完全不消耗本地服务器带宽与流量**，且无需磁盘缓存。
- **👑 全网榜单白嫖 (v1.0.0 新特性)**：对接 LX-Server，每日自动拉取全网 5 大核心音乐榜单（热歌/新歌/抖音神曲等）。即便你网盘没有这些歌，依然能通过流媒体劫持在客户端直接秒播！扫描网盘时，还会自动将虚拟在线歌曲替换为你的本地无损高音质版本。
- **🖼️ 智能缩略图优化**：内建基于 Pillow 的图像引擎，自动将提取的歌曲封面压缩为极小体积的 WebP 格式，配合前端虚拟列表与懒加载，万首曲库依然丝滑。
- **💻 现代化 Web 控制台**：基于 Vue 3 构建的响应式控制台。支持曲库管理、批量删除清理、播放历史统计与扫描任务实时监控。
- **📦 极简部署体验**：后端 FastAPI + 前端静态托管同源打包。只需配合一个 PostgreSQL 数据库，一键 Docker Compose 拉起。官方提供 `amd64` / `arm64` 双架构镜像。

## 📸 界面预览
![首页](https://github.com/gefl24/music-on/blob/main/%E9%A1%B5%E9%9D%A21.png)
![歌单页](https://github.com/gefl24/music-on/blob/main/%E9%A1%B5%E9%9D%A22.png)
## 🚀 快速部署 (Docker Compose)

最简单、最推荐的部署方式是使用 Docker Compose。

1. **新建配置文件**：
   创建一个目录，并在其中创建 `docker-compose.yml` 和 `.env`。

   **docker-compose.yml**
   ```yaml
   version: '3.8'

   services:
     musicon:
       image: geelonn/musicon:latest
       container_name: musicon-app
       restart: always
       ports:
         - "8000:8000"
       depends_on:
         postgres:
           condition: service_healthy
       environment:
         DATABASE_URL: postgresql+psycopg://musicon:${POSTGRES_PASSWORD}@postgres:5432/musicon
         SECRET_KEY: ${SECRET_KEY} # 会话签名密钥
       volumes:
         - ./musicon_cover_cache:/app/data/covers
   
     postgres:
       image: postgres:15-alpine
       container_name: musicon-postgres
       restart: always
       environment:
         POSTGRES_USER: musicon
         POSTGRES_PASSWORD: ${POSTGRES_PASSWORD} # 从 .env 加密读取
         POSTGRES_DB: musicon
       volumes:
         - ./musicon_postgres_data:/var/lib/postgresql/data
       healthcheck:
         test: ["CMD-SHELL", "pg_isready -U musicon -d musicon"]
         interval: 5s
         timeout: 5s
         retries: 5

   volumes:
     musicon_postgres_data:
     musicon_cover_cache:
   ```

   **.env**
   ```env
   # 强烈建议使用随机长字符串替换以下默认值
   POSTGRES_PASSWORD=your_super_secret_db_password_here
   SECRET_KEY=your_super_secret_session_key_here
   ```

2. **一键启动**：
   ```bash
   docker compose up -d
   ```

3. **访问控制台**：
   服务启动后，打开浏览器访问 `http://localhost:8000` 即可进入 Web 管理后台。

## ⚙️ 初始化配置与使用

1. **安全配置**：默认密码 changeme ，首次登录后请立即前往【系统设置】配置你的 **Web 管理密码** 和 **Subsonic 客户端连接密码**。
2. **对接 AList**：在【系统设置】中填入你的 AList 根地址（如 `http://192.168.x.x:5244`）以及具有相应目录读取权限的 AList Token。
3. **导入音乐**：前往【扫描任务】页面，填入 AList 中存放音乐的挂载路径（如 `/阿里云盘/Music`）。点击新建扫描，后台会自动异步提取元数据。
4. **配置 LX-Server (选填)**：若你想白嫖全网热歌榜单的在线流媒体播放，请在【扫描任务】顶部填入你部署的 LX-Server 地址，并点击“刷新网络榜单”。
5. **客户端连接 (Subsonic)**：
   - **服务器地址**: `http://你的IP:8000`
   - **用户名**: 默认 `admin`（可在系统设置中修改）
   - **密码**: 在【系统设置】中配置的 Subsonic 密码



## 🤝 参与贡献

非常欢迎提交 Pull Request (PR) 或者提出 Issue。无论是功能建议、Bug 修复，还是文档完善，都能帮助 MusicOn 变得更好！

## 📄 开源协议

本项目基于 [MIT License](LICENSE) 开源。
