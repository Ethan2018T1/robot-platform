# 🤖 空地协同管理平台 (Robot Platform)

> 集无人机、机器狗、无人车、机器人、无人船、机械手臂等设备统一管理的综合平台

## 🏗️ 系统架构

```
┌────────────────────────────────────────────────────────────────┐
│                    React 管理前端 (:3000)                       │
│  态势总览 │ 设备管理 │ 任务调度 │ 实时监控 │ 业务应用中心          │
├────────────────────────────────────────────────────────────────┤
│               Spring Boot 后端 API (:8080)                     │
│  /api/devices │ /api/tasks │ /api/drones │ WebSocket            │
├──────────┬──────────┬──────────┬──────────┬────────────────────┤
│ Open-RMF │  MAVSDK  │ ROS 2    │ Vizanti  │     Zenoh          │
│ 总调度    │ 无人机   │ Nav2     │ 3D大屏   │    通信链路         │
│ (:8083)  │ (:50051) │ (:9090)  │ (:5000)  │   (:7447)         │
├──────────┴──────────┴──────────┴──────────┴────────────────────┤
│           MySQL (:3306)  │  Redis (:6379)                      │
└────────────────────────────────────────────────────────────────┘
```

## 📦 技术栈与开源集成

| 层级 | 项目 | 作用 | 端口 |
|------|------|------|------|
| 总调度 | **Open-RMF** | 多机器人任务分发、防撞 | 8083, 3001 |
| 无人机 | **MAVSDK** (Python) | 控制无人机起飞/巡航/拍照 | 50051 |
| 机器狗/车 | **ROS 2 + Nav2** | 底盘移动、避障、SLAM建图 | 9090 |
| 可视化 | **Vizanti** / Foxglove | 3D地图、视频流 | 5000 |
| 通信 | **Zenoh** | 断网容忍、低延迟通信 | 7447, 8000 |
| 前端 | React + Ant Design | 管理平台 Web 界面 | 3000 |
| 后端 | Spring Boot 3 | REST API + WebSocket | 8080 |
| 数据库 | MySQL 8 + Redis | 数据持久化 + 缓存 | 3306, 6379 |

## 🚀 快速开始 (Docker)

### 前置要求

- Docker Desktop (Mac)
- Docker Compose v2

### 一键启动

```bash
# 克隆项目
cd robot-platform

# 启动所有服务 (首先启动基础设施)
docker compose up -d mysql redis zenoh-router

# 等待 MySQL 就绪后启动全部
docker compose up -d
```

### 分步启动 (推荐开发时)

```bash
# 1. 基础设施
docker compose up -d mysql redis zenoh-router

# 2. ROS 2 + RMF (较重，需要较长构建时间)
docker compose up -d ros2-nav2 rmf-core

# 3. MAVSDK + Vizanti
docker compose up -d mavsdk-service vizanti

# 4. 后端 + 前端
docker compose up -d backend frontend
```

### 访问地址

| 服务 | 地址 |
|------|------|
| 📊 管理平台 | http://localhost:3000 |
| 📡 API 文档 (Swagger) | http://localhost:8080/swagger-ui.html |
| 🗺️ Vizanti 3D | http://localhost:5000 |
| 🤖 RMF Dashboard | http://localhost:3001 |
| 🔗 Zenoh REST | http://localhost:8000 |

## 📋 业务应用

| 应用 | 状态 | 涉及设备 | 使用组件 |
|------|------|----------|----------|
| ✅ 机器狗巡检 | 已上线 | 机器狗 | ROS 2 Nav2 + Open-RMF |
| ✅ 厂区巡逻 | 已上线 | 机器狗 + 无人车 | Open-RMF 协同调度 |
| 🧪 空地协同采样 | 测试中 | 无人机 + 机器狗 | MAVSDK + Nav2 + Zenoh |
| ✅ 管道航拍巡检 | 已上线 | 无人机 | MAVSDK |
| 🔜 水域巡查 | 即将推出 | 无人船 | — |
| 🔜 自动化装配 | 即将推出 | 机械手臂 | — |

## 📁 项目结构

```
robot-platform/
├── docker-compose.yml          # Docker 编排
├── .env                        # 环境变量
├── ros2/                       # ROS 2 + Nav2 控制
│   ├── Dockerfile
│   ├── entrypoint.sh
│   ├── config/nav2_params.yaml
│   └── maps/
├── rmf/                        # Open-RMF 调度
│   ├── Dockerfile
│   ├── entrypoint.sh
│   └── config/fleet_config.yaml
├── mavsdk/                     # MAVSDK 无人机
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── src/main.py
│   └── missions/
├── zenoh/                      # Zenoh 通信
│   └── zenoh.json5
├── vizanti/                    # Vizanti 大屏
│   ├── Dockerfile
│   └── entrypoint.sh
├── frontend/                   # React 前端
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── backend/                    # Spring Boot 后端
│   ├── Dockerfile
│   ├── pom.xml
│   ├── sql/init.sql
│   └── src/
└── README.md
```

## 🔧 开发指南

### 离线/弱网场景

通过 Zenoh 桥接实现通信可靠性:
- 设备端运行 Zenoh Client，连接到 Zenoh Router
- 支持 WiFi/4G/5G 多链路切换
- 断网后自动缓存，恢复后同步

### 添加新设备类型

1. 后端: 在 `DeviceType` 枚举中添加新类型
2. 前端: 在 `types/index.ts` 中同步更新
3. ROS 2: 创建对应的驱动节点

### 开发新业务应用

1. 在前端 `pages/` 下创建应用页面
2. 在后端创建对应的 Controller/Service
3. 通过 Open-RMF API 编排多设备任务

## 📄 License

MIT
