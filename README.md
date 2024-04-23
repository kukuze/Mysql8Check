## 查询出所有的字段名是否用了关键字
WITH all_columns AS (
    SELECT table_name, column_name
    FROM information_schema.columns
    WHERE table_schema = 'acdc'  -- 替换 'your_database_name' 为实际数据库名称
),keywords AS (
    SELECT word 
    FROM information_schema.keywords
)

SELECT table_name, column_name
FROM all_columns
WHERE column_name IN (SELECT word FROM keywords);

## 查询出字段名是否使用了is开头
个人认为应该使用 flag_main避开这种关键字
WITH all_columns AS (
    SELECT table_name, column_name
    FROM information_schema.columns
    WHERE table_schema = 'acdc'  -- 替换 'your_database_name' 为实际数据库名称
)

SELECT table_name, column_name
FROM all_columns
WHERE column_name like "is%"

## 查询所有复数的表名

SELECT
	* 
FROM
	information_schema.TABLES 
WHERE
	table_schema = 'acdc' and (table_name like "%s" or table_name like "%es" or table_name like "%ies");


## 查询出所有表名字段名以数字开头或用大写字母的

WITH all_columns AS (
    SELECT table_name, column_name
    FROM information_schema.columns
    WHERE table_schema = 'acdc'  -- 替换 'acdc' 为你的数据库名称
)

SELECT table_name, column_name
FROM all_columns
WHERE 
    LOWER(table_name) != table_name  
    or LOWER(column_name) != column_name  
    or table_name REGEXP '^[0-9]' 
    or column_name REGEXP '^[0-9]' 


## 查询出所有用了double 或者float的字段名

SELECT
	table_name,column_name,data_type
FROM
	information_schema.COLUMNS 
WHERE
	table_schema = 'acdc'
	and (data_type = "float" or data_type ="double")

## id类型没有特殊要求，必须使用bigint ，禁止使用int，即使现在的数据量很小。id如果是数字类型的话，必须是8个字节。


SELECT
	table_name,column_name,column_key,data_type
FROM
	information_schema.COLUMNS 
WHERE
	table_schema = 'acdc' and column_key='PRI' and data_type !='bigint'


## mysql索引优化
#1. 查询冗余索引
select * from sys.schema_redundant_indexes where table_schema="dbname";
#2. 查询未使用过的索引
select * from sys.schema_unused_indexes where object_schema="dbname";
#3. 查询索引的使用情况
select *  from sys.schema_index_statistics where table_schema='dbname'


## 设置合适的字符存储长度，不但可以节省数据表空间和索引存储，更重要的是能提升检索速度。
SELECT
    table_name,
    column_name,
    column_type
FROM
    information_schema.COLUMNS 
WHERE
    table_schema = 'acdc' 
    AND column_type LIKE '%int%' 
    AND column_type NOT LIKE '%unsigned%'

unsingned tinyint 1个字节 0-255
unsingned smallint 2个字节 0-65535
unsingned int 4个字节 0至42.9亿
unsingned bigint 8个字节 0至10的19次方。

## 关于字符集
查询所有不是 utf8mb4_general_ci排序规则的表
SELECT 
    table_schema AS database_name,
    table_name,
    character_set_name AS character_set,
    collation_name AS collation
FROM 
    information_schema.tables
    JOIN information_schema.collation_character_set_applicability 
    ON tables.table_collation = collation_character_set_applicability.collation_name
WHERE 
    table_schema = 'acdc';

查询数据库的字符集还有排序规则
SELECT
	* 
FROM
	information_schema.SCHEMATA 
WHERE
	SCHEMA_NAME = 'acdc'

查询表的列的字符集还有排序规则

SELECT
	table_schema,
	table_name,
	column_name,
	data_type,
	character_set_name,
	collation_name 
FROM
	information_schema.COLUMNS 
WHERE
	table_schema = 'acdc' 
	AND ( character_set_name != 'utf8mb4' OR collation_name != 'utf8mb4_general_ci' );


## 找出所有没有注释的字段

SELECT 
    columns.table_schema AS database_name,
    columns.table_name,
    columns.column_name,
    columns.data_type,
    columns.character_set_name AS character_set,
    columns.collation_name AS collation,
    columns.column_comment
FROM 
    information_schema.columns
JOIN 
    information_schema.tables 
ON 
    columns.table_schema = tables.table_schema 
    AND columns.table_name = tables.table_name
WHERE 
    columns.table_schema = 'acdc'
    AND tables.table_type = 'BASE TABLE' -- 排除视图
    AND (columns.column_comment IS NULL OR columns.column_comment = ''); -- 没有注释的字段

## 找出所有可能用char类型做主键的表。不要用uuid或者身份证做主键

SELECT COLUMNS
	.table_schema,
	COLUMNS.table_name,
	COLUMNS.column_name,
	COLUMNS.data_type,
	COLUMNS.column_key 
FROM
	information_schema.
	COLUMNS JOIN information_schema.table_constraints ON COLUMNS.table_schema = table_constraints.table_schema 
	AND COLUMNS.table_name = table_constraints.table_name 
	AND COLUMNS.column_key = 'PRI' 
WHERE
	COLUMNS.column_name LIKE '%id%' 
	AND ( COLUMNS.data_type = 'char' OR COLUMNS.data_type = 'varchar' ) 
	AND COLUMNS.table_schema = 'acdc'
