# CRUDSQL

Current research focuses mainly on read operations and ignores other aspects of database operations such as create, update, and delete operations. The benchmark datasets as well as models that have been proposed also fail to cover these operations, limiting the development and practical applications in the field. To bridge this gap, we propose CRUDSQL, a large-scale cross-domain single-table CRUD operations Chinese Text-to-SQL dataset. The dataset contains **10,000**  question/SQL pairs involving **625** tables from different domains. 



## Citation

If you use CRUDSQL，please cite the following work:

[Beyond Read-Only: Crafting a Comprehensive Chinese Text-to-SQL Dataset for Database Manipulation and Query](https://aclanthology.org/2024.findings-naacl.214.pdf)

```
@inproceedings{chen2024beyond,
  title={Beyond Read-Only: Crafting a Comprehensive Chinese Text-to-SQL Dataset for Database Manipulation and Query},
  author={Chen, Xi and You, Jinguo and Likun, Likun and Li, Xiang},
  booktitle={Findings of the Association for Computational Linguistics: NAACL 2024},
  pages={3383--3393},
  year={2024}
}
```

[TableQA: a Large-Scale Chinese Text-to-SQL Dataset for Table-Aware SQL Generation](https://arxiv.org/abs/2006.06434)

```
@misc{sun2020tableqa,
    title={TableQA: a Large-Scale Chinese Text-to-SQL Dataset for Table-Aware SQL Generation},
    author={Ningyuan Sun and Xuefeng Yang and Yunfeng Liu},
    year={2020},
    eprint={2006.06434},
    archivePrefix={arXiv},
    primaryClass={cs.DB}
}
```



## Data content and format

### Tables

Table data is divided into `json` and `db` format. The former can be read line by line, where each line is a serialized JSON object. The latter is a SQLite3 database.

```
{
	"rows":[
		["建国校区", "土木工程学院", "工1110", "桥梁工程", "14周", "2014/5/27", "8:00-9:50", "综103", "土木工程学院"],
		......
	],
	"name":"Table_aaa4ea4c3b0611e9b3b9f40f24344a08",
	"header":["校区", "学生学院", "班级名称", "课程名称", "考试周时间", "考试日期", "考试时间", "考试地点", "开课系"],
	"id":"aaa4ea4c3b0611e9b3b9f40f24344a08",
	"types":["text", "text", "text", "text", "text", "real", "text", "text", "text"]
}
```

The fields represent the following:

+ `rows` ：a list of rows. Each row is a list of row entries.
+ `name` ：the table name.
+ `header` :a list of column names in the table.
+ `id` : the table ID.
+ `types` : a list of types corresponding to each column in the table.



### Question and SQL

```
{
	"table_id":"aaa4ea4c3b0611e9b3b9f40f24344a08",
	"question":"你好，我是土木工程学院工专121班的一名学生，我想查询土力学与基础工程这门课的考试周是在哪一周，考试又是在哪个教室？",
	"sql":{
		"agg":[0, 0],
		"cond_conn_op":1,
		"sel":[4, 7],
		"conds":[
			[1, 2, "土木工程学院"],
            [2, 2, "工专121"],
            [3, 2, "土力学与基础工程"]
		]
		"u_express":[
			[-1, 2, [-1, 0, ""]]
		],
		"type":3
	}
}

{
	"table_id": "aaa4ea4c3b0611e9b3b9f40f24344a08",
    "question": "工111班的建筑混凝土结构设计考试教室需要更换到北306",
    "sql": {
        "agg": [0], 
        "cond_conn_op": 1, 
        "sel": [-1], 
        "conds": [
            [2, 2, "工111"], 
            [3, 2, "建筑混凝土结构设计"]
        ], 
        "u_express": [
            [7, 2, [-1, 0, "北306"]]
        ], 
        "type": 2
    }
}

{
	"table_id": "aaa4ea4c3b0611e9b3b9f40f24344a08",
    "question": "工116在15周需要进行岩土工程这门课的考试，你需要把通知写到表中",
    "sql": {
        "agg": [0], 
        "cond_conn_op": 1, 
        "sel": [-1], 
        "conds": [
            [2, 2, "工116"], 
            [3, 2, "岩土工程"], 
            [4, 2, "15周"]
        ], 
        "u_express": [
            [-1, 2, [-1, 0, ""]]
        ], 
        "type": 0
    }
}

{
	"table_id": "aaa4ea4c3b0611e9b3b9f40f24344a08", 
	"question": "删除所有考试时间在8:00到9:50这个时间段的记录", 
	"sql": {
		"agg": [0], 
		"cond_conn_op": 0, 
		"sel": [-1], 
		"conds": [
			[6, 2, "8:00-9:50"]
		], 
		"u_express": [
			[-1, 2, [-1, 0, ""]]
		], 
		"type": 1
	}
}
```

The fields represent the following:

+ `table_id` : the table ID.
+ `question` : the natural language question.
+ `sql` : the SQL query corresponding to the question. This has the following subfields:
  + `agg` : the numerical index of the aggregation operator that is being used.
  + `cond_conn_op` :  the relationship between conditions.
  + `sel` : the numerical index of the column that is being selected.
  + `conds` : a list of triplets `(column_index, operator_index, condition_value)` where:
    + `column_index` : the numerical index of the condition column that is being used.
    + `operator_index` : the numerical index of the condition operator that is being used.
    + `condition_value` : the comparison value for the condition.
  + `u_express` : a list of triplets `(column_index, operator_index, update_list)` where:
    + `column_index` : the numerical index of the condition column that is being used.
    + `operator_index` : the numerical index of the condition operator that is being used（always "="）.
    + `condition_list` : a list of triplets `(column_index, update_operator_index, update_value)` where:
      + `column_index` : the numerical index of the condition column that is being used.
      + `update_operator_index` : the numerical index of the update condition operator that is being used.
      + `update_value` : the update value.
  + `type` :  the classification of Create，Read，Update，and Delete operations.



### Explanation of SQL expression dictionary

```
operator_dict = {0:">", 1:"<", 2:"=", 3:"!="}
agg_dict = {0:"", 1:"AVG", 2:"MAX", 3:"MIN", 4:"COUNT", 5:"SUM"}
cond_conn_op_dict = {0:"", 1:"and", 2:"or"}
update_operator_dict = {0:"", 1:"+", 2:"-"}
```



