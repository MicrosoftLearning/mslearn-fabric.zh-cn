---
lab:
  title: Microsoft Fabric 中的数据科学入门
  module: Get started with data science in Microsoft Fabric
---

# Microsoft Fabric 中的数据科学入门

在本实验室中，你将引入数据、浏览笔记本中的数据、使用 Data Wrangler 处理数据、训练两种类型的模型。 通过执行所有这些步骤，你将能够探索 Microsoft Fabric 中的数据科学功能。

完成此实验后，你将获得机器学习和模型跟踪的实践经验，并了解如何在 Microsoft Fabric 中使用笔记本、Data Wrangler、试验和模型   。

完成本实验室大约需要 20 分钟。

> **注意**：完成本练习需要 [Microsoft Fabric 试用版](https://learn.microsoft.com/fabric/get-started/fabric-trial)。

## 创建工作区

在 Fabric 中处理数据之前，创建一个已启用的 Fabric 试用版的工作区。

1. 在浏览器中 [https://app.fabric.microsoft.com](https://app.fabric.microsoft.com) 导航到 Microsoft Fabric 主页。
1. 选择 **Synapse 数据科学**。
1. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
1. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
1. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 创建笔记本

要运行代码，可以创建一个笔记本。 笔记本提供了一个交互式环境，可在其中编写和运行（多种语言的）代码。

1. 在“Synapse 数据科学”主页中，创建新的笔记本********。

    几秒钟后，一个包含单个单元格的新笔记本将会打开。 笔记本由一个或多个单元格组成，这些单元格可以包含代码或 markdown（格式化文本） 。

1. 选择第一个单元格（当前是代码单元格），然后在其右上角的动态工具栏中，使用 M&#8595; 按钮将单元格转换为 markdown 单元格。

    当单元格更改为 markdown 单元格时，它包含的文本将会呈现。

1. 使用 &#128393;（“编辑”）按钮将单元格切换到编辑模式，然后删除内容并输入以下文本：

    ```text
   # Data science in Microsoft Fabric
    ```

## 获取数据

现在，你已准备好运行代码来获取数据和训练模型。 将使用 Azure 开放数据集中的[糖尿病数据集](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true)。 加载数据后，将数据转换为 Pandas 数据帧：处理行和列中数据的常见结构。

1. 在笔记本中，使用最新单元格输出下方的“+ 代码****”图标将新代码单元格添加到笔记本中。

    > **提示**：若要查看“+ 代码”图标，请将鼠标移到当前单元格输出的正下方和左侧。**** 或者在菜单栏中的“编辑”选项卡上，选择“+ 添加代码单元格”。********

1. 在新代码单元格中输入以下代码：

    ```python
   # Azure storage access info for open dataset diabetes
   blob_account_name = "azureopendatastorage"
   blob_container_name = "mlsamples"
   blob_relative_path = "diabetes"
   blob_sas_token = r"" # Blank since container is Anonymous access
    
   # Set Spark config to access  blob storage
   wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
   spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
   print("Remote blob path: " + wasbs_path)
    
   # Spark read parquet, note that it won't load any data yet by now
   df = spark.read.parquet(wasbs_path)
    ```

1. 使用单元格左侧的“&#9655; 运行单元格”按钮运行单元格。 或者，可以按键盘上的 `SHIFT` + `ENTER` 来运行单元格。

    > 注意：由于这是你第一次在此会话中运行 Spark 代码，因此必须启动 Spark 池。 这意味着会话中的第一次运行可能需要一分钟左右才能完成。 后续运行速度会更快。

1. 使用单元格输出下方的“+ 代码”图标将新的代码单元格添加到笔记本，并在其中输入以下代码：

    ```python
   display(df)
    ```

1. 单元格命令完成后，查看单元格下方的输出，输出应如下所示：

    |年龄|性别|BMI|BP|S1|S2|S3|S4|S5|S6|Y|
    |---|---|---|--|--|--|--|--|--|--|--|
    |59|2|32.1|101.0|157|93.2|38.0|4.0|4.8598|87|151|
    |48|1|21.6|87.0|183|103.2|70.0|3.0|3.8918|69|75|
    |72|2|30.5|93.0|156|93.6|41.0|4.0|4.6728|85|141|
    |24|1|25.3|84.0|198|131.4|40.0|5.0|4.8903|89|206|
    |50|1|23.0|101.0|192|125.4|52.0|4.0|4.2905|80|135|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    输出显示糖尿病数据集的行和列。

1. 呈现的表顶部有两个选项卡：“表格”和“图表”。 选择“图表”。
1. 选择图表右上角的“视图选项”以更改可视化效果。
1. 将图表更改为以下设置：
    * 图表类型：`Box plot`
    * 键：留空
    * 值：`Y`
1. 选择“应用”以呈现新的可视化效果并浏览输出。

## 准备数据

引入并浏览数据后，可以转换数据。 可以在笔记本中运行代码，也可以使用 Data Wrangler 为你生成代码。

1. 数据作为 Spark 数据帧加载。 要启动 Data Wrangler，需要将数据转换为 Pandas 数据帧。 在笔记本中运行以下代码：

    ```python
   df = df.toPandas()
   df.head()
    ```

1. 在笔记本功能区中选择“数据”，然后选择“在数据整理器中转换 DataFrame”下拉列表。********
1. 选择 `df` 数据集。 当数据整理器启动时，它会在“摘要”面板中生成所显示的数据帧的描述性概述。

    目前，标签列为 `Y`，这是一个连续变量。 要训练会预测 Y 的机器学习模型，需要训练回归模型。 Y 的（预测）值可能难以解释。 相反，我们可以尝试训练分类模型，它能够预测某人患糖尿病的风险是低还是高。 为了能够训练分类模型，需要基于 `Y` 中的值创建二进制标签列。

1. 选择 Data Wrangler 中的 `Y` 列。 请注意，`220-240` 箱的频率会降低。 第 75 百分位数 `211.5` 与直方图中两个区域的转换大致一致。 让我们将此值用作低风险和高风险的阈值。
1. 导航到“操作”面板，展开“公式”，然后选择“从公式创建列”。
1. 创建新的列，设置如下：
    * 列名：`Risk`
    * 列公式：`(df['Y'] > 211.5).astype(int)`
1. 查看添加到预览的新列 `Risk`。 进行验证，具有值 `1` 的行计数应大约为所有行的 25%（因为它是 `Y` 的第 75 百分位数）。
1. 选择“应用”。
1. 选择“将代码添加到笔记本”。****
1. 使用 Data Wrangler 生成的代码运行单元格。
1. 在新单元格中运行以下代码，验证 `Risk` 列的形状是否符合预期：

    ```python
   df_clean.describe()
    ```

## 训练机器学习模型

现在你已准备好数据，可以使用它来训练机器学习模型以预测糖尿病了。 可以使用我们的数据集训练两种不同类型的模型：回归模型（预测 `Y`）或分类模型（预测 `Risk`）。 你将使用 scikit-learn 库训练模型，并使用 MLflow 跟踪模型。

### 训练回归模型

1. 运行以下代码，将数据拆分为训练数据集和测试数据集，并将特征与要预测的标签 `Y` 分开：

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df_clean[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df_clean['Y'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. 在笔记本中添加另一个新代码单元格，在其中输入以下代码并运行它：

    ```python
   import mlflow
   experiment_name = "diabetes-regression"
   mlflow.set_experiment(experiment_name)
    ```

    该代码创建名为 `diabetes-regression` 的 MLflow 试验。 在此试验中跟踪模型。

1. 在笔记本中添加另一个新代码单元格，在其中输入以下代码并运行它：

    ```python
   from sklearn.linear_model import LinearRegression
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = LinearRegression()
      model.fit(X_train, y_train)
    ```

    该代码使用线性回归训练回归模型。 使用 MLflow 自动记录参数、指标和项目。

### 训练分类模型

1. 运行以下代码，将数据拆分为训练数据集和测试数据集，并将特征与要预测的标签 `Risk` 分开：

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df_clean[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df_clean['Risk'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. 在笔记本中添加另一个新代码单元格，在其中输入以下代码并运行它：

    ```python
   import mlflow
   experiment_name = "diabetes-classification"
   mlflow.set_experiment(experiment_name)
    ```

    该代码创建名为 `diabetes-classification` 的 MLflow 试验。 在此试验中跟踪模型。

1. 在笔记本中添加另一个新代码单元格，在其中输入以下代码并运行它：

    ```python
   from sklearn.linear_model import LogisticRegression
    
   with mlflow.start_run():
       mlflow.sklearn.autolog()

       model = LogisticRegression(C=1/0.1, solver="liblinear").fit(X_train, y_train)
    ```

    代码使用逻辑回归训练分类模型。 使用 MLflow 自动记录参数、指标和项目。

## 探索试验

Microsoft Fabric 将跟踪所有试验，并支持直观地探索它们。

1. 从左侧的中心菜单栏导航到工作区。
1. 选择 `diabetes-regression` 试验将其打开。

    > 提示：如果看不到任何记录的试验运行，请刷新页面。

1. 查看运行指标，了解回归模型的准确性。
1. 回到主页并选择 `diabetes-classification` 试验以将其打开。
1. 查看运行指标以了解分类模型的准确性。 请注意，指标的类型是不同的，因为你训练了不同类型的模型。

## 保存模型

比较了在试验中训练的机器学习模型后，可以选择性能最佳的模型。 若要使用性能最佳的模型，请保存模型并使用它生成预测。

1. 在“试验”功能区中选择“另存为 ML 模型”。****
1. 在新打开的弹出窗口中，选择“创建新的 ML 模型”。****
1. 选择 `model` 文件夹。
1. 将模型命名为“`model-diabetes`”，然后选择“保存”****。
1. 在创建模型时屏幕右上角显示的通知中选择“查看 ML 模型”。**** 还可刷新窗口。 保存的模型链接在“ML 模型版本”下。****

请注意，模型、试验和试验运行是链接的，以便你查看模型的训练方式。

## 保存笔记本并结束 Spark 会话

完成模型训练和评估后，可以使用有意义的名称保存笔记本并结束 Spark 会话。

1. 在笔记本菜单栏中，使用 ⚙️“设置”图标查看笔记本设置。
2. 将笔记本的“名称”设置为“训练和比较模型”，然后关闭设置窗格 。
3. 在笔记本菜单上，选择“停止会话”以结束 Spark 会话。

## 清理资源

在本练习中，你已创建笔记本并训练了机器学习模型。 你已使用 scikit-learn 来训练模型，并使用 MLflow 来跟踪其性能。

如果已完成模型和试验，可以删除为本练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工作区页中，选择“工作区设置”****。
3. 在“常规”部分底部，选择“删除此工作区”********。
