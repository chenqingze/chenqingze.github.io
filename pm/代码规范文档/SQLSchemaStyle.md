# 数据库设计规约

## 一、命名：不要过度使用简写，字段名不长尽量完整书写

### 1.    统一使用小写字母、数字或下划线

### 2.    主键：统一使用“id”命名，一对一子表可使用主表主键作为主键

### 3.    外键：统一使用tablename_id。示例：假如 user 表与package表有外键关联，则外键名为 package_id

### 4.    索引：统一使用“idx_”前缀。示例：idx_columnname

## 二、字段类型

### 1.    布尔类型：



### 2.    数字类型

 

### 3.    字符串类型

varchar是可变长字符串，不预先分配存储空间，长度不要超过5000，如果存储长度大于此值，定义字段类型为text，独立出来一张表，用主键来对应，避免影响其它字段索引效率。

### 4.    时间类型：

1)统一使用datetime utc时间
2)精确到时分秒使用“_at”作为后缀
3)精确到日期不含时分秒使用“_on”作为后缀
4)数据库统一使用utc 时间

### 5.    数据库必备字段

下面这些字段不需要手动更新，系统默认初始化、自动更新
创建时间：created_at    自动初始化
`created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP;
修改时间：modified_at    自动更新和初始化
`updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP;

## 三、DDL规范

1)表查询不要使用 "*"，明确查询字段。
2)更新是不要更新无需更新的字段
3)尽量避免使用in操作
4)直接操作数据库时对于更新和删除操作必须先查询后修改，避免误操作【强制】
5)超过三张表不要使用jion方式查询
6)尽量不要使用子查询
