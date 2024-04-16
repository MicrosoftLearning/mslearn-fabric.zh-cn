---
lab:
  title: 在 Microsoft Fabric 中查询数据仓库
  module: Query a data warehouse in Microsoft Fabric
---

# 在 Microsoft Fabric 中查询数据仓库

在 Microsoft Fabric 中，数据仓库提供用于大规模分析的关系数据库。 通过 Microsoft Fabric 工作区中内置的一组丰富的体验，客户能够通过一个在 DirectLake 模式下与 Power BI 集成且易于使用、始终连接的语义模型来更快地获得见解。 

完成本实验室大约需要 30 分钟。

> **注意**：需要 [Microsoft Fabric 试用版](https://learn.microsoft.com/fabric/get-started/fabric-trial) 才能完成本练习。

## 创建工作区

在 Fabric 中处理数据之前，创建一个已启用的 Fabric 试用版的工作区。

1. 在 Microsoft Fabric 主页上，选择“Synapse 数据仓库”。[](https://app.fabric.microsoft.com)
1. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
1. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
1. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 创建示例数据仓库

现已有了工作空间，可以创建数据仓库了。

1. 在左下角，确保已选择“数据仓库”体验****。
1. 在主页上，选择“示例仓库”，并创建一个命名为 sample-dw 的新数据仓库************。

    大约一分钟后，系统将创建一个新仓库，并填充出租车行程分析场景的示例数据。

    ![新仓库的屏幕截图。](./Images/sample-data-warehouse.png)

## 查询数据仓库

SQL 查询编辑器支持 IntelliSense、代码完成、语法突出显示、客户端分析和验证。 可以运行数据定义语言 (DDL)、数据操作语言 (DML) 和数据控制语言 (DCL) 语句。

1. 在 sample-dw 数据仓库页的“新建 SQL 查询”下拉列表中，选择“新建 SQL 查询”************。

1. 在此新建的空白查询窗格中，输入以下 Transact-SQL 代码：

    ```sql
    SELECT 
    D.MonthName, 
    COUNT(*) AS TotalTrips, 
    SUM(T.TotalAmount) AS TotalRevenue 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.MonthName;
    ```

1. 使用“&#9655; 运行”按钮运行 SQL 脚本并查看结果，结果中按月显示了行程总数和总收入****。

1. 输入以下 Transact-SQL 代码：

    ```sql
   SELECT 
    D.DayName, 
    AVG(T.TripDurationSeconds) AS AvgDuration, 
    AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.DayName;
    ```

1. 运行修改后的查询并查看结果，结果显示了一个星期的某一天的平均行程时长和距离。

1. 输入以下 Transact-SQL 代码：

    ```sql
    SELECT TOP 10 
    G.City, 
    COUNT(*) AS TotalTrips 
    FROM dbo.Trip AS T
    JOIN dbo.Geography AS G
        ON T.PickupGeographyID=G.GeographyID
    GROUP BY G.City
    ORDER BY TotalTrips DESC;
    
    SELECT TOP 10 
        G.City, 
        COUNT(*) AS TotalTrips 
    FROM dbo.Trip AS T
    JOIN dbo.Geography AS G
        ON T.DropoffGeographyID=G.GeographyID
    GROUP BY G.City
    ORDER BY TotalTrips DESC;
    ```

1. 运行修改后的查询并查看结果，结果显示了前 10 个最火爆的上下客地点。

1. 关闭所有查询选项卡。

## 验证数据一致性

验证数据一致性对于确保数据准确可靠以便进行分析和决策非常重要。 不一致的数据可能导致不正确的分析和误导性结果。 

让我们查询数据仓库以检查一致性。

1. 在“新建 SQL 查询”下拉列表中，选择“新建 SQL 查询”********。

1. 在此新建的空白查询窗格中，输入以下 Transact-SQL 代码：

    ```sql
    -- Check for trips with unusually long duration
    SELECT COUNT(*) FROM dbo.Trip WHERE TripDurationSeconds > 86400; -- 24 hours
    ```

1. 运行修改后的查询并查看结果，结果显示了时长异常长的所有行程的详细信息。

1. 在“新建 SQL 查询”下拉列表中，选择“新建 SQL 查询”以添加第二个查询选项卡********。然后，在新的空查询选项卡中运行以下代码：

    ```sql
    -- Check for trips with negative trip duration
    SELECT COUNT(*) FROM dbo.Trip WHERE TripDurationSeconds < 0;
    ```

1. 在此新建的空白查询窗格中，输入并运行以下 Transact-SQL 代码：

    ```sql
    -- Remove trips with negative trip duration
    DELETE FROM dbo.Trip WHERE TripDurationSeconds < 0;
    ```

    > **注意：** 有多种方法可以处理不一致的数据。 除了移除数据外，另一种方法是将数据替换为其他值，例如平均值或中值。

1. 关闭所有查询选项卡。

## 另存为视图

假设你需要筛选一组用户的特定行程，这些用户将使用数据生成报告。

让我们根据之前使用的查询创建视图，并向其添加筛选器。

1. 在“新建 SQL 查询”下拉列表中，选择“新建 SQL 查询”********。

1. 在新的空白查询窗格中，重新输入并运行以下 Transact-SQL 代码：

    ```sql
    SELECT 
        D.DayName, 
        AVG(T.TripDurationSeconds) AS AvgDuration, 
        AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.DayName;
    ```

1. 修改查询以添加 `WHERE D.Month = 1`。 此操作会筛选数据，以仅包含 1 月的记录。 最终的查询应如下所示：

    ```sql
    SELECT 
        D.DayName, 
        AVG(T.TripDurationSeconds) AS AvgDuration, 
        AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    WHERE D.Month = 1
    GROUP BY D.DayName
    ```

1. 在查询中选择 SELECT 语句的文本。 然后，在“&#9655 运行”按钮旁边，选择“另存为视图”********。

1. 创建名为 vw_JanTrip 的新视图****。

1. 在资源管理器中，导航到“架构”>>“dbo”>>“视图”********。 记下刚刚创建的 vw_JanTrip 视图**。

1. 关闭所有查询选项卡。

> 其他信息****：有关查询数据仓库的详细信息，请参阅 Microsoft Fabric 文档中的[使用 SQL 查询编辑器查询](https://learn.microsoft.com/fabric/data-warehouse/sql-query-editor)。

## 清理资源

在本练习中，你已使用查询来获取对 Microsoft Fabric 数据仓库中的数据的见解。

如果已完成数据仓库探索，可以删除为本练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工作区页中，选择“工作区设置”****。
3. 在“常规”部分底部，选择“删除此工作区”********。
