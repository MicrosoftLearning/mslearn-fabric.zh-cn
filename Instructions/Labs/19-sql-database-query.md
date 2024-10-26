---
lab:
  title: 在 Microsoft Fabric 中使用 SQL 数据库
  module: Get started with SQL Database in Microsoft Fabric
---

# 在 Microsoft Fabric 中使用 SQL 数据库

Microsoft Fabric 中的 SQL 数据库是对开发人员友好的事务数据库，它基于Azure SQL 数据库，可以在 Fabric 中轻松创建操作数据库。 Fabric 中的 SQL 数据库使用 SQL 数据库引擎作为 Azure SQL 数据库。

完成本实验室大约需要 30 分钟。

> **注意**：需要 [Microsoft Fabric 试用版](https://learn.microsoft.com/fabric/get-started/fabric-trial) 才能完成本练习。

## 创建工作区

在 Fabric 中处理数据之前，创建一个已启用的 Fabric 试用版的工作区。

1. 在 [Microsoft Fabric 主页](https://app.fabric.microsoft.com/home?experience=fabric) (`https://app.fabric.microsoft.com/home?experience=fabric`) 上，选择“**Synapse 数据仓库**”。
1. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
1. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
1. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 使用示例数据创建 SQL 数据库

现在有了工作区，可以创建 SQL 数据库了。

1. 在左侧面板上选择“**+ 创建**”。
1. 导航到“**数据库**”部分，然后选择“**SQL 数据库**”。
1. 输入 **AdventureWorksLT** 作为数据库名称，然后选择“**创建**”。
1. 创建数据库后，可以从**示例数据库**卡将示例数据加载到数据库中。

    一分钟左右之后，数据库将填充场景的示例数据。

    ![使用示例数据加载的新数据库的屏幕截图。](./Images/sql-database-sample.png)

## 查询 SQL 数据库

SQL 查询编辑器支持 IntelliSense、代码完成、语法突出显示、客户端分析和验证。 可以运行数据定义语言 (DDL)、数据操作语言 (DML) 和数据控制语言 (DCL) 语句。

1. 在 **AdventureWorksLT** 数据库页中，导航到“**主页**”，然后选择“**新建查询**”。

1. 在新的空白查询窗格中，输入并运行以下 T-SQL 代码。

    ```sql
    SELECT 
        p.Name AS ProductName,
        pc.Name AS CategoryName,
        p.ListPrice
    FROM 
        SalesLT.Product p
    INNER JOIN 
        SalesLT.ProductCategory pc ON p.ProductCategoryID = pc.ProductCategoryID
    ORDER BY 
    p.ListPrice DESC;
    ```
    
    此查询联接 `Product` 和 `ProductCategory` 表以显示产品名称、类别及其列表价格，按价格降序排序。

1. 在新的查询编辑器中，输入并运行以下 T-SQL 代码。

    ```sql
   SELECT 
        c.FirstName,
        c.LastName,
        soh.OrderDate,
        soh.SubTotal
    FROM 
        SalesLT.Customer c
    INNER JOIN 
        SalesLT.SalesOrderHeader soh ON c.CustomerID = soh.CustomerID
    ORDER BY 
        soh.OrderDate DESC;
    ```

    此查询检索客户列表及其订单日期和分类汇总，按订单日期降序排序。 

1. 关闭所有查询选项卡。

## 将数据与外部数据源集成

你将把有关公共假日的外部数据与销售订单集成。 然后，你将确定与公共假日相吻合的销售订单，从而深入了解假日如何影响销售活动。

1. 导航到“**主页**”，然后选择“**新建查询**”。

1. 在新的空白查询窗格中，输入并运行以下 T-SQL 代码。

    ```sql
    CREATE TABLE SalesLT.PublicHolidays (
        CountryOrRegion NVARCHAR(50),
        HolidayName NVARCHAR(100),
        Date DATE,
        IsPaidTimeOff BIT
    );
    ```

    此查询将创建 `SalesLT.PublicHolidays` 表，为下一步做准备。

1. 在新的查询编辑器中，输入并运行以下 T-SQL 代码。

    ```sql
    INSERT INTO SalesLT.PublicHolidays (CountryOrRegion, HolidayName, Date, IsPaidTimeOff)
    SELECT CountryOrRegion, HolidayName, Date, IsPaidTimeOff
    FROM OPENROWSET 
    (BULK 'abs://holidaydatacontainer@azureopendatastorage.blob.core.windows.net/Processed/*.parquet'
    , FORMAT = 'PARQUET') AS [PublicHolidays]
    WHERE countryorRegion in ('Canada', 'United Kingdom', 'United States')
        AND YEAR([date]) = 2024
    ```
    
    此查询从 Azure Blob 存储 中的 Parquet 文件读取假日数据，对其进行筛选，使其仅包括 加拿大、英国和美国 2024 年的假日，然后将此筛选的数据插入 `SalesLT.PublicHolidays` 表中。    

1. 在新的或现有的查询编辑器中，输入并执行以下 T-SQL 代码。

    ```sql
    -- Insert new addresses into SalesLT.Address
    INSERT INTO SalesLT.Address (AddressLine1, City, StateProvince, CountryRegion, PostalCode, rowguid, ModifiedDate)
    VALUES
        ('123 Main St', 'Seattle', 'WA', 'United States', '98101', NEWID(), GETDATE()),
        ('456 Maple Ave', 'Toronto', 'ON', 'Canada', 'M5H 2N2', NEWID(), GETDATE()),
        ('789 Oak St', 'London', 'England', 'United Kingdom', 'EC1A 1BB', NEWID(), GETDATE());
    
    -- Insert new orders into SalesOrderHeader
    INSERT INTO SalesLT.SalesOrderHeader (
        SalesOrderID, RevisionNumber, OrderDate, DueDate, ShipDate, Status, OnlineOrderFlag, 
        PurchaseOrderNumber, AccountNumber, CustomerID, ShipToAddressID, BillToAddressID, 
        ShipMethod, CreditCardApprovalCode, SubTotal, TaxAmt, Freight, Comment, rowguid, ModifiedDate
    )
    VALUES
        (1001, 1, '2024-12-25', '2024-12-30', '2024-12-26', 1, 1, 'PO12345', 'AN123', 1, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '123 Main St'), 'Ground', '12345', 100.00, 10.00, 5.00, 'New Order 1', NEWID(), GETDATE()),
        (1002, 1, '2024-11-28', '2024-12-03', '2024-11-29', 1, 1, 'PO67890', 'AN456', 2, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '123 Main St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '456 Maple Ave'), 'Air', '67890', 200.00, 20.00, 10.00, 'New Order 2', NEWID(), GETDATE()),
        (1003, 1, '2024-02-19', '2024-02-24', '2024-02-20', 1, 1, 'PO54321', 'AN789', 3, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '456 Maple Ave'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), 'Sea', '54321', 300.00, 30.00, 15.00, 'New Order 3', NEWID(), GETDATE()),
        (1004, 1, '2024-05-27', '2024-06-01', '2024-05-28', 1, 1, 'PO98765', 'AN321', 4, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), 'Ground', '98765', 400.00, 40.00, 20.00, 'New Order 4', NEWID(), GETDATE());
    ```

    此代码将新地址和订单添加到数据库，模拟来自不同国家/地区的虚构订单。

1. 在新的或现有的查询编辑器中，输入并执行以下 T-SQL 代码。

    ```sql
    SELECT DISTINCT soh.SalesOrderID, soh.OrderDate, ph.HolidayName, ph.CountryOrRegion
    FROM SalesLT.SalesOrderHeader AS soh
    INNER JOIN SalesLT.Address a
        ON a.AddressID = soh.ShipToAddressID
    INNER JOIN SalesLT.PublicHolidays AS ph
        ON soh.OrderDate = ph.Date AND a.CountryRegion = ph.CountryOrRegion
    ```

    花点时间观察结果，注意查询如何标识与相应国家/地区的公共假日相吻合的销售订单。 这可以提供关于订单模式和假日对销售活动的潜在影响的有价值的见解。

1. 关闭所有查询选项卡。

## 保护数据

假设特定用户组只能访问来自美国的数据以生成报表。

让我们根据之前使用的查询创建视图，并向其添加筛选器。

1. 在新的空白查询窗格中，输入并运行以下 T-SQL 代码。

    ```sql
    CREATE VIEW SalesLT.vw_SalesOrderHoliday AS
    SELECT DISTINCT soh.SalesOrderID, soh.OrderDate, ph.HolidayName, ph.CountryOrRegion
    FROM SalesLT.SalesOrderHeader AS soh
    INNER JOIN SalesLT.Address a
        ON a.AddressID = soh.ShipToAddressID
    INNER JOIN SalesLT.PublicHolidays AS ph
        ON soh.OrderDate = ph.Date AND a.CountryRegion = ph.CountryOrRegion
    WHERE a.CountryRegion = 'United Kingdom';
    ```

1. 在新的或现有的查询编辑器中，输入并执行以下 T-SQL 代码。

    ```sql
    -- Create the role
    CREATE ROLE SalesOrderRole;
    
    -- Grant select permission on the view to the role
    GRANT SELECT ON SalesLT.vw_SalesOrderHoliday TO SalesOrderRole;
    ```

    任何以成员身份添加到 `SalesOrderRole` 角色的用户仅有权访问筛选的视图。 如果此角色中的用户尝试访问任何其他用户对象，他们将收到如下所示的错误消息：

    ```
    Msg 229, Level 14, State 5, Line 1
    The SELECT permission was denied on the object 'ObjectName', database 'DatabaseName', schema 'SchemaName'.
    ```

> **详细信息**：请参阅 Microsoft Fabric 文档中的[什么是 Microsoft Fabric？](https://learn.microsoft.com/fabric/get-started/microsoft-fabric-overview)，以了解有关平台中其他可用组件的详细信息。

在本练习中，你已在 Microsoft Fabric 的 SQL 数据库中创建、导入外部数据、查询和保护数据。

## 清理资源

如果已完成数据库探索，可删除为本练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工具栏上的“...”菜单中，选择“工作区设置” 。
3. 在“常规”部分中，选择“删除此工作区”。********
