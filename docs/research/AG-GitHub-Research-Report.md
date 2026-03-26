# GitHub 植物识别与病虫害检测开源项目调研报告

**调研日期**：2026年3月23日  
**调研范围**：GitHub 公开仓库  
**项目代号**：AG（AeroGrow Pro）

---

## 一、调研概述

### 1.1 搜索统计
| 类别 | 搜索结果数 | 高星项目数(>100) |
|:---|:---|:---|
| 病虫害检测 (plant disease detection) | 12,100+ | 8个 |
| 植物种类识别 (plant species identification) | 239 | 2个 |

### 1.2 核心发现
- **病虫害检测领域成熟度高**：项目数量多、 star数高、更新活跃
- **植物种类识别相对冷门**：项目少、技术门槛高、数据集稀缺
- **技术栈趋同**：PyTorch + CNN/ResNet 成为主流方案

---

## 二、病虫害检测项目详细分析

### 2.1 Top 10 项目清单

| 排名 | 项目名称 | Stars | 技术栈 | 特点 | 更新状态 |
|:---|:---|:---|:---|:---|:---|
| 1 | pratikkayal/PlantDoc-Dataset | 386 | Dataset | PlantDoc权威数据集，CODS-COMAD 2020 | 2021年停更 |
| 2 | manthan89-py/Plant-Disease-Detection | 359 | PyTorch, CNN | CNN叶片图像预测，代码完整 | **2025年活跃** |
| 3 | imskr/Plant_Disease_Detection | 355 | PyTorch, fastai | Web应用架构，多CNN支持 | 2023年 |
| 4 | spytensor/plants_disease_detection | 340 | Python | 2018 AI Challenger农作物病害 | 2020年停更 |
| 5 | johri-lab/Automatic-leaf-infection-identifier | 308 | Python, OpenCV | 叶片感染自动识别，轻量级 | 2023年 |
| 6 | farmassistX/farmassist | 234 | Flutter, Firebase | IoT+AI综合农业App | **2024年活跃** |
| 7 | mehra-deepak/Plant-Disease-Detection | 208 | Jupyter | 农业技术综述，理论性强 | 2022年 |
| 8 | soumyajit4419/Plant_AI | 185 | PyTorch, CNN | 多种CNN架构对比实验 | 2021年 |
| 9 | nandakishormpai/Plant_Disease_Detector | 130 | Flutter, Flask | 移动端App+后端API | 2022年 |
| 10 | arpit0891/Plant-Disease-Detection-Web-application | 114 | FastAI, ResNet34 | FastAI框架Web应用 | **2024年更新** |

### 2.2 技术路线分析

**主流技术栈**：
- **深度学习框架**：PyTorch (60%) > TensorFlow (25%) > FastAI (15%)
- **模型架构**：ResNet > CNN > AlexNet > VGG
- **部署方式**：Web (Flask/Django) > Mobile (Flutter/Android) > Desktop

**数据集情况**：
- PlantDoc：最权威，包含多种作物病害
- AI Challenger 2018：农作物病害大规模数据集
- 自定义叶片数据集：轻量级项目常用

---

## 三、植物种类识别项目分析

### 3.1 Top 项目清单

| 排名 | 项目名称 | Stars | 技术栈 | 特点 |
|:---|:---|:---|:---|:---|
| 1 | joergmlpts/nature-id | 54 | Python, ML | 植物/鸟类/昆虫综合识别，分类学层级 |
| 2 | MohFahmi27/Identification-of-Herbal-Medicine-Plant-Leaf | 23 | Android | 草药植物叶片识别（印尼3万+物种） |
| 3 | hughpearse/Sato-folium | 19 | Shell | 植物叶片图像识别 |
| 4 | manuchopra/TreeID | 16 | C | 斯坦福校园6万棵树识别 |
| 5 | naneja/plants | 15 | PyTorch, AlexNet | CNN迁移学习 |
| 6 | Tejas242/FloraFauna-ai | 15 | JavaScript, GenAI | GenAI驱动，2024年新项目 |
| 7 | Mahaning/ML_Plant_Clasification_Using_Leaf | 12 | Flask, ResNet9 | Web应用 |
| 8 | jayesh-srivastava/GreenStems | 9 | Kotlin | 综合App（病害+花卉+土壤） |

### 3.2 技术难点

1. **物种数量庞大**：全球植物30万+物种，分类粒度难把握
2. **特征差异小**：同属植物叶片形态相似度高
3. **数据集稀缺**：大规模标注数据集极少
4. **跨域问题**：不同光照/角度/季节影响识别准确率

---

## 四、关键项目深度解析

### 4.1 farmassistX/farmassist ⭐ 推荐
- **定位**：IoT+AI综合农业应用
- **技术**：Flutter + Firebase + 边缘AI
- **特点**：完整的移动端+云端架构，与AG产品形态最匹配
- **参考价值**：整体架构设计、移动端模型部署

### 4.2 manthan89-py/Plant-Disease-Detection ⭐ 推荐
- **定位**：轻量级CNN病害检测
- **技术**：PyTorch + CNN
- **特点**：代码简洁、训练流程完整、2025年活跃维护
- **参考价值**：模型训练逻辑、数据预处理流程

### 4.3 joergmlpts/nature-id
- **定位**：多物种自然识别（植物/鸟/昆虫）
- **技术**：Python + 分类学数据库
- **特点**：分类学层级设计，支持科/属/种多级识别
- **参考价值**：AG可借鉴其分类学层级架构

---

## 五、对 AG 项目的建议

### 5.1 技术选型建议

| 模块 | 推荐方案 | 理由 |
|:---|:---|:---|
| **深度学习框架** | PyTorch Mobile | 移动端部署成熟，支持iOS/Android |
| **病害检测模型** | ResNet18/34 轻量化 | 精度与速度平衡，适合边缘设备 |
| **种类识别模型** | MobileNetV3 | 超轻量级，适合实时识别 |
| **部署方案** | 云端+边缘混合 | 复杂模型放云端，简单识别本地执行 |

### 5.2 开发路线建议

**Phase 1：病害检测（优先）**
- 基于 PlantDoc 数据集训练 ResNet 模型
- 支持常见室内观叶植物病害（红蜘蛛、白粉病、脱水）
- 目标：5-10种病害，识别准确率>85%

**Phase 2：种类识别（其次）**
- 缩小范围至 Top 50 常见室内植物
- 采用迁移学习，基于 ImageNet 预训练模型微调
- 目标：50种植物，Top-5准确率>90%

**Phase 3：综合优化**
- 整合两个模型，构建统一识别流水线
- 结合 ToF 距离信息，提升近景识别准确率

### 5.3 参考项目优先级

| 优先级 | 项目 | 参考重点 |
|:---|:---|:---|
| P0 | farmassistX/farmassist | 移动端架构、IoT集成 |
| P0 | manthan89-py/Plant-Disease-Detection | CNN模型、训练流程 |
| P1 | imskr/Plant_Disease_Detection | Web后端架构 |
| P1 | joergmlpts/nature-id | 分类学层级设计 |
| P2 | PlantDoc-Dataset | 数据集构建 |

---

## 六、风险与挑战

| 风险 | 影响 | 缓解措施 |
|:---|:---|:---|
| 训练数据不足 | 模型泛化能力差 | 利用数据增强、迁移学习 |
| 移动端算力限制 | 推理速度慢 | 模型量化、剪枝、使用轻量级架构 |
| 多物种识别精度低 | 用户体验差 | 先聚焦常见植物，逐步扩展 |
| 与光控系统集成复杂 | 开发周期长 | 先独立开发AI模块，后集成 |

---

## 七、结论

GitHub 上**病虫害检测**开源生态成熟，有大量可参考项目；**植物种类识别**项目较少，技术难度较高。

**建议 AG 项目**：
1. **优先实现病害检测功能**，利用现有成熟方案快速落地
2. **种类识别采用保守策略**，先支持 Top 50 常见室内植物
3. **参考 farmassist 架构**，构建 Flutter + 边缘AI + 云端混合方案

---

**报告完成时间**：2026-03-23 11:30  
**调研者**：卡维斯 (KAVIS)
