---
lab:
  title: 生成并保存批量预测
  module: Generate batch predictions using a deployed model in Microsoft Fabric
---

# 生成并保存批量预测

在本实验室中，你将使用机器学习模型来预测糖尿病的定量度量值。 你将使用 Fabric 中的 PREDICT 函数通过已注册的模型生成预测。

完成本实验室后，你将获得生成预测和可视化结果的动手实践经验。

完成本实验室大约需要 20 分钟。

> 注意：完成本练习需要 Microsoft Fabric 许可证。 有关如何启用免费 Fabric 试用版许可证的详细信息，请参阅 [Fabric 入门](https://learn.microsoft.com/fabric/get-started/fabric-trial)。 执行此操作需要 Microsoft 学校或工作帐户 。 如果没有，可以[注册 Microsoft Office 365 E3 或更高版本的试用版](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans)。

## 创建工作区

在 Fabric 中使用模型之前，在已启用的 Fabric 试用版中创建工作区。

1. 登录到 [Microsoft Fabric](https://app.fabric.microsoft.com) (`https://app.fabric.microsoft.com`)，然后选择 Power BI。
2. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
3. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
4. 打开新工作区时，它应为空，如下所示：

    ![Power BI 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 上传 Notebook

要引入数据、训练和注册模型，需要在笔记本中运行单元格。 可以将笔记本上传到工作区。

1. 在新的浏览器选项卡中，导航到“[Generate-Predictions](https://github.com/MicrosoftLearning/mslearn-fabric/blob/main/Allfiles/Labs/08/Generate-Predictions.ipynb)”笔记本并将其下载到任选的文件夹。
1. 返回到 Fabric 浏览器选项卡，在 Fabric 门户的左下角，选择 Power BI 图标并切换到“数据科学”体验 。
1. 在“**数据科学**”主页中，选择“**导入笔记本**”。

    成功导入笔记本后，你将收到通知。

1. 导航到名为 `Generate-Predictions` 的已导入笔记本。
1. 请仔细阅读笔记本中的说明，并单独运行每个单元格。

## 清理资源

在本练习中，你已使用模型生成批量预测。

如果已完成对笔记本的探索，则可以删除为此练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工具栏上的“...”菜单中，选择“工作区设置” 。
3. 在“其他”部分中，选择“删除此工作区” 。
