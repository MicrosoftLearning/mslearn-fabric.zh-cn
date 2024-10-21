---
lab:
  title: 保护数据仓库中的数据
  module: Get started with data warehouses in Microsoft Fabric
---

# 保护数据仓库中的数据

Microsoft Fabric 权限和精细 SQL 权限协同工作，以控制仓库访问和用户权限。 在本练习中，你将使用精细权限、列级安全性、行级安全性和动态数据掩码来保护数据。

完成本实验室大约需要 45 分钟。

> **注意**：需要 [Microsoft Fabric 试用版](https://learn.microsoft.com/fabric/get-started/fabric-trial) 才能完成本练习。

## 创建工作区

在 Fabric 中处理数据之前，创建一个已启用的 Fabric 试用版的工作区。

1. 在 Microsoft Fabric 主页上，选择“Synapse 数据仓库”。[](https://app.fabric.microsoft.com)
1. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
1. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
1. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-empty-workspace.png)

## 创建数据仓库

接下来，在刚刚创建的工作区中创建数据仓库。 “Synapse 数据仓库”主页包含创建新仓库的快捷方式：

1. 在“Synapse 数据仓库”主页中，使用所选的名称创建新的仓库********。

    大约一分钟后，一个新的仓库创建完成：

    ![新仓库的屏幕截图。](./Images/new-empty-data-warehouse.png)

## 将动态数据掩码应用于表中的列

动态数据掩码规则在表级别的各个列上应用，因此所有查询都会受到掩码的影响。 没有显式权限查看机密数据的用户将在查询结果中看到掩码值，而具有查看数据权限的用户将看到未隐藏的数据。 有四种类型的掩码：默认、电子邮件、随机和自定义字符串。 在本练习中，你将应用默认掩码、电子邮件掩码和自定义字符串掩码。

1. 在仓库中，选择“T-SQL”磁贴，并将默认 SQL 代码替换为以下 T-SQL 语句，以创建**** 表并插入和查看数据。  `CREATE TABLE` 语句中应用的掩码执行以下操作：

    ```sql
    CREATE TABLE dbo.Customer
    (   
        CustomerID INT NOT NULL,   
        FirstName varchar(50) MASKED WITH (FUNCTION = 'partial(1,"XXXXXXX",0)') NULL,     
        LastName varchar(50) NOT NULL,     
        Phone varchar(20) MASKED WITH (FUNCTION = 'default()') NULL,     
        Email varchar(50) MASKED WITH (FUNCTION = 'email()') NULL   
    );
    GO
    --Users restricted from seeing masked data will see the following when they query the table
    --The FirstName column shows the first letter of the string with XXXXXXX and none of the last characters.
    --The Phone column shows xxxx
    --The Email column shows the first letter of the email address followed by XXX@XXX.com.
    
    INSERT dbo.Customer (CustomerID, FirstName, LastName, Phone, Email) VALUES
    (29485,'Catherine','Abel','555-555-5555','catherine0@adventure-works.com'),
    (29486,'Kim','Abercrombie','444-444-4444','kim2@adventure-works.com'),
    (29489,'Frances','Adams','333-333-3333','frances0@adventure-works.com');
    GO

    SELECT * FROM dbo.Customer;
    GO
    ```

2. 使用“&#9655;运行”按钮运行 SQL 脚本，在数据仓库的 dbo 架构中创建名为 Customer 的新表************。

3. 然后，在“资源管理器”窗格中，展开“架构” > “dbo” > “Tables”，并验证是否已创建 Customer 表********************。 SELECT 语句返回未屏蔽的数据，因为你作为工作区管理员进行连接，可以看到未屏蔽的数据。

4. 作为查看器工作区角色成员的测试用户进行连接，**** 并运行以下 T-SQL 语句。

    ```sql
    SELECT * FROM dbo.Customer;
    GO
    ```

    此用户尚未获得 UNMASK 权限，因此返回的 FirstName、Phone 和 Email 列的数据将被屏蔽，因为这些列是在 `CREATE TABLE` 语句中使用掩码定义的。

5. 作为工作区管理员重新连接，并运行以下 T-SQL 以取消掩码测试用户的数据。

    ```sql
    GRANT UNMASK ON dbo.Customer TO [testUser@testdomain.com];
    GO
    ```

6. 再次作为测试用户进行连接，并运行以下 T-SQL 语句。

    ```sql
    SELECT * FROM dbo.Customer;
    GO
    ```

    由于测试用户已被授予 `UNMASK` 权限，因此将返回未屏蔽的数据。

## 应用行级安全性

行级安全性 (RLS) 可用于根据执行查询的用户的标识或角色来限制对行的访问。  在此练习中，你将通过创建一项安全策略和一个已定义为内联表值函数的安全谓词来限制对行的访问。

1. 在上一个练习所创建的仓库中，选择“新建 SQL 查询”**** 下拉列表。  在标题“空白”下的下拉列表**** 下，选择“新建 SQL 查询”****。

2. 创建表并插入数据。 为了可以在后面的步骤中测试行级安全性，请将 testuser1@mydomain.com 替换为你环境中的用户名，并将 testuser2@mydomain.com 替换为你的用户名。
    ```sql
    CREATE TABLE dbo.Sales  
    (  
        OrderID INT,  
        SalesRep VARCHAR(60),  
        Product VARCHAR(10),  
        Quantity INT  
    );
    GO
     
    --Populate the table with 6 rows of data, showing 3 orders for each test user. 
    INSERT dbo.Sales (OrderID, SalesRep, Product, Quantity) VALUES
    (1, 'testuser1@mydomain.com', 'Valve', 5),   
    (2, 'testuser1@mydomain.com', 'Wheel', 2),   
    (3, 'testuser1@mydomain.com', 'Valve', 4),  
    (4, 'testuser2@mydomain.com', 'Bracket', 2),   
    (5, 'testuser2@mydomain.com', 'Wheel', 5),   
    (6, 'testuser2@mydomain.com', 'Seat', 5);  
    GO
   
    SELECT * FROM dbo.Sales;  
    GO
    ```

3. 使用“&#9655;运行”按钮运行 SQL 脚本，在数据仓库的 dbo 架构中创建名为 DimProduct 的新表************。

4. 然后，在“资源管理器”窗格中，展开“架构” > “dbo” > “表”，并验证是否已创建 Sales 表********************。
5. 创建一个新的结构、一个定义为函数的安全谓词和一项安全策略。  

    ```sql
    --Create a separate schema to hold the row-level security objects (the predicate function and the security policy)
    CREATE SCHEMA rls;
    GO
    
    --Create the security predicate defined as an inline table-valued function. A predicate evalutes to true (1) or false (0). This security predicate returns 1, meaning a row is accessible, when a row in the SalesRep column is the same as the user executing the query.

    --Create a function to evaluate which SalesRep is querying the table
    CREATE FUNCTION rls.fn_securitypredicate(@SalesRep AS VARCHAR(60)) 
        RETURNS TABLE  
    WITH SCHEMABINDING  
    AS  
        RETURN SELECT 1 AS fn_securitypredicate_result   
    WHERE @SalesRep = USER_NAME();
    GO 
    
    --Create a security policy to invoke and enforce the function each time a query is run on the Sales table. The security policy has a Filter predicate that silently filters the rows available to read operations (SELECT, UPDATE, and DELETE). 
    CREATE SECURITY POLICY SalesFilter  
    ADD FILTER PREDICATE rls.fn_securitypredicate(SalesRep)   
    ON dbo.Sales  
    WITH (STATE = ON);
    GO
6. Use the **&#9655; Run** button to run the SQL script
7. Then, in the **Explorer** pane, expand **Schemas** > **rls** > **Functions**, and verify that the function has been created.
7. Confirm that you're logged as another user by running the following T-SQL.

    ```sql
    SELECT USER_NAME();
    GO
5. Query the sales table to confirm that row-level security works as expected. You should only see data that meets the conditions in the security predicate defined for the user you're logged in as.

    ```sql
    SELECT * FROM dbo.Sales;
    GO

## Implement column-level security

Column-level security allows you to designate which users can access specific columns in a table. It is implemented by issuing a GRANT statement on a table specifying a list of columns and the user or role that can read them. To streamline access management, assign permissions to roles in lieu of individual users. In this exercise, you will create a table, grant access to a subset of columns on the table, and test that restricted columns are not viewable by a user other than yourself.

1. In the warehouse you created in the earlier exercise, select the **New SQL Query** dropdown.  Under the dropdown under the header **Blank**, select **New SQL Query**.  

2. Create a table and insert data into the table.

 ```sql
    CREATE TABLE dbo.Orders
    (   
        OrderID INT,   
        CustomerID INT,  
        CreditCard VARCHAR(20)      
        );
    GO

    INSERT dbo.Orders (OrderID, CustomerID, CreditCard) VALUES
    (1234, 5678, '111111111111111'),
    (2341, 6785, '222222222222222'),
    (3412, 7856, '333333333333333');
    GO

    SELECT * FROM dbo.Orders;
    GO
 ```

3. 拒绝查看表中列的权限。 下面的 Transact SQL 会阻止 <testuser@mydomain.com> 查看 Orders 表中的 CreditCard 列。 在下面的 `DENY` 语句中，将 testuser@mydomain.com 替换为系统中对工作区具有查看者权限的用户名。

 ```sql
DENY SELECT ON dbo.Orders (CreditCard) TO [testuser@mydomain.com];
 ```

4. 以特定用户（你拒绝了对该用户的选定权限）的身份登录 Fabric 来测试列级安全性。

5. 查询 Orders 表以确认列级安全性是否按预期发挥作用。 以下查询将仅返回 OrderID 和 CustomerID 列，不返回 CreditCard 列。  

    ```sql
    SELECT * FROM dbo.Orders;
    GO

    --You'll receive an error because access to the CreditCard column has been restricted.  Try selecting only the OrderID and CustomerID fields and the query will succeed.

    SELECT OrderID, CustomerID from dbo.Orders
    ```

## 使用 T-SQL 配置 SQL 精细权限

Fabric 仓库有一个权限模型，允许你在工作区级别和项级别控制对数据的访问。 需要更精细地控制用户对 Fabric 仓库中的安全对象执行的操作时，可以使用标准 SQL 数据控制语言 (DCL) 命令 `GRANT`、`DENY` 和 `REVOKE`。 在此练习中，你将创建对象，使用 `GRANT` 和 `DENY` 保护它们，然后运行查询以查看应用精细权限的效果。

1. 在之前的练习所创建的仓库中，选择“新建 SQL 查询”**** 下拉列表。  在标题“空白”**** 下，选择“新建 SQL 查询”****。  

2. 创建存储过程和表。

 ```
    CREATE PROCEDURE dbo.sp_PrintMessage
    AS
    PRINT 'Hello World.';
    GO
  
    CREATE TABLE dbo.Parts
    (
        PartID INT,
        PartName VARCHAR(25)
    );
    GO
    
    INSERT dbo.Parts (PartID, PartName) VALUES
    (1234, 'Wheel'),
    (5678, 'Seat');
    GO  
    
    --Execute the stored procedure and select from the table and note the results you get because you're a member of the Workspace Admin. Look for output from the stored procedure on the 'Messages' tab.
      EXEC dbo.sp_PrintMessage;
    GO
    
    SELECT * FROM dbo.Parts
    GO
  ```

3. 接下来，将对表的 `DENY SELECT` 权限授予属于“工作区查看者”角色成员的用户，并将对过程的 `GRANT EXECUTE` 授予同一用户。

 ```sql
    DENY SELECT on dbo.Parts to [testuser@mydomain.com];
    GO

    GRANT EXECUTE on dbo.sp_PrintMessage to [testuser@mydomain.com];
    GO

 ```

4. 以你在上面 DENY 和 GRANT 语句中指定的用户（代替 testuser@mydomain.com）的身份登录 Fabric。 然后通过执行存储过程和查询表来测试所应用的精细权限。  

 ```sql
    EXEC dbo.sp_PrintMessage;
    GO
    
    SELECT * FROM dbo.Parts
 ```

## 清理资源

在此练习中，你将动态数据掩码应用于表中的列，应用了行级安全性，实现了列级安全性，并使用 T-SQL 配置了 SQL 精细权限。

1. 在左侧栏中，选择工作区的图标以查看其包含的所有项。
2. 在工具栏上的“...”菜单中，选择“工作区设置” 。
3. 在“常规”部分中，选择“删除此工作区”。********
