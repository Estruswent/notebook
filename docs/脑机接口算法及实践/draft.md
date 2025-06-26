# 第二讲：Python脑电信号预处理实践

## 课程提纲

- Python环境搭建
- Python基本使用
- 常用算法库
- EEG预处理技术
- 脑电信号预处理实践

---

## 一、Python环境搭建

### 1.1 Python简介

- **什么是Python？**  
  Python是一种解释型、面向对象、动态数据类型的高级编程语言。

- **为什么选择Python？**  
    - 语法简单，易于学习，可读性强。
    - 强大的库生态，适合构建一体化的应用。
    - 拥有活跃的社区支持。

### 1.2 Anaconda简介

- **什么是Anaconda？**  
  Anaconda是一个集成了Python、环境管理工具和科学计算包的发行版。

- **为什么选择Anaconda？**  
    - 提供Conda包管理器，便于安装和管理库。
    - 支持多环境隔离，避免项目冲突。

- **Miniconda**：Anaconda的轻量级替代，仅包含Python和Conda。

### 1.3 Miniconda安装

- **下载途径**：  
    - 官方网站：[https://www.anaconda.com/download](https://www.anaconda.com/download)  
    - 清华镜像站（国内更快）：[https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda)

- **安装步骤**：  
    - 下载安装程序。  
    - 运行安装程序，选择安装路径（如`C:\Miniconda3`）。  
    - 将Conda添加到系统环境变量，以便命令行访问。

### 1.4 Miniconda配置

- **更换镜像源**：  
  编辑`.condarc`文件，使用更快的镜像源：  
    - Linux/macOS：`~/.condarc`  
    - Windows：`C:\Users\<YourUserName>\.condarc`  

  示例配置：  
  ```yaml
  channels:
    - defaults
  show_channel_urls: true
  default_channels:
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
  ```

- **验证环境变量**：确保终端中可运行`conda`命令。

### 1.5 Conda环境管理

- **常用命令**：  
    - 查看Conda版本：`conda -v`  
    - 列出已有环境：`conda env list`  
    - 创建新环境：`conda create -n [环境名称] python=[版本号]`  
      示例：`conda create -n bci python=3.12`  
    - 激活环境：`conda activate [环境名称]`  
    - 退出环境：`conda deactivate`  
    - 删除环境：`conda remove --name [环境名称] --all`

- **导出/导入环境**：  
    - 导出：`conda env export --name [环境名称] > env.yml`  
    - 导入：`conda env create -f env.yml`

### 1.6 包管理

- **Conda包管理**：  
    - 列出已安装包：`conda list`  
    - 安装包：`conda install [包名称]`（可选：`=版本号`）  
    - 更新包：`conda update [包名称]`  
    - 删除包：`conda uninstall [包名称]`

- **Pip包管理**：  
    - 列出已安装包：`pip list`  
    - 安装包：`pip install [包名称]`（可选：`==版本号`）  
    - 更新包：`pip install --upgrade [包名称]`  
    - 删除包：`pip uninstall [包名称]`

- **导出/导入Pip环境**：  
    - 导出：`pip freeze > requirements.txt`  
    - 导入：`pip install -r requirements.txt`

### 1.7 任务一

- **目标**：搭建用于脑电信号处理的Python环境。

- **步骤**：  
    - 安装Miniconda。  
    - 创建名为`bci`的环境，Python版本为3.12：  
      ```bash
      conda create -n bci python=3.12
      ```  
    - 激活环境：`conda activate bci`  
    - 安装第三方库：  
      ```bash
      conda install numpy pandas matplotlib scipy mne mne_icalabel torch
      ```

---

## 二、Python基本使用

### 2.1 基础语法

- **打印输出**：`print("Hello, EEG!")`

- **字符串操作**：  
    - `str.upper()`：转为大写。  
    - `str.lower()`：转为小写。  
    - `str.split()`：分割为列表。

- **容器**：  
    - **List**：有序、可变：`[1, 2, 3]`  
    - **Set**：无序、唯一：`{1, 2, 3}`  
    - **Dict**：键值对：`{"channel": "Fz"}`

- **条件语句**：`if`、`elif`、`else`

- **循环**：`for item in list:`、`while condition:`

- **函数**：  

  ```python
  def preprocess_data(data):
      return data * 2
  ```

---

## 三、常用算法库

### 3.1 Numpy

- **用途**：数组数值计算。

- **示例**：  
    - 创建数组：`np.array([1, 2, 3])`  
    - 切片：`arr[1:]`  
    - 广播：`arr + 2`

### 3.2 Matplotlib

- **用途**：数据可视化。

- **示例**：  
    - 折线图：`plt.plot(x, y)`  
    - 散点图：`plt.scatter(x, y)`  
    - 直方图：`plt.hist(data)`

### 3.3 Pandas

- **用途**：数据分析和处理（DataFrame）。

- **示例**：  
    - 读取CSV：`pd.read_csv("data.csv")`  
    - 过滤：`df[df["value"] > 0]`  
    - 清理：`df.dropna()`

---

## 四、EEG预处理技术

### 4.1 EEG简介

- **什么是EEG？** 记录大脑皮层电活动。

- **伪迹类型**：  
    - 眨眼伪迹  
    - 眼动伪迹  
    - 肌电污染  
    - 心电干扰  
    - 工频干扰  
    - Alpha波形

### 4.2 EEG预处理流程

- 导入数据
- 加载电极
- 滤波
- 分段
- 降采样
- 插值坏导
- 剔除坏段

### 4.3 EEG数据读取

- EEG文件格式多样（如`.cnt`、`.edf`），需用特定函数（如MNE的`read_raw_edf()`）加载。

### 4.4 EEG数据结构

- **元数据**：通道数量、名称、类型、坏通道、采样频率等。

- **脑电数据**：二维数组（通道数 × 采样点数）。

- **查看形状**：`raw.get_data().shape`

### 4.5 EEG数据可视化

- **波形图**：展示信号随时间变化。

- **功率谱密度图（PSD）**：显示频率成分。

- **通道布局图**：展示电极位置。

### 4.6 导入电极

- 根据通道名称设置类型（如EEG、EOG）。
- 按类型筛选通道。
- 重命名通道（若需要）。
- 设置电极布局（如10-20系统）。

### 4.7 设置坏通道

- **手动**：`raw.info["bads"] = ["Fz"]`

- **交互式**：在波形图中点击标记坏通道。

### 4.8 重采样

- 调整采样频率：`raw.resample(256)`（如降到256 Hz）。

### 4.9 滤波

- **带通滤波**：保留特定频率范围（如1-40 Hz）。

- **陷波滤波**：去除特定频率（如50 Hz工频噪声）。

### 4.10 EEG重参考

- **共平均参考**：减去所有通道均值。

- **双侧乳突参考**：使用耳部电极（如A1、A2）。

- **REST参考**：参考电极标准化技术。

### 4.11 插值坏导

- 利用周围正常通道的空间信息重建坏通道数据。

### 4.12 独立成分分析（ICA）

- **用途**：分离混合信号，去除伪迹（如眨眼、心跳）。

- **常见成分**：眨眼、眼动、头动、心电、工频干扰。

---

## 五、脑电信号预处理实践

### 5.1 数据集

- **来源**：MDD抑郁症公开数据集（EDF格式）

- **链接**：[https://figshare.com/articles/dataset/EEG_Data_New/4244171](https://figshare.com/articles/dataset/EEG_Data_New/4244171)

### 5.2 注意事项

- EDF文件需用特定方法读取（如`mne.io.read_raw_edf()`）。

- 清理通道名称（删除`A2-A1`、`23A-23R`、`24A-24R`）。

- 使用共平均参考：`raw.set_eeg_reference(ref_channels="average")`。

### 5.3 实践步骤

- 导入数据：使用MNE加载EDF文件。
- 清理通道名称：删除不需要的后缀。
- 设置电极类型与布局：匹配10-20系统。
- 滤波：应用带通滤波（1-40 Hz）和陷波滤波（50 Hz）。
- 重参考：使用共平均参考。
- 检测与插值坏通道：标记并修复坏通道。
- ICA：运行ICA去除伪迹。
- 分段与剔除坏段：分割数据并移除噪声片段。

export: `conda env export > environment.yml`

安装第三方库：

- numpy
- pandas
- matplotlib
- scipy
- mne
- mne_icalabel

存在各种干扰

fif mne.io.read_raw_fif


脑电数据：一个二维数组，形状为（通道数，采样点数）。

plot绘制可视化
compute_psd()功率谱密度图
plot_sensors()

设置坏通道raw_info['bads']
重采样resample()改变采样频率

带通滤波filter()
陷波滤波notch_filter()

处理时都要做，但顺序不重要

重参考：

共平均参考

双侧乳突参考

REST重参考



插值坏导：基于周围正常的信息，通过插值修复“坏”的数据。

ICA：分离混合信号中的独立源成分，除去伪迹。
