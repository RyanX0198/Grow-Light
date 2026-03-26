# AeroGrow Pro - 智慧植物养护系统

🌱 智能植物照明灯控制与植物养护管理 APP

**产品定位**: 北美室内园艺智慧补光系统  
**目标价位**: $69.99 (BOM $38-42)  
**核心技术**: ESP32-S3 + PyTorch Mobile + AI视觉识别

---

## 🚀 快速开始

### 在线演示
GitHub Pages: https://ryanx0198.github.io/AeroGrow-Pro/

### 本地演示
```bash
# 方式1：直接打开
demos/demo-app-v1.0.html

# 方式2：本地服务器
cd demos
python3 -m http.server 8080
# 访问 http://localhost:8080
```

---

## 📁 项目结构

```
AeroGrow-Pro/
├── README.md                          # 本文件
├── DEPLOY.md                          # 部署指南
├── AeroGrow Pro - 智慧植物养护 APP PRD文档.md  # 产品需求文档
├── demos/                             # ⭐ 四个演示文件
│   ├── demo-app-v1.0.html            # 正式1.0 APP (64KB)
│   ├── demo-pricing.html             # 三档订阅对比 (8.2KB)
│   ├── demo-lab.html                 # 试验台演示 (23KB)
│   └── demo-experiment.html          # 实验方案演示 (15KB)
└── docs/                              # 📚 文档中心
    ├── README.md                     # 文档索引
    ├── PRD/APP-PRD-v1.0.md           # PRD文档副本
    └── deployment/DEPLOY.md          # 部署文档副本
```

---

## ✨ 核心功能

### 1. 植物管理
- DLI 光环境实时监测
- PPFD 时间曲线图（Canvas可视化）
- 植物健康状态追踪
- 位置优化建议

### 2. 灯光控制
- AR风格空间热力图（PPFD分布）
- 光强调节（100-800 μmol/m²s）
- 光谱控制（蓝光/色温/远红光）

### 3. 订阅服务（三档体系）

| 档位 | 价格 | 核心功能 |
|:---|:---|:---|
| **Seed 免费版** | $0 | 手动控制、基础光谱 |
| **萌芽版** | $4.99/月 | DLI智能闭环、AI识别 |
| **Bloom 盛放版** | $9.99/月 | AI病虫害医生、大模型问答 |

---

## 🛠️ 技术栈

- HTML5 + Tailwind CSS
- Canvas API 数据可视化
- 原生 JavaScript（单页应用）
- PWA 适配（移动端友好）

---

## 📚 文档

| 文档 | 路径 | 说明 |
|:---|:---|:---|
| **产品需求** | `AeroGrow Pro - 智慧植物养护 APP PRD文档.md` | 完整PRD规范 |
| **部署指南** | `DEPLOY.md` | 部署说明 |
| **文档索引** | `docs/README.md` | 文档中心导航 |

---

## 🔗 相关链接

- **GitHub仓库**: https://github.com/RyanX0198/AeroGrow-Pro
- **在线演示**: https://ryanx0198.github.io/AeroGrow-Pro/

---

*项目代号: AG | 核心理念: 全场景AI智慧生态补光系统*
