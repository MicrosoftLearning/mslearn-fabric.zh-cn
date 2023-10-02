---
lab:
  title: 训练分类模型以预测客户流失
  module: Get started with data science in Microsoft Fabric
---

# 使用笔记本在 Microsoft Fabric 中训练模型

在本实验室中，我们将使用 Microsoft Fabric 创建笔记本并训练机器学习模型来预测客户流失。 我们将使用 Scikit-Learn 来训练模型，并使用 MLflow 来跟踪其性能。 客户流失是许多公司面临的一个关键业务问题，预测哪些客户可能会流失，可以帮助公司留住客户并增加收入。 完成本实验室后，你将获得机器学习和模型跟踪的实践经验，并了解如何使用 Microsoft Fabric 为项目创建笔记本。

完成本实验室大约需要 45 分钟。

> 注意：完成本练习需要 Microsoft Fabric 许可证。 有关如何启用免费 Fabric 试用版许可证的详细信息，请参阅 [Fabric 入门](https://learn.microsoft.com/fabric/get-started/fabric-trial)。 执行此操作需要 Microsoft 学校或工作帐户 。 如果没有，可以[注册 Microsoft Office 365 E3 或更高版本的试用版](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans)。

## 创建工作区

在 Fabric 中处理数据之前，在已启用的 Fabric 试用版中创建工作区。

1. 登录到 [Microsoft Fabric](https://app.fabric.microsoft.com) (`https://app.fabric.microsoft.com`)，然后选择 Power BI。
2. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
3. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
4. 打开新工作区时，它应为空，如下所示：

    ![Power BI 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 创建湖屋并上传文件

有了工作区后，可在门户中切换到数据科学体验，并为要分析的数据文件创建一个数据湖屋。

1. 在 Power BI 门户左下角，选择 Power BI 图标并切换到“数据工程”体验 。
1. 在“数据工程”主页中，使用所选名称创建一个新的湖屋 。

    大约一分钟后，将完成创建一个不包含表或文件的新湖屋 。 需要将一些数据引入数据湖屋进行分析。 可通过多种方法执行此操作，但在本练习中，只需将文本文件的文件夹下载并解压缩到本地计算机或实验室 VM（如果适用），然后将其上传到湖屋。

1. 从 [https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/churn.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/churn.csv) 下载此练习的 `churn.csv` CSV 文件并将其保存。


1. 返回到包含湖屋的 Web 浏览器标签页，在“湖屋”窗格的“文件”节点的“...”菜单中，依次选择“上传”和“上传文件”，然后将 churn.csv 文件从本地计算机（或实验室 VM，如适用）上传到湖屋     。
6. 上传文件后，展开“文件”并验证 CSV 文件是否已上传。

## 创建笔记本

若要训练模型，可以创建笔记本。 笔记本提供了一个交互式环境，你可在其中编写和运行作为试验的（多种语言的）代码。

1. 在 Power BI 门户左下角，选择“数据工程”图标并切换到“数据科学”体验 。

1. 在“数据科学”主页中，创建新的笔记本 。

    几秒钟后，一个包含单个单元格的新笔记本将会打开。 笔记本由一个或多个单元格组成，这些单元格可以包含代码或 markdown（格式化文本） 。

1. 选择第一个单元格（当前是代码单元格），然后在其右上角的动态工具栏中，使用 M&#8595; 按钮将单元格转换为 markdown 单元格。

    当单元格更改为 markdown 单元格时，它包含的文本将会呈现。

1. 使用 &#128393;（“编辑”）按钮将单元格切换到编辑模式，然后删除内容并输入以下文本：

    ```text
   # Train a machine learning model and track with MLflow

   Use the code in this notebook to train and track models.
    ``` 

## 将数据加载到数据帧中

现在，你已准备好运行代码来准备数据和训练模型。 若要处理数据，需要使用数据帧。 Spark 中的数据帧类似于 Python 中的 Pandas 数据帧，提供一种用于处理行和列中的数据的通用结构。

1. 在“添加湖屋”窗格中，选择“添加”以添加湖屋。 
1. 选择“现有湖屋”，然后选择“添加” 。
1. 选择在上一部分创建的湖屋。
1. 展开“文件”文件夹，以便在笔记本编辑器旁边列出 CSV 文件。
1. 在“churn.csv”的“...”菜单中，选择“加载数据” > “Pandas”   。 应在笔记本中添加包含以下代码的新代码单元格：

    ```python
   import pandas as pd
   # Load data into pandas DataFrame from "/lakehouse/default/" + "Files/churn.csv"
   df = pd.read_csv("/lakehouse/default/" + "Files/churn.csv")
   display(df)
    ```

    > 提示：可以隐藏左侧包含文件的窗格，方法是使用其 << 图标 。 这样做有助于专注于笔记本。

1. 使用单元格左侧的“&#9655; 运行单元格”按钮运行单元格。

    > 注意：由于这是你第一次在此会话中运行 Spark 代码，因此必须启动 Spark 池。 这意味着会话中的第一次运行可能需要一分钟左右才能完成。 后续运行速度会更快。

1. 单元格命令完成后，查看单元格下方的输出，该输出应如下所示：

    |Index|CustomerID|years_with_company|total_day_calls|total_eve_calls|total_night_calls|total_intl_calls|average_call_minutes|total_customer_service_calls|age|流失|
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    |1|1000038|0|117|88|32|607|43.90625678|0.810828179|34|0|
    |2|1000183|1|164|102|22|40|49.82223317|0.294453889|35|0|
    |3|1000326|3|116|43|45|207|29.83377967|1.344657937|57|1|
    |4|1000340|0|92|24|11|37|31.61998183|0.124931779|34|0|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    输出显示 churn.csv 文件中客户数据的行和列。

## 训练机器学习模型

现在，你已加载数据，可以使用它来训练机器学习模型并预测客户流失。 你将使用 Scikit-Learn 库训练模型，并使用 MLflow 跟踪模型。 

1. 使用单元格输出下方的“+ 代码”图标将新的代码单元格添加到笔记本，并在其中输入以下代码：

    ```python
   from sklearn.model_selection import train_test_split

   print("Splitting data...")
   X, y = df[['years_with_company','total_day_calls','total_eve_calls','total_night_calls','total_intl_calls','average_call_minutes','total_customer_service_calls','age']].values, df['churn'].values
   
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. 运行添加的代码单元格，请注意，需要在数据集中省略“CustomerID”，并将数据拆分为训练和测试数据集。
1. 在笔记本中添加另一个新代码单元格，在其中输入以下代码并运行它：
    
    ```python
   import mlflow
   experiment_name = "experiment-churn"
   mlflow.set_experiment(experiment_name)
    ```
    
    该代码创建名为 `experiment-churn` 的 MLflow 试验。 在此试验中跟踪模型。

1. 在笔记本中添加另一个新代码单元格，在其中输入以下代码并运行它：

    ```python
   from sklearn.linear_model import LogisticRegression
   
   with mlflow.start_run():
       mlflow.autolog()

       model = LogisticRegression(C=1/0.1, solver="liblinear").fit(X_train, y_train)

       mlflow.log_param("estimator", "LogisticRegression")
    ```
    
    代码使用逻辑回归训练分类模型。 使用 MLflow 自动记录参数、指标和项目。 此外，还需要记录一个名为 `estimator` 的参数，其值为 `LogisticRegression`。

1. 在笔记本中添加另一个新代码单元格，在其中输入以下代码并运行它：

    ```python
   from sklearn.tree import DecisionTreeClassifier
   
   with mlflow.start_run():
       mlflow.autolog()

       model = DecisionTreeClassifier().fit(X_train, y_train)
   
       mlflow.log_param("estimator", "DecisionTreeClassifier")
    ```

    代码使用决策树分类器训练分类模型。 使用 MLflow 自动记录参数、指标和项目。 此外，还需要记录一个名为 `estimator` 的参数，其值为 `DecisionTreeClassifier`。

## 使用 MLflow 搜索和查看试验

使用 MLflow 训练和跟踪模型后，可以使用 MLflow 库检索试验及其详细信息。

1. 若要列出所有试验，请使用以下代码：

    ```python
   import mlflow
   experiments = mlflow.search_experiments()
   for exp in experiments:
       print(exp.name)
    ```

1. 若要检索特定试验，可按名称获取试验：

    ```python
   experiment_name = "experiment-churn"
   exp = mlflow.get_experiment_by_name(experiment_name)
   print(exp)
    ```

1. 使用试验名称，可以检索该试验的所有作业：

    ```python
   mlflow.search_runs(exp.experiment_id)
    ```

1. 若要更轻松地比较作业运行和输出，可以配置搜索以对结果进行排序。 例如，以下单元格按 `start_time` 对结果进行排序，且仅显示最多 `2` 个结果： 

    ```python
   mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)
    ```

1. 最后，可以绘制多个模型的评估指标，以便轻松比较模型：

    ```python
   import matplotlib.pyplot as plt
   
   df_results = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)[["metrics.training_accuracy_score", "params.estimator"]]
   
   fig, ax = plt.subplots()
   ax.bar(df_results["params.estimator"], df_results["metrics.training_accuracy_score"])
   ax.set_xlabel("Estimator")
   ax.set_ylabel("Accuracy")
   ax.set_title("Accuracy by Estimator")
   for i, v in enumerate(df_results["metrics.training_accuracy_score"]):
       ax.text(i, v, str(round(v, 2)), ha='center', va='bottom', fontweight='bold')
   plt.show()
    ```

    输出应如下图所示：

    ![绘制的评估指标的屏幕截图。](./Images/plotted-metrics.png)

## 探索试验

Microsoft Fabric 将跟踪所有试验，并支持直观地探索它们。

1. 在左侧窗格上，导航到你的工作区。
1. 从列表中选择 `experiment-churn` 试验。

    > 提示：如果看不到任何记录的试验运行，请刷新页面。

1. 选择“视图”选项卡。
1. 选择“运行列表”。 
1. 通过选中每个框来选择两个最新运行。
    因此，最后两个运行将在“指标比较”窗格中相互比较。 默认情况下，指标按运行名称绘制。 
1. 选择图形的 &#128393;（“编辑”）按钮，以可视化每个运行的准确性。 
1. 将可视化效果类型更改为 `bar`。 
1. 将 X 轴更改为 `estimator`。 
1. 选择“替换”并浏览新图形。

通过绘制每个记录的估算器的准确性，可以查看哪种算法可生成更好的模型。

## 保存模型

比较了在试验运行中训练的机器学习模型后，可以选择性能最佳的模型。 若要使用性能最佳的模型，请保存模型并使用它生成预测。

1. 在试验概述中，确保已选择“视图”选项卡。
1. 选择“运行详细信息”。
1. 选择准确度最高的运行。 
1. 在“另存为模型”框中选择“保存” 。
1. 在新打开的弹出窗口中选择“创建新模型”。
1. 将模型命名为 `model-churn`，然后选择“创建”。 
1. 在创建模型时屏幕右上角显示的通知中选择“查看模型”。 还可刷新窗口。 已保存的模型在“已注册版本”下链接。 

请注意，模型、试验和试验运行是链接的，以便你查看模型的训练方式。 

## 保存笔记本并结束 Spark 会话

完成模型训练和评估后，可以使用有意义的名称保存笔记本并结束 Spark 会话。

1. 在笔记本菜单栏中，使用 ⚙️“设置”图标查看笔记本设置。
2. 将笔记本的“名称”设置为“训练和比较模型”，然后关闭设置窗格 。
3. 在笔记本菜单上，选择“停止会话”以结束 Spark 会话。

## 清理资源

在本练习中，你已创建笔记本并训练了机器学习模型。 你已使用 Scikit-Learn 来训练模型，并使用 MLflow 来跟踪其性能。

如果已完成模型和试验探索，可以删除为本练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工具栏上的“...”菜单中，选择“工作区设置” 。
3. 在“其他”部分中，选择“删除此工作区” 。
