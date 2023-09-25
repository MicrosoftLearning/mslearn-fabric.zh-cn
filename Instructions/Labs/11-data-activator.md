---
lab:
  title: 在 Fabric 中使用 Data Activator
  module: Get started with Data Activator in Microsoft Fabric
---

# 在 Fabric 中使用 Data Activator

在 Microsoft Fabric 中，数据仓库提供用于大规模分析的关系数据库。 与湖屋中定义的表的默认只读 SQL 终结点不同，数据仓库提供完整的 SQL 语义；包括插入、更新和删除表中数据的功能。

完成本实验室大约需要 30 分钟。

> 注意：需要 Microsoft Fabric 许可证才能完成本练习。 有关如何启用免费 Fabric 试用版许可证的详细信息，请参阅 [Fabric 入门](https://learn.microsoft.com/fabric/get-started/fabric-trial)。 执行此操作需要 Microsoft 学校或工作帐户 。 如果没有，可以[注册 Microsoft Office 365 E3 或更高版本的试用版](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans)。

## 创建工作区

在 Fabric 中处理数据之前，在已启用的 Fabric 试用版中创建工作区。

1. 登录到 [Microsoft Fabric](https://app.fabric.microsoft.com) (`https://app.fabric.microsoft.com`)，然后选择 Power BI。
2. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
3. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
4. 打开新工作区时，它应为空，如下所示：

    ![Power BI 中空工作区的屏幕截图。](./Images/new-workspace.png)
