title: 开发中踩过的那些坑...
date: '2020-04-04 18:43:28'
updated: '2023-05-08 18:44:19'
tags:
  - debug
categories:
  - C#
---
# 表达式树
表达式树以树形数据结构表示代码，其中每一个节点都是一种表达式，逻辑以表达式的方式存储在树状结构里，从而可以在运行时去解析这个树，然后执行，实现动态的编辑和执行代码，在不同数据库中执行 LINQ 查询以及创建动态查询。
<!--more -->
1.使用lambda表达式创建表达式树
``` C#
Expression<Func<int, int>> expr = num => num * 2;
Func<int, int> result = expr.Compile();
Console.Write(result(5));
```
打印输出：
> 10

*以上表达式树编译后，返回一个带输入输出参数的委托对象。*

2.使用LINQ查询数据时，传递错误的表达式参数导致数据库全表查询。
这里只演示LINQ查询时的核心部分代码。
- 错误使用：
``` C#
Func<UserInfo, bool> filter = u => u.Id == 1 && u.IsDelete == false;
var users = dbContext.UserInfos.Where(filter).ToList();
```
SQL
> SELECT Id, Name... FROM UserInfo

	*以上式例中传递的查询条件并非表示式树，而是一个委托对象。虽然编译时编译器并不会报错，但在实际查询时，监控SQL查询语句你会发现**查询条件并没有包含在SQL语句中**，这样就会导致每次查询时会全表查询，把所有数据加载到内存中，然后使用委托过滤掉。虽然业务实现了，但在大量并发请求时导致的内存消耗无疑是一场灾难。*
	
- 正确使用：
``` C#
Expression<Func<UserInfo, bool>> filter = u => u.Id == 1 && u.IsDelete == false;
var users = dbContext.UserInfos.Where(filter).ToList();
```
SQL
> SELECT Id, Name... FROM UserInfo WHERE Id = 1 AND IsDelete = 0

# Uri.EscapeDataString
> *.Net HttpClient Invalid URI: The Uri string is too long*

在使用HttpClient通过Post发送大量数据请求时，报错“无效的URI：Uri字符串太长”，由于转义字符串超过最大长度限制。
不同版本（.NET）限制长度：

| 版本  | 限制长度  | 有效长度  |
| :------------ | :------------ | :------------ |
|  < 4.5   |  32766  | 32765   |
|  >= 4.5 | 65520   | 65519   | 

解决方案：
``` C#
public static string EncodeString(string content){
    // maxLengthAllowed .NET < 4.5 = 32765;
    // maxLengthAllowed .NET >= 4.5 = 65519;
    int maxLengthAllowed = 65519;
    StringBuilder sb = new StringBuilder();
	int loops = content.Length / maxLengthAllowed;
	
	for(int i = 0; i <= loops; i++){
		sb.Append(Uri.EscapeDataString(i < loops 
		? content.Substring(i * maxLengthAllowed, maxLengthAllowed)
		: content.Substring(i * maxLengthAllowed)
		));
		
		return sb.ToString();
	}
}
```
*使用Post发送对象数据时，content格式为“key=value&key1=value1&....”*