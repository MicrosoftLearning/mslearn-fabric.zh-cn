---
lab:
  title: 在 Microsoft Fabric 中监视数据仓库
  module: Monitor a data warehouse in Microsoft Fabric
---

# 在 Microsoft Fabric 中监视数据仓库

在 Microsoft Fabric 中，数据仓库提供用于大规模分析的关系数据库。 Microsoft Fabric 中的数据仓库包含可用于监视活动和查询的动态管理视图。

完成本实验室大约需要 30 分钟。

> **注意**：需要 [Microsoft Fabric 试用版](https://learn.microsoft.com/fabric/get-started/fabric-trial) 才能完成本练习。

## 创建工作区

在 Fabric 中处理数据之前，创建一个已启用的 Fabric 试用版的工作区。

1. 在 Microsoft Fabric 主页上，选择“Synapse 数据仓库”。[](https://app.fabric.microsoft.com)
1. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
1. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
1. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-workspace.png)

## 创建示例数据仓库

现已有了工作空间，可以创建数据仓库了。

1. 在左下角，确保已选择“数据仓库”体验****。
1. 在主页上，选择“示例仓库”，并创建一个命名为 sample-dw 的新数据仓库************。

    大约一分钟后，系统将创建一个新仓库，并填充出租车行程分析场景的示例数据。

    ![新仓库的屏幕截图。](./Images/sample-data-warehouse.png)

## 浏览动态管理视图

Microsoft Fabric 数据仓库包含动态管理视图 (DMV)，可用于标识数据仓库实例中的当前活动。

1. 在 sample-dw 数据仓库页的“新建 SQL 查询”下拉列表中，选择“新建 SQL 查询”************。
1. 在新空白查询窗格中，输入以下 Transact-SQL 代码查询 sys.dm_exec_connections DMV****：

    ```sql
   SELECT * FROM sys.dm_exec_connections;
    ```

1. 使用“&#9655;运行”按钮运行 SQL 脚本并查看结果，其中包括与数据仓库的所有连接的详细信息****。
1. 修改 SQL 代码以查询 sys.dm_exec_sessions DMV，如下所示****：

    ```sql
   SELECT * FROM sys.dm_exec_sessions;
    ```

1. 运行修改后的查询并查看结果，其中显示了所有经过验证的会话的详细信息。
1. 修改 SQL 代码以查询 sys.dm_exec_requests DMV，如下所示****：

    ```sql
   SELECT * FROM sys.dm_exec_requests;
    ```

1. 运行修改后的查询并查看结果，其中显示了在数据仓库中执行的所有请求的详细信息。
1. 修改 SQL 代码以加入 DMV 并返回有关当前在同一数据库中运行的请求的信息，如下所示：

    ```sql
   SELECT connections.connection_id,
    sessions.session_id, sessions.login_name, sessions.login_time,
    requests.command, requests.start_time, requests.total_elapsed_time
   FROM sys.dm_exec_connections AS connections
   INNER JOIN sys.dm_exec_sessions AS sessions
       ON connections.session_id=sessions.session_id
   INNER JOIN sys.dm_exec_requests AS requests
       ON requests.session_id = sessions.session_id
   WHERE requests.status = 'running'
       AND requests.database_id = DB_ID()
   ORDER BY requests.total_elapsed_time DESC;
    ```

1. 运行修改后的查询并查看结果，其中显示了数据库中所有正在运行的查询的详细信息（包括此查询）。
1. 在“新建 SQL 查询”下拉列表中，选择“新建 SQL 查询”以添加第二个查询选项卡********。然后，在新的空查询选项卡中运行以下代码：

    ```sql
   WHILE 1 = 1
       SELECT * FROM Trip;
    ```

1. 使查询保持运行状态，然后返回到包含代码的选项卡，以查询 DMV 并重新运行它。 这一次，结果应包括另一个选项卡中正在运行的第二个查询。记下该查询的运行时间。
1. 等待几秒钟，然后重新运行代码以再次查询 DMV。 其他选项卡中查询的运行时间应增加。
1. 返回到查询仍在运行的第二个查询选项卡，然后选择“X 取消”以取消运行****。
1. 返回到包含用于查询 DMV 的代码的选项卡，重新运行查询以确认第二个查询不再运行。
1. 关闭所有查询选项卡。

> 其他信息****：有关如何使用 DMV 的详细信息，请参阅 Microsoft Fabric 文档中的 [使用 DMV 监视连接、会话和请求](https://learn.microsoft.com/fabric/data-warehouse/monitor-using-dmv)。

## 浏览查询见解

Microsoft Fabric 数据仓库提供查询见解，这是一组特殊的视图，提供有关正在数据仓库中运行的查询的详细信息**。

1. 在 sample-dw 数据仓库页的“新建 SQL 查询”下拉列表中，选择“新建 SQL 查询”************。
1. 在新空白查询窗格中，输入以下 Transact-SQL 代码查询 exec_requests_history 视图****：

    ```sql
   SELECT * FROM queryinsights.exec_requests_history;
    ```

1. 使用“&#9655;运行”按钮运行 SQL 脚本并查看结果，其中包括以前执行的查询的详细信息****。
1. 修改 SQL 代码以查询 frequently_run_queries 视图，如下所示****：

    ```sql
   SELECT * FROM queryinsights.frequently_run_queries;
    ```

1. 运行修改后的查询并查看结果，其中显示了频繁运行的查询的详细信息。
1. 修改 SQL 代码以查询 long_running_queries 视图，如下所示****：

    ```sql
   SELECT * FROM queryinsights.long_running_queries;
    ```

1. 运行修改后的查询并查看结果，其中显示了所有查询及其持续时间的详细信息。

> 其他信息****：有关使用查询见解的详细信息，请参阅 Microsoft Fabric 文档中的 [Fabric 数据仓库中的查询见解](https://learn.microsoft.com/fabric/data-warehouse/query-insights)。


## 清理资源

在本练习中，你使用了动态管理视图和查询见解来监视 Microsoft Fabric 数据仓库中的活动。

如果已完成数据仓库探索，可以删除为本练习创建的工作区。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工作区页中，选择“工作区设置”****。
3. 在“常规”部分底部，选择“删除此工作区”********。
