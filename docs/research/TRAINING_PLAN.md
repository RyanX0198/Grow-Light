# AG 项目 - 室内植物病害检测模型训练方案

## 一、数据集下载

### 1.1 数据集信息
- **数据集名称**: Indoor Plant Disease Detection Dataset
- **Kaggle链接**: https://www.kaggle.com/datasets/abdulahad0296/indoor-plant-disease-detection-dataset
- **大小**: 2 GB
- **图片数量**: 21,097 张
- **类别**: 16 类（5种植物 + 健康/病害状态）

### 1.2 下载方法

#### 方法 1: 使用 Kaggle CLI (推荐)

```bash
# 1. 安装 Kaggle CLI
pip3 install kaggle

# 2. 配置 API 密钥
# 访问 https://www.kaggle.com/account  -> Create New API Token
# 将下载的 kaggle.json 移动到 ~/.kaggle/
mkdir -p ~/.kaggle
mv ~/Downloads/kaggle.json ~/.kaggle/
chmod 600 ~/.kaggle/kaggle.json

# 3. 下载数据集
cd ~/.openclaw/workspace-backend-lead/aerogrow-demo/dataset
kaggle datasets download -d abdulahad0296/indoor-plant-disease-detection-dataset

# 4. 解压
unzip indoor-plant-disease-detection-dataset.zip
```

#### 方法 2: 手动下载
1. 访问 https://www.kaggle.com/datasets/abdulahad0296/indoor-plant-disease-detection-dataset
2. 点击 "Download" 按钮
3. 将下载的 zip 文件移动到 `dataset/` 目录
4. 解压: `unzip indoor-plant-disease-detection-dataset.zip`

### 1.3 数据集结构

```
dataset/
└── indoor/
    ├── train/          # 训练集
    │   ├── Aloe_Healthy/
    │   ├── Aloe_Rust/
    │   ├── Aloe_Anthracnose/
    │   ├── Aloe_LeafSpot/
    │   ├── Aloe_SunBurn/
    │   ├── Cactus_Healthy/
    │   ├── Cactus_DactylopiusOpuntia/
    │   ├── MoneyPlant_Healthy/
    │   ├── MoneyPlant_BacterialWilt/
    │   ├── MoneyPlant_ManganeseToxicity/
    │   ├── SnakePlant_Healthy/
    │   ├── SnakePlant_Anthracnose/
    │   ├── SnakePlant_LeafWithering/
    │   ├── SpiderPlant_Healthy/
    │   ├── SpiderPlant_FungalLeafSpot/
    │   └── SpiderPlant_LeafTipNecrosis/
    ├── test/           # 测试集 (⚠️ 需要过滤增强图像)
    └── valid/          # 验证集 (⚠️ 需要过滤增强图像)
```

---

## 二、数据预处理

### 2.1 过滤增强图像（重要！）

测试集和验证集中的芦荟类别包含增强图像，需要过滤掉：

```python
# filter_augmented.py
import os
import shutil
from pathlib import Path

def remove_augmented_images(directory):
    """删除以 'augmented' 开头的图像文件"""
    removed_count = 0
    for root, dirs, files in os.walk(directory):
        # 只处理芦荟相关文件夹
        if 'Aloe' in root:
            for file in files:
                if file.lower().startswith('augmented'):
                    file_path = os.path.join(root, file)
                    os.remove(file_path)
                    removed_count += 1
                    print(f"Removed: {file_path}")
    print(f"\nTotal removed: {removed_count} augmented images")

# 执行过滤
if __name__ == "__main__":
    base_path = "dataset/indoor"
    remove_augmented_images(os.path.join(base_path, "test"))
    remove_augmented_images(os.path.join(base_path, "valid"))
```

### 2.2 数据统计

```python
# dataset_stats.py
import os
from collections import Counter

def count_images(directory):
    """统计各文件夹图像数量"""
    class_counts = {}
    for class_name in sorted(os.listdir(directory)):
        class_path = os.path.join(directory, class_name)
        if os.path.isdir(class_path):
            count = len([f for f in os.listdir(class_path) 
                        if f.lower().endswith(('.jpg', '.jpeg', '.png'))])
            class_counts[class_name] = count
    return class_counts

# 统计
base_path = "dataset/indoor"
for split in ['train', 'test', 'valid']:
    print(f"\n{split.upper()}:")
    counts = count_images(os.path.join(base_path, split))
    for cls, count in sorted(counts.items()):
        print(f"  {cls}: {count}")
    print(f"  Total: {sum(counts.values())}")
```

---

## 三、模型训练

### 3.1 环境准备

```bash
# 创建虚拟环境
python3 -m venv ag_venv
source ag_venv/bin/activate

# 安装依赖
pip install torch torchvision torchaudio
pip install timm  # 用于 EfficientNet
pip install onnx onnxruntime
pip install tensorflow  # 用于 TFLite 转换
pip install matplotlib seaborn scikit-learn
pip install tqdm tensorboard
```

### 3.2 训练脚本

```python
# train.py
import os
import torch
import torch.nn as nn
from torch.utils.data import DataLoader
from torchvision import datasets, transforms, models
import timm
from tqdm import tqdm
import json
from sklearn.metrics import classification_report, confusion_matrix
import matplotlib.pyplot as plt
import seaborn as sns

# 配置
CONFIG = {
    'data_dir': 'dataset/indoor',
    'batch_size': 32,
    'num_epochs': 50,
    'learning_rate': 1e-4,
    'image_size': 224,
    'num_classes': 16,
    'device': 'cuda' if torch.cuda.is_available() else 'cpu',
    'model_name': 'efficientnet_b0',  # 轻量级，适合边缘设备
    'save_dir': 'models'
}

# 数据增强
train_transform = transforms.Compose([
    transforms.Resize((CONFIG['image_size'], CONFIG['image_size'])),
    transforms.RandomRotation(20),
    transforms.RandomHorizontalFlip(),
    transforms.RandomVerticalFlip(),
    transforms.ColorJitter(brightness=0.2, contrast=0.2),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], 
                        std=[0.229, 0.224, 0.225])
])

val_transform = transforms.Compose([
    transforms.Resize((CONFIG['image_size'], CONFIG['image_size'])),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], 
                        std=[0.229, 0.224, 0.225])
])

# 加载数据
def load_data():
    train_dataset = datasets.ImageFolder(
        os.path.join(CONFIG['data_dir'], 'train'),
        transform=train_transform
    )
    val_dataset = datasets.ImageFolder(
        os.path.join(CONFIG['data_dir'], 'valid'),
        transform=val_transform
    )
    test_dataset = datasets.ImageFolder(
        os.path.join(CONFIG['data_dir'], 'test'),
        transform=val_transform
    )
    
    train_loader = DataLoader(train_dataset, batch_size=CONFIG['batch_size'], 
                              shuffle=True, num_workers=4)
    val_loader = DataLoader(val_dataset, batch_size=CONFIG['batch_size'], 
                           shuffle=False, num_workers=4)
    test_loader = DataLoader(test_dataset, batch_size=CONFIG['batch_size'], 
                            shuffle=False, num_workers=4)
    
    return train_loader, val_loader, test_loader, train_dataset.classes

# 创建模型
def create_model():
    # 使用 EfficientNet-B0 (轻量级，适合移动端)
    model = timm.create_model(CONFIG['model_name'], pretrained=True)
    
    # 修改分类头
    if 'efficientnet' in CONFIG['model_name']:
        model.classifier = nn.Linear(model.classifier.in_features, CONFIG['num_classes'])
    else:
        model.fc = nn.Linear(model.fc.in_features, CONFIG['num_classes'])
    
    return model.to(CONFIG['device'])

# 训练函数
def train_model(model, train_loader, val_loader, classes):
    criterion = nn.CrossEntropyLoss()
    optimizer = torch.optim.AdamW(model.parameters(), lr=CONFIG['learning_rate'])
    scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(optimizer, 'min', patience=3)
    
    best_val_acc = 0.0
    history = {'train_loss': [], 'train_acc': [], 'val_loss': [], 'val_acc': []}
    
    for epoch in range(CONFIG['num_epochs']):
        # 训练阶段
        model.train()
        train_loss = 0.0
        train_correct = 0
        train_total = 0
        
        pbar = tqdm(train_loader, desc=f'Epoch {epoch+1}/{CONFIG['num_epochs']}')
        for inputs, labels in pbar:
            inputs, labels = inputs.to(CONFIG['device']), labels.to(CONFIG['device'])
            
            optimizer.zero_grad()
            outputs = model(inputs)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()
            
            train_loss += loss.item()
            _, predicted = outputs.max(1)
            train_total += labels.size(0)
            train_correct += predicted.eq(labels).sum().item()
            
            pbar.set_postfix({'loss': f'{loss.item():.4f}'})
        
        train_loss = train_loss / len(train_loader)
        train_acc = 100. * train_correct / train_total
        
        # 验证阶段
        model.eval()
        val_loss = 0.0
        val_correct = 0
        val_total = 0
        
        with torch.no_grad():
            for inputs, labels in val_loader:
                inputs, labels = inputs.to(CONFIG['device']), labels.to(CONFIG['device'])
                outputs = model(inputs)
                loss = criterion(outputs, labels)
                
                val_loss += loss.item()
                _, predicted = outputs.max(1)
                val_total += labels.size(0)
                val_correct += predicted.eq(labels).sum().item()
        
        val_loss = val_loss / len(val_loader)
        val_acc = 100. * val_correct / val_total
        
        scheduler.step(val_loss)
        
        # 记录历史
        history['train_loss'].append(train_loss)
        history['train_acc'].append(train_acc)
        history['val_loss'].append(val_loss)
        history['val_acc'].append(val_acc)
        
        print(f'Epoch {epoch+1}: Train Loss={train_loss:.4f}, Train Acc={train_acc:.2f}%, '
              f'Val Loss={val_loss:.4f}, Val Acc={val_acc:.2f}%')
        
        # 保存最佳模型
        if val_acc > best_val_acc:
            best_val_acc = val_acc
            os.makedirs(CONFIG['save_dir'], exist_ok=True)
            torch.save({
                'epoch': epoch,
                'model_state_dict': model.state_dict(),
                'optimizer_state_dict': optimizer.state_dict(),
                'val_acc': val_acc,
                'classes': classes
            }, os.path.join(CONFIG['save_dir'], 'best_model.pth'))
            print(f'Saved best model with val_acc: {val_acc:.2f}%')
    
    return history

# 评估函数
def evaluate_model(model, test_loader, classes):
    model.eval()
    all_preds = []
    all_labels = []
    
    with torch.no_grad():
        for inputs, labels in tqdm(test_loader, desc='Testing'):
            inputs = inputs.to(CONFIG['device'])
            outputs = model(inputs)
            _, predicted = outputs.max(1)
            all_preds.extend(predicted.cpu().numpy())
            all_labels.extend(labels.numpy())
    
    # 打印分类报告
    print("\nClassification Report:")
    print(classification_report(all_labels, all_preds, target_names=classes))
    
    # 绘制混淆矩阵
    cm = confusion_matrix(all_labels, all_preds)
    plt.figure(figsize=(12, 10))
    sns.heatmap(cm, annot=True, fmt='d', cmap='Blues', xticklabels=classes, yticklabels=classes)
    plt.title('Confusion Matrix')
    plt.xlabel('Predicted')
    plt.ylabel('True')
    plt.tight_layout()
    plt.savefig('confusion_matrix.png')
    print("Confusion matrix saved to confusion_matrix.png")

# 主函数
if __name__ == '__main__':
    print(f"Using device: {CONFIG['device']}")
    
    # 加载数据
    train_loader, val_loader, test_loader, classes = load_data()
    print(f"Classes: {classes}")
    
    # 创建模型
    model = create_model()
    print(f"Model: {CONFIG['model_name']}")
    print(f"Total parameters: {sum(p.numel() for p in model.parameters()) / 1e6:.2f}M")
    
    # 训练
    history = train_model(model, train_loader, val_loader, classes)
    
    # 加载最佳模型并测试
    checkpoint = torch.load(os.path.join(CONFIG['save_dir'], 'best_model.pth'))
    model.load_state_dict(checkpoint['model_state_dict'])
    evaluate_model(model, test_loader, classes)
    
    print("\nTraining completed!")
```

---

## 四、模型转换 (PyTorch -> TFLite)

### 4.1 导出为 ONNX

```python
# export_onnx.py
import torch
import torch.onnx

def export_to_onnx(model_path, output_path):
    # 加载模型
    checkpoint = torch.load(model_path, map_location='cpu')
    
    # 重新创建模型结构
    import timm
    model = timm.create_model('efficientnet_b0', pretrained=False, num_classes=16)
    model.load_state_dict(checkpoint['model_state_dict'])
    model.eval()
    
    # 创建虚拟输入
    dummy_input = torch.randn(1, 3, 224, 224)
    
    # 导出 ONNX
    torch.onnx.export(
        model,
        dummy_input,
        output_path,
        export_params=True,
        opset_version=11,
        do_constant_folding=True,
        input_names=['input'],
        output_names=['output'],
        dynamic_axes={'input': {0: 'batch_size'}, 'output': {0: 'batch_size'}}
    )
    print(f"ONNX model exported to: {output_path}")

if __name__ == '__main__':
    export_to_onnx('models/best_model.pth', 'models/plant_disease.onnx')
```

### 4.2 ONNX -> TFLite (INT8 量化)

```python
# convert_tflite.py
import onnx
from onnx_tf.backend import prepare
import tensorflow as tf
import numpy as np

def representative_dataset():
    """代表性数据集用于INT8量化"""
    for _ in range(100):
        data = np.random.rand(1, 224, 224, 3).astype(np.float32)
        yield [data]

def convert_to_tflite(onnx_path, tflite_path):
    # 加载 ONNX 模型
    onnx_model = onnx.load(onnx_path)
    
    # 转换为 TensorFlow
    tf_rep = prepare(onnx_model)
    tf_rep.export_graph('models/tf_model')
    
    # 转换为 TFLite 并量化
    converter = tf.lite.TFLiteConverter.from_saved_model('models/tf_model')
    converter.optimizations = [tf.lite.Optimize.DEFAULT]
    converter.representative_dataset = representative_dataset
    converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
    converter.inference_input_type = tf.uint8
    converter.inference_output_type = tf.float32
    
    tflite_model = converter.convert()
    
    # 保存
    with open(tflite_path, 'wb') as f:
        f.write(tflite_model)
    
    print(f"TFLite model saved to: {tflite_path}")
    print(f"Model size: {len(tflite_model) / 1024 / 1024:.2f} MB")

if __name__ == '__main__':
    convert_to_tflite('models/plant_disease.onnx', 'models/plant_disease_int8.tflite')
```

---

## 五、模型验证

```python
# validate_tflite.py
import tensorflow as tf
import numpy as np
from PIL import Image

def load_tflite_model(model_path):
    interpreter = tf.lite.Interpreter(model_path=model_path)
    interpreter.allocate_tensors()
    return interpreter

def predict(interpreter, image_path, class_names):
    # 获取输入输出索引
    input_details = interpreter.get_input_details()
    output_details = interpreter.get_output_details()
    
    # 加载和预处理图像
    img = Image.open(image_path).resize((224, 224))
    img_array = np.array(img, dtype=np.uint8)
    img_array = np.expand_dims(img_array, axis=0)
    
    # 推理
    interpreter.set_tensor(input_details[0]['index'], img_array)
    interpreter.invoke()
    output = interpreter.get_tensor(output_details[0]['index'])
    
    # 获取预测结果
    predicted_idx = np.argmax(output[0])
    confidence = output[0][predicted_idx]
    
    return class_names[predicted_idx], confidence

if __name__ == '__main__':
    # 类别名称
    classes = [
        'Aloe_Anthracnose', 'Aloe_Healthy', 'Aloe_LeafSpot', 'Aloe_Rust', 'Aloe_SunBurn',
        'Cactus_DactylopiusOpuntia', 'Cactus_Healthy',
        'MoneyPlant_BacterialWilt', 'MoneyPlant_Healthy', 'MoneyPlant_ManganeseToxicity',
        'SnakePlant_Anthracnose', 'SnakePlant_Healthy', 'SnakePlant_LeafWithering',
        'SpiderPlant_FungalLeafSpot', 'SpiderPlant_Healthy', 'SpiderPlant_LeafTipNecrosis'
    ]
    
    # 加载模型
    interpreter = load_tflite_model('models/plant_disease_int8.tflite')
    
    # 测试
    test_image = 'dataset/indoor/test/Aloe_Healthy/sample.jpg'
    prediction, confidence = predict(interpreter, test_image, classes)
    print(f"Prediction: {prediction}, Confidence: {confidence:.4f}")
```

---

## 六、部署到 ESP32-S3

### 6.1 模型大小优化

转换后的 INT8 量化模型预期大小：
- **EfficientNet-B0**: ~5 MB (原始) -> ~1.3 MB (INT8量化)
- **MobileNet-V3**: ~2.5 MB (原始) -> ~0.8 MB (INT8量化) ⭐ 推荐

如果模型仍然太大，考虑：
1. 使用 **MobileNet-V3-Small** 替代 EfficientNet
2. 进一步剪枝 (Pruning)
3. 知识蒸馏 (Knowledge Distillation)

### 6.2 ESP32-S3 部署代码框架

```cpp
// main.cpp (ESP32-S3)
#include <TensorFlowLite_ESP32.h>
#include "model_data.h"  // 包含转换后的 TFLite 模型

// 模型配置
constexpr int kTensorArenaSize = 2 * 1024 * 1024;  // 2MB Tensor Arena
alignas(16) uint8_t tensor_arena[kTensorArenaSize];

// 类别名称
const char* kClassNames[] = {
    "Aloe_Anthracnose", "Aloe_Healthy", "Aloe_LeafSpot", "Aloe_Rust", "Aloe_SunBurn",
    "Cactus_DactylopiusOpuntia", "Cactus_Healthy",
    "MoneyPlant_BacterialWilt", "MoneyPlant_Healthy", "MoneyPlant_ManganeseToxicity",
    "SnakePlant_Anthracnose", "SnakePlant_Healthy", "SnakePlant_LeafWithering",
    "SpiderPlant_FungalLeafSpot", "SpiderPlant_Healthy", "SpiderPlant_LeafTipNecrosis"
};

void setup() {
    // 初始化串口
    Serial.begin(115200);
    
    // 加载模型
    const tflite::Model* model = tflite::GetModel(g_plant_disease_model);
    
    // 创建解释器
    tflite::MicroInterpreter interpreter(
        model, resolver, tensor_arena, kTensorArenaSize);
    
    // 分配张量
    interpreter.AllocateTensors();
    
    // 获取输入输出张量
    TfLiteTensor* input = interpreter.input(0);
    TfLiteTensor* output = interpreter.output(0);
}

void loop() {
    // 从摄像头获取图像 (224x224 RGB)
    uint8_t* image_data = capture_image();
    
    // 复制到输入张量
    memcpy(input->data.uint8, image_data, 224 * 224 * 3);
    
    // 推理
    interpreter.Invoke();
    
    // 获取结果
    float* predictions = output->data.f;
    int predicted_class = argmax(predictions, 16);
    float confidence = predictions[predicted_class];
    
    // 输出结果
    Serial.printf("Class: %s, Confidence: %.4f\n", 
                  kClassNames[predicted_class], confidence);
    
    delay(1000);
}
```

---

## 七、执行步骤总结

```bash
# 1. 进入项目目录
cd ~/.openclaw/workspace-backend-lead/aerogrow-demo

# 2. 下载数据集（手动或使用 Kaggle CLI）
# 将数据集解压到 dataset/indoor/

# 3. 过滤增强图像
python3 filter_augmented.py

# 4. 统计数据集
python3 dataset_stats.py

# 5. 训练模型
python3 train.py

# 6. 导出 ONNX
python3 export_onnx.py

# 7. 转换为 TFLite
python3 convert_tflite.py

# 8. 验证 TFLite 模型
python3 validate_tflite.py
```

---

## 八、预期结果

| 指标 | 目标 |
|:---|:---|
| **Top-1 准确率** | > 85% |
| **推理速度 (ESP32-S3)** | < 500ms |
| **模型大小 (INT8)** | < 2 MB |
| **功耗** | < 1W |

---

如需我执行具体的训练步骤，或需要更详细的某个环节说明，请指示，老板。
