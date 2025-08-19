
# 扬声器声场控制模型

本项目是一个基于数据驱动建模的声场控制算法实现。其主要功能是通过深度学习网络，对微型扬声器阵列进行精确建模和控制，以在特定区域（暗区）实现声音抑制。

## 1 环境配置

### 系统要求
- Python 3.9+
- CUDA 11.0+ (可选，用于GPU加速)
- MATLAB R2020a+ (可选，用于结果可视化)

###  依赖安装
```bash
pip install -r requirements.txt
```
包括：
- numpy==1.22.3
- torch==2.0.1
- h5py==3.14.0
- matplotlib==3.5.1
- scipy==1.6.2
- torchinfo==1.8.0
- einops==0.8.1

## 2 项目结构

```
lsk_control/
│
├── dataset/                    # 训练/测试数据存储目录
│   └── *.mat                   # 音频数据文件 (被.gitignore忽略)
│
├── model_set/                  # 神经网络模型定义
│   ├── __init__.py             # 模块接口导出
│   ├── Forward_WN.py           # 前向传播膨胀卷积网络 (WaveNet)
│   ├── LinearCNN.py            # 1D卷积FIR滤波器建模网络
│   ├── VNN.py                  # Volterra非线性网络实现
│   ├── VQE.py                  # 语音质量增强网络
│   ├── wavenet_control.py      # WaveNet与Volterra组合控制网络
│   └── utils.py                # 音频序列处理工具函数
│
├── trainModel/                 # 训练模型权重存储
│   ├── trained_*.pth           # 扬声器建模模型权重
│   └── control_*.pth           # 声场控制模型权重
│
├── util_func/                  # 工具函数库
│   ├── __init__.py             # 模块接口导出
│   ├── calc_freq_loss.py       # 频域损失函数计算
│   ├── calc_MD5.py             # 文件完整性校验
│   ├── LoadData.py             # MATLAB数据文件读取
│   └── Warmup_scheduler.py     # 学习率预热调度器
│
├── Results_plot/               # 结果可视化脚本
│   ├── LoadData.m              # 谱分析函数
    ├── fre2bin.m              # 谱分析函数
│   └── ctrl_datashow.m         # MATLAB建模结果可视化
│
├── ctrl_results/               # 控制网络输出数据
│   └── *.mat                   # 控制结果数据文件
│
├── main_ctrl.py                # 主程序入口
├── .gitignore                  # Git忽略文件配置
├── README.md                   # 项目说明文档
└── requirements.txt            # Python依赖列表
```

## 3 核心模块说明

### 神经网络架构
- Forward_WN.py: 基于WaveNet的膨胀卷积网络，用于扬声器非线性特性建模和非线性控制
- LinearCNN.py: 1D卷积实现的线性FIR滤波器，处理线性声学响应
- VNN.py: Volterra级数网络，建模扬声器非线性失真
- VQE.py: 语音质量增强网络，提升输出音频质量
- wavenet_control.py: 混合架构，结合膨胀卷积与Volterra网络

### 工具函数模块

- calc_freq_loss.py: 实现频域损失函数，确保频率响应准确性
- LoadData.py: 处理MATLAB .mat格式的音频数据文件
- Warmup_scheduler.py: 学习率预热策略，提升训练稳定性


## 4 代码使用说明

完整的执行流程分为 （1） Python数据处理 和（2） MATLAB结果可视化 两个步骤。
### 步骤一：运行主脚本 (Python)

本项目通过主脚本 main_ctrl.py 来驱动，用于加载模型并生成声场控制结果。
#### （1）准备数据与模型:
数据: 将.mat 格式训练/测试数据放入 lsk_control/dataset/ 文件；

预训练模型: 确保 lsk_control/trainModel/ 文件夹中包含必要的预训练模型权重 (.pth 文件)
#### （2）运行脚本:
直接执行主脚本：
```
python lsk_control/main_ctrl.py
```
####  (3) 输出数据格式
控制网络运行后，在 ctrl_results/ 目录下生成 .mat 文件，包含以下字段：
- ypred: 网络输出 - 控制后的音源信号经建模模型得到的暗区麦克风信号。
- yout: 实测参考 - 在VAST系统工作下实测的明/暗区麦克风信号。
- lsk2: 未控制基线 - 2号扬声器单独工作时，明/暗区麦克风接收到的信号。
- control: 控制信号 - 经控制网络生成的1, 3, 4号扬声器的驱动信号或叫音源信号。
- xin: 输入源信号 - VAST系统工作下，采集的原始输入音频。
  
### 步骤二：结果可视化 (MATLAB)

打开 MATLAB;
将当前工作目录切换至 lsk_control/Results_plot/;
运行 ctrl_datashow.m 脚本;
该脚本会自动加载 ../ctrl_results/ 目录下的结果文件，并生成最终的可视化图表。

## 5 参数与配置
### (1) 命令行参数
注意: 当前版本的 main_ctrl.py 默认使用脚本内预设的参数。如果您希望通过命令行动态指定参数（例如，输入/输出路径、模型名称等），您需要自行修改脚本，引入并使用 Python 的 argparse 模块来实现。
### (2) 网络参数配置
您可以根据需要调整网络和算法的参数。我们提供了两种修改方式：
全局配置 (推荐): 直接修改 lsk_control/model_set/ 目录下具体网络模型文件中的默认配置参数。这是进行系统性实验的推荐方式。
临时配置: 在 main_ctrl.py 脚本中，我们预留了参数配置接口。您可以在该脚本的指定区域临时修改参数，以便快速测试和验证。 


