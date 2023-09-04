---
lab:
  title: 使用 Spark 和 Microsoft Fabric 笔记本引入数据
  module: Ingest data with Spark and Microsoft Fabric notebooks
---

# 使用 Spark 和 Microsoft Fabric 笔记本引入数据

在此实验室中，你将创建一个 Microsoft Fabric 笔记本，并使用 PySpark 连接到 Azure Blob 存储路径，然后使用写入优化将数据加载到湖屋。

完成本实验室大约需要 30 分钟。

对于此体验，你将跨多个笔记本代码单元格生成代码，这可能无法反映出你在你的环境中执行此操作的方式，但是，它对于调试是有用的。

由于你也在使用示例数据集，因此优化并不能反映出在大规模生产中你可能看到的情况。但是，你仍然可以看到改进，而当每一毫秒都很重要时，优化就是关键。

> 注意：完成本练习需要 Microsoft Fabric 许可证 。 有关如何启用免费 Fabric 试用版许可证的详细信息，请参阅 [Fabric 入门](https://learn.microsoft.com/fabric/get-started/fabric-trial)。
>
> 执行此操作还需要 Microsoft 学校或工作帐户 。 如果没有，可以[注册 Microsoft Office 365 试用版](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans)。

## 创建工作区和湖屋目标

首先创建一个启用了 Fabric 试用版的工作区、一个新的湖屋以及湖屋中的目标文件夹。

1. 登录 [Microsoft Fabric](https://app.fabric.microsoft.com) (`https://app.fabric.microsoft.com`)，然后选择“**数据工程**”体验。

    ![Synapse 数据工程体验的屏幕截图](Images/data-engineering-home.png)

1. 在左侧菜单栏中，选择“工作区”。

1. 使用你选择的名称创建一个新工作区，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。

1. 打开新工作区时，该工作区应为空，工作区名称旁边有一个钻石图标，如下所示：

    ![新的空工作区的屏幕截图](Images/new-workspace.png)

1. 在工作区中，选择“+ 新建”>“湖屋”，提供一个名称，然后选择“创建” 。

    > **注意：** 创建一个没有**表**或**文件**的新湖屋可能需要几分钟时间。

    ![新湖屋的屏幕截图](Images/new-lakehouse.png)

1. 从“文件”中选择“[...]”，创建名为 RawData 的新子文件夹   。

1. 从湖屋内的湖屋资源管理器中，选择“文件”>“...”>“属性”。

1. 将 RawData 文件夹的 ABFS 路径复制到空记事本中以供以后使用，该路径应如下所示：`abfss://{workspace_name}@onelake.dfs.fabric.microsoft.com/{lakehouse_name}.Lakehouse/Files/{folder_name}/{file_name}` 

现在应该有一个包含湖屋和 RawData 目标文件夹的工作区。

## 创建 Fabric 笔记本并加载外部数据

创建新的 Fabric 笔记本，并使用 PySpark 连接到外部数据源。

1. 从湖屋的顶部菜单中，选择“打开笔记本”>“新建笔记本”，新建的笔记本会立即打开。

    >  **提示：** 可以从此笔记本中访问湖屋资源管理器，并且可以在完成本练习时刷新以查看进度。

1. 请注意，在默认单元格中，代码设置为“PySpark (Python)”。

1. 将以下代码插入代码单元格，该代码将：
    - 声明连接字符串的参数
    - 生成连接字符串
    - 将数据读取到 DataFrame

    ```Python
    # Azure Blob Storage access info
    blob_account_name = "azureopendatastorage"
    blob_container_name = "nyctlc"
    blob_relative_path = "yellow"
    
    # Construct connection path
    wasbs_path = f'wasbs://{blob_container_name}@{blob_account_name}.blob.core.windows.net/{blob_relative_path}'
    print(wasbs_path)
    
    # Read parquet data from Azure Blob Storage path
    blob_df = spark.read.parquet(wasbs_path)
    ```

1. 选择代码单元格旁边的“ &#9655; 运行单元格”，以连接数据并将其读入 DataFrame 中。

    预期结果：命令应会成功并输出 `wasbs://nyctlc@azureopendatastorage.blob.core.windows.net/yellow`

    > **注意：** Spark 会话从第一次代码运行时开始，因此可能需要更长时间才能完成。

1. 若要将数据写入文件，现在需要 RawData 文件夹的 ABFS 路径 。

1. 在新代码单元格中插入以下代码：

    ```python
        # Declare file name    
        file_name = "yellow_taxi"
    
        # Construct destination path
        output_parquet_path = f"**InsertABFSPathHere**/{file_name}"
        print(output_parquet_path)
        
        # Load the first 1000 rows as a Parquet file
        blob_df.limit(1000).write.mode("overwrite").parquet(output_parquet_path)
    ```

1. 添加你的 **RawData** ABFS 路径并选择“ **&#9655; 运行单元格**”以将 1000 行写入 yellow_taxi.parquet 文件。

1. output_parquet_path应如下所示：`abfss://Spark@onelake.dfs.fabric.microsoft.com/DPDemo.Lakehouse/Files/RawData/yellow_taxi`

1. 若要确认从湖屋资源管理器加载数据，请选择“文件”>“...”>“刷新”。

现在应该会看到包含 yellow_taxi.parquet“文件”的新文件夹 RawData，它显示为一个包含分区文件的文件夹 。

## 转换数据并将其加载到 Delta 表

数据引入任务很可能不会仅以加载文件结束。 湖屋中的 Delta 表支持可缩放的灵活查询和存储，因此我们也会创建一个。

1. 新建一个代码单元格并插入以下代码：

    ```python
    from pyspark.sql.functions import col, to_timestamp, current_timestamp, year, month
    
    # Read the parquet data from the specified path
    raw_df = spark.read.parquet(output_parquet_path)   
    
    # Add dataload_datetime column with current timestamp
    filtered_df = raw_df.withColumn("dataload_datetime", current_timestamp())
    
    # Filter columns to exclude any NULL values in storeAndFwdFlag
    filtered_df = filtered_df.filter(raw_df["storeAndFwdFlag"].isNotNull())
    
    # Load the filtered data into a Delta table
    table_name = "yellow_taxi"  # Replace with your desired table name
    filtered_df.write.format("delta").mode("append").saveAsTable(table_name)
    
    # Display results
    display(filtered_df.limit(1))
    ```

1. 选择代码单元格旁边的“&#9655; 运行单元格”。

    - 这会添加一个时间戳列 dataload_datetime，记录数据加载到 Delta 表的时间
    - 筛选 storeAndFwdFlag 中的 NULL 值
    - 将筛选的数据加载到 Delta 表中
    - 显示单行进行验证

1. 查看并确认显示的结果，如下图所示：

    ![显示单行的成功输出的屏幕截图](Images/notebook-transform-result.png)

现在，你已成功连接到外部数据、将其写入 parquet 文件、将数据加载到 DataFrame、转换了数据并将其加载到 Delta 表。

## 优化 Delta 表写入

你可能在组织中使用大数据，并且这就是你选择 Fabric 笔记本进行数据引入的原因，因此我们还会介绍如何优化数据的引入和读取。 首先，我们将重复转换和写入 Delta 表的步骤，其中包括写入优化。

1. 新建一个代码单元格并插入以下代码：

    ```python
    from pyspark.sql.functions import col, to_timestamp, current_timestamp, year, month
 
    # Read the parquet data from the specified path
    raw_df = spark.read.parquet(output_parquet_path)    

    # Add dataload_datetime column with current timestamp
    opt_df = raw_df.withColumn("dataload_datetime", current_timestamp())
    
    # Filter columns to exclude any NULL values in storeAndFwdFlag
    opt_df = opt_df.filter(opt_df["storeAndFwdFlag"].isNotNull())
    
    # Enable V-Order
    spark.conf.set("spark.sql.parquet.vorder.enabled", "true")
    
    # Enable automatic Delta optimized write
    spark.conf.set("spark.microsoft.delta.optimizeWrite.enabled", "true")
    
    # Load the filtered data into a Delta table
    table_name = "yellow_taxi_opt"  # New table name
    opt_df.write.format("delta").mode("append").saveAsTable(table_name)
    
    # Display results
    display(opt_df.limit(1))
    ```

1. 确认结果与优化代码之前的结果相同。

现在，记下这两个代码块的运行时间。 运行时间会有所不同，但可以看到使用优化的代码可以显著提高性能。

## 使用 SQL 查询分析 Delta 表数据

本实验室侧重于数据引入，它真正解释了“提取、转换、加载”过程，但预览数据也很有用。

1. 新建一个代码单元格并插入以下代码：

    ```python
    # Load table into df
    delta_table_name = "yellow_taxi"
    table_df = spark.read.format("delta").table(delta_table_name)
    
    # Create temp SQL table
    table_df.createOrReplaceTempView("yellow_taxi_temp")
    
    # SQL Query
    table_df = spark.sql('SELECT * FROM yellow_taxi_temp')
    
    # Display 10 results
    display(table_df.limit(10))
    ```

1. 创建另一个代码单元格，并同样插入以下代码：

    ```python
    # Load table into df
    delta_table_name = "yellow_taxi_opt"
    opttable_df = spark.read.format("delta").table(delta_table_name)
    
    # Create temp SQL table
    opttable_df.createOrReplaceTempView("yellow_taxi_opt")
    
    # SQL Query to confirm
    opttable_df = spark.sql('SELECT * FROM yellow_taxi_opt')
    
    # Display results
    display(opttable_df.limit(10))
    ```

1. 现在，为这两个查询中的第一个查询选择“**运行单元格**”按钮旁边的 &#9660; 箭头，然后从下拉列表中选择“**运行此单元格及其之下**”。

    这将运行最后两个代码单元格。 请注意查询具有非优化数据的表和具有优化数据的表之间的执行时间差异。

## 清理资源

在本练习中，你在 Fabric 中将笔记本与 PySpark 配合使用来加载数据，并将其保存到 Parquet。 然后，你使用该 Parquet 文件进一步转换数据，并优化了 Delta 表写入。 最后，你使用 SQL 查询 Delta 表。

如果已完成探索，可删除为本练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工具栏上的“...”菜单中，选择“工作区设置” 。
3. 在“其他”部分中，选择“删除此工作区” 。
