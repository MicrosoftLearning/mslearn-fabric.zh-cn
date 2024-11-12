---
lab:
  title: 使用 T-SQL 将数据加载到仓库中
  module: Load data into a warehouse in Microsoft Fabric
---

# 使用 T-SQL 将数据加载到仓库中

在 Microsoft Fabric 中，数据仓库提供用于大规模分析的关系数据库。 与湖屋中定义的表的默认只读 SQL 终结点不同，数据仓库提供完整的 SQL 语义；包括插入、更新和删除表中数据的功能。

完成本实验室大约需要 30 分钟。

> **注意**：需要 [Microsoft Fabric 试用版](https://learn.microsoft.com/fabric/get-started/fabric-trial) 才能完成本练习。

## 创建工作区

在 Fabric 中处理数据之前，创建一个已启用的 Fabric 试用版的工作区。

1. 在 [Microsoft Fabric 主页](https://app.fabric.microsoft.com/home?experience=fabric) (`https://app.fabric.microsoft.com/home?experience=fabric`) 上，选择“**Synapse 数据仓库**”。
1. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
1. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
1. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 创建湖屋并上传文件

由于方案中没有任何可用数据，因此必须引入用于加载仓库的数据。 将为将用于加载仓库的数据文件创建数据仓库。

1. 在“Synapse 数据工程”主页中，新建湖屋并为其指定名称 。

    大约一分钟后，一个新的空湖屋创建完成。 需要将一些数据引入数据湖屋进行分析。 有多种方法可以执行此操作，但在本练习中，需要把 CSV 文件下载到本地计算机（或实验室 VM，如果适用），然后上传到湖屋。

1. 从 `https://github.com/MicrosoftLearning/dp-data/raw/main/sales.csv` 下载本练习的文件。

1. 返回到包含你的湖屋的 Web 浏览器标签页，在“资源管理器”窗格的“Files”文件夹的“...”菜单中，依次选择“上传”和“上传文件”，然后将 sales.csv 文件从本地计算机（或实验室 VM，如果适用）上传到湖屋。************************

1. 上传文件后，选择“**Files**”。 验证 CSV 文件是否已上传，如下所示：

    ![湖屋中已上传文件的屏幕截图。](./Images/sales-file-upload.png)

## 在湖屋中创建表格

1. 在“**资源管理器**”窗格中 **sales.csv** 文件的“**...**”菜单中，选择“**加载到表**”，然后选择“**新建表**”。

1. 在“**将文件加载到新表**”对话框中提供以下信息。
    - **新表名：** staging_sales
    - **将标题用于列名：** 选定
    - **** 分隔符：,

1. 选择“加载”****。

## 创建仓库

现已有工作区、湖屋和包含所需数据的销售表，是时候创建数据仓库了。 “Synapse 数据仓库”主页包含创建新仓库的快捷方式：

1. 在“Synapse 数据仓库”主页中，使用所选的名称创建新的仓库 。

    大约一分钟后，一个新的仓库创建完成：

    ![新仓库的屏幕截图。](./Images/new-data-warehouse.png)

## 创建事实表、维度和视图

为“销售”数据创建事实表和维度。 还将创建指向湖屋的视图，这简化了将用于加载的存储过程中的代码。

1. 从工作区中，选择创建的仓库。

1. 在仓库“**资源管理器**”中，选择“**新建 SQL 查询**”，然后复制并运行以下查询。

    ```sql
    CREATE SCHEMA [Sales]
    GO
        
    IF NOT EXISTS (SELECT * FROM sys.tables WHERE name='Fact_Sales' AND SCHEMA_NAME(schema_id)='Sales')
        CREATE TABLE Sales.Fact_Sales (
            CustomerID VARCHAR(255) NOT NULL,
            ItemID VARCHAR(255) NOT NULL,
            SalesOrderNumber VARCHAR(30),
            SalesOrderLineNumber INT,
            OrderDate DATE,
            Quantity INT,
            TaxAmount FLOAT,
            UnitPrice FLOAT
        );
    
    IF NOT EXISTS (SELECT * FROM sys.tables WHERE name='Dim_Customer' AND SCHEMA_NAME(schema_id)='Sales')
        CREATE TABLE Sales.Dim_Customer (
            CustomerID VARCHAR(255) NOT NULL,
            CustomerName VARCHAR(255) NOT NULL,
            EmailAddress VARCHAR(255) NOT NULL
        );
        
    ALTER TABLE Sales.Dim_Customer add CONSTRAINT PK_Dim_Customer PRIMARY KEY NONCLUSTERED (CustomerID) NOT ENFORCED
    GO
    
    IF NOT EXISTS (SELECT * FROM sys.tables WHERE name='Dim_Item' AND SCHEMA_NAME(schema_id)='Sales')
        CREATE TABLE Sales.Dim_Item (
            ItemID VARCHAR(255) NOT NULL,
            ItemName VARCHAR(255) NOT NULL
        );
        
    ALTER TABLE Sales.Dim_Item add CONSTRAINT PK_Dim_Item PRIMARY KEY NONCLUSTERED (ItemID) NOT ENFORCED
    GO
    ```

    > **重要提示：** 在数据仓库中，表级别并不总是需要外键约束。 虽然外键约束有助于确保数据完整性，但它们也会增加 ETL（提取、转换、加载）过程的开销，并减缓数据加载。 在数据仓库中使用外键约束的决定应该基于对数据完整性和性能之间权衡的仔细考虑。

1. 在“**资源管理器**”中，导航到“**架构 >> 销售 >> 表格**”。 请注意刚刚创建的 *Fact_Sales*、*Dim_Customer* 和 *Dim_Item* 表。

1. 打开新的“**新建 SQL 查询**”编辑器，然后复制并运行以下查询。 使用创建的湖屋更新 *<your lakehouse name>*。

    ```sql
    CREATE VIEW Sales.Staging_Sales
    AS
    SELECT * FROM [<your lakehouse name>].[dbo].[staging_sales];
    ```

1. 在“**资源管理器**”中，导航到“**架构 >> 销售 >> 视图**”。 请注意创建的 *Staging_Sales* 视图。

## 将数据加载到仓库

现已创建了事实表和维度表，让我们创建存储过程，将数据从湖屋加载到仓库中。 由于在创建湖屋时会创建自动 SQL 端点，因此可以使用 T-SQL 和跨数据库查询从仓库直接访问湖屋中的数据。

为了简化本案例研究，将使用客户名称和商品名称作为主键。

1. 创建新的“**新建 SQL 查询**”编辑器，然后复制并运行以下查询。

    ```sql
    CREATE OR ALTER PROCEDURE Sales.LoadDataFromStaging (@OrderYear INT)
    AS
    BEGIN
        -- Load data into the Customer dimension table
        INSERT INTO Sales.Dim_Customer (CustomerID, CustomerName, EmailAddress)
        SELECT DISTINCT CustomerName, CustomerName, EmailAddress
        FROM [Sales].[Staging_Sales]
        WHERE YEAR(OrderDate) = @OrderYear
        AND NOT EXISTS (
            SELECT 1
            FROM Sales.Dim_Customer
            WHERE Sales.Dim_Customer.CustomerName = Sales.Staging_Sales.CustomerName
            AND Sales.Dim_Customer.EmailAddress = Sales.Staging_Sales.EmailAddress
        );
        
        -- Load data into the Item dimension table
        INSERT INTO Sales.Dim_Item (ItemID, ItemName)
        SELECT DISTINCT Item, Item
        FROM [Sales].[Staging_Sales]
        WHERE YEAR(OrderDate) = @OrderYear
        AND NOT EXISTS (
            SELECT 1
            FROM Sales.Dim_Item
            WHERE Sales.Dim_Item.ItemName = Sales.Staging_Sales.Item
        );
        
        -- Load data into the Sales fact table
        INSERT INTO Sales.Fact_Sales (CustomerID, ItemID, SalesOrderNumber, SalesOrderLineNumber, OrderDate, Quantity, TaxAmount, UnitPrice)
        SELECT CustomerName, Item, SalesOrderNumber, CAST(SalesOrderLineNumber AS INT), CAST(OrderDate AS DATE), CAST(Quantity AS INT), CAST(TaxAmount AS FLOAT), CAST(UnitPrice AS FLOAT)
        FROM [Sales].[Staging_Sales]
        WHERE YEAR(OrderDate) = @OrderYear;
    END
    ```
1. 创建新的“**新建 SQL 查询**”编辑器，然后复制并运行以下查询。

    ```sql
    EXEC Sales.LoadDataFromStaging 2021
    ```

    > **注意：** 在这种情况下，只加载 2021 年的数据。 但是，可以选择修改它以加载前几年的数据。

## 运行分析查询

运行一些分析查询来验证仓库中的数据。

1. 在顶部菜单上，选择“**新建 SQL 查询**”，然后复制并运行以下查询。

    ```sql
    SELECT c.CustomerName, SUM(s.UnitPrice * s.Quantity) AS TotalSales
    FROM Sales.Fact_Sales s
    JOIN Sales.Dim_Customer c
    ON s.CustomerID = c.CustomerID
    WHERE YEAR(s.OrderDate) = 2021
    GROUP BY c.CustomerName
    ORDER BY TotalSales DESC;
    ```

    > **注意：** 此查询按 2021 年的总销售额显示客户。 指定年份总销售额最高的客户是**谢德**，总销售额为 **14686.69**。 

1. 在顶部菜单上，选择“**新建 SQL 查询**”或重用同一编辑器，然后复制并运行以下查询。

    ```sql
    SELECT i.ItemName, SUM(s.UnitPrice * s.Quantity) AS TotalSales
    FROM Sales.Fact_Sales s
    JOIN Sales.Dim_Item i
    ON s.ItemID = i.ItemID
    WHERE YEAR(s.OrderDate) = 2021
    GROUP BY i.ItemName
    ORDER BY TotalSales DESC;

    ```

    > **注意：** 此查询显示 2021 年总销售额排名前几的销售项目。 这些结果表明，*Mountain-200 自行车*型号有黑色和银色两种颜色，是 2021 年最受客户欢迎的产品。

1. 在顶部菜单上，选择“**新建 SQL 查询**”或重用同一编辑器，然后复制并运行以下查询。

    ```sql
    WITH CategorizedSales AS (
    SELECT
        CASE
            WHEN i.ItemName LIKE '%Helmet%' THEN 'Helmet'
            WHEN i.ItemName LIKE '%Bike%' THEN 'Bike'
            WHEN i.ItemName LIKE '%Gloves%' THEN 'Gloves'
            ELSE 'Other'
        END AS Category,
        c.CustomerName,
        s.UnitPrice * s.Quantity AS Sales
    FROM Sales.Fact_Sales s
    JOIN Sales.Dim_Customer c
    ON s.CustomerID = c.CustomerID
    JOIN Sales.Dim_Item i
    ON s.ItemID = i.ItemID
    WHERE YEAR(s.OrderDate) = 2021
    ),
    RankedSales AS (
        SELECT
            Category,
            CustomerName,
            SUM(Sales) AS TotalSales,
            ROW_NUMBER() OVER (PARTITION BY Category ORDER BY SUM(Sales) DESC) AS SalesRank
        FROM CategorizedSales
        WHERE Category IN ('Helmet', 'Bike', 'Gloves')
        GROUP BY Category, CustomerName
    )
    SELECT Category, CustomerName, TotalSales
    FROM RankedSales
    WHERE SalesRank = 1
    ORDER BY TotalSales DESC;
    ```

    > **注意：** 此查询的结果显示每个类别的顶级客户：自行车、头盔和手套，基于其总销售额。 例如，Joan Coleman 是“手套”类别的最大客户********。
    >
    > 类别信息是使用字符串操作从 `ItemName` 列中提取的，因为维度表中没有单独的类别列。 这种方法假设项目名称遵循一致的命名约定。 如果项目名称不遵循一致的命名约定，则结果可能无法准确反映每个项目的真实类别。

在本练习中，创建了湖屋和包含多个表的数据仓库。 已经获取了数据，并使用跨数据库查询将数据从湖屋加载到仓库。 此外，还使用了查询工具来执行分析查询。

## 清理资源

如果已完成数据仓库探索，可以删除为本练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工具栏上的“...”菜单中，选择“工作区设置” 。
3. 在“常规”部分中，选择“删除此工作区”。********
