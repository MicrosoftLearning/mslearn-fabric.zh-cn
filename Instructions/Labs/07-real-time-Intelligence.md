---
lab:
  title: Microsoft Fabric 中的实时智能入门
  module: Get started with Real-Time Intelligence in Microsoft Fabric
---

# Microsoft Fabric 中的实时智能入门

Microsoft Fabric 提供一个运行时，可用于使用 Kusto 查询语言 (KQL) 存储和查询数据。 Kusto 针对包含时间序列组件的数据（例如日志文件或 IoT 设备的实时数据）进行了优化。

完成本实验室大约需要 30 分钟。

> **注意**：需要 [Microsoft Fabric 试用版](https://learn.microsoft.com/fabric/get-started/fabric-trial) 才能完成本练习。

## 创建工作区

在 Fabric 中处理数据之前，创建一个已启用的 Fabric 试用版的工作区。

1. 在 [Microsoft Fabric 主页](https://app.fabric.microsoft.com)中，选择“实时智能”。****
1. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
1. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
1. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 下载 KQL 数据库的文件

现在已经有了工作区，可以下载要分析的数据文件了。

1. 从 [https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv) 下载用于本练习的数据文件，在本地计算机上（或者实验室 VM 上，如果适用）将其另存为 sales.csv
1. 返回到具有 Microsoft Fabric 体验的浏览器窗口。****

## 创建 KQL 数据库

Kusto 查询语言 (KQL) 用于查询 KQL 数据库中定义的表中的静态或流数据。 若要分析销售数据，必须在 KQL 数据库中创建一个表，并从文件引入数据。

1. 在 **Microsoft Fabric** 体验门户中，选择“实时智能”体验图像，如下所示：****

    ![所选 Fabric 体验主页的屏幕截图，其中选择了“实时智能”](./Images/fabric-experience-home.png)

2. 在“实时智能”体验的“主页”上，选择“KQL 数据库”，并使用所选的名称创建新数据库************。
3. 创建新数据库后，选择从本地文件获取数据的选项。 然后，通过选择以下选项，使用向导将数据导入新表：
    - 目标：
        - 数据库：已选择创建的数据库
        - **Table**：单击“新建表”左侧的 + 符号，创建名为“sales”的新表************

        ![新建表向导步骤 1](./Images/import-wizard-local-file-1.png?raw=true)

        - 现在，你将看到“将文件拖动到此处或浏览文件”超链接出现在同一窗口中****。

        ![新建表向导步骤 2](./Images/import-wizard-local-file-2.png?raw=true)

        - 浏览或将 sales.csv 拖到屏幕上，等待“状态”框变为绿色复选框，然后选择“下一步”********

        ![新建表向导步骤 3](./Images/import-wizard-local-file-3.png?raw=true)

        - 在此屏幕中，你将看到列标题位于第一行，尽管系统检测到列标题，但我们仍然需要移动这些行上方的“第一行是列标题”滑块，以避免出现任何错误****。
        
        ![新建表向导步骤 4](./Images/import-wizard-local-file-4.png?raw=true)

        - 选择此滑块后，将看到一切正常，选择面板右下角的“完成”按钮****。

        ![新建表向导步骤 5](./Images/import-wizard-local-file-5.png?raw=true)

        - 等待“摘要”屏幕中的步骤完成，其中包括：
            - 创建表 (sales)
            - 创建映射 (sales_mapping)
            - 数据排队
            - 引流
        - 选择“关闭”按钮****

        ![新建表向导步骤 6](./Images/import-wizard-local-file-6.png?raw=true)

> 注意：在本示例中，从文件导入了非常少量的静态数据，出于本练习的目的考虑，这是可以的。 实际上，可以使用 Kusto 来分析更大量的数据；包括来自 Azure 事件中心等流式处理源的实时数据。

## 使用 KQL 查询销售表

现在，数据库中已有一个数据表，可以使用 KQL 代码对其进行查询。

1. 确保突出显示了销售表。 在菜单栏中，选择“查询表”下拉列表，并从中选择“显示任意 100 条记录” 。

2. 一个新窗格会打开，其中包含查询及其结果。 

3. 按照以下方式更改查询：

    ```kusto
   sales
   | where Item == 'Road-250 Black, 48'
    ```

4. 运行查询。 然后查看结果，其中应只包含“Road-250 Black, 48”产品的销售订单行。

5. 按照以下方式更改查询：

    ```kusto
   sales
   | where Item == 'Road-250 Black, 48'
   | where datetime_part('year', OrderDate) > 2020
    ```

6. 运行查询并查看结果，其中应仅包含 2020 年之后制作的“Road-250 Black, 48”的销售订单。

7. 按照以下方式更改查询：

    ```kusto
   sales
   | where OrderDate between (datetime(2020-01-01 00:00:00) .. datetime(2020-12-31 23:59:59))
   | summarize TotalNetRevenue = sum(UnitPrice) by Item
   | sort by Item asc
    ```

8. 运行查询并查看结果，其中应包含 2020 年 1 月 1 日至 12 月 31 日期间每种产品的总净收入，按产品名称的升序排列。
9. 选择“另存为 KQL 查询集”，并将查询保存为“按产品划分的收入” 。

## 从 KQL 查询集创建 Power BI 报表

可以使用 KQL 查询集作为 Power BI 报表的基础。

1. 在查询集的查询工作台编辑器中，运行查询并等待结果。
2. 选择“生成 Power BI 报表”并等待报表编辑器打开。
3. 在报表编辑器的“数据”窗格中，展开“Kusto 查询结果”，然后选择“Item”和“TotalRevenue”   字段。
4. 在报表设计画布上，选择已添加的表可视化效果，然后在“可视化效果”窗格中，选择“簇状条形图” 。

    ![KQL 查询中的报表的屏幕截图。](./Images/kql-report.png)

5. 在 Power BI 窗口的“文件”菜单中，选择“保存”  。 然后在工作区中将报表保存为 Revenue by Item.pbix，其中湖屋和 KQL 数据库是使用非业务敏感度标签定义的 。
6. 关闭 Power BI 窗口，然后在左侧栏中选择工作区的图标。

    如有必要，请刷新“工作区”页以查看其包含的所有项。

7. 在工作区中的项列表中，请注意“Revenue by Item”报表已列出。

## 清理资源

在本练习中，你创建了一个湖屋，这是一个 KQL 数据库，用于分析上传到湖屋中的数据。 你使用 KQL 查询数据并创建查询集，然后使用该查询集创建 Power BI 报表。

如果已完成 KQL 数据库探索，可删除为本练习创建的工作区。

1. 在左侧栏中，选择你的工作区的图标。
2. 在工具栏上的“...”菜单中，选择“工作区设置” 。
3. 在“常规”部分中，选择“删除此工作区”。********
