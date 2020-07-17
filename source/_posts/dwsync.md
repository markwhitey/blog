---
title: 美团数据建设
mathjax: false
date: 2020-01-05 12:10:07
tags:
- 数据仓库
categories:
- Big Data
top:
photo:
---



{% cq %}

美团在现有大数据平台的基础上，借鉴业界成熟 OneData 方法论，构建合理的数据体系架构、数据规范、模型标准和开发模式，以保障数据快速支撑不断变化的业务并驱动业务的发展

{% endcq %}

<!-- more -->

<br>

>  原文地址 https://tech.meituan.com/2019/10/17/meituan-saas-data-warehouse.html

背景
--

随着业务的发展，频繁迭代和跨部门的垂直业务单元变得越来越多。但由于缺乏前期规划，导致后期数仓出现了严重的数据质量问题，这给数据治理工作带来了很大的挑战。在数据仓库建设过程中，我们总结的问题包括如下几点：

*   缺乏统一的业务和技术标准，如：开发规范、指标口径和交付标准不统一。
*   缺乏有效统一的数据质量监控，如：列值信息不完整和不准确，SLA 时效无法保障等。
*   业务知识体系散乱不集中，导致不同研发人员对业务理解存在较大的偏差，造成产品的开发成本显著增加。
*   数据架构不合理，主要体现在数据层之间的分工不明显，缺乏一致的基础数据层，缺失统一维度和指标管理。

目标
--

在现有大数据平台的基础上，借鉴业界成熟 OneData 方法论，构建合理的数据体系架构、数据规范、模型标准和开发模式，以保障数据快速支撑不断变化的业务并驱动业务的发展，最终形成我们自己的 OneData 理论体系与实践体系。

OneData 探索
----------

### OneData：行业经验

在数据建设方面，阿里巴巴提出了一种 OneData 标准，如图 - 1 所示：

![](https://p1.meituan.net/travelcube/484e5c6296bad966d71b59e63932a881155070.png)

图 1 OneData 标准

### OneData：我们的思考

他山之石，可以攻玉。我们结合实际情况和业界经验，进行了如下思考：

1. 对阿里巴巴 OneData 的思考

*   整个 OneData 体系覆盖范围广，包含数据规范定义体系、数据模型规范设计、ETL 规范研发以及支撑整个体系从方法到实施的工具体系。
*   实施周期较长，人力投入成本较高。
*   推广落地的工作比较依赖工具。

2. 对现有实际的思考

*   现阶段工具保障方面偏弱，人力比较缺乏。
*   现有开发流程不能全部推翻。

经过综合考量，我们发现直接全盘复用他人经验是不合理的。那我们如何定义自己的 OneData，即能在达到目标的前提下，又能避免上述的难题呢？

### OneData：我们的想法

首先，结合行业经验，自身阶段的实践及以往的数仓经验，我们预先定义了 OneData 核心思想与 OneData 核心特点。

OneData 核心思想：从设计、开发、部署和使用层面，避免重复建设和指标冗余建设，从而保障数据口径的规范和统一，最终实现数据资产全链路关联、提供标准数据输出以及建立统一的数据公共层。

**OneData 核心特点：三特性和三效果。**

*   三特性：统一性、唯一性、规范性。
*   三效果：高扩展性、强复用性、低成本性。

![](https://p0.meituan.net/travelcube/9d71ab3500ecb71102dba48d9fbb79a2255425.png)

图 2 OneData 的六个特性

### OneData：我们的策略

OneData 即有核心思想又有核心特点，到底怎么来实现核心思想又能满足其核心特点呢？通过以往经验的沉淀，我们提出两个统一方法策略：统一归口、统一出口。

![](https://p0.meituan.net/travelcube/9645687e124fc3f90f5b641ca5e21b58289517.png)

根据两个统一的方法策略，我们开启了 OneData 的实践之路。

OneData 实践
----------

### 统一业务归口

数据来源于业务并支撑着业务的发展。因此，数仓建设的基石就是对于业务的把控，数仓建设者即是技术专家，也应该是 “大半个” 业务专家。基于互联网行业的特点，我们基本上采用需求推动数据的建设，这也带来了一些问题，包括：数据对业务存在一定的滞后性；业务知识沉淀在各个需求里，导致业务知识体系分散。针对这些问题，我们提出统一业务归口，构建全局知识库，进而保障对业务认知的一致性。

![](https://p1.meituan.net/travelcube/9927abc46f99a74f3216636d5f7f4275280106.png)

![](https://p0.meituan.net/travelcube/de9834657fee8d07db9843fecf6524b3567061.png)

图 3 支持业务的数据源知识库

### 设计统一归口

为了解决数据仓库建设过程中出现的各种痛点，我们从模型与规范两个方面进行建设，并提出设计统一归口。

**1. 模型**

规范化模型分层、数据流向和主题划分，从而降低研发成本，增强指标复用性，并提高业务的支撑能力。

**(1) 模型分层**

优秀可靠的数仓体系，往往需要清晰的数据分层结构，即要保证数据层的稳定又要屏蔽对下游的影响，并且要避免链路过长。结合这些原则及以往的工作经验，我们将分层进行统一定义为四层：

![](https://p0.meituan.net/travelcube/659540d42003522a8607b6b7e6ea75b6402081.png)

图 4 数据分层架构

**(2) 模型数据流向**

重构前，存在大量的烟囱式开发、分层应引用不规范性及数据链路混乱、血缘关系很难追溯和 SLA 时效难保障等问题。

![](https://p0.meituan.net/travelcube/e9cfbb9f6a4e13ce759d6409ca522378184732.png)

图 5 重构前和重构后的数据流向图

重构之后，稳定业务按照标准的数据流向进行开发，即 ODS–>DWD–>DWA–>APP。非稳定业务或探索性需求，可以遵循 ODS->DWD->APP 或者 ODS->DWD->DWT->APP 两个模型数据流。在保障了数据链路的合理性之后，又在此基础上确认了模型分层引用原则：

*   正常流向：ODS>DWD->DWT->DWA->APP，当出现 ODS >DWD->DWA->APP 这种关系时，说明主题域未覆盖全。应将 DWD 数据落到 DWT 中，对于使用频度非常低的表允许 DWD->DWA。
*   尽量避免出现 DWA 宽表中使用 DWD 又使用（该 DWD 所归属主题域）DWT 的表。
*   同一主题域内对于 DWT 生成 DWT 的表，原则上要尽量避免，否则会影响 ETL 的效率。
*   DWT、DWA 和 APP 中禁止直接使用 ODS 的表， ODS 的表只能被 DWD 引用。
*   禁止出现反向依赖，例如 DWT 的表依赖 DWA 的表。

**2. 主题划分**

传统行业如银行、制造业、电信、零售等行业中，都有比较成熟的主题划分，如 BDWM、FS-LDM、MLDM 等等。但从实际调研情况来看，主题划分太抽象会造成对业务理解和开发成本较高，不适用互联网行业。因此，结合各层的特性，我们提出了两类主题的划分：**面向业务**、**面向分析**。

*   面向业务：按照业务进行聚焦，降低对业务理解的难度，并能解耦复杂的业务。我们将实体关系模型进行变种处理为实体与业务过程模型。实体定义为业务过程的参与体；业务过程定义是由多个实体作用的结果，实体与业务过程都带有自己特有的属性。根据业务的聚合性，我们把业务进行拆分，形成了七大核心主题。
*   面向分析：按照分析聚焦，提升数据易用性，提高数据的共享与一致性。按照分析主体对象不同及分析特征，形成分析域主题在 DWA 进行应用，例如销售分析域、组织分析域。

**3. 规范**

模型是整个数仓建设基石，规范是数仓建设的保障。为了避免出现指标重复建设和数据质量差的情况，我们统一按照最详细、可落地的方法进行规范建设。

**(1) 词根**

词根是维度和指标管理的基础，划分为普通词根与专有词根，提高词根的易用性和关联性。

*   普通词根：描述事物的最小单元体，如：交易 - trade。
*   专有词根：具备约定成俗或行业专属的描述体，如：美元 - USD。

**(2) 表命名规范**

**通用规范**

*   表名、字段名采用一个下划线分隔词根（示例：clienttype->client_type）。
*   每部分使用小写英文单词，属于通用字段的必须满足通用字段信息的定义。
*   表名、字段名需以字母为开头。
*   表名、字段名最长不超过 64 个英文字符。
*   优先使用词根中已有关键字（数仓标准配置中的词根管理），定期 Review 新增命名的不合理性。
*   在表名自定义部分禁止采用非标准的缩写。

**表命名规则**

`表名称 = 类型 + 业务主题 + 子主题 + 表含义 + 存储格式 + 更新频率 +结尾`，如下图所示：

![](https://p0.meituan.net/travelcube/8d39076f1895ea0fd6e5936709151390243201.png)

图 6 统一的表命名规范

**(3) 指标命名规范**

结合指标的特性以及词根管理规范，将指标进行结构化处理。

A. 基础指标词根，即所有指标必须包含以下基础词根：

| 基础指标词根 | 英文全称 | Hive 数据类型 | MySQL 数据类型 | 长度 | 精度 | 词根  | 样例   |
| ------------ | -------- | ------------- | -------------- | ---- | ---- | ----- | ------ |
| 数量         | count    | Bigint        | Bigint         | 10   | 0    | cnt   |        |
| 金额类       | amout    | Decimal       | Decimal        | 20   | 4    | amt   |        |
| 比率 / 占比  | ratio    | Decimal       | Decimal        | 10   | 4    | ratio | 0.9818 |
| ……           | ……       | ……            | ……             |      |      | ……    |        |

B. 业务修饰词，用于描述业务场景的词汇，例如 trade - 交易。

C. 日期修饰词，用于修饰业务发生的时间区间。

| 日期类型 | 全称   | 词根 | 备注 |
| -------- | ------ | ---- | ---- |
| 日       | daily  | d    |      |
| 周       | weekly | w    |      |
| ……       | ……     | ……   |      |

D. 聚合修饰词，对结果进行聚集操作。

| 聚合类型 | 全称    | 词根 | 备注                 |
| -------- | ------- | ---- | -------------------- |
| 平均     | average | avg  |                      |
| 周累计   | wtd     | wtd  | 本周一截止到当天累计 |
| ……       | ……      | ……   |                      |

E. 基础指标，单一的业务修饰词 + 基础指标词根构建基础指标 ，例如：交易金额 - trade_amt。

F. 派生指标，多修饰词 + 基础指标词根构建派生指标。派生指标继承基础指标的特性，例如：安装门店数量 - install_poi_cnt。

G. 普通指标命名规范，与字段命名规范一致，由词汇转换即可以。

![](https://p0.meituan.net/travelcube/4462d67eeef43504ca22ec2f523a812963726.png)

图 7 普通指标规范

H. 日期类型指标命名规范，命名时要遵循：业务修饰词 + 基础指标词根 + 日期修饰词 / 聚合修饰词。将日期后缀加到名称后面，如下图所示：

![](https://p0.meituan.net/travelcube/7975e49d4a0c32f3dc4268a15f417baa82605.png)

图 8 日期类型指标规范

I. 聚合类型指标，命名时要遵循：业务修饰词 + 基础指标词根 + 聚合类型 + 日期修饰词。将累积标记加到名称后面，如下图所示：

![](https://p1.meituan.net/travelcube/45fb192b0204154dd9ba93be07336f74102303.png)

图 9 聚合类指标规范

(4) 清洗规范

确认了字段命名和指标命名之后，根据指标与字段的部分特性，我们整理出了整个数仓可预知的 24 条清洗规范：

| 数据类型 | 数据类别   | Hive 类型 | MySQL 类型 | 长度 | 精度 | 词根 | 格式说明     | 备注                 |
| -------- | ---------- | --------- | ---------- | ---- | ---- | ---- | ------------ | -------------------- |
| 日期类型 | 字符日期类 | string    | varchar    | 10   |      | date | YYYY-MM-DD   | 日期清洗为相应的格式 |
| 数据类型 | 数量类     | bigint    | bigint     | 10   | 0    | cnt  | 活跃门店数量 |                      |
| ……       | ……         | ……        | ……         | ……   | ……   | ……   | ……           |                      |

结合模型与规范，形成模型设计及模型评审两者的工作职责，如下图所示：

![](https://p1.meituan.net/travelcube/7841211ccc7aec623a13596dcf8a7180332984.png)

图 10 模型设计和审计职责

### 统一应用归口

在对原有的应用支持流程进行梳理的时候，我们发现整个研发过程是烟囱式。如果不进行改善就会导致前面的建设” 毁于一旦 “，所以需对原有应用支持流程进行改造，如下图所示：

![](https://p0.meituan.net/travelcube/6aacfd8f4c29b15f9950efa69057a91b320101.png)

图 11 应用归口

从图中可以看出，重构前一个应用存在多次迭代，每次迭代都各自维护自己的文档。烟囱式开发会导致业务信息混乱、应用无法与文档对齐、知识传递成本、维护成本和迭代成本大大增加等问题。重构后，应用与知识库相对应，保证一个应用只对应一份文档，且应用统一要求在一份文档上进行迭代，从根源上改变应用支持流程。同时，针对核心业务细节和所支撑的数据信息，进行了全局调研并归纳到知识库。综合统一的知识库，降低了知识传递、理解、维护和迭代成本。

统一归口策略包含业务归口统一、设计归口统一和应用归口统一，从底层保证了数仓建设的**三特性**和**三效果**。

### 统一数据出口

数仓建设不仅仅是为了数据内容而建设，同时也为了提高交付的数据质量与数据使用的便利性。如何保证数据质量以及推广数据的使用，我们提出了统一数据出口策略。在进行数据资产管理和统一数据出口之前，必须高质量地保障输出的数据质量，从而树立 OneData 数据服务体系的权威性。

**1. 交付标准化**

如何保证数据质量，满足什么样的数据才是可交付的，是数据建设者一直探索的问题。为了保证交付的严谨性，在具体化测试方案之前，我们结合数仓的特点明确了数据交付标准的五个特性，如下图所示：

![](https://p0.meituan.net/travelcube/3d7f4a6ce97acefd301686d6befcb7a364211.png)

图 12 交付标准化

《交付标准化》完善了整个交付细节，从根本上保证了数据的质量，如：业务测试保障数据的合理性、一致性；技术测试保障数据的唯一性、准确性；数据平台的稳定性和后期人工维护保障数据的及时性。

**2. 数据资产管理**

针对如何解决数据质量中维度与指标一致性以及如何提高数据易用性的问题，我们提出数据资产的概念，借助公司内部平台工具 “起源数据平台” 实现了整个数据资产管理，它的功能如下图所示：

![](https://p1.meituan.net/travelcube/643c9115e8abd7145c633a8f78dc0b27235617.png)

图 13 起源功能体系

借用起源数据平台，我们实现了：

*   统一指标管理，保证了指标定义、计算口径、数据来源的一致性。
*   统一维度管理，保证了维度定义、维度值的一致性。
*   统一数据出口，实现了维度和指标元数据信息的唯一出口，维值和指标数据的唯一出口。

通过交付标准化和数据资产管理，保证了数据质量与数据的易用性，同时我们也构建出 OneData 数据体系中数据指标管理的核心。

实践的成果
-----

### 流程改善

我们对开发过程进行梳理，服务于整个 OneData 体系。对需求分析、指标管理、模型设计、数据验证进行了改善，并结合 OneData 模型策略，改善了数仓管理流程。

![](https://p0.meituan.net/travelcube/e4e81434367f293d68274763ea2cec86360224.png)

图 14 数仓管理流程

### 数仓全景图

基于 OneData 主题建设，我们采用**面向业务**、**面向分析**的建设策略，形成数仓全景图，如下图所示：

![](https://p1.meituan.net/travelcube/5e2805db5f9ae659348d946b6b2e6d88419432.png)

图 15 数仓全景图

### 资产管理列表

基于起源数据平台形成的资产管理体系，如下图所示：

![](https://p0.meituan.net/travelcube/dd8ec354e31ad14f33da7e36a811ac68108113.png)

图 16 数据资产管理

### 项目收益

基于 OneData 建设成果，我们结合实际项目建设样例，对比以前未进行 OneData 建设时的收益。如下图所示：

![](https://p0.meituan.net/travelcube/f12748cfac4549c74bea54899417d8bd365668.png) ![](https://p0.meituan.net/travelcube/68ba42f3db45022eac7a94076ce23dca302119.png)

图 17 价值收益

总结和展望
-----

我们结合了 OneData 核心思想与特点，构建一种稳定、可靠的基础数据仓库，从底层保障了数据质量，同时完成 OneData 实践，形成自有的 OneData 理论体系。未来，我们还将在技术上引入实时数据仓库，满足灵活多样、低延时的数据需求；在业务层面会横向拓展其他业务领域，不间断地支撑核心业务的决策与分析。下一步，我们将为企业级 One Entity 数据中台（以 Data As a Service 为理念），提供强有力的数据支撑。在后续数仓维护过程中，不断地发现问题、解决问题和总结问题，保障数据稳定性、一致性和有效性，为核心业务构建价值链，最终形成企业级的数据资产。

作者
--

禄平，周成，黄浪，健平，高谦，美团数据研发工程师。