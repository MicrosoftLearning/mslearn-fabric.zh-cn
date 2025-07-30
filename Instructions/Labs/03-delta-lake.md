---
lab:
  title: 在 Apache Spark 中使用 Delta 表
  module: Work with Delta Lake tables in Microsoft Fabric
---

# 在 Apache Spark 中使用 Delta 表

Microsoft Fabric 湖屋中的表基于开源 Delta Lake 格式。 Delta Lake 添加了对批处理和流式数据的关系语义的支持。 在本练习中，你将创建 Delta 表并使用 SQL 查询浏览数据。

完成此练习大约需要 **45** 分钟

> [!Note] 
> 你需要访问 [Microsoft Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) 租户才能完成本练习。

## 创建工作区

在 Fabric 中处理数据之前，请在启用 Fabric 容量的租户中创建一个工作区。

1. 在浏览器中，导航到 [Microsoft Fabric 主页](https://app.fabric.microsoft.com/home?experience=fabric-developer) (`https://app.fabric.microsoft.com/home?experience=fabric-developer`)，使用 Fabric 凭据登录。
1. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
1. 新建一个工作区并为其指定名称，并在“高级”部分选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
1. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 创建湖屋并上传文件

现在已经有了工作区，可以为数据创建数据湖屋了。

1. 在左侧菜单上，选择“**创建**”。 在“*新建*”页的 *“数据工程”* 部分下，选择“**湖屋**”。 为其指定唯一的名称。

    >**备注**：如果未将“**创建**”选项固定到边栏，则需要首先选择省略号 (**...**) 选项。

    大约一分钟后，一个新的湖屋创建完成：

    ![新湖屋的屏幕截图。](./Images/new-lakehouse.png)

1. 查看新的湖屋，注意左侧的**资源管理器**窗格可用于浏览湖屋中的表和文件：

现在可以将数据引入到湖屋中。 有多种方法可以实现此操作，此处先将文本文件下载到本地计算机（或实验室虚拟机，如适用），然后上传到湖屋。 

1. 从 `https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv` 中下载[数据文件](https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv)，将其另存为 *products.csv*。
1. 返回到包含湖屋的 Web 浏览器选项卡，然后在“资源管理器”窗格中，选择 **Files** 文件夹旁边的 ... 。  创建名为 *products* 的**新子文件夹**。
1. 在 在产品文件夹的菜单上，从本地计算机（或实验室 VM，如适用）**上传***products.csv*文件。
1. 上传文件后，选择 **products** 文件夹并验证是否已上传文件，如下所示：

    ![上传到湖屋的 products.csv 的屏幕图片。](Images/upload-products.png)
  
## 探索数据帧中的数据

现在可以创建一个 Fabric 笔记本来处理数据。 笔记本提供了一个交互式环境，可在其中编写和运行代码。

1. 在左侧菜单上，选择“**创建**”。 在“*新建*”页的“*数据工程*”部分，选择“**笔记本**”。

    创建并打开名为“**笔记本 1**”的新笔记本。

    ![新建笔记本的屏幕截图。](./Images/new-notebook.png)

1. Fabric 会为你创建的每个笔记本分配一个名称，例如笔记本 1、笔记本 2 等。点击菜单上**主页**选项卡上方的名称面板，将名称更改为更具描述性的名称。
1. 选择第一个单元格（当前是代码单元格），然后在其右上角的动态工具栏中，使用 **M↓** 按钮将单元格转换为 Markdown 单元格。 然后，单元格中包含的文本将显示为带格式的文本。
1. 使用 🖉（编辑）按钮将单元格切换到编辑模式，然后修改 Markdown，如下所示：

    ```markdown
    # Delta Lake tables 
    Use this notebook to explore Delta Lake functionality 
    ```

1. 单击笔记本中单元格外的任意位置，即可停止编辑。
1. 在“资源管理器”窗格中，选择“添加数据项”，然后选择“现有数据源”。************ 连接到之前创建的湖屋。
1. 添加新代码单元格，并添加以下代码，以使用定义的架构将产品数据读取到 DataFrame 中：

    ```python
   from pyspark.sql.types import StructType, IntegerType, StringType, DoubleType

   # define the schema
   schema = StructType() \
   .add("ProductID", IntegerType(), True) \
   .add("ProductName", StringType(), True) \
   .add("Category", StringType(), True) \
   .add("ListPrice", DoubleType(), True)

   df = spark.read.format("csv").option("header","true").schema(schema).load("Files/products/products.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/products/products.csv".
   display(df)
    ```

> [!TIP]
> 使用 V 形 « 图标隐藏或显示资源管理器窗格。 这使你可以专注于笔记本或文件。

1. 使用单元格左侧的**运行单元格** (▷) 按钮运行单元格。

> [!NOTE]
> 由于这是你第一次在此笔记本中运行代码，因此必须启动 Spark 会话。 这意味着第一次运行可能需要一分钟左右才能完成。 后续运行速度会更快。

1. 单元格完成后，查看单元格下方的输出，输出应如下所示：

    ![products.csv 数据的屏幕图片。](Images/products-schema.png)
 
## 创建 Delta 表

可以使用 *saveAsTable* 方法将数据帧保存为 Delta 表。 Delta Lake 支持创建托管表和外部表 。

   * **托管的**增量表受益于更高的性能，因为 Fabric 同时管理架构元数据和数据文件。
   * **外部**表允许使用 Fabric 管理的元数据在外部存储数据。

### 创建托管表

数据文件是在 **Tables** 文件夹中创建的。

1. 在第一个代码单元格返回的结果下，使用“+ 代码”图标添加新的代码单元格。

> [!TIP]
> 若要查看“+ 代码”图标，请将鼠标移到当前单元格输出的正下方和左侧。 或者在菜单栏中的“编辑”选项卡上，选择“**+ 添加代码单元格**”。

1. 要创建托管的 Delta 表，请添加一个新单元格，输入以下代码，然后运行该单元格：

    ```python
   df.write.format("delta").saveAsTable("managed_products")
    ```

1. 在资源管理器窗格中，“**刷新**” Tables 文件夹，并展开该节点，验证是否已创建 **managed_products** 表。

> [!NOTE]
> 文件名旁边的三角形图标表示 Delta 表。

托管表的文件存储在湖屋的 **Tables** 文件夹中。 已创建名为 *managed_products* 的文件夹，用于存储表的 Parquet 文件和 delta_log 文件夹 。

### 创建外部表

还可以创建外部表，这些表可能存储在湖屋以外的某个位置，架构元数据存储在湖屋中。

1. 在资源管理器窗格的“...”菜单中， **Files** 文件夹的菜单，选择“**复制 ABFS 路径**”。 ABFS 路径是湖屋 Files 文件夹的完全限定的路径。

1. 在新代码单元中，粘贴 ABFS 路径。 添加以下代码，使用剪切和粘贴将 abfs_path 插入代码中的正确位置：

    ```python
   df.write.format("delta").saveAsTable("external_products", path="abfs_path/external_products")
    ```

1. 完整路径应如下所示：

    ```python
   abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files/external_products
    ```

1. **运行**单元格，将数据帧保存为 Files/external_products 文件夹中的外部表。

1. 在资源管理器窗格中，“**刷新**” Tables 文件夹，并展开该节点，验证已创建包含架构元数据的external_products 表。

1. 在资源管理器窗格的“...”菜单中， Files 文件夹的菜单，选择“ **刷新**”。 然后展开“Files”节点，并验证是否已为表的数据文件创建 external_products 文件夹 。

### 比较托管表与外部表

让我们使用 %%sql magic 命令来探讨托管表与外部表之间的差异。

1. 在新的代码单元格中，运行以下代码：

    ```python
   %%sql
   DESCRIBE FORMATTED managed_products;
    ```

1. 在结果中，查看表的位置属性。 单击“数据类型”列中的“位置”值以查看完整路径。 请注意，OneLake 存储位置以 /Tables/managed_products 结尾。

1. 修改 DESCRIBE 命令以显示 external_products 表的详细信息，如下所示：

    ```python
   %%sql
   DESCRIBE FORMATTED external_products;
    ```

1. 运行单元格并在结果中查看表的位置属性。 扩大数据类型列以查看完整路径，并注意 OneLake 存储位置以 /Files/external_products 结尾。

1. 在新的代码单元格中，运行以下代码：

    ```python
   %%sql
   DROP TABLE managed_products;
   DROP TABLE external_products;
    ```

1. 在资源管理器窗格中，“**刷新**” Tables 文件夹，验证该节点中是否未列出任何表。
1. 在资源管理器窗格中，“**刷新**” Files 文件夹，并验证是否*尚未*删除 external_products。 选择此文件夹可查看 Parquet 数据文件和_delta_log 文件夹。 

外部表的元数据已删除，但数据文件未删除。

## 使用 SQL 创建 Delta 表

现在，你将使用 %%sql magic 命令创建 Delta 表。 

1. 添加另一个代码单元格，并运行以下代码：

    ```python
   %%sql
   CREATE TABLE products
   USING DELTA
   LOCATION 'Files/external_products';
    ```

1. 在资源管理器窗格的“...”菜单中， **Tables** 文件夹的菜单，选择“**刷新**”。 然后展开“Tables”节点并验证是否列出了名为 *products* 的新表。 然后展开表以查看架构。
1. 添加另一个代码单元格，并运行以下代码：

    ```python
   %%sql
   SELECT * FROM products;
    ```

## 探索表版本控制

Delta 表的事务历史记录存储在 delta_log 文件夹的 JSON 文件中。 可以使用此事务日志来管理数据版本控制。

1. 将新的代码单元添加到笔记本，并运行以下代码，以实现山地自行车价格降低 10%：

    ```python
   %%sql
   UPDATE products
   SET ListPrice = ListPrice * 0.9
   WHERE Category = 'Mountain Bikes';
    ```

1. 添加另一个代码单元格，并运行以下代码：

    ```python
   %%sql
   DESCRIBE HISTORY products;
    ```

结果显示为该表记录的事务历史记录。

1. 添加另一个代码单元格，并运行以下代码：

    ```python
   delta_table_path = 'Files/external_products'
   # Get the current data
   current_data = spark.read.format("delta").load(delta_table_path)
   display(current_data)

   # Get the version 0 data
   original_data = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
   display(original_data)
    ```

返回两个结果集 - 一个包含降价后的数据，另一个显示数据的原始版本。

## 使用 SQL 查询分析 Delta 表数据

使用 SQL magic 命令，可以使用 SQL 语法而不是 Pyspark。 在这里，你将使用 `SELECT` 语句从产品表创建临时视图。

1. 添加新的代码单元，并运行以下代码来创建和显示临时视图：

    ```python
   %%sql
   -- Create a temporary view
   CREATE OR REPLACE TEMPORARY VIEW products_view
   AS
       SELECT Category, COUNT(*) AS NumProducts, MIN(ListPrice) AS MinPrice, MAX(ListPrice) AS MaxPrice, AVG(ListPrice) AS AvgPrice
       FROM products
       GROUP BY Category;

   SELECT *
   FROM products_view
   ORDER BY Category;    
    ```

1. 添加新的代码单元，并运行以下代码，按产品数返回前 10 个类别：

    ```python
   %%sql
   SELECT Category, NumProducts
   FROM products_view
   ORDER BY NumProducts DESC
   LIMIT 10;
    ```

1. 返回数据后，选择“ **+ 新建图表**“以展示其中建议的图表之一。

    ![SQL 选择语句和结果的屏幕图片。](Images/sql-select.png)

或者，可以使用 PySpark 运行 SQL 查询。

1. 添加新代码单元格并运行以下代码：

    ```python
   from pyspark.sql.functions import col, desc

   df_products = spark.sql("SELECT Category, MinPrice, MaxPrice, AvgPrice FROM products_view").orderBy(col("AvgPrice").desc())
   display(df_products.limit(6))
    ```

## 使用 Delta 表对数据进行流式处理

Delta Lake 支持流式处理数据。 Delta 表可以是接收器，也可以是使用 Spark 结构化流式处理 API 创建的数据流的数据源 。 在此示例中，你将使用 Delta 表作为模拟物联网 (IoT) 场景中部分流式处理数据的接收器。

1.  添加新代码单元格，然后添加以下代码并运行：

    ```python
    from notebookutils import mssparkutils
    from pyspark.sql.types import *
    from pyspark.sql.functions import *

    # Create a folder
    inputPath = 'Files/data/'
    mssparkutils.fs.mkdirs(inputPath)

    # Create a stream that reads data from the folder, using a JSON schema
    jsonSchema = StructType([
    StructField("device", StringType(), False),
    StructField("status", StringType(), False)
    ])
    iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

    # Write some event data to the folder
    device_data = '''{"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"error"}
    {"device":"Dev2","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}'''

    mssparkutils.fs.put(inputPath + "data.txt", device_data, True)

    print("Source stream created...")
    ```

确保消息“*已创建源数据流...*” 的消息。 刚刚运行的代码创建了一个基于文件夹的流数据源，其中保存了一些数据，表示来自虚拟 IoT 设备的读数。

1. 在新的代码单元格中，添加并运行以下代码：

    ```python
   # Write the stream to a delta table
   delta_stream_table_path = 'Tables/iotdevicedata'
   checkpointpath = 'Files/delta/checkpoint'
   deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
   print("Streaming to delta sink...")
    ```

此代码以 Delta 格式将流式处理设备数据写入名为 iotdevicedata 的文件夹。 由于该文件夹位置的路径位于 Tables 文件夹中，因此将自动为其创建一个表。

1. 在新的代码单元格中，添加并运行以下代码：

    ```python
   %%sql
   SELECT * FROM IotDeviceData;
    ```

此代码查询 IotDeviceData 表，其中包含来自流式处理源的设备数据。

1. 在新的代码单元格中，添加并运行以下代码：

    ```python
   # Add more data to the source stream
   more_data = '''{"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"error"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}'''

   mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
    ```

此代码将更多假设的设备数据写入流式处理源。

1. 重新运行包含以下代码的单元格：

    ```python
   %%sql
   SELECT * FROM IotDeviceData;
    ```

此代码再次查询 IotDeviceData 表，该表现在应包括已添加到流式处理源的其他数据。

1. 在新代码单元格中，添加代码以停止数据流并运行单元格：

    ```python
   deltastream.stop()
    ```

## 清理资源

在本练习中，你已了解如何在 Microsoft Fabric 中使用 Delta 表。

如果已完成湖屋探索，可删除为本练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
1. 在 在工具栏上的菜单中，选择“**工作区设置**”。
1. 在“常规”部分中，选择“**删除此工作区**”。
