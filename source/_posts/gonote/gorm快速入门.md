---
title: 【Go】gorm学习
toc: true
date: 2025-03-03 00:00:00
tags: 
  - go
categories: 
  - Golang

---

点击阅读更多查看文章内容<!--more-->

## 什么是ORM

ORM（Object-Relational Mapping，面向对象关系映射）是一种程序设计技术，它将面向对象编程中的对象和关系型数据库中的表格结构进行映射。通过ORM，开发者可以使用面向对象的方式来操作数据库，而不需要编写复杂的SQL语句。

具体来说，ORM提供了一个抽象层，允许开发者通过类和对象来表示数据库表和记录，通常这种映射是自动完成的，ORM框架会负责将对象的属性与数据库中的字段对应起来，并生成相应的SQL语句执行数据库操作。

[go-gorm/gorm: The fantastic ORM library for Golang, aims to be developer friendly](https://github.com/go-gorm/gorm)

[GORM 指南 | GORM - The fantastic ORM library for Golang, aims to be developer friendly.](https://gorm.io/zh_CN/docs/)

## gorm连接数据库

```go
package main

import (
	"database/sql"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

func main() {
	// 参考 https://github.com/go-sql-driver/mysql#dsn-data-source-name 获取详情
	dsn := "root:root@tcp(127.0.0.1:3306)/gorm_test?charset=utf8mb4&parseTime=True&loc=Local"
	_, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
	if err != nil {
		panic(err)
	}
}
```

---

## 生成表结构

db.AutoMigrate(&Product{})，根据Product结构体建表

```go
package main

import (
	"database/sql"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
	"log"
	"os"
	"time"
)

type Product struct {
	gorm.Model
	Code  sql.NullString
	Price uint
}

func main() {
	// 参考 https://github.com/go-sql-driver/mysql#dsn-data-source-name 获取详情
	dsn := "root:shn001221@tcp(127.0.0.1:3306)/gorm_test?charset=utf8mb4&parseTime=True&loc=Local"
	//设置全局的logger，这个logger在我们执行每个sql语句的时候会打印每一行sql
	//sql才是最重要的，本着这个原则我尽量的给大家看到每个api背后的sql语句是什么
	newLogger := logger.New(
		log.New(os.Stdout, "\r\n", log.LstdFlags), // io writer
		logger.Config{
			SlowThreshold: time.Second, // 慢 SQL 阈值
			LogLevel:      logger.Info, // Log level
			Colorful:      true,        // 禁用彩色打印
		},
	)

	// 全局模式
	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
		Logger: newLogger,
	})
	if err != nil {
		panic(err)
	}

	//定义一个表结构， 将表结构直接生成对应的表 - migrations
	// 迁移 schema
	_ = db.AutoMigrate(&Product{}) //此处应该有sql语句
}
```

输出内容（实际执行语句）

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130003943.png" alt="image-20250206230527958" style="zoom:50%;" />

执行结果：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130003944.png" alt="image-20250206230658221" style="zoom: 67%;" />

---

## 模型

gorm.Model

在定义持久化模型 PO(persist object) 时，推荐组合使用 gorm.Model 中预定义的几个通用字段，包括主键、增删改时间等：

```go
type PO struct {
    gorm.Model
}
package gorm
type Model struct {
    // 主键 id
    ID        uint `gorm:"primarykey"`
    // 创建时间
    CreatedAt time.Time
    // 更新时间
    UpdatedAt time.Time
    // 删除时间
    DeletedAt DeletedAt `gorm:"index"`
}
```

值得一提的是，在 gorm 体系中，一个 po 模型只要启用了 deletedAt 字段，则默认会开启**软删除机制：在执行删除操作时，不会立刻物理删除数据，而是仅仅将 po 的 deletedAt 字段置为非空.**

---

## 标签

```go
type PO struct{
   // 组合使用 gorm Model，引用 id、createdAt、updatedAt、deletedAt 等字段
   gorm.Model
  // 列名为 name；列类型字符串；使用该列作为唯一索引
   Name string `gorm:"column:name;type:varchar(15);unique_index"` 
   // 该列默认值为 18
   Age int `gorm:"default:18"` 
   // 该列值不为空
   Email string `gorm:"not null"` 
   // 该列的数值逐行递增
   Num int `gorm:"auto_increment"` 
}
```

几类常用的标签及对应的用途展示如下表：

| 标签           | 作用   |
| -------------- | ------ |
| primarykey     | 主键   |
| unique_index   | 唯一键 |
| index          | 键     |
| auto_increment | 自增列 |
| column         | 列名   |
| type           | 列类型 |
| default        | 默认值 |
| not null       | 非空   |

---

## 零值

在 **golang 中一些基础类型都存在对应的零值，即便用户未显式给字段赋值，字段在初始化时也会首先赋上零值**. 比如 bool 类型的零值为 false；string 类型为 ""，int 类型为 0.

这样就会导致，在我们执行创建、更新等操作时，倘若 po 模型中存在零值字段，此时 **gorm 无法区分到底是用户显式声明的零值，还是未显式声明而被编译器默认赋予的零值. 在无法区分的情况下，gorm 会统一按照后者，采取忽略处理的方式**.

> 为什么只能更新非零值字段？如果我们去更新一个product 只设置了price：200，那么其他字段默认为零值，也就是说在更新数据库时会将其他字段都更新为零值，因此选择将零值忽略掉。

倘若此时我们想要明确是显式将字段设置为零值的，对应可以采取以下两种处理方式：

- **使用指针类型：**

我们将 age 字段类型设定为 *int，只要指针非空，就代表使用方进行了显式赋值.

```go
type PO struct{
   gorm.Model
   Age *int `gorm:"column:age"` // 默认值为 18
}
```

- **使用 [sql.Nullxx](https://zhida.zhihu.com/search?content_id=235243154&content_type=Article&match_order=1&q=sql.Nullxx&zhida_source=entity) 类型：**

我们将 age 字段类型设定为 sql.NullInt64，只要 Valid 标识为 true，就代表使用方进行了显式赋值.

```go
type PO struct{
   gorm.Model
   Age sql.NullInt64 `gorm:"column:age"` // 默认值为 18
}


type NullInt64 struct {
    Int64 int64
    Valid bool // Valid is true if Int64 is not NULL
}
```

---

## 增删改查

```go
// 创建单个记录
user := User{Name: "John", Age: 30}
result := db.Create(&user)
```

```go
// 查询多个记录
db.Where("age > ?", 25).Find(&users) // 带条件查询
```

```go
// 更新多个字段
db.Model(&user).Updates(User{Name: "Jane", Age: 31})
```

```go
// 删除单个记录
db.Delete(&user)
```

在 GORM 中，表名通常是通过模型（结构体）自动推断的，而不是直接在语句中指定的。GORM 会根据模型的结构体名称和约定规则自动推断表名。

- 增：db.Create(&Product{Code: sql.NullString{"D42", true}, Price: 100}) 
  - INSERT INTO `products` (`created_at`,`updated_at`,`deleted_at`,`code`,`price`) VALUES ('2025-02-06 23:22:57.421','2025-02-06 23:22:57.421',NULL,'D42',100)
- 删：db.Delete(&product) （删除product对应的id的记录）
  - UPDATE `products` SET `deleted_at`='2025-02-06 23:26:32.629' WHERE `products`.`id` = 3 AND `products`.`deleted_at` IS NULL
- 改：db.Model(&product).Update("Price", 200) （修改product对应的id的记录）
  -  UPDATE `products` SET `price`=200,`updated_at`='2025-02-06 23:24:49.857' WHERE `id` = 3
- 查：db.First(&product, "code = ?", "D42")  （查找 code 字段值为 D42 的记录 赋值给 product）
  - SELECT * FROM `products` WHERE code = 'D42' AND `products`.`deleted_at` IS NULL ORDER BY `products`.`id` LIMIT 1

---

## 表结构定义细节

通过字段标签声明表结构定义的细节（[字段标签](https://gorm.io/zh_CN/docs/models.html#字段标签)）

定义user表

- 将userid设为主键
- 将name列名设置为user_name，类型设置为varchar(50)，设置索引idx_user_name，设置唯一索引，设置默认值“bobby”

```go
type User struct {
	UserID uint   `gorm:"primarykey"`
	Name   string `gorm:"column:user_name;type:varchar(50);index:idx_user_name;unique;default:'bobby'"`
}
```

---

## 插入记录

```go
user := User{Name: "Jinzhu", Age: 18, Birthday: time.Now()}

result := db.Create(&user) // 通过数据的指针来创建

user.ID             // 返回插入数据的主键
result.Error        // 返回 error
result.RowsAffected // 返回插入记录的条数
```

**批量插入**

要高效地插入大量记录，请将切片传递给`Create`方法。 GORM 将生成**一条** SQL 来插入所有数据

```go
var users = []User{{Name: "jinzhu1"}, {Name: "jinzhu2"}, {Name: "jinzhu3"}}
db.Create(&users)

for _, user := range users {
  user.ID // 1,2,3
}
```

你可以通过`db.CreateInBatches`方法来指定批量插入的批次大小，以下代币会执行两次insert，第一次插入两条，第二次插入一条

sql语句有长度限制，一次性插入的数据是有上限的，所以需要限制每次插入的数量

```go
var users = []User{{Name: "jinzhu1"}, {Name: "jinzhu2"}, {Name: "jinzhu3"}}

// batch size 100
db.CreateInBatches(users, 2)
```

**通过map插入**

GORM支持通过 `map[string]interface{}` 与 `[]map[string]interface{}{}`来创建记录。

```go
db.Model(&User{}).Create(map[string]interface{}{
  "Name": "jinzhu", "Age": 18,
})

// batch insert from `[]map[string]interface{}{}`
db.Model(&User{}).Create([]map[string]interface{}{
  {"Name": "jinzhu_1", "Age": 18},
  {"Name": "jinzhu_2", "Age": 20},
})
```

---

## 查询记录

**检索单个对象**

GORM 提供了 `First`、`Take`、`Last` 方法，以便从数据库中检索单个对象。当查询数据库时它添加了 `LIMIT 1` 条件，且没有找到记录时，它会返回 `ErrRecordNotFound` 错误

```go
var user User
// 获取第一条记录（主键升序）
db.First(&user)
// SELECT * FROM users ORDER BY id LIMIT 1;

// 获取一条记录，没有指定排序字段
db.Take(&user)
// SELECT * FROM users LIMIT 1;

// 获取最后一条记录（主键降序）
db.Last(&user)
// SELECT * FROM users ORDER BY id DESC LIMIT 1;

result := db.First(&user)
result.RowsAffected // 返回找到的记录数
result.Error        // returns error or nil

// 检查 ErrRecordNotFound 错误
errors.Is(result.Error, gorm.ErrRecordNotFound)
```

 **检索全部对象**

```go
	var users []User
	result := db.Find(&users)
// SELECT * FROM users;
	fmt.Println("总共记录:", result.RowsAffected)
	for _, user := range users {
		fmt.Println(user.ID)
	}
```

---

## gorm的基本查询

**String 条件**

where设置查询条件，First指定表名以及limit 1，find不会添加limit

```go
// Get first matched record
db.Where("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE name = 'jinzhu' ORDER BY id LIMIT 1;

// Get all matched records
db.Where("name <> ?", "jinzhu").Find(&users)
// SELECT * FROM users WHERE name <> 'jinzhu';

// IN
db.Where("name IN ?", []string{"jinzhu", "jinzhu 2"}).Find(&users)
// SELECT * FROM users WHERE name IN ('jinzhu','jinzhu 2');

// LIKE
db.Where("name LIKE ?", "%jin%").Find(&users)
// SELECT * FROM users WHERE name LIKE '%jin%';

// AND
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' AND age >= 22;

// Time
db.Where("updated_at > ?", lastWeek).Find(&users)
// SELECT * FROM users WHERE updated_at > '2000-01-01 00:00:00';

// BETWEEN
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
// SELECT * FROM users WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';
```

**Struct & Map 条件**

```go
// Struct
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 ORDER BY id LIMIT 1;

// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;

// Slice of primary keys
db.Where([]int64{20, 21, 22}).Find(&users)
// SELECT * FROM users WHERE id IN (20, 21, 22);
```

## 更新记录

**Save更新**

`Save` 会保存所有的字段，即使字段是零值

```go
db.First(&user)

user.Name = "jinzhu 2"
user.Age = 100
db.Save(&user)
// UPDATE users SET name='jinzhu 2', age=100, birthday='2016-01-01', updated_at = '2013-11-17 21:34:10' WHERE id=111;
```

`Save` 是一个组合函数。 如果保存值不包含主键，它将执行 `Create`，否则它将执行 `Update` (包含所有字段)。

```go
db.Save(&User{Name: "jinzhu", Age: 100})
// INSERT INTO `users` (`name`,`age`,`birthday`,`update_at`) VALUES ("jinzhu",100,"0000-00-00 00:00:00","0000-00-00 00:00:00")

db.Save(&User{ID: 1, Name: "jinzhu", Age: 100})
// UPDATE `users` SET `name`="jinzhu",`age`=100,`birthday`="0000-00-00 00:00:00",`update_at`="0000-00-00 00:00:00" WHERE `id` = 1
```

**更新单个列**

Model中传入空的User则指定表名，如果user有值则传入where子句

```go
// 根据条件更新
db.Model(&User{}).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE active=true;

// User 的 ID 是 `111`
db.Model(&user).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// 根据条件和 model 的值进行更新
db.Model(&user).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;
```

---

## gorm的软删除细节

删除一条记录时，删除对象需要指定主键，否则会触发 [批量删除](https://gorm.io/zh_CN/docs/delete.html#batch_delete)，例如：

```go
// Email 的 ID 是 `10`
db.Delete(&email)
// DELETE from emails where id = 10;

// 带额外条件的删除
db.Where("name = ?", "jinzhu").Delete(&email)
// DELETE from emails where id = 10 AND name = "jinzhu";
```

**软删除**

如果你的模型包含了 `gorm.DeletedAt`字段（该字段也被包含在`gorm.Model`中），那么该模型将会自动获得软删除的能力！

当调用`Delete`时，GORM并不会从数据库中删除该记录，而是将该记录的`DeleteAt`设置为当前时间，而后的一般查询方法将无法查找到此条记录。

```go
// user's ID is `111`
db.Delete(&user)
// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE id = 111;

// Batch Delete
db.Where("age = ?", 20).Delete(&User{})
// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE age = 20;

// Soft deleted records will be ignored when querying
db.Where("age = 20").Find(&user)
// SELECT * FROM users WHERE age = 20 AND deleted_at IS NULL;
```

如果你并不想嵌套`gorm.Model`，你也可以像下方例子那样开启软删除特性：

```go
type User struct {
  ID      int
  Deleted gorm.DeletedAt
  Name    string
}
```

**查找被软删除的记录**

你可以使用`Unscoped`来查询到被软删除的记录

```go
db.Unscoped().Where("age = 20").Find(&users)
// SELECT * FROM users WHERE age = 20;
```

**永久删除**

你可以使用 `Unscoped`来永久删除匹配的记录

```go
db.Unscoped().Delete(&order)
// DELETE FROM orders WHERE id=10;
```

---





