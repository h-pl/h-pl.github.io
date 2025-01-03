---
title: '评估 HVAC 系统在碳中和的优化方向'
date: 2022-01-04 11:26:19
tags: [BIM]
published: true
hideInList: false
feature: 
isTop: false
---

关于这一议题，大多数已发表的研究都是基于假设和经验法。首次评估和制定了基于BIM的HVAC系统详细的**全生命周期评估**（LCA）的要求和方法。以瑞士大楼及现有的能效路径作为基础，LCA结果表明，暖通空调系统的影响是原来的三倍😲，且占总影响的15-36%。

## 研究目标
**本文目标**是：优化现有工作流程，因大量参数化工作，借助了可视化编程（VPL）将外部产品数据信息连接到BIM构件的方法。
- 建筑行业占全球碳排放的40%
    - 高效的建筑技术和可再生能源代替传统能源
    - 隐含碳是指建筑材料在制造、运输、施工和生命周期结束阶段释放的温室气体，占建筑行业的11%
    - 在瑞士，暖通空调系统（供暖、通风、热分配）的制造和维护过程中的温室气体排放约占新办公建筑总排放量的13%😕
- 到目前为止，只有少数研究测量了HVAC系统的内在影响[17]
    - 其中，只有一个[31]使用了一个建成的或几乎建成的BIM HVAC模型
    - 一般来说，这类研究可大致分为两种类型的**HVAC系统的LCA研究**。
        - 1️⃣第一种类型的研究在两个或更多的HVAC系统之间进行比较，以确定哪个系统对环境的影响更小，这些研究对详细的评估不感兴趣
        - 2️⃣第二类HVAC的LCA是指对HVAC系统的**详细评估**📃，类似的研究有四项
            - 其中，最具代表的一项研究发现
  
  |影响因素|影响占比|重要性|
  |----|----|----|
  |室内（天花板、门、家具、栏杆）|43%|最大|
  |技术设备🔌|24%||
  |结构框架|21%||
  |围护结构|12%||

## 研究方法
**基于BIM的LCA**被用于许多研究中，试图摆脱传统的LCA（基于手工的计算），并为设计者提供一个工具，以方便在设计阶段对建筑进行环境影响评估，以及在施工后进行评估（即符合标准或用于知识生成）。也有两种方法被大量使用，但它们🔨**忽略了或过度简化**了HVAC系统。在体现环境影响方面，以及在如何使用BIM模型和现有工具或方法评估暖通空调系统方面，存在着**研究空白**👀。
- 1️⃣第一种，目前主要借助于商业工具是LCA 与 BIM 相结合（如 Tally 或 OneClickLCA），主要原理是从BIM模型中提取信息（IFC或gbXML格式），然后导入LCA软件中
- 2️⃣第二种，使用BIM与可视化编程语言（VPL）。这种方法能够从BIM模型中自动提取信息，并创建可更新的链接到LCA数据库。
    - BIM模型提取的数据主要有几何数据、材料数据（数量和名称）和LCA数据。
        - 几何信息是直接从BIM模型中提取的。
        - 材料信息可以直接从BIM模型中提取。
            - 在某些情况下，材料数量信息可以直接从BIM模型或产品数据表中获取。
            - 在其他情况下，数量需要通过结合数学公式和来自BIM模型的几何信息来计算。
            - 如果模型中**没有**这些信息，那么就用产品数据表来代替。

![](https://h-pl.github.io/post-images/1641361724264.png)
####### 图1描述了HVAC系统的综合BIM和LCA工作流程。该示意图显示了与VPL相连的不同数据源。具体来说，来自BIM模型（1）、产品数据表（2）和LCA数据库（3）的信息在VPL环境中被结合起来，计算影响，并将结果以所需格式导出。本研究中使用的BIM软件是Revit 2019，而VPL是Dynamo 2.0。
从而实现以下功能🌈：
- 直接从BIM模型中提取物体和材料数据。
- 在BIM对象和产品数据表/目录之间以及材料和LCA数据库之间建立双向链接。
- 进行LCA计算并将结果导出到Excel文件中。
该工具的灵活性使其有可能根据所需的（LCA）边界条件、可用的数据以及数据格式和其结构来定制拟议的工作流程。

### 直接从BIM模型中提取物体和材料数据。
分析从对象层面开始，通过三个步骤将计算细化到材料层面。
- 1️⃣根据元素的复杂程度和性质，对元素进行高级分组和排序。管道与管道被归类在一起，因为它们都是线性元素，具有类似的细节（复杂性）。
- 2️⃣第二个层次，元素被进一步细分。例如，风管被细分为矩形风管和圆形风管。
- 3️⃣第三层也是最底层的分组/排序是根据材料类型进行的，例如，圆钢风管和圆铜风管。

根据共同特征，即几何形状和复杂程度，创建了四个不同的组。这四组是风管和管道，配件，机械设备和空气终端，以及管道和风管配件。总的来说，评估组的材料数量计算采用了三种方法。
- 1️⃣第一种方法中，从BIM模型中提取材料和几何信息，并利用科学公式（BIM数据与科学公式相结合）在Dynamo中计算材料的重量。
- 2️⃣第二种方法中，材料和几何信息从BIM模型中提取，在Dynamo中与产品数据表信息相结合（BIM数据与产品数据表数据相结合）。在大多数情况下，**来自产品数据表的重量信息是按物体而不是按材料提供的。**因此，必须假定👓各种物体材料在总重量中所占的百分比。
- 3️⃣第三种方法包括将物体与来自产品数据表的重量信息直接映射（BIM物体链接到产品数据表），也必须假设产品材料占总重量的百分比。

⚠️如果上述的方法都不适用，比如配件的情况，那么就采用**经验法则**的估计方法

### 各类构件的计算方法
1. 风管和管道的质量
- 两者统称为线性图元。可采用
![](https://h-pl.github.io/post-images/1641361658984.png)
其中，M是单位长度质量 kg/m，D是外径 mm，T是壁厚 mm
- 异形构件
实体对象，即无参数对象。计算直径与总面积的比值。根据面积比通过**经验公式**推导异形构件与总管道及风管的质量比。
2. 机械设备与终端
- 来源于制造商
3. 管道附件及新材料（组合材料）
- 来源于制造商
- 新材料（组合材料）来源于类似材料

### LCA 数据来源
主要是由KBOB和Ecoinvent数据库检索而得。其中，LCA值主要基于材料映射。

## 案例分析
案例研究是位于瑞士的西门子国际总部办公大楼。该办公楼于 2018 年竣工，是位于楚格的西门子新园区的一部分。它拥有 LEED 白金认证和瑞士 Minergie 标签，重点关注建筑外壳和能源消耗。该建筑共有七层，总建筑面积（​​GFA）为 32,000 平方米，包括地下两层，主要用作车库。在暖通空调系统方面，该建筑以水为热泵运行，并使用楚格湖的水进行加热和自然冷却。这是首批使用 BIM 的西门子建筑项目之一。 HVAC 系统的 BIM 模型（图 2）在施工完成后经历了广泛的修订。它的开发水平 (LOD) 高于 300，尽管观察到某些元素比其他元素更详细。此外，该项目得到了很好的参数记录，包括大多数 HVAC 设备的产品数据表和详细信息。可以说，结合制造商信息的施工后BIM模型满足了竣工要求。

## 结论
材料数量提取结果表明，镀锌钢（66%）、铝（13%）和矿棉（10%）是主要材料。该案例钢材总量，包括镀锌钢、不锈钢、钢材，共计356吨，约占总材料量的80%（图3）。
![](https://h-pl.github.io/post-images/1641366062708.png)

当考虑到建筑的整个生命周期时，每个部件的LCA变得更加耐人寻味。与制造阶段相比，管道系统和机械设备的更换阶段产生的温室气体排放表明，与制造阶段的影响（15.3 kgCO2eq/m2）相比，机械设备在**使用阶段**的影响几乎翻倍（35 kgCO2eq/m2）。这种增加与建筑使用阶段的设备🌵**更换频率**有关。
例如，热泵是每20年更换一次。因此，在60年的建筑寿命中，它们被更换两次。材料数量的增加所产生的环境影响十分明显，两倍。总的来说，**机械设备与风管和管道**一起，是暖通空调系统生命周期内温室气体排放总量的主要贡献者。在更高的层面上，关于调查模块的结果显示，更换（B4）的年化排放量为1.70千克二氧化碳当量/平方米，是建筑生命周期阶段中**碳密集度最高**的；而制造（A1-A3）的年化排放量为1.32千克二氧化碳当量/平方米，以及运行（B6）的年化排放量为1.25千克二氧化碳当量/平方米，其重要性相近。运行影响的计算是基于公用事业公司提供的能源消耗数据，结合能源工程师提供的按计划的能源分配图。暖通空调系统的能源消耗占总能源消耗的57%。最后，弃置影响非常小（0.4 kgCO2eq/m2），主要来自于绝缘和辅助材料。

值得注意的是，大型空气处理机组AHU内的过滤器更换数量是😱**相当可观**的，占AHU更换总影响数的65%。而整个HVAC系统中，过滤器的更换占整个系统总更换影响的😱11%。其中，管道附件中，蝶阀，多叶风阀，流量控制器是碳排放量最大的。


## 参考文献
- [HVAC and BIM](/pdf/HVACandBIM.pdf)


