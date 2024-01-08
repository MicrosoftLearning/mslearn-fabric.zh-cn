---
lab:
  title: 使用 Apache Spark 分析数据
  module: Use Apache Spark to work with files in a lakehouse
---

# 使用 Apache Spark 分析数据

Apache Spark 是用于分布式数据处理的开放源代码引擎，广泛用于探索、处理和分析 Data Lake Storage 中的大量数据。 Spark 在许多数据平台产品中作为处理选项提供，包括 Azure HDInsight、Azure Databricks、Azure Synapse Analytics 和 Microsoft Fabric。 Spark 的优点之一是支持各种编程语言，包括 Java、Scala、Python 和 SQL；这让 Spark 一种非常灵活的数据处理工作负载（包括数据清理和操作、统计分析和机器学习以及数据分析和可视化）解决方案。

完成本实验室大约需要 45 分钟。

> 注意：需要 Microsoft 学校或工作帐户才能完成本练习。 如果没有该帐户，可以[注册 Microsoft Office 365 E3 或更高版本的试用版](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans)。

## 创建工作区

在 Fabric 中处理数据之前，创建一个已启用的 Fabric 试用版的工作区。

1. 在 [Microsoft Fabric 主页](https://app.fabric.microsoft.com)中，选择“Synapse 数据工程”。****
1. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
1. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
1. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 创建湖屋并上传文件

现在已经有了工作区，可以为要分析的数据文件创建数据湖屋了。

1. 在“Synapse 数据工程”主页中，新建湖屋并为其指定名称 。

    大约一分钟后，一个新的空湖屋创建完成。 需要将一些数据引入数据湖屋进行分析。 可通过多种方法执行此操作，但在本练习中，只需将文本文件的文件夹下载并解压缩到本地计算机（或实验室 VM，如适用），然后将其上传到湖屋。

1. 从 [https://github.com/MicrosoftLearning/dp-data/raw/main/orders.zip](https://github.com/MicrosoftLearning/dp-data/raw/main/orders.zip) 下载并解压缩本练习的数据文件。

1. 解压缩存档文件后，验证是否有名为 orders 的文件夹，其中是否包含名为 2019.csv、2020.csv 和 2021.csv 的 CSV 文件   。
1. 返回到包含湖屋的 Web 浏览器标签页，在“资源管理器”窗格的 Files 文件夹的“...”菜单中，依次选择“上传”和“上传文件夹”，然后将从 orders 文件夹从本地计算机（或实验室 VM，如适用）上传到湖屋     。
1. 上传文件后，展开 Files 并选择 orders 文件夹；然后验证 CSV 文件是否已上传，如下所示 ：

    ![湖屋中已上传文件的屏幕截图。](./Images/uploaded-files.png)

## 创建笔记本

要在 Apache Spark 中处理数据，可创建笔记本。 笔记本提供了一个交互式环境，你可在其中编写和运行多种语言的代码，并添加注释来记录它。

1. 在主页上查看数据湖中 orders 文件夹的内容时，在“打开笔记本”菜单中选择“新建笔记本”   。

    几秒钟后，一个包含单个单元格的新笔记本将会打开。 笔记本由一个或多个单元格组成，这些单元格可以包含代码或 markdown（格式化文本） 。

2. 选择第一个单元格（当前是代码单元格），然后在其右上角的动态工具栏中，使用 M&#8595; 按钮将单元格转换为 markdown 单元格。

    当单元格更改为 markdown 单元格时，它包含的文本将会呈现。

3. 使用 &#128393;（编辑）按钮将单元格切换到编辑模式，然后修改 markdown，如下所示：

    ```
   # Sales order data exploration

   Use the code in this notebook to explore sales order data.
    ```

4. 单击笔记本中单元格以外的任意位置，即可停止编辑并查看呈现的 markdown。

## 将数据加载到数据帧中

现在你可运行将数据加载到数据帧中的代码。 Spark 中的数据帧类似于 Python 中的 Pandas 数据帧，提供一种用于处理行和列中的数据的通用结构。

> 注意：Spark 支持多种编码语言，包括 Scala、Java 等。 本练习将使用 Python 的 Spark 优化变体 PySpark。 PySpark 是 Spark 中最常用的语言之一，也是 Fabric 笔记本中的默认语言。

1. 在笔记本可见的情况下，展开 Files 列表并选择 orders 文件夹，以便 CSV 文件列在笔记本编辑器旁边，如下所示 ：

    ![包含 Files 窗格的笔记本的屏幕截图。](./Images/notebook-files.png)

2. 在 2019.csv 的“...”菜单中，选择“加载数据” > “Spark”   。 应在笔记本中添加包含以下代码的新代码单元格：

    ```python
   df = spark.read.format("csv").option("header","true").load("Files/orders/2019.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/orders/2019.csv".
   display(df)
    ```

    > 提示：可使用 << 图标隐藏左侧的湖屋资源管理器窗格 。 这样做有助于专注于笔记本。

3. 使用单元格左侧的“&#9655; 运行单元格”按钮运行单元格。

    > 注意：由于这是你第一次运行 Spark 代码，因此必须启动 Spark 会话。 这意味着会话中的第一次运行可能需要一分钟左右才能完成。 后续运行速度会更快。

4. 单元格命令完成后，查看单元格下方的输出，该输出应如下所示：

    | Index | SO43701 | 11 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com | Mountain-100 Silver, 44 | 16 | 3399.99 | 271.9992 |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO43704 | 1 | 2019-07-01 | Julio Ruiz | julio1@adventure-works.com | Mountain-100 Black, 48 | 1 | 3374.99 | 269.9992 |
    | 2 | SO43705 | 1 | 2019-07-01 | Curtis Lu | curtis9@adventure-works.com | Mountain-100 Silver, 38 | 1 | 3399.99 | 271.9992 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    输出显示 2019.csv 文件中的数据行和列。 但请注意，列标题看起来不正常。 用于将数据加载到数据帧的默认代码假定 CSV 文件的第一行包含列名，但在本例中，CSV 文件只包含数据，没有标头信息。

5. 修改代码以将 header 选项设置为 false，如下所示 ：

    ```python
   df = spark.read.format("csv").option("header","false").load("Files/orders/2019.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/orders/2019.csv".
   display(df)
    ```

6. 重新运行单元格并查看输出，输出应如下所示：

   | Index | _c0 | _c1 | _c2 | _c3 | _c4 | _c5 | _c6 | _c7 | _c8 |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO43701 | 11 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com | Mountain-100 Silver, 44 | 16 | 3399.99 | 271.9992 |
    | 2 | SO43704 | 1 | 2019-07-01 | Julio Ruiz | julio1@adventure-works.com | Mountain-100 Black, 48 | 1 | 3374.99 | 269.9992 |
    | 3 | SO43705 | 1 | 2019-07-01 | Curtis Lu | curtis9@adventure-works.com | Mountain-100 Silver, 38 | 1 | 3399.99 | 271.9992 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    现在，数据帧正确地将第一行作为数据值包含在内，但列名称是自动生成的，并不是很有用。 要理解数据，需要为文件中的数据值显式定义正确的架构和数据类型。

7. 按如下所示修改代码以定义架构，并在加载数据时应用该架构：

    ```python
   from pyspark.sql.types import *

   orderSchema = StructType([
       StructField("SalesOrderNumber", StringType()),
       StructField("SalesOrderLineNumber", IntegerType()),
       StructField("OrderDate", DateType()),
       StructField("CustomerName", StringType()),
       StructField("Email", StringType()),
       StructField("Item", StringType()),
       StructField("Quantity", IntegerType()),
       StructField("UnitPrice", FloatType()),
       StructField("Tax", FloatType())
       ])

   df = spark.read.format("csv").schema(orderSchema).load("Files/orders/2019.csv")
   display(df)
    ```

8. 运行修改后的单元格并查看输出，输出应如下所示：

   | Index | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | 电子邮件 | 项 | 数量 | 单价 | 税款 |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO43701 | 11 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com | Mountain-100 Silver, 44 | 16 | 3399.99 | 271.9992 |
    | 2 | SO43704 | 1 | 2019-07-01 | Julio Ruiz | julio1@adventure-works.com | Mountain-100 Black, 48 | 1 | 3374.99 | 269.9992 |
    | 3 | SO43705 | 1 | 2019-07-01 | Curtis Lu | curtis9@adventure-works.com | Mountain-100 Silver, 38 | 1 | 3399.99 | 271.9992 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    现在，数据帧包含正确的列名（除了 Index，这是所有数据帧中基于每一行的序号位置的内置列）。 列的数据类型是使用 Spark SQL 库中定义的一组标准类型指定的，这些类型是在单元格开始时导入的。

9. 通过查看数据帧确认更改是否已应用于数据。 运行以下单元格：

    ```python
   display(df)
    ```

10. 数据帧仅包含 2019.csv 文件中的数据。 修改代码，使文件路径使用 \* 通配符从 orders 文件夹的所有文件中读取销售订单数据：

    ```python
    from pyspark.sql.types import *

    orderSchema = StructType([
       StructField("SalesOrderNumber", StringType()),
       StructField("SalesOrderLineNumber", IntegerType()),
       StructField("OrderDate", DateType()),
       StructField("CustomerName", StringType()),
       StructField("Email", StringType()),
       StructField("Item", StringType()),
       StructField("Quantity", IntegerType()),
       StructField("UnitPrice", FloatType()),
       StructField("Tax", FloatType())
       ])

    df = spark.read.format("csv").schema(orderSchema).load("Files/orders/*.csv")
    display(df)
    ```

11. 运行修改后的代码单元并查看输出，该输出现应包括 2019 年、2020 年和 2021 年的销售情况。

    注意：仅显示一部分行，因此可能无法查看所有年份的示例。

## 探索数据帧中的数据

数据帧对象包含各种可用于筛选、分组或以其他方式操作其所含数据的函数。

### 筛选数据帧

1. 使用单元格输出下方的“+ 代码”图标将新的代码单元格添加到笔记本，并在其中输入以下代码。

    ```Python
   customers = df['CustomerName', 'Email']
   print(customers.count())
   print(customers.distinct().count())
   display(customers.distinct())
    ```

2. 运行新的代码单元格，并查看结果。 观察以下详细信息：
    - 对数据帧执行操作时，结果是一个新数据帧（在本例中，是一个新客户数据帧，它是通过从 df 数据帧中选择特定的列子集来创建的）
    - 数据帧提供 count 和 distinct 等函数，可用于汇总和筛选它们包含的数据 。
    - `dataframe['Field1', 'Field2', ...]` 语法是用于定义列子集的快速方法。 还可以使用 select 方法，所以上述代码的第一行可以编写为 `customers = df.select("CustomerName", "Email")`

3. 按如下所示修改代码：

    ```Python
   customers = df.select("CustomerName", "Email").where(df['Item']=='Road-250 Red, 52')
   print(customers.count())
   print(customers.distinct().count())
   display(customers.distinct())
    ```

4. 运行修改后的代码来查看已购买 Road-250 Red, 52 产品的客户。 请注意，可以“链接”多个函数，使一个函数的输出成为下一个函数的输入；在这种情况下，由 select 方法创建的数据帧是用于应用筛选条件的 where 方法的源数据帧 。

### 在数据帧中对数据进行聚合和分组

1. 在笔记本中新增一个代码单元格，并在其中输入以下代码：

    ```Python
   productSales = df.select("Item", "Quantity").groupBy("Item").sum()
   display(productSales)
    ```

2. 运行添加的代码单元格，并注意结果显示按产品分组的订单数量之和。 groupBy 方法按“项”对行进行分组，随后将 sum 聚合函数应用于所有剩余的数值列（在本例中为“数量”）

3. 在笔记本中再次新增一个代码单元格，并在其中输入以下代码：

    ```Python
   from pyspark.sql.functions import *

   yearlySales = df.select(year(col("OrderDate")).alias("Year")).groupBy("Year").count().orderBy("Year")
   display(yearlySales)
    ```

4. 运行添加的代码单元格，并注意结果显示每年的销售订单数。 请注意，select 方法包含一个 SQL year 函数以提取 OrderDate 字段的年份部分（这也是代码包含 import 语句以从 Spark SQL 库导入函数的原因） 。 然后它使用 alias 方法向提取的年份值分配一个列名。 接下来，按派生的“年份”列对数据进行分组，并计算每个组中的行计数，最后使用 orderBy 方法对生成的数据帧进行排序。

## 使用 Spark 转换数据文件

数据工程师的常见任务是引入特定格式或结构的数据并对其进行转换，以便进行进一步下游处理或分析。

### 使用数据帧方法和函数转换数据

1. 在笔记本中再次新增一个代码单元格，并在其中输入以下代码：

    ```Python
   from pyspark.sql.functions import *

   ## Create Year and Month columns
   transformed_df = df.withColumn("Year", year(col("OrderDate"))).withColumn("Month", month(col("OrderDate")))

   # Create the new FirstName and LastName fields
   transformed_df = transformed_df.withColumn("FirstName", split(col("CustomerName"), " ").getItem(0)).withColumn("LastName", split(col("CustomerName"), " ").getItem(1))

   # Filter and reorder columns
   transformed_df = transformed_df["SalesOrderNumber", "SalesOrderLineNumber", "OrderDate", "Year", "Month", "FirstName", "LastName", "Email", "Item", "Quantity", "UnitPrice", "Tax"]

   # Display the first five orders
   display(transformed_df.limit(5))
    ```

2. 运行代码，通过以下转换从原始订单数据创建新的数据帧：
    - 基于 OrderDate 列添加 Year 和 Month 列  。
    - 基于 CustomerName 列添加 FirstName 和 LastName 列  。
    - 对列进行筛选并重新排序，删除 CustomerName 列。

3. 查看输出并验证是否已对数据进行了转换。

    你可使用 Spark SQL 库的完备功能通过筛选行、派生、删除、重命名列以及应用其他所需的数据修改来转换数据。

    > 提示：请参阅 [Spark 数据帧文档](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/dataframe.html)，详细了解数据帧对象的方法。

### 保存转换后的数据

1. 添加包含以下代码的新单元格，以 Parquet 格式保存转换后的数据帧（如果数据已存在，则覆盖数据）：

    ```python
   transformed_df.write.mode("overwrite").parquet('Files/transformed_data/orders')
   print ("Transformed data saved!")
    ```

    > 注意：对于用于进一步分析或引入到分析存储的数据文件，通常首选 Parquet 格式。 Parquet 是一种非常高效的格式，大多数大规模数据分析系统都支持这种格式。 事实上，有时数据转换要求可能只是将数据从其他格式（如 CSV）转换为 Parquet！

2. 运行单元格并等待数据已保存的消息。 然后在左侧的“资源管理器”窗格中，在 Files 节点的“...”菜单中，选择“刷新”；然后选择 transformed_orders 文件夹以验证它是否包含名为 orders 的新文件夹，该文件夹是否又包含一个或多个 Parquet 文件     。

    ![包含 parquet 文件的文件夹的屏幕截图。](./Images/saved-parquet.png)

3. 添加包含以下代码的新单元格，以从 transformed_orders/orders 文件夹中的 parquet 文件加载新的数据帧：

    ```python
   orders_df = spark.read.format("parquet").load("Files/transformed_data/orders")
   display(orders_df)
    ```

4. 运行单元格并验证结果是否显示已从 parquet 文件加载的订单数据。

### 以分区文件的形式保存数据

1. 添加包含以下代码的新单元格；该代码保存数据帧，并按年份和月份对数据进行分区 ：

    ```python
   orders_df.write.partitionBy("Year","Month").mode("overwrite").parquet("Files/partitioned_data")
   print ("Transformed data saved!")
    ```

2. 运行单元格并等待数据已保存的消息。 然后在左侧的“资源管理器”窗格中，在 Files 节点的“...”菜单中，选择“刷新”；然后展开 partitioned_orders 文件夹以验证它是否包含名为 *Year=xxxx*** 的文件夹层次结构，每个文件夹是否包含名为 *Month=xxxx*** 的文件夹     。 每个月份文件夹均包含一个 parquet 文件，该文件包含该月的订单。

    ![分区数据文件层次结构的屏幕截图。](./Images/partitioned-files.png)

    在处理大量数据时，对数据文件进行分区是优化性能的常用方法。 此方法可以显著提高性能，并使数据筛选更加容易。

3. 添加包含以下代码的新单元格，以从 orders.parquet 文件加载新的数据帧：

    ```python
   orders_2021_df = spark.read.format("parquet").load("Files/partitioned_data/Year=2021/Month=*")
   display(orders_2021_df)
    ```

4. 运行单元格并验证结果是否显示 2021 年的销售订单数据。 请注意，数据帧不包含路径中指定的分区列（“年份”和“月份”） 。

## 使用表和 SQL

如你所见，数据帧对象的本机方法能让你非常有效地查询和分析文件中的数据。 但是，许多数据分析师更习惯使用可通过 SQL 语法查询的表。 Spark 提供可在其中定义关系表的元存储。 提供数据帧对象的 Spark SQL 库还支持使用 SQL 语句查询元存储中的表。 借助 Spark 的这些功能，可将数据湖的灵活性与关系数据仓库的结构化数据架构和基于 SQL 的查询相结合，因此有了“数据湖屋”。

### 创建表

Spark 元存储中的表是数据湖中文件的关系抽象。 表可以管理（在这种情况下，文件由元存储管理），也可以是外部的（在这种情况下，表引用数据湖中独立于元存储进行管理的文件位置） 。

1. 在笔记本中新增一个代码单元格，然后输入以下代码，该代码将销售订单数据的数据帧保存为名为 salesorders 的表：

    ```Python
   # Create a new table
   df.write.format("delta").saveAsTable("salesorders")

   # Get the table description
   spark.sql("DESCRIBE EXTENDED salesorders").show(truncate=False)
    ```

    > 注意：此示例有几个注意事项。 首先，未提供显式路径，因此表的文件将由元存储管理。 其次，表以增量格式保存。 可基于多种文件格式（包括 CSV、Parquet、Avro 等）创建表，但 Delta Lake 是一种将关系数据库功能添加到表的 Spark 技术；包括对事务、行版本控制和其他有用功能的支持。 对于 Fabric 中的数据湖屋，以增量格式创建表是首选。

2. 运行代码单元格并查看输出，该输出描述新表的定义。

3. 在“资源管理器”窗格中 Tables 文件夹的“...”菜单中，选择“刷新”   。 然后展开 Tables 节点并验证是否已创建 salesorders 表 。

    ![资源管理器中 salesorder 表的屏幕截图。](./Images/table-view.png)

5. 在 salesorders 表的“...”菜单中，选择“加载数据” > “Spark”   。

    在笔记本中添加包含类似于以下示例的代码的新代码单元格：

    ```Python
   df = spark.sql("SELECT * FROM [your_lakehouse].salesorders LIMIT 1000")
   display(df)
    ```

6. 运行新代码，该代码使用 Spark SQL 库在 PySpark 代码中嵌入针对 salesorder 表的 SQL 查询，并将查询结果加载到数据帧中。

### 在单元格中运行 SQL

虽然能够将 SQL 语句嵌入到包含 PySpark 代码的单元格中非常有用，但数据分析师通常只想直接使用 SQL。

1. 在笔记本中新增一个代码单元格，并在其中输入以下代码：

    ```sql
   %%sql
   SELECT YEAR(OrderDate) AS OrderYear,
          SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue
   FROM salesorders
   GROUP BY YEAR(OrderDate)
   ORDER BY OrderYear;
    ```

2. 运行单元格并查看结果。 观察以下情况：
    - 单元格开头的 `%%sql` 行（称为 magic）指示应使用 Spark SQL 语言运行时来运行此单元格中的代码，而不是 PySpark。
    - SQL 代码引用以前创建的 salesorders 表。
    - SQL 查询的输出将自动显示为单元格下的结果。

> 注意：有关 Spark SQL 和数据帧的详细信息，请参阅 [Spark SQL 文档](https://spark.apache.org/docs/2.2.0/sql-programming-guide.html)。

## 使用 Spark 直观呈现数据

众所周知，一张图片胜过千言万语，而图表通常胜过千行数据。 虽然 Fabric 中的笔记本包含一个内置图表视图，用于从数据帧或 Spark SQL 查询显示的数据，但它并非专为全面的图表而设计。 但是，可以使用 Python 图形库（如 matplotlib 和 seaborn）根据数据帧中的数据创建图表 。

### 以图表形式查看结果

1. 在笔记本中新增一个代码单元格，并在其中输入以下代码：

    ```sql
   %%sql
   SELECT * FROM salesorders
    ```

2. 运行代码并观察它是否从之前创建的 salesorders 视图返回数据。
3. 在单元格下方的结果部分中，将“视图”选项从“表格”更改为“图表”  。
4. 使用图表右上角的“视图选项”按钮显示图表的选项窗格。 然后按如下方式设置选项并选择“应用”：
    - **图表类型**：条形图
    - **键**：项
    - **值**：数量
    - **序列组**：留空
    - **聚合**：Sum
    - **堆积**：未选中

5. 验证图表是否如下所示：

    ![按订单总数排列的产品条形图的屏幕截图](./Images/notebook-chart.png)

### matplotlib 入门

1. 在笔记本中新增一个代码单元格，并在其中输入以下代码：

    ```Python
   sqlQuery = "SELECT CAST(YEAR(OrderDate) AS CHAR(4)) AS OrderYear, \
                   SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue \
               FROM salesorders \
               GROUP BY CAST(YEAR(OrderDate) AS CHAR(4)) \
               ORDER BY OrderYear"
   df_spark = spark.sql(sqlQuery)
   df_spark.show()
    ```

2. 运行代码并观察它是否返回包含年收入的 Spark 数据帧。

    若要将数据可视化为图表，要首先使用 matplotlib Python 库。 此库是其他许多库所基于的核心绘图库，在创建图表方面提供了极大的灵活性。

3. 在笔记本中新增一个代码单元格，并在其中添加以下代码：

    ```Python
   from matplotlib import pyplot as plt

   # matplotlib requires a Pandas dataframe, not a Spark one
   df_sales = df_spark.toPandas()

   # Create a bar plot of revenue by year
   plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'])

   # Display the plot
   plt.show()
    ```

4. 运行单元格并查看结果，结果中包含每年总收入的柱形图。 请注意用于生成此图表的代码的以下功能：
    - matplotlib 库需要 Pandas 数据帧，因此需要将 Spark SQL 查询返回的 Spark 数据帧转换为此格式 。
    - matplotlib 库的核心是 pyplot 对象 。 这是大多数绘图功能的基础。
    - 默认设置会生成一个可用图表，但它有很大的自定义空间

5. 修改代码以绘制图表，如下所示：

    ```Python
   from matplotlib import pyplot as plt

   # Clear the plot area
   plt.clf()

   # Create a bar plot of revenue by year
   plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

   # Customize the chart
   plt.title('Revenue by Year')
   plt.xlabel('Year')
   plt.ylabel('Revenue')
   plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
   plt.xticks(rotation=45)

   # Show the figure
   plt.show()
    ```

6. 重新运行代码单元格并查看结果。 图表现在包含更多信息。

    严格来说，绘图包含图。 在前面的示例中，图是隐式创建的；但也可以显式创建它。

7. 修改代码以绘制图表，如下所示：

    ```Python
   from matplotlib import pyplot as plt

   # Clear the plot area
   plt.clf()

   # Create a Figure
   fig = plt.figure(figsize=(8,3))

   # Create a bar plot of revenue by year
   plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

   # Customize the chart
   plt.title('Revenue by Year')
   plt.xlabel('Year')
   plt.ylabel('Revenue')
   plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
   plt.xticks(rotation=45)

   # Show the figure
   plt.show()
    ```

8. 重新运行代码单元格并查看结果。 图确定绘图的形状和大小。

    图可以包含多个子图，每个子图都其自己的轴上。

9. 修改代码以绘制图表，如下所示：

    ```Python
   from matplotlib import pyplot as plt

   # Clear the plot area
   plt.clf()

   # Create a figure for 2 subplots (1 row, 2 columns)
   fig, ax = plt.subplots(1, 2, figsize = (10,4))

   # Create a bar plot of revenue by year on the first axis
   ax[0].bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')
   ax[0].set_title('Revenue by Year')

   # Create a pie chart of yearly order counts on the second axis
   yearly_counts = df_sales['OrderYear'].value_counts()
   ax[1].pie(yearly_counts)
   ax[1].set_title('Orders per Year')
   ax[1].legend(yearly_counts.keys().tolist())

   # Add a title to the Figure
   fig.suptitle('Sales Data')

   # Show the figure
   plt.show()
    ```

10. 重新运行代码单元格并查看结果。 图包含代码中指定的子图。

> 注意：若要详细了解如何使用 matplotlib 绘图，请参阅 [matplotlib 文档](https://matplotlib.org/)。

### 使用 seaborn 库

虽然 matplotlib 可创建多种类型的复杂图表，但它可能需要一些复杂的代码才能获得最佳结果。 因此，多年来，许多新库都建立在 matplotlib 的基础之上，移除了其复杂性，增强了其功能。 seaborn 就是这样的一种库。

1. 在笔记本中新增一个代码单元格，并在其中输入以下代码：

    ```Python
   import seaborn as sns

   # Clear the plot area
   plt.clf()

   # Create a bar chart
   ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
   plt.show()
    ```

2. 运行代码并观察它是否使用 seaborn 库显示条形图。
3. 按如下所示修改代码：

    ```Python
   import seaborn as sns

   # Clear the plot area
   plt.clf()

   # Set the visual theme for seaborn
   sns.set_theme(style="whitegrid")

   # Create a bar chart
   ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
   plt.show()
    ```

4. 运行修改后的代码，注意 seaborn 能够让你为绘图设置一致的颜色主题。

5. 按如下所示再次修改代码：

    ```Python
   import seaborn as sns

   # Clear the plot area
   plt.clf()

   # Create a line chart
   ax = sns.lineplot(x="OrderYear", y="GrossRevenue", data=df_sales)
   plt.show()
    ```

6. 运行修改后的代码，查看折线图形式的年收入。

> 注意：若要详细了解如何使用 seaborn 绘图，请参阅 [seaborn 文档](https://seaborn.pydata.org/index.html)。

## 保存笔记本并结束 Spark 会话

处理完数据后，可使用有意义的名称保存笔记本并结束 Spark 会话。

1. 在笔记本菜单栏中，使用 ⚙️“设置”图标查看笔记本设置。
2. 将笔记本名称设置为“探索销售订单”，然后关闭设置窗格 。
3. 在笔记本菜单上，选择“停止会话”以结束 Spark 会话。

## 清理资源

在本练习中，你已了解如何使用 Spark 在 Microsoft Fabric 中处理数据。

如果已完成湖屋探索，可删除为本练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工具栏上的“...”菜单中，选择“工作区设置” 。
3. 在“其他”部分中，选择“删除此工作区” 。
