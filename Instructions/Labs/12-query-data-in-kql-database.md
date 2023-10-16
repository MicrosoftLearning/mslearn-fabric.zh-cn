---
lab:
  title: 在 KQL 数据库中查询数据
  module: Query data from a Kusto Query database in Microsoft Fabric
---
# 在 Microsoft Fabric 中查询 Kusto 数据库入门
KQL 查询集是一种工具，可用于查询 KQL 数据库、修改和显示来自其中的查询结果。 可以将 KQL 查询集中的每个选项卡链接到不同的 KQL 数据库，并保存查询以供将来使用或与他人共享进行数据分析。 还可以更换任何选项卡的 KQL 数据库，以比较来自不同数据源的查询结果。

KQL 查询集使用 Kusto 查询语言（与许多 SQL 函数兼容）来创建查询。 为深入了解 [kusto 查询 (KQL) 语言](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/?context=%2Ffabric%2Fcontext%2Fcontext)， 

## 创建工作区

在 Fabric 中处理数据之前，在已启用的 Fabric 试用版中创建工作区。

1. 登录到 [Microsoft Fabric](https://app.fabric.microsoft.com) (`https://app.fabric.microsoft.com`)，然后选择 Power BI。
2. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
3. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
4. 打开新工作区时，它应为空，如下所示：

    ![Power BI 中空工作区的屏幕截图。](./Images/new-workspace.png)

在本实验室中，你将在 Fabric 中使用实时分析 (RTA)，通过示例事件流创建 KQL 数据库。 实时分析 (RTA) 可以方便地提供一个示例数据集，供你用来探索 RTA 的功能。 你将使用此示例数据创建 KQL | SQL 查询和查询集，用于分析一些实时数据，还可以在下游进程中使用。


## 方案
在此方案中，你是一名分析师，负责查询一个要从 Fabric 环境实现的示例数据集。



Kusto 查询是读取数据、处理数据和显示结果的一种方法。 查询以易于使用的纯文本编写。 Kusto 查询可以有一个或多个以表或图的形式显示数据的语句。

Table 语句包含一些处理表数据的运算符。 每个运算符采用表作为输入，并输出一个表。 运算符之间由竖线 (|) 联接。 数据从一个运算符转移到另一个运算符。 每个运算符都以某种方式更改并传递数据。

可以将其想象成漏斗，一开始是整个数据表。 各个运算符筛选、排序或汇总数据。 运算符的顺序会产生影响，因为它们是一个接一个地按顺序工作。 在漏斗末尾，会获得最终输出。

这些运算符特定于 KQL，但可能与 SQL 或其他语言中的情况类似。