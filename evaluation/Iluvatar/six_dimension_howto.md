# 天垓100（BI-V100）六维度评测方法

## 前置条件

1. 获取天数智芯天垓100（BI-V100）算力。

   DeepSpark社区客户想要使用天垓100加速卡，可以通过填写[表单](https://order.iluvatar.com/)进行申请。

2. 安装天数智芯软件栈。

## 使用工具

- ixSMI：GPU实时状态检测工具，包含在天数智芯软件栈中，当您安装好天数智芯软件栈后即可通过**ixsmi**命令进行调用。
- DeepSpark模型训练脚本：脚本会打印训练时每秒处理的单位样本数量、模型精度值等数据。

## 六维度数据计算方法

### 速度

定义：模型稳定训练时每秒处理的单位样本的算力。

数据来源：DeepSpark模型训练脚本，脚本会打印训练过程中每个epoch的每秒处理的单位样本数量。

计算方法：即指定迭代轮次为总共5次，去掉5次中的速度的最高最低值，取中间3次速度值的平均值（或mean中值）。如取稳定训练的第6到第10个epoch共5个epoch，最高是第6个、最低是第10个epoch，则取第7个、第8个和第9个epoch的值相加除以3。

### 准确性

定义：模型收敛的精度值。

数据来源：DeepSpark模型训练脚本，脚本会打印训练过程中每个epoch的训练准确度。

计算方法：随着训练迭代而模型的loss变化不再显著时，评估模型是否达到判定为收敛所要求的准确度指标，记录此时的精度值。

### 线性度

定义：模型集群规模化训练的scalability，即算力的线性扩展性能。

数据来源：DeepSpark模型训练脚本，脚本会打印训练过程中每个epoch的每秒处理的单位样本数量。

计算方法：线性度具体还可以分为如下两种：

- 卡线性度：在一台服务器上使用多张BI-V100训练时每秒处理的单位样本数量（即速度）除以多卡的卡数，再对比使用1张BI-V100训练时的速度。例如，使用8张BI-V100时的训练速度除以卡的倍数（即8），再除以使用1张BI-V100时的训练速度，得到8卡对单卡的线性度。

- 节点线性度：在每台服务器（即节点）安装相同数量的BI-V100卡前提下，使用多台服务器训练的每秒处理的单位样本数量（即速度）除以多台服务器的台数，再对比使用1台服务器训练时的速度。例如，使用4台服务器（每台都分别安装了8张BI-V100）时的训练速度，除以服务器的总台数（即节点数4），再除以使用1台服务器时的训练速度，得到4节点对单节点的线性度。

### 功耗

定义：模型稳定训练时候实际消耗的GPU平均功耗。

数据来源：GPU实时状态检测工具ixSMI。

计算方法：模型稳定训练时（通常是排除训练刚开始的数据处理和预热阶段），在终端输入以下命令，ixSMI工具会每间隔5秒显示此时BI-V100的功耗，单位为W。取多次的数据取平均值作为模型训练的功耗。

- 如您环境中仅安装了1张BI-V100，输入以下命令：

```
$ ixsmi -q -l | grep -E 'Power Draw'
```

- 如您环境中安装了多张BI-V100（可使用**ixsmi**命令获取多张卡的概要信息，包括index信息），输入以下命令，使用${gpu_index}指定具体读取哪张BI-V100的信息，注意index从0开始：

```
$ ixsmi -q -i ${gpu_index} -l | grep -E 'Power Draw'
```

例如，以下命令可以得到index为0的BI-V100的功耗信息：

```
$ ixsmi -q -i 0 -l | grep -E 'Power Draw'
```


### 显存占用

定义：模型稳定训练时实际消耗的GPU平均显存占用量。

数据来源：GPU实时状态检测工具ixSMI。

计算方法：模型稳定训练时（通常是排除训练刚开始的数据处理和预热阶段），在终端输入以下命令，ixSMI工具会每间隔5秒显示此时BI-V100显存的使用量，单位为MiB。取多次的数据取平均值作为模型训练的显存占用量。

- 如您环境中仅安装了1张BI-V100，输入以下命令：

```
$ ixsmi -q -l | grep Used.[^G]
```

- 如您环境中安装了多张BI-V100（可使用**ixsmi**命令获取多张卡的概要信息，包括index信息），输入以下命令，使用${gpu_index}指定具体读取哪张BI-V100的信息，注意index从0开始：

```
$ ixsmi -q -i ${gpu_index} -l | grep Used.[^G]
```

例如，以下命令可以得到index为0的BI-V100的显存占用量：

```
$ ixsmi -q -i 0 -l | grep Used.[^G]
```


### 稳定度

定义：取模型多次完整训练最终所达到的收敛值，比较多次收敛值的稳定程度。

数据来源：DeepSpark模型训练脚本，脚本会打印训练过程中每个epoch的结束时达到的准确度。

计算方法：模型采用5次完整训练，每次都最终达到标准收敛值，取5次的收敛值的中值做为基准值，然后其他的4次收敛值对比基准值的差值百分比应分布在（-0.01，+0.01）的合理区间，此时达到满分1。当5个数据有1次不在该范围内，稳定度则递减20%。

## 六维度数据计算示例

[天垓100六维度评测方法示例：ResNet50](six_dimension_howto_example.md)
