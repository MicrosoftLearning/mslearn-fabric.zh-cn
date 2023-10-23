---
lab:
  title: 在 KQL 数据库中查询数据
  module: Query data from a Kusto Query database in Microsoft Fabric
---
# 在 Microsoft Fabric 中查询 Kusto 数据库入门
KQL 查询集是一种工具，可用于查询 KQL 数据库、修改和显示来自其中的查询结果。 可以将 KQL 查询集中的每个选项卡链接到不同的 KQL 数据库，并保存查询以供将来使用或与他人共享进行数据分析。 还可以更换任何选项卡的 KQL 数据库，以比较来自不同数据源的查询结果。

KQL 查询集使用 Kusto 查询语言（与许多 SQL 函数兼容）来创建查询。 为深入了解 [kusto 查询 (KQL) 语言](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/?context=%2Ffabric%2Fcontext%2Fcontext)， 

完成本实验室大约需要 25 分钟。

## 创建工作区

在 Fabric 中处理数据之前，在已启用的 Fabric 试用版中创建工作区。

1. 登录到 [Microsoft Fabric](https://app.fabric.microsoft.com) (`https://app.fabric.microsoft.com`)，然后选择 Power BI。
2. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
3. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
4. 打开新工作区时，它应为空，如下所示：

    ![Power BI 中空工作区的屏幕截图。](./Images/new-workspace.png)

在本实验室中，你将在 Fabric 中使用实时分析 (RTA)，通过示例事件流创建 KQL 数据库。 实时分析 (RTA) 可以方便地提供一个示例数据集，供你用来探索 RTA 的功能。 你将使用此示例数据创建 KQL | SQL 查询和查询集，用于分析一些实时数据，还可以在下游进程中使用。

## 创建 KQL 数据库

1. 在“实时分析”中，选择“KQL 数据库”框 。

   ![选择 kqldatabase 的图像](./Images/select-kqldatabase.png)

2. 系统会提示为 KQL 数据库命名

   ![名称 kqldatabase 的图像](./Images/name-kqldatabase.png)

3. 为 KQL 数据库指定一个你能记住的名称（如 MyStockData），然后按“创建” 。

4. 在“数据库详细信息”面板中，选择铅笔图标以在 OneLake 中打开可用性。

   ![“启用 onelake”的图像](./Images/enable-onelake-availability.png)

5. 从“开始获取数据”的选项中选择“示例数据”框。
 
   ![图像显示了一系列选择选项，并突出显示了示例数据](./Images/load-sample-data.png)

6. 从示例数据的选项中选择“指标分析”框。

   ![“选择实验室分析数据”的图像](./Images/create-sample-data.png)

7. 加载数据后，数据加载到 KQL 数据库中。 为此，可以选择表右侧的省略号，导航到“查询表”，然后选择“显示任意 100 条记录”。 

    ![图像显示从 RawServerMetrics 表中选择前 100 个文件](./Images/rawservermetrics-top-100.png)

> 注：首次运行此功能时，可能需要几秒钟来分配计算资源。

## 方案
在此方案中，你是一名分析师，负责从假定的 SQL Server 中查询要通过 Fabric 环境实现的原始指标的示例数据集。 使用 KQL 和 T-SQL 查询此数据并收集所需信息，以获取有关数据的信息化见解。

