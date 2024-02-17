---
lab:
  title: 使用 Microsoft Fabric 中的笔记本探索数据科学数据
  module: Explore data for data science with notebooks in Microsoft Fabric
---

# 使用 Microsoft Fabric 中的笔记本探索数据科学数据

在此实验中，将使用笔记本进行数据探索。 笔记本是强大的工具，用于交互式地探索和分析数据。 在本练习中，将了解如何创建和使用笔记本来探索数据集、生成摘要统计信息以及创建可视化效果以更好地理解数据。 此实验结束时，你将对如何使用笔记本进行数据探索和分析有深入的理解。

完成本实验室大约需要 30 分钟。

> 注意：需要 Microsoft 学校或工作帐户才能完成本练习。 如果没有该帐户，可以[注册 Microsoft Office 365 E3 或更高版本的试用版](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans)。

## 创建工作区

在 Fabric 中处理数据之前，创建一个已启用的 Fabric 试用版的工作区。

1. 在浏览器中导航到 Microsoft Fabric 主页 (`https://app.fabric.microsoft.com`)，如有必要，请使用 Fabric 凭据登录。
1. 在 Fabric 主页中，选择“Synapse 数据科学”。****
1. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
1. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
1. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 创建笔记本

若要训练模型，可以创建笔记本。 笔记本提供了一个交互式环境，你可在其中编写和运行作为试验的（多种语言的）代码。

1. 在“Synapse 数据科学”主页中，创建新的笔记本********。

    几秒钟后，一个包含单个单元格的新笔记本将会打开。 笔记本由一个或多个单元格组成，这些单元格可以包含代码或 markdown（格式化文本） 。

1. 选择第一个单元格（当前是代码单元格），然后在其右上角的动态工具栏中，使用 M&#8595; 按钮将单元格转换为 markdown 单元格。

    当单元格更改为 markdown 单元格时，它包含的文本将会呈现。

1. 如有必要，使用 &#128393;（“编辑”）按钮将单元格切换到编辑模式，然后删除内容并输入以下文本：****

    ```text
   # Perform data exploration for data science

   Use the code in this notebook to perform data exploration for data science.
    ```

## 将数据加载到数据帧中

现在可以运行代码来获取数据了。 将使用 Azure 开放数据集中的[**糖尿病数据集**](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true)。 加载数据后，将数据转换为 Pandas 数据帧，这是处理行和列中数据的常见结构。

1. 在笔记本中，使用最新单元格下方的“ **+ 代码**”图标将新代码单元格添加到笔记本中。

    > **提示**：若要查看“+ 代码”图标，请将鼠标移到当前单元格输出的正下方和左侧。**** 或者在菜单栏中的“编辑”选项卡上，选择“+ 添加代码单元格”。********

1. 输入以下代码以将数据集加载到数据帧中。

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

1. 使用单元格左侧的“&#9655; 运行单元格”按钮运行单元格。 或者，可以按键盘上的 Shift + Enter 来运行单元格。********

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

    输出显示糖尿病数据集的行和列。 数据包括 10 个基线变量，年龄、性别、身体质量指数、平均血压、糖尿病患者的 6 个血清测量值，以及感兴趣的反应（基线后一年疾病进展的定量度量值），标记为 **Y**。

1. 数据作为 Spark 数据帧加载。 Scikit-learn 将期望输入数据集是 Pandas 数据帧。 运行以下代码将数据集转换为 Pandas 数据帧：

    ```python
   df = df.toPandas()
   df.head()
    ```

## 检查数据的形状

既然已经加载了数据，就可以检查数据集的结构，例如行和列的数量、数据类型和缺少的值。

1. 使用单元格输出下方的“+ 代码”图标将新的代码单元格添加到笔记本，并在其中输入以下代码：

    ```python
   # Display the number of rows and columns in the dataset
   print("Number of rows:", df.shape[0])
   print("Number of columns:", df.shape[1])

   # Display the data types of each column
   print("\nData types of columns:")
   print(df.dtypes)
    ```

    数据集包含 **442 行**和 **11 列**。 这表示数据集中有 442 个样本和 11 个特征或变量。 **** SEX 变量可能包含分类或字符串数据。

## 检查是否缺少数据

1. 使用单元格输出下方的“+ 代码”图标将新的代码单元格添加到笔记本，并在其中输入以下代码：

    ```python
   missing_values = df.isnull().sum()
   print("\nMissing values per column:")
   print(missing_values)
    ```

    代码检查是否缺少值。 请注意，数据集中没有缺少的数据。

## 为数值变量生成描述性统计信息

现在，让我们生成描述性统计数据来理解数值变量的分布。

1. 使用单元格输出下方的“ **+ 代码**”图标将新代码单元格添加到笔记本中，然后输入以下代码。

    ```python
   df.describe()
    ```

    平均年龄约为 48.5 年，标准偏差为 13.1 年。 年龄最小的是 19 岁，年龄最大的是 79 岁。 平均 BMI 约为 26.4，根据 [WHO 标准](https://www.who.int/health-topics/obesity#tab=tab_1)，属于**超重**类别。 最小 BMI 为 18，最大值为 42.2。

## 绘制数据分布

验证 BMI 特征，并绘制其分布图，以更好地理解其特征。

1. 将另一个代码单元格添加到笔记本。 然后，在此单元格中输入以下代码并执行它。

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
   import numpy as np
    
   # Calculate the mean, median of the BMI variable
   mean = df['BMI'].mean()
   median = df['BMI'].median()
   
   # Histogram of the BMI variable
   plt.figure(figsize=(8, 6))
   plt.hist(df['BMI'], bins=20, color='skyblue', edgecolor='black')
   plt.title('BMI Distribution')
   plt.xlabel('BMI')
   plt.ylabel('Frequency')
    
   # Add lines for the mean and median
   plt.axvline(mean, color='red', linestyle='dashed', linewidth=2, label='Mean')
   plt.axvline(median, color='green', linestyle='dashed', linewidth=2, label='Median')
    
   # Add a legend
   plt.legend()
   plt.show()
    ```

    从此图中可以观察到数据集中 BMI 的范围和分布。 例如，大多数 BMI 都在 23.2 和 29.2 之间，数据是右偏的。

## 执行多变量分析

让我们生成诸如散点图和方框图之类的可视化效果，以揭示数据中的模式和关系。

1. 使用单元格输出下方的“ **+ 代码**”图标将新代码单元格添加到笔记本中，然后输入以下代码。

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns

   # Scatter plot of Quantity vs. Price
   plt.figure(figsize=(8, 6))
   sns.scatterplot(x='BMI', y='Y', data=df)
   plt.title('BMI vs. Target variable')
   plt.xlabel('BMI')
   plt.ylabel('Target')
   plt.show()
    ```

    可以看到，随着 BMI 的增加，目标变量也会增加，这表明这两个变量之间存在正线性关系。

1. 将另一个代码单元格添加到笔记本。 然后，在此单元格中输入以下代码并执行它。

    ```python
   import seaborn as sns
   import matplotlib.pyplot as plt
    
   fig, ax = plt.subplots(figsize=(7, 5))
    
   # Replace numeric values with labels
   df['SEX'] = df['SEX'].replace({1: 'Male', 2: 'Female'})
    
   sns.boxplot(x='SEX', y='BP', data=df, ax=ax)
   ax.set_title('Blood pressure across Gender')
   plt.tight_layout()
   plt.show()
    ```

    这些观察结果表明，男性和女性患者的血压状况存在差异。 平均而言，女性患者的血压高于男性患者。

1. 聚合数据可以使可视化效果和分析更易于管理。 将另一个代码单元格添加到笔记本。 然后，在此单元格中输入以下代码并执行它。

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
    
   # Calculate average BP and BMI by SEX
   avg_values = df.groupby('SEX')[['BP', 'BMI']].mean()
    
   # Bar chart of the average BP and BMI by SEX
   ax = avg_values.plot(kind='bar', figsize=(15, 6), edgecolor='black')
    
   # Add title and labels
   plt.title('Avg. Blood Pressure and BMI by Gender')
   plt.xlabel('Gender')
   plt.ylabel('Average')
    
   # Display actual numbers on the bar chart
   for p in ax.patches:
       ax.annotate(format(p.get_height(), '.2f'), 
                   (p.get_x() + p.get_width() / 2., p.get_height()), 
                   ha = 'center', va = 'center', 
                   xytext = (0, 10), 
                   textcoords = 'offset points')
    
   plt.show()
    ```

    此图显示女性患者的平均血压高于男性患者。 此外，研究表明，女性的平均体重指数 (BMI) 略高于男性。

1. 将另一个代码单元格添加到笔记本。 然后，在此单元格中输入以下代码并执行它。

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
    
   plt.figure(figsize=(10, 6))
   sns.lineplot(x='AGE', y='BMI', data=df, errorbar=None)
   plt.title('BMI over Age')
   plt.xlabel('Age')
   plt.ylabel('BMI')
   plt.show()
    ```

    19 至 30 岁年龄组的平均 BMI 值最低，而 65 至 79 岁年龄组平均 BMI 值最高。 此外，观察大多数年龄组的平均 BMI 都在超重范围内。

## 相关性分析

计算不同特征之间的相关性，以理解它们的关系和依赖项。

1. 使用单元格输出下方的“ **+ 代码**”图标将新代码单元格添加到笔记本中，然后输入以下代码。

    ```python
   df.corr(numeric_only=True)
    ```

1. 热度地图是有用的工具，可以快速可视化变量对之间关系的强度和方向。 它可以突出强的正相关性或负相关性，以及识别缺乏任何相关性的配对。 若要创建热度地图，请在笔记本中添加另一个代码单元，然后输入以下代码。

    ```python
   plt.figure(figsize=(15, 7))
   sns.heatmap(df.corr(numeric_only=True), annot=True, vmin=-1, vmax=1, cmap="Blues")
    ```

    S1 和 S2 变量具有 **0.89** 的高正关联性，表明它们在同一方向上移动。 当 S1 增加时，S2 也趋于增加，反之亦然。 此外，S3 与 S4 具有 **-0.73** 的强负关联性。 这表示，随着 S3 增加，S4 往往会减少。

## 保存笔记本并结束 Spark 会话

现在已经完成了对数据的探索，可以用有意义的名称保存笔记本并结束 Spark 会话。

1. 在笔记本菜单栏中，使用 ⚙️“设置”图标查看笔记本设置。
2. 将笔记本的**名称**设置为“**为数据科学探索数据**”，然后关闭设置窗格。
3. 在笔记本菜单上，选择“停止会话”以结束 Spark 会话。

## 清理资源

在本练习中，已创建并使用笔记本进行数据探索。 还执行了代码来计算摘要统计信息，并创建可视化效果以更好地理解数据中的模式和关系。

如果已完成模型和试验探索，可以删除为本练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工具栏上的“...”菜单中，选择“工作区设置” 。
3. 在“其他”部分中，选择“删除此工作区” 。
