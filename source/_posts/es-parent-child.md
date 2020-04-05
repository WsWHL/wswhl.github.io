---
title: Elasticsearch parent child索引构建
date: 2020-04-05 14:49:16
categories: 
  - 每日一记
tags:
  - Elasticsearch
  - Indexer
---
## 注意事项

- 我们在使用es查询客户端时，需要注意客户端版本是否与服务端版本兼容，因为不同版本所生成的查询语法可能会有很大的差别，以至于出现查询异常，或完全无效的情况，所以要使用对应版本和对应语法编写。<!--more -->
- 有些特殊的查询语法，对索引的创建方式严格的要求，且无法直接在创建后去进行修改实现。例如：默认字段查询时是以`Match`分词进行匹配，如果要使用`term`查询方式，则需要将对于字段设置为`keyword`类型，某则该查询语法无效，或者匹配到不完全一致的结果，部分特殊字段也需要特殊处理，比如ip、邮箱、编号等这一类。
- Parent Child Query结构，只能建立一种文档关联关系，从而避免一个文档冗余数据太多影响查询效率。


## Parent Child Query

### 创建索引
这里就简单的使用es官网的一个例子。
- 编写模型类
	``` C# 
	public abstract class MyDocument
	{
		public int Id { get; set; }
		public JoinField MyJoinField { get; set; }
	}

	public class MyParent : MyDocument
	{
		[Text]
		public string ParentProperty { get; set; }
	}

	public class MyChild : MyDocument
	{
		[Text]
		public string ChildProperty { get; set; }
	}
	
	public class MyChild1 : MyDocument
	{
		[Text]
		public string ChildProperty { get; set; }
	}
	```

- 创建索引
``` C#
var createIndexResponse = client.Indices.Create("index", c => c
    .Index<MyDocument>()
    .Map<MyDocument>(m => m
        .RoutingField(r => r.Required()) 
        .AutoMap<MyParent>() 
        .AutoMap<MyChild>() 
		.AutoMap<MyChild1>()
        .Properties(props => props
            .Join(j => j 
                .Name(p => p.MyJoinField)
                .Relations(r => r
                    .Join<MyParent>(typeof(MyChild), typeof(MyChild1))
                )
            )
			.Keyword(s => s.Name(f => f.Id))
			.Keyword(s => new KeywordPropertyDescriptor<MyChild1>().Name(f => f.ChildProperty))
        )
    )
);
```
*** 查询索引映射信息：`/index/_mapping`，所以要想修改索引映射信息只能删除重建 ***

### 查询
主要有四种查询方式：
1. Parent Id Query
``` C#
q
.ParentId(p => p
    .Type<MyDocument>()
    .Id(MyParent.Id)
)
```
> 可以根据父Id，找出所有相关的子类型数据。

2. Has Parent Query
``` C#
q
.HasParent<MyParent>(c => c
    .Query(qq => qq.MatchAll())
)
```
> 可以根据父类的相关属性匹配出对应关联的子类数据。

3. Has Child Query
``` C#
q
.HasChild<MyChild>(c => c
    .MaxChildren(5)
    .MinChildren(1)
    .ScoreMode(ChildScoreMode.Average)
    .Query(qq => qq.MatchAll())
)
```
> 可以根据子类的相关属性，匹配出子类关联的父类数据。

4. Find JoinField Type
``` C#
q
.HasRelationName<MyParent>(x => x.MyJoinField)
.MatchAll()
```
> 可以根据指定JoinField类型筛选相关数据。

	备注：查询时并不能像常规的关系型数据库进行联合查询，因此当我们需要根据父类型和子类型属性进行数据筛选时，最好将相关字段数据冗余到一个类型，这样可以方便的查询。