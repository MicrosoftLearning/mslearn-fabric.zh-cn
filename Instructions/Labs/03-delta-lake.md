---
lab:
  title: 在 Apache Spark 中使用 Delta 表
  module: Work with Delta Lake tables in Microsoft Fabric
---

# 在 Apache Spark 中使用 Delta 表

Microsoft Fabric 湖屋中的表基于 Apache Spark 的开源 Delta Lake 格式。 Delta Lake 为批处理和流式处理数据操作添加了对关系语义的支持，并支持创建湖屋体系结构。在该体系结构中，Apache Spark 可用于处理和查询基于数据湖中基础文件的表中的数据。

完成此练习大约需要 40 分钟

> **注意**：需要 [Microsoft Fabric 试用版](https://learn.microsoft.com/fabric/get-started/fabric-trial) 才能完成本练习。

## 创建工作区

在 Fabric 中处理数据之前，创建一个已启用的 Fabric 试用版的工作区。

1. 在 `https://app.fabric.microsoft.com` 的 [Microsoft Fabric 主页](https://app.fabric.microsoft.com)中，选择“Synapse 数据工程”****。
2. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
3. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
4. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 创建湖屋并上传数据

现在已经有了工作区，可以为要分析的数据创建数据湖屋了。

1. 在“Synapse 数据工程”主页中，使用所选名称创建一个新的湖屋 。

    大约一分钟后，一个新的空湖屋创建完成。 需要将一些数据引入数据湖屋进行分析。 有多种方法可以执行此操作，但在本练习中，只需将文本文件下载到本地计算机（或者实验室 VM，如果适用），然后将其上传到湖屋。

1. 从 `https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv` 下载本练习的[数据文件](https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv)，并在本地计算机上（或者实验室 VM 上，如果适用）将其另存为 **products.csv**。

1. 返回到包含湖屋的 Web 浏览器选项卡，在“资源管理器”窗格中“Files”文件夹的“...”菜单中，选择“新建子文件夹”并创建名为“products”的文件夹    。

1. 在 products 文件夹的“...”菜单中，选择“上传”和“上传文件”，然后将本地计算机（或者实验室 VM，如果适用）中的 products.csv 文件上传到湖屋    。
1. 上传文件后，选择 products 文件夹；并验证是否已上传 products.csv 文件，如下所示 ：

    ![湖屋中已上传的 products.csv 文件的屏幕截图。](./Images/products-file.png)

## 探索数据帧中的数据

1. 在主页上查看湖屋中 products 文件夹的内容时，在“打开笔记本”菜单中选择“新建笔记本”   。

    几秒钟后，一个包含单个单元格的新笔记本将会打开。 笔记本由一个或多个单元格组成，这些单元格可以包含代码或 markdown（格式化文本） 。

2. 在笔记本中选择包含一些简单代码的现有单元格，然后使用其 右上方的 &#128465;（删除）图标将其删除，这不需要此代码。
3. 在左侧的“湖屋资源管理器”窗格中，展开“Files”并选择“products”，以显示一个新窗格，其中显示了之前上传的 products.csv 文件   ：

    ![包含“Files”窗格的笔记本的屏幕截图。](./Images/notebook-products.png)

4. 在 products.csv 的“...”菜单中，选择“加载数据” > “Spark”   。 应将包含以下代码的新代码单元格添加到笔记本：

    ```python
   df = spark.read.format("csv").option("header","true").load("Files/products/products.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/products/products.csv".
   display(df)
    ```

    > 提示：可以隐藏左侧包含文件的窗格，方法是使用其 << 图标 。 这样做有助于专注于笔记本。

5. 使用单元格左侧的 &#9655;（运行单元格）按钮运行单元格。

    > 注意：由于这是你第一次在此笔记本中运行 Spark 代码，因此必须启动 Spark 会话。 这意味着第一次运行可能需要一分钟左右才能完成。 后续运行速度会更快。

6. 单元格命令完成后，查看单元格下方的输出，输出应如下所示：

    | 索引 | ProductID | ProductName | 类别 | ListPrice |
    | -- | -- | -- | -- | -- |
    | 1 | 771 | Mountain-100 Silver, 38 | 山地自行车 | 3399.9900 |
    | 2 | 772 | Mountain-100 Silver, 42 | 山地自行车 | 3399.9900 |
    | 3 | 773 | Mountain-100 Silver, 44 | 山地自行车 | 3399.9900 |
    | ... | ... | ... | ... | ... |

## 创建 Delta 表

可以使用 `saveAsTable` 方法将数据帧保存为 delta 表。 Delta Lake 支持创建托管表和外部表 。

### 创建托管表

托管表是其架构元数据和数据文件均由 Fabric 管理的表。 表的数据文件是在 Tables 文件夹中创建的。

1. 如果尚不存在代码单元格，请在第一个代码单元格返回的结果下，使用“+ 代码”图标添加新的代码单元格****。

    > **提示**：若要查看“+ 代码”图标，请将鼠标移到当前单元格输出的正下方和左侧。**** 或者在菜单栏中的“编辑”选项卡上，选择“+ 添加代码单元格”。********

2. 在新单元格中输入并运行以下代码：

    ```python
   df.write.format("delta").saveAsTable("managed_products")
    ```

3. 在“湖屋资源管理器”窗格中，在“Tables”文件夹的“...”菜单中，选择“刷新”   。 然后展开“Tables”节点并验证是否已创建managed_products 表 。

### 创建外部表

还可以创建外部表，其架构元数据在湖屋的元存储中定义，但数据文件存储在外部位置。

1. 添加另一个新代码单元格并向其中添加以下代码：

    ```python
   df.write.format("delta").saveAsTable("external_products", path="abfs_path/external_products")
    ```

2. 在“湖屋资源管理器”窗格中，在“Files”文件夹的“...”菜单中，选择“复制 ABFS 路径”   。

    ABFS 路径是湖屋的 OneLake 存储中 Files 文件夹的完全限定的路径，如下所示：

    *abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files*

3. 在输入到代码单元格的代码中，将 abfs_path 替换为复制到剪贴板的路径，以便代码将数据帧保存为外部表，其中的数据文件位于 Files 文件夹位置中名为 external_products 的文件夹中************。 完整路径应如下所示：

    *abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files/external_products*

4. 在“湖屋资源管理器”窗格中，在“Tables”文件夹的“...”菜单中，选择“刷新”   。 然后展开“Tables”节点并验证是否已创建 external_products 表 。

5. 在“湖屋资源管理器”窗格中，在 Files 文件夹的“...”菜单中，选择“刷新”   。 然后展开“Files”节点，并验证是否已为表的数据文件创建 external_products 文件夹 。

### 比较托管表与外部表

我们来探讨托管表与外部表之间的差异。

1. 添加另一个代码单元格，并运行以下代码：

    ```sql
   %%sql

   DESCRIBE FORMATTED managed_products;
    ```

    在结果中，查看表的 Location 属性，该属性应为湖屋的 OneLake 存储的路径，以 /Tables/managed_products 结尾（可能需要加宽 Data type 列才可查看完整路径）  。

2. 修改 `DESCRIBE` 命令以显示 external_products 表的详细信息，如下所示：

    ```sql
   %%sql

   DESCRIBE FORMATTED external_products;
    ```

    在结果中，查看表的 Location 属性，该属性应为湖屋的 OneLake 存储的路径，以 /Files/external_products 结尾（可能需要加宽 Data type 列才可查看完整路径）  。

    托管表的文件存储在湖屋的 OneLake 存储的 Tables 文件夹中。 在本例中，已创建名为 managed_products 的文件夹，用于存储所创建的表的 Parquet 文件和 delta_log 文件夹 。

3. 添加另一个代码单元格，并运行以下代码：

    ```sql
   %%sql

   DROP TABLE managed_products;
   DROP TABLE external_products;
    ```

4. 在“湖屋资源管理器”窗格中，在“Tables”文件夹的“...”菜单中，选择“刷新”   。 然后展开“Tables”节点并验证是否未列出任何表。

5. 在“湖屋资源管理器”窗格中，展开“Files”文件夹，并验证是否尚未删除 external_products  。 选择此文件夹可查看 Parquet 数据文件和 _delta_log 文件夹，该文件夹中包含先前位于 external_products 表中的数据 。 外部表的表元数据已删除，但文件不受影响。

### 使用 SQL 创建表

1. 添加另一个代码单元格，并运行以下代码：

    ```sql
   %%sql

   CREATE TABLE products
   USING DELTA
   LOCATION 'Files/external_products';
    ```

2. 在“湖屋资源管理器”窗格中，在“Tables”文件夹的“...”菜单中，选择“刷新”   。 然后展开“Tables”节点并验证是否列出了名为 products 的新表 。 然后展开表以验证其架构是否与保存在 external_products 文件夹中的原始数据帧匹配。

3. 添加另一个代码单元格，并运行以下代码：

    ```sql
   %%sql

   SELECT * FROM products;
   ```

## 探索表版本控制

Delta 表的事务历史记录存储在 delta_log 文件夹的 JSON 文件中。 可以使用此事务日志来管理数据版本控制。

1. 在笔记本中新增一个代码单元格，并运行以下代码：

    ```sql
   %%sql

   UPDATE products
   SET ListPrice = ListPrice * 0.9
   WHERE Category = 'Mountain Bikes';
    ```

    此代码实现了山地自行车价格降低 10%。

2. 添加另一个代码单元格，并运行以下代码：

    ```sql
   %%sql

   DESCRIBE HISTORY products;
    ```

    结果显示为该表记录的事务历史记录。

3. 添加另一个代码单元格，并运行以下代码：

    ```python
   delta_table_path = 'Files/external_products'

   # Get the current data
   current_data = spark.read.format("delta").load(delta_table_path)
   display(current_data)

   # Get the version 0 data
   original_data = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
   display(original_data)
    ```

    结果显示两个数据帧 - 一个包含降价后的数据，另一个显示数据的原始版本。

## 使用 Delta 表对数据进行流式处理

Delta Lake 支持流式处理数据。 Delta 表可以是接收器，也可以是使用 Spark 结构化流式处理 API 创建的数据流的数据源 。 在此示例中，你将使用 Delta 表作为模拟物联网 (IoT) 方案中部分流式处理数据的接收器。

1. 在笔记本中添加一个新的代码单元格。 然后，在新单元格中添加以下代码并运行它：

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

    确保打印消息“已创建源流...”。 刚刚运行的代码创建了一个基于文件夹的流数据源，其中保存了一些数据，表示来自虚拟 IoT 设备的读数。

2. 在新的代码单元格中，添加并运行以下代码：

    ```python
   # Write the stream to a delta table
   delta_stream_table_path = 'Tables/iotdevicedata'
   checkpointpath = 'Files/delta/checkpoint'
   deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
   print("Streaming to delta sink...")
    ```

    此代码以 delta 格式将流式处理设备数据写入名为 iotdevicedata 的文件夹。 由于该文件夹位置的路径位于 Tables 文件夹中，因此将自动为其创建一个表。

3. 在新的代码单元格中，添加并运行以下代码：

    ```sql
   %%sql

   SELECT * FROM IotDeviceData;
    ```

    此代码查询 IotDeviceData 表，其中包含来自流式处理源的设备数据。

4. 在新的代码单元格中，添加并运行以下代码：

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

5. 重新运行包含以下代码的单元格：

    ```sql
   %%sql

   SELECT * FROM IotDeviceData;
    ```

    此代码再次查询 IotDeviceData 表，该表现在应包括已添加到流式处理源的其他数据。

6. 在新的代码单元格中，添加并运行以下代码：

    ```python
   deltastream.stop()
    ```

    此代码停止流。

## 清理资源

在本练习中，你学习了如何在 Microsoft Fabric 中使用 delta 表。

如果已完成湖屋探索，可删除为本练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工作区页中，选择“工作区设置”****。
3. 在“常规”部分底部，选择“删除此工作区”********。
