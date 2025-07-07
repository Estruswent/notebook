# Lecture6

肌电分类：

- 表面肌电
- 针极肌电
- 超声肌电


- 
- 肌电+脑电联合分析

## 肌电反解和肌电解码


下面是这个数据集的简介：

## Introduction
The database contains EEG recordings of subjects before and during the performance of mental arithmetic tasks.

## Study Methods
The EEGs were recorded monopolarly using Neurocom EEG 23-channel system (Ukraine, XAI-MEDICA). The silver/silver chloride electrodes were placed on the scalp according to the International 10/20 scheme. All electrodes were referenced to the interconnected ear reference electrodes.

A high-pass filter with a 30 Hz cut-off frequency and a power line notch filter (50 Hz) were used. All recordings are artifact-free EEG segments of 60 seconds duration. At the stage of data preprocessing, the Independent Component Analysis (ICA) was used to eliminate the artifacts (eyes, muscle, and cardiac overlapping of the cardiac pulsation). The arithmetic task was the serial subtraction of two numbers. Each trial started with the communication orally 4-digit (minuend) and 2-digit (subtrahend) numbers (e.g. 3141 and 42).

The participants were eligible to enroll in the study if they had normal or corrected-to-normal visual acuity, normal color vision, had no clinical manifestations of mental or cognitive impairment, verbal or non-verbal learning disabilities. Exclusion criteria were the use of psychoactive medication, drug or alcohol addiction and psychiatric or neurological complaints.

## Data
The data files with EEG are provided in EDF (European Data Format) format. Each subject has 2 files:

with "_1" suffix -- the recording of the background EEG of a subject (before mental arithmetic task)
with "_2" suffix -- the recording of EEG during the mental arithmetic task.
The recording datetime information has been set to Jan 01 for all files.

In this experiment all subjects are divided into two groups:

Group "G" (24 subjects) performing good quality count (Mean number of operations per 4 minutes = 21, SD = 7.4).
Group "B" (12 subjects) performing bad quality count (Mean number of operations per 4 minutes = 7, SD = 3.6).
In subject-info.csv, the "Count quality" column indicates which subjects correspond to which group (0 - Group "B", 1 - Group "G"). Additionally, subject-info.csv provides basic information about each subject (gender, age, job, date of recording).

train.py 是我的训练和检测的代码，我将1-25号数据用作训练模型，26-35号用于检测模型，但是目前出现结果：

RANDOM_FOREST Results:
accuracy: 0.7481
pr_auc: 0.4884
roc_auc: 0.7542

显然准确度是不够的。

另外，在运行时还出现和数据集信息有关的下述输出，我希望这能帮助到你：

Training samples (original): 6094
Training label distribution (original): {np.int64(0): np.int64(4544), np.int64(1): np.int64(1550)}
Training samples (after SMOTE): 9088
Training label distribution (after SMOTE): {np.int64(0): np.int64(4544), np.int64(1): np.int64(4544)}
Test samples: 2338
Test label distribution: {np.int64(0): np.int64(1718), np.int64(1): np.int64(620)}

