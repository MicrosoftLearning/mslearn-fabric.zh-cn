---
lab:
  title: 在 Microsoft Fabric 中使用 API for GraphQL
  module: Get started with GraphQL in Microsoft Fabric
---

# 在 Microsoft Fabric 中使用 API for GraphQL

Microsoft Fabric API for GraphQL 是一个数据访问层，它支持使用广泛采用且熟悉的 API 技术快速高效地查询多个数据源。 使用 API 可以抽象化后端数据源的具体信息，以便可以专注于应用程序的逻辑，并在单个调用中提供客户端所需的所有数据。 GraphQL 使用简单的查询语言和易于操作的结果集，这样可以最大限度地减少应用程序在 Fabric 中访问数据所需的时间。

完成本实验室大约需要 30 分钟。

> **注意**：需要 [Microsoft Fabric 试用版](https://learn.microsoft.com/fabric/get-started/fabric-trial) 才能完成本练习。

## 创建工作区

在 Fabric 中处理数据之前，创建一个已启用的 Fabric 试用版的工作区。

1. 在 [Microsoft Fabric 主页](https://app.fabric.microsoft.com/home?experience=fabric)上，网址为：`https://app.fabric.microsoft.com/home?experience=fabric`。
1. 在左侧菜单栏中，选择“**新建工作区**”。
1. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
1. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 使用示例数据创建 SQL 数据库

现在有了工作区，可以创建 SQL 数据库了。

1. 在 Fabric 门户的左侧面板上，选择“**+ 新建物料项目**”。
1. 导航到“**存储数据**”部分，然后选择“**SQL 数据库**”。
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

1. 关闭所有查询选项卡。

## 创建 GraphQL API

首先，设置 GraphQL 终结点以公开销售订单数据。 此终结点允许基于各种参数（如日期、客户和产品）查询销售订单。

1. 在 Fabric 门户上，导航到工作区，然后选择“**+ 新建物料项目**”。
1. 导航到“**开发数据**”部分，然后选择“**API for GraphQL**”。
1. 提供名称，然后选择“**创建**”。
1. 在 API for GraphQL 主页上，选择“**选择数据源**”。
1. 如果系统提示选择连接选项，请选择“**使用单一登录 (SSO) 身份验证连接到 Fabric 数据源**”。
1. 在“**选择要连接的数据**”页上，选择之前创建的 `AdventureWorksLT` 数据库。
1. 选择“连接” 。
1. 在“**选择数据**”页上，选择“`SalesLT.Product`”表。 
1. 预览数据并选择“**加载**”。
1. 选择“**复制终结点**”并记下公共 URL 链接。 我们不需要此链接，但复制 API 地址时需要使用此链接。

## 禁用突变

创建 API 后，我们只想公开用于在此方案中读取操作的销售数据。

1. 在 API for GraphQL 的“**架构资源管理器**”上，展开“**突变**”。
1. 选择每个突变旁边的 **...**（省略号），然后选择“**禁用**”。

此时将阻止通过 API 对数据进行任何修改或更新。 这意味着数据将是只读的，用户只能查看或查询数据，但不能对其进行任何更改。

## 使用 GraphQL 查询数据

现在，让我们使用 GraphQL 查询数据，以查找名称以 *“HL Road Frame”* 开头的所有产品。

1. 在 GraphQL 查询编辑器中，输入并运行以下查询。

```json
query {
  products(filter: { Name: { startsWith: "HL Road Frame" } }) {
    items {
      ProductModelID
      Name
      ListPrice
      Color
      Size
      ModifiedDate
    }
  }
}
```

在此查询中，产品是主要类型，它包括 `ProductModelID`、`Name`、`ListPrice`、`Color`、`Size` 和 `ModifiedDate` 字段。 此查询将返回名称以 *“HL Road Frame”* 开头的产品列表。

> **详细信息**：请参阅 Microsoft Fabric 文档中[什么是 Microsoft Fabric API for GraphQL？](https://learn.microsoft.com/fabric/data-engineering/api-graphql-overview)，以了解平台中有关其他可用组件的详细信息。

在本练习中，你已在 Microsoft Fabric 中使用 GraphQL 从 SQL 数据库创建、查询和公开数据。

## 清理资源

如果已完成数据库探索，可删除为本练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工具栏上的“...”菜单中，选择“工作区设置” 。
3. 在“常规”部分中，选择“删除此工作区”。********

