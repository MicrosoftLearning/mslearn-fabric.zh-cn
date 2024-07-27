---
lab:
  title: 保护 Microsoft Fabric 数据仓库
  module: Secure a Microsoft Fabric data warehouse
---

# 保护数据仓库中的数据

Microsoft Fabric 权限和精细 SQL 权限协同工作，以控制仓库访问和用户权限。 在此练习中，你将使用精细权限、列级安全性、行级安全性和动态数据掩码来保护数据。

> **注意**：若要完成本实验室中的练习，你需要两个用户：一个用户应分配有工作区管理员角色，另一个用户应具有工作区查看者角色。 若要为工作区分配角色，请参阅[授予对工作区的访问权限](https://learn.microsoft.com/fabric/get-started/give-access-workspaces)。

完成本实验室大约需要 **45** 分钟。

## 创建工作区

在 Fabric 中处理数据之前，创建一个已启用的 Fabric 试用版的工作区。

1. 在 Microsoft Fabric 主页上，选择“Synapse 数据仓库”。[](https://app.fabric.microsoft.com)
1. 在左侧菜单栏中，选择“工作区”（图标类似于 &#128455;）。
1. 新建一个工作区并为其指定名称，并选择包含 Fabric 容量（试用版、高级版或 Fabric）的许可模式  。
1. 打开新工作区时，它应为空。

    ![Fabric 中空工作区的屏幕截图。](./Images/new-empty-workspace.png)

> **注意**：创建工作区时，你将自动成为工作区管理员角色的成员。 可以将环境中的第二个用户添加到工作区查看者角色，以测试这些练习中配置的功能。 若要完成此操作，可先选择工作区中的“管理访问权限”****，然后选择“添加人员或组”****。 这样第二个用户就可以查看工作区内容。

## 创建数据仓库

接下来，在创建的工作区中创建数据仓库。 “Synapse 数据仓库”主页包含创建新仓库的快捷方式：

1. 在“Synapse 数据仓库”主页中，使用所选的名称创建新的仓库********。

    大约一分钟后，一个新的仓库创建完成：

    ![新仓库的屏幕截图。](./Images/new-empty-data-warehouse.png)

## 将动态数据掩码规则应用于表中的列

动态数据掩码规则在表级别的各个列上应用，因此所有查询都会受到掩码的影响。 没有显式权限查看机密数据的用户会在查询结果中看到掩码值，而具有查看数据权限的用户会看到未隐藏的数据。 有四种类型的掩码：默认、电子邮件、随机和自定义字符串。 在本练习中，你将应用默认掩码、电子邮件掩码和自定义字符串掩码。

1. 在仓库中，选择“T-SQL”磁贴，并将默认 SQL 代码替换为以下 T-SQL 语句，以创建**** 表并插入和查看数据。  

    ```T-SQL
   CREATE TABLE dbo.Customers
   (   
       CustomerID INT NOT NULL,   
       FirstName varchar(50) MASKED WITH (FUNCTION = 'partial(1,"XXXXXXX",0)') NULL,     
       LastName varchar(50) NOT NULL,     
       Phone varchar(20) MASKED WITH (FUNCTION = 'default()') NULL,     
       Email varchar(50) MASKED WITH (FUNCTION = 'email()') NULL   
   );
   
   INSERT dbo.Customers (CustomerID, FirstName, LastName, Phone, Email) VALUES
   (29485,'Catherine','Abel','555-555-5555','catherine0@adventure-works.com'),
   (29486,'Kim','Abercrombie','444-444-4444','kim2@adventure-works.com'),
   (29489,'Frances','Adams','333-333-3333','frances0@adventure-works.com');
   
   SELECT * FROM dbo.Customers;
    ```

    当被限制查看未屏蔽数据的用户查询此表时，FirstName **** 列会显示带 XXXXXXX 的字符串的首字母，但不显示任何尾字符。 Phone **** 列会显示 xxxx。 Email **** 列会显示电子邮件地址的首字母，后跟 `XXX@XXX.com`。 此方法可确保让敏感数据保密，同时仍允许受限用户查询表。

2. 使用“&#9655;运行”按钮运行 SQL 脚本，在数据仓库的 dbo 架构中创建名为 Customers 的新表************。

3. 然后，在“资源管理器”窗格中，展开“架构” > “dbo” > “Tables”，并验证是否已创建 Customers 表********************。 `SELECT` 语句为你返回未屏蔽的数据，因为作为工作区创建者，你是可以看到未屏蔽数据的工作区管理员角色的成员。

4. 作为查看者工作区角色成员的测试用户进行连接，**** 并运行以下 T-SQL 语句。

    ```T-SQL
    SELECT * FROM dbo.Customers;
    ```
    
    此测试用户尚未获得 UNMASK 权限，因此返回的 FirstName、Phone 和 Email 列的数据会被屏蔽，因为这些列是在 `CREATE TABLE` 语句中使用掩码定义的。

5. 作为工作区管理员重新进行连接，并运行以下 T-SQL 以取消屏蔽测试用户的数据。 将 `<username>@<your_domain>.com` 替换为正在测试的属于查看者**** 工作区角色的用户的名称。 

    ```T-SQL
    GRANT UNMASK ON dbo.Customers TO [<username>@<your_domain>.com];
    ```

6. 再次作为测试用户进行连接，并运行以下 T-SQL 语句。

    ```T-SQL
    SELECT * FROM dbo.Customers;
    ```

    由于测试用户已被授予 `UNMASK` 权限，因此将返回未屏蔽的数据。

## 应用行级安全性

行级安全性 (RLS) 可用于根据执行查询的用户的标识或角色来限制对行的访问。 在此练习中，你将通过创建一项安全策略和一个已定义为内联表值函数的安全谓词来限制对行的访问。

1. 在上一个练习所创建的仓库中，选择“新建 SQL 查询”**** 下拉列表。  在标题“空白”**** 下，选择“新建 SQL 查询”****。

2. 创建表并将数据插入其中。 为了可以在后面的步骤中测试行级安全性，请将 `username1@your_domain.com` 替换为你环境中的用户名，并将 `username2@your_domain.com` 替换为你的用户名。

    ```T-SQL
   CREATE TABLE dbo.Sales  
   (  
       OrderID INT,  
       SalesRep VARCHAR(60),  
       Product VARCHAR(10),  
       Quantity INT  
   );
    
   --Populate the table with 6 rows of data, showing 3 orders for each test user. 
   INSERT dbo.Sales (OrderID, SalesRep, Product, Quantity) VALUES
   (1, '<username1>@<your_domain>.com', 'Valve', 5),   
   (2, '<username1>@<your_domain>.com', 'Wheel', 2),   
   (3, '<username1>@<your_domain>.com', 'Valve', 4),  
   (4, '<username2>@<your_domain>.com', 'Bracket', 2),   
   (5, '<username2>@<your_domain>.com', 'Wheel', 5),   
   (6, '<username2>@<your_domain>.com', 'Seat', 5);  
    
   SELECT * FROM dbo.Sales;  
    ```

3. 使用“&#9655; 运行”按钮运行 SQL 脚本，在数据仓库的 dbo 架构中创建名为 Sales 的新表************。

4. 然后，在“资源管理器”窗格中，展开“架构” > “dbo” > “表”，并验证是否已创建 Sales 表********************。
5. 创建一个新的结构、一个定义为函数的安全谓词和一项安全策略。  

    ```T-SQL
   --Create a separate schema to hold the row-level security objects (the predicate function and the security policy)
   CREATE SCHEMA rls;
   GO
   
   /*Create the security predicate defined as an inline table-valued function.
   A predicate evaluates to true (1) or false (0). This security predicate returns 1,
   meaning a row is accessible, when a row in the SalesRep column is the same as the user
   executing the query.*/   
   --Create a function to evaluate who is querying the table
   CREATE FUNCTION rls.fn_securitypredicate(@SalesRep AS VARCHAR(60)) 
       RETURNS TABLE  
   WITH SCHEMABINDING  
   AS  
       RETURN SELECT 1 AS fn_securitypredicate_result   
   WHERE @SalesRep = USER_NAME();
   GO   
   /*Create a security policy to invoke and enforce the function each time a query is run on the Sales table.
   The security policy has a filter predicate that silently filters the rows available to 
   read operations (SELECT, UPDATE, and DELETE). */
   CREATE SECURITY POLICY SalesFilter  
   ADD FILTER PREDICATE rls.fn_securitypredicate(SalesRep)   
   ON dbo.Sales  
   WITH (STATE = ON);
   GO
    ```

6. 使用“&#9655; 运行”按钮运行 SQL 脚本****
7. 然后，在“资源管理器”窗格中展开“架构” > “rls” > “函数”，验证是否已创建该函数****************。
8. 以你在 Sales 表 `INSERT` 语句中替换了 `<username1>@<your_domain>.com` 的用户的身份登录 Fabric。 通过运行以下 T-SQL 确认你已经以该用户的身份登录。

    ```T-SQL
   SELECT USER_NAME();
    ```

9. 查询 Sales **** 表以确认行级安全性是否按预期发挥作用。 你应该只会看到符合安全谓词中为用户（你以此用户的身份登录）定义的条件的数据。

    ```T-SQL
   SELECT * FROM dbo.Sales;
    ```

## 实现列级安全性

使用列级安全性，你可以指定哪些用户可以访问表中的特定列。 它是通过发出针对表的 `GRANT` 或 `DENY` 语句来实现的，该语句指定列的列表以及可以或不可以读取这些列的用户或角色。 为了简化访问管理，请将权限分配给角色而不是单个用户。 在此练习中，你将创建一个表，授予对表中部分列的访问权限，并测试除你自己之外的其他用户是否无法查看受限列。

1. 在之前的练习所创建的仓库中，选择“新建 SQL 查询”**** 下拉列表。 在标题“空白”**** 下，选择“新建 SQL 查询”****。  

2. 创建表并将数据插入表中。

    ```T-SQL
   CREATE TABLE dbo.Orders
   (   
       OrderID INT,   
       CustomerID INT,  
       CreditCard VARCHAR(20)      
   );   
   INSERT dbo.Orders (OrderID, CustomerID, CreditCard) VALUES
   (1234, 5678, '111111111111111'),
   (2341, 6785, '222222222222222'),
   (3412, 7856, '333333333333333');   
   SELECT * FROM dbo.Orders;
     ```

3. 拒绝查看表中列的权限。 T-SQL 语句会阻止 `<username>@<your_domain>.com` 查看 Orders 表中的 CreditCard 列。 在 `DENY` 语句中，将 `<username>@<your_domain>.com` 替换为系统中对工作区具有查看者**** 权限的用户名。

     ```T-SQL
   DENY SELECT ON dbo.Orders (CreditCard) TO [<username>@<your_domain>.com];
     ```

4. 以特定用户（你拒绝了对该用户的选定权限）的身份登录 Fabric 来测试列级安全性。

5. 查询 Orders 表以确认列级安全性是否按预期发挥作用。 以下查询将仅返回 OrderID 和 CustomerID 列，不返回 CreditCard 列。  

    ```T-SQL
   SELECT * FROM dbo.Orders;
    ```

    你会收到错误，因为对 CreditCard 列的访问受到了限制。  请尝试仅选择 OrderID 和 CustomerID 字段，查询会成功。

    ```T-SQL
   SELECT OrderID, CustomerID from dbo.Orders
    ```

## 使用 T-SQL 配置 SQL 精细权限

Fabric 有一个权限模型，允许你在工作区级别和项级别控制对数据的访问。 需要更精细地控制用户对 Fabric 仓库中的安全对象执行的操作时，可以使用标准 SQL 数据控制语言 (DCL) 命令 `GRANT`、`DENY` 和 `REVOKE`。 在此练习中，你将创建对象，使用 `GRANT` 和 `DENY` 保护它们，然后运行查询以查看应用精细权限的效果。

1. 在之前的练习所创建的仓库中，选择“新建 SQL 查询”**** 下拉列表。 在标题“空白”**** 下，选择“新建 SQL 查询”****。  

2. 创建存储过程和表。 然后执行该过程并查询表。

     ```T-SQL
   CREATE PROCEDURE dbo.sp_PrintMessage
   AS
   PRINT 'Hello World.';
   GO   
   CREATE TABLE dbo.Parts
   (
       PartID INT,
       PartName VARCHAR(25)
   );
   
   INSERT dbo.Parts (PartID, PartName) VALUES
   (1234, 'Wheel'),
   (5678, 'Seat');
    GO
   
   /*Execute the stored procedure and select from the table and note the results you get
   as a member of the Workspace Admin role. Look for output from the stored procedure on 
   the 'Messages' tab.*/
   EXEC dbo.sp_PrintMessage;
   GO   
   SELECT * FROM dbo.Parts
     ```

3. 接下来，将对表的 `DENY SELECT` 权限授予属于“工作区查看者”**** 角色成员的用户，并将对过程的 `GRANT EXECUTE` 权限授予同一用户。 将 `<username>@<your_domain>.com` 替换为你环境中属于“工作区查看者”**** 角色成员的用户名。

     ```T-SQL
   DENY SELECT on dbo.Parts to [<username>@<your_domain>.com];

   GRANT EXECUTE on dbo.sp_PrintMessage to [<username>@<your_domain>.com];
     ```

4. 以你在 `DENY` 和 `GRANT` 语句中指定的用户（代替 `<username>@<your_domain>.com`）的身份登录 Fabric。 然后通过执行存储过程和查询表来测试所应用的精细权限。  

     ```T-SQL
   EXEC dbo.sp_PrintMessage;
   GO
   
   SELECT * FROM dbo.Parts;
     ```

## 清理资源

在此练习中，你将动态数据掩码规则应用于表中的列，应用了行级安全性，实现了列级安全性，并使用 T-SQL 配置了 SQL 精细权限。

1. 在左侧导航栏中，选择工作区的图标以查看其包含的所有项。
2. 在顶部工具栏上的菜单中，选择“工作区设置”****。
3. 在“常规”部分中，选择“删除此工作区”。********
