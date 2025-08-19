# 项目标题：扬声器控制模型

本项目提供了一个用于 **[基于微型扬声器数据驱动模型下的声场控制算法]** 的实现。

## 1. 环境配置

本项目基于 Python3.9 开发，请按照以下步骤配置运行环境。

### 安装依赖
所有依赖项都已列在 requirements.txt 文件中，使用下面命令安装：

pip install -r requirements.txt

### 代码结构
- lsk_control/
  - dataset/:存放训练数据的文件夹，目前在.gitignore中屏蔽了该文件夹下.mat类型文件的上传
  - model_set/:定义模型的代码
    - __init__.py:用于向外部暴露接口
    - Forward_WN.py:前向传播的膨胀卷积网络
    - LinearCNN.py:1D卷积层实现的线性FIR滤波器建模网络
    - VNN.py:Volterra网络的实现
    - VQE:Voice Quality Enhancement网络实现
    - wavenet_control:膨胀卷积网络与Volterra网络的组合网络
    - utils:对音频序列进行精确切片
  - trainModel/:保存训练好的模型文件
    - tained_*.pth:训练好的扬声器建模模型
    - control_*.pth:训练好的控制模型
  - util_func/:一些常用的工具函数
    - __init__.py:用于向外部暴露接口
    - calc_freq_loss.py:计算频域Loss
    - calc_MD5.py:计算文件的MD5值
    - LoadData.py:读取mat文件
    - Warmup_scheduler.py：学习率调度器
  - Results_plot
    -ctrl_datashow.m:用于建模结果的可视化
  - ctrl_results:控制网络结果的数据存放目录
  - .gitignore
  - README.md
  - main_ctrl.py:主程序


## 3. 如何运行
本项目通过主脚本 main_ctrl.py 来执行

