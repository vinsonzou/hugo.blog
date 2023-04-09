---
title: "[Go] 使用database/sql操作MySQL数据库"
subtitle: ""
date: 2023-04-08T17:11:03+08:00
lastmod: 2023-04-08T17:11:03+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

## 基础用法

Go语言中的`database/sql`包提供了保证SQL或类SQL数据库的泛用接口，并不提供具体的数据库驱动。使用`database/sql`包时必须注入（至少）一个数据库驱动。 我们常用的数据库基本上都有完整的第三方实现。例如：[MySQL驱动](https://github.com/go-sql-driver/mysql)

### 初始化连接

Open函数只是验证其参数格式是否正确，实际上并不创建与数据库的连接。如果要检查数据源的名称是否真实有效，应该调用Ping方法。 返回的DB对象可以安全地被多个goroutine并发使用，并且维护其自己的空闲连接池。因此，Open函数仅需调用一次，无需关闭这个DB对象。

```go
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/go-sql-driver/mysql"
)

// 声明一个全局变量db
var db *sql.DB

// 创建一个初始化数据库的函数
func initDB(dsn string) (err error) {
	db, err = sql.Open("mysql", dsn)
	if err != nil {
		return
	}
	err = db.Ping()
	if err != nil {
		return
	}
	db.SetMaxIdleConns(10)
	db.SetMaxOpenConns(10)
	fmt.Println("数据库连接成功")
	return
}

func main() {
	dsn := "demo:pass@tcp(127.0.0.1:3306)/demo"
	err := initDB(dsn)
	if err != nil {
		fmt.Println("数据库初始化失败. err:", err)
		return
	}
}
```

其中`sql.DB`是一个数据库（操作）句柄，代表一个具有零到多个底层连接的连接池。它可以安全地被多个goroutine同时使用。`database/sql`包会自动创建和释放连接；它也会维护一个闲置连接的连接池。

-  `SetMaxOpenConns`设置与数据库建立连接的最大数目。 如果n大于0且小于最大闲置连接数，会将最大闲置连接数减小到匹配最大开启连接数的限制。 如果n<=0，不会限制最大开启连接数，默认为0（无限制）。
-  `SetMaxIdleConns`设置连接池中的最大闲置连接数。 如果n大于最大开启连接数，则新的最大闲置连接数会减小到匹配最大开启连接数的限制。 如果n<=0，不会保留闲置连接。

### 插入数据

插入、更新和删除操作都使用`Exec`方法。

```go
func (db *DB) Exec(query string, args ...interface{}) (Result, error)
```

Exec执行一次命令（包括查询、删除、更新、插入等），返回的Result是对已执行的SQL命令的总结。参数args表示query中的占位参数。

具体插入数据示例代码如下：

```go
// 插入单条数据
func insertData() {
	sqlStr := "insert into user(name, age) values (?,?)"
	ret, err := db.Exec(sqlStr, "张三", 23)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	theID, err := ret.LastInsertId() // 新插入数据的id
	if err != nil {
		fmt.Printf("get lastinsert ID failed, err:%v\n", err)
		return
	}
	fmt.Printf("insert success, the id is %d.\n", theID)
}

// 插入多条数据
func insertManyData() {
	sqlStr := "insert into user(name, age) values (?,?)"
	insertInfo := map[string]int{
		"张三":  23,
		"李四":  24,
		"王五":  25,
	}
	for k, v := range insertInfo {
		ret, err := db.Exec(sqlStr, k, v)
		if err != nil {
			fmt.Printf("insert failed, err:%v\n", err)
			return
		}
		theID, err := ret.LastInsertId() // 新插入数据的id
		if err != nil {
			fmt.Printf("get lastinsert ID failed, err:%v\n", err)
			return
		}
		fmt.Printf("insert success, the id is %d.\n", theID)
	}
}
```

### 查询数据

#### 单行查询

单行查询`db.QueryRow()`执行一次查询，并期望返回最多一行结果（即Row）。QueryRow总是返回非nil的值，直到返回值的Scan方法被调用时，才会返回被延迟的错误。（如：未找到结果）

```go
func (db *DB) QueryRow(query string, args ...interface{}) *Row
```

具体示例代码：

```go
// 查询数据
func selectData(id int) {
	var u user
	sqlStr := "select id,name,age from user where id=?"
	row := db.QueryRow(sqlStr, id)
	err := row.Scan(&u.id, &u.name, &u.age)
	if err != nil {
		fmt.Println("数据读取失败. err:", err)
		return
	}
	// 输出数据
	fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age)
}

// 返回int，可能为空的情况
func selectDataByName(filterName string) (int, error) {
	var ret_id int
  var retInt sql.NullInt64
  sqlStr := "SELECT id FROM user WHERE name = ?"
  if err := db.QueryRow(sqlStr, filterName).Scan(&retInt); err != nil {
    if err == sql.ErrNoRows {
      return ret_id, err
    }
    return ret_id, err
  }

  if retInt.Valid {
    ret_id = int(retInt.Int64)
  }
  
  return ret_id, nil
}

// 返回字符串，可能为空的情况
func selectDataById(id int) (string, error) {
  var ret_str string
  var retStr sql.NullString
  sqlStr := "SELECT name FROM user WHERE id = ?"
  if err := db.QueryRow(sqlStr, id).Scan(&retStr); err != nil {
    if err == sql.ErrNoRows {
      return ret_str, err
    }
    return ret_str, err
  }

  if retStr.Valid {
    ret_str = retStr.String
  }
  
  return ret_str, nil
}
```

> 注意
>
> - 在QueryRow后要调用Scan方法，不然数据库连接不会被释放
> - 这里有个特殊情况要注意，对于那种没有返回结果的SQL语句，千万不要使用Query方法去执行，这会导致无法回收连接，这时候推荐使用Exec方法去执行。

#### 多行查询

多行查询`db.Query()`执行一次查询，返回多行结果（即Rows），一般用于执行select命令。参数args表示query中的占位参数。

```go
// 查询多行数据
func selectManyData(id int) {
	sqlStr := "select id,name,age from user where id>?"
	rows, err := db.Query(sqlStr, id)
	if err != nil {
		fmt.Println("数据查询失败. err:", err)
	}
	defer rows.Close()
	for rows.Next() {
		var u user
		err := rows.Scan(&u.id, &u.name, &u.age)
		if err != nil {
			fmt.Println("数据读取失败. err:", err)
			return
		}
		// 输出数据
		fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age)
	}
}
```

> 查询多行数据必须调用`Close()`方法来释放数据库连接

#### IN查询

```go
// 查询多行数据
func selectManyDataWithIN(ids []interface) ([]string, error) {
	var names []string
	sqlStr := "SELECT name FROM user WHERE id IN (?" + strings.Repeat(",?", len(ids)-1) + ")"
	rows, err := db.Query(sqlStr, ids...)
	if err != nil {
		return names, err
	}
	defer rows.Close()

	for rows.Next() {
		var u user
		if err := rows.Scan(&u.name); err != nil {
			// 允许为空
			if err == sql.ErrNoRows {
				continue
			}
      
			return "", err
		}

		names := append(names, u.name)
	}
  
	return names, nil
}
```

### 更新数据

更新数据也是用上面介绍的Exec()方法。

示例代码如下：

```go
// 更新数据
func updateData(age, id int) {
	sqlStr := "update user set age=? where id = ?"
	ret, err := db.Exec(sqlStr, age, id)
	if err != nil {
		fmt.Println("数据更新失败. err:", err)
		return
	}
	// 操作影响的行数
	rowNum, err := ret.RowsAffected()
	if err != nil {
		fmt.Println("获取影响的行数失败. err:", err)
		return
	}
	// 打印影响的行数
	fmt.Printf("修改成功，影响行数:%d\n", rowNum)
}
```

### 删除数据

删除数据也是用Exec()方法。

示例代码如下:

```go
// 删除数据
func deleteData(id int) {
	sqlStr := "delete from user where id = ?"
	ret, err := db.Exec(sqlStr, id)
	if err != nil {
		fmt.Printf("数据删除失败, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("获取受影响行数失败, err:%v\n", err)
		return
	}
	fmt.Printf("删除成功，影响行数:%d\n", n)
}
```

### 完整代码

```go
package main

import (
	"database/sql"
  "strings"
	"fmt"

	_ "github.com/go-sql-driver/mysql"
)

// 声明一个全局变量db
var db *sql.DB

// 声明一个user结构体，用来保存查询数据
type user struct {
	id   int
	name string
	age  int
}

// 创建一个初始化数据库的函数
func initDB(dsn string) (err error) {
	db, err = sql.Open("mysql", dsn)
	if err != nil {
		return
	}
	err = db.Ping()
	if err != nil {
		return
	}
	db.SetMaxIdleConns(10)
	db.SetMaxOpenConns(10)
	fmt.Println("数据库连接成功")
	return
}

// 插入单条数据
func insertData() {
	sqlStr := "insert into user(name, age) values (?,?)"
	ret, err := db.Exec(sqlStr, "王五", 25)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	theID, err := ret.LastInsertId() // 新插入数据的id
	if err != nil {
		fmt.Printf("get lastinsert ID failed, err:%v\n", err)
		return
	}
	fmt.Printf("insert success, the id is %d.\n", theID)
}

// 插入多条数据
func insertManyData() {
	sqlStr := "insert into user(name, age) values (?,?)"
	insertInfo := map[string]int{
		"张三":  23,
		"李四":  24,
		"王五":  25,
	}
	for k, v := range insertInfo {
		ret, err := db.Exec(sqlStr, k, v)
		if err != nil {
			fmt.Printf("insert failed, err:%v\n", err)
			return
		}
		theID, err := ret.LastInsertId() // 新插入数据的id
		if err != nil {
			fmt.Printf("get lastinsert ID failed, err:%v\n", err)
			return
		}
		fmt.Printf("insert success, the id is %d.\n", theID)
	}
}

// 查询数据
func selectData(id int) {
	var u user
	sqlStr := "select id,name,age from user where id=?"
	row := db.QueryRow(sqlStr, id)
	err := row.Scan(&u.id, &u.name, &u.age)
	if err != nil {
		fmt.Println("数据读取失败. err:", err)
		return
	}
	// 输出数据
	fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age)
}

// 查询多行数据
func selectManyData(id int) {
	sqlStr := "select id,name,age from user where id>?"
	rows, err := db.Query(sqlStr, id)
	if err != nil {
		fmt.Println("数据查询失败. err:", err)
	}
	defer rows.Close()
	// 循环读数据
	for rows.Next() {
		var u user
		err := rows.Scan(&u.id, &u.name, &u.age)
		if err != nil {
			fmt.Println("数据读取失败. err:", err)
			return
		}
		// 输出数据
		fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age)
	}
}

// 返回int，可能为空的情况
func selectDataByName(filterName string) (int, error) {
	var ret_id int
  var retInt sql.NullInt64
  sqlStr := "SELECT id FROM user WHERE name = ?"
  if err := db.QueryRow(sqlStr, filterName).Scan(&retInt); err != nil {
    if err == sql.ErrNoRows {
      return ret_id, err
    }
    return ret_id, err
  }

  if retInt.Valid {
    ret_id = int(retInt.Int64)
  }
  
  return ret_id, nil
}

// 返回字符串，可能为空的情况
func selectDataById(id int) (string, error) {
  var ret_str string
  var retStr sql.NullString
  sqlStr := "SELECT name FROM user WHERE id = ?"
  if err := db.QueryRow(sqlStr, id).Scan(&retStr); err != nil {
    if err == sql.ErrNoRows {
      return ret_str, err
    }
    return ret_str, err
  }

  if retStr.Valid {
    ret_str = retStr.String
  }
  
  return ret_str, nil
}

// 查询多行数据
func selectManyDataWithIN(ids []interface) ([]string, error) {
  var names []string
  sqlStr := "SELECT name FROM user WHERE id IN (?" + strings.Repeat(",?", len(ids)-1) + ")"
	rows, err := db.Query(sqlStr, ids...)
	if err != nil {
    return names, err
	}
	defer rows.Close()

	for rows.Next() {
		var u user
		if err := rows.Scan(&u.name); err != nil {
      // 允许为空
      if err == sql.ErrNoRows {
        continue
      }
      
    	return "", err
    }

    names := append(names, u.name)
  }
  
  return names, nil
}

// 更新数据
func updateData(age, id int) {
	sqlStr := "update user set age=? where id = ?"
	ret, err := db.Exec(sqlStr, age, id)
	if err != nil {
		fmt.Println("数据更新失败. err:", err)
		return
	}
	// 操作影响的行数
	rowNum, err := ret.RowsAffected()
	if err != nil {
		fmt.Println("获取影响的行数失败. err:", err)
		return
	}
	// 打印影响的行数
	fmt.Printf("修改成功，影响行数:%d\n", rowNum)
}

// 删除数据
func deleteData(id int) {
	sqlStr := "delete from user where id = ?"
	ret, err := db.Exec(sqlStr, id)
	if err != nil {
		fmt.Printf("数据删除失败, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("获取受影响行数失败, err:%v\n", err)
		return
	}
	fmt.Printf("删除成功，影响行数:%d\n", n)
}

func main() {
	dsn := "demo:pass@tcp(127.0.0.1:3306)/demo"
	err := initDB(dsn)
	if err != nil {
		fmt.Println("数据库初始化失败. err:", err)
		return
	}
	// 插入数据操作
	// insertData()
	// insertManyData()
	// selectData(1)
	// selectManyData(0)
	// updateData(25, 3)
	deleteData(3)
}
```

## 进阶用法

### MySQL预处理

#### 什么是预处理？

普通SQL语句执行过程：

1. 客户端对SQL语句进行占位符替换得到完整的SQL语句。
2. 客户端发送完整SQL语句到MySQL服务端
3. MySQL服务端执行完整的SQL语句并将结果返回给客户端。

预处理执行过程：

1. 把SQL语句分成两部分，命令部分与数据部分。
2. 先把命令部分发送给MySQL服务端，MySQL服务端进行SQL预处理。
3. 然后把数据部分发送给MySQL服务端，MySQL服务端对SQL语句进行占位符替换。
4. MySQL服务端执行完整的SQL语句并将结果返回给客户端。

#### 为什么要预处理？

- 优化MySQL服务器重复执行SQL的方法，可以提升服务器性能，提前让服务器编译，一次编译多次执行，节省后续编译的成本。

- 避免SQL注入问题。

#### 使用预处理

在Go中使用`*sql.DB.prepare()`来处理预处理问题。 方法如下：

```go
func (db *DB) Prepare(query string) (*Stmt, error)
```

`Prepare`方法会先将sql语句发送给MySQL服务端，返回一个准备好的状态用于之后的查询和命令。返回值可以同时执行多个查询和命令。

示例代码如下：

```go
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/go-sql-driver/mysql"
)

// 声明一个全局变量db
var db *sql.DB

// 声明一个user结构体，用来保存查询数据
type user struct {
	id   int
	name string
	age  int
}

// 创建一个初始化数据库的函数
func initDB(dsn string) (err error) {
	db, err = sql.Open("mysql", dsn)
	if err != nil {
		return
	}
	err = db.Ping()
	if err != nil {
		return
	}
	db.SetMaxIdleConns(10)
	db.SetMaxOpenConns(10)
	fmt.Println("数据库连接成功")
	return
}

// 插入单条数据
func prepareInsertData() {
	sqlStr := "insert into user(name, age) values (?,?)"
	stmt, err := db.Prepare(sqlStr)
	if err != nil {
		fmt.Println("预处理失败. err:", err)
		return
	}
	_, err = stmt.Exec("赵六", 36)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	fmt.Println("数据插入成功")
}

// 查询数据
func prepareSelectData(id int) {
	var u user
	sqlStr := "select id,name,age from user where id=?"
	stmt, err := db.Prepare(sqlStr)
	if err != nil {
		fmt.Println("预处理失败. err:", err)
		return
	}
	defer stmt.Close()
	rows, err := stmt.Query(id)
	if err != nil {
		fmt.Println("预处理查询失败. err:", err)
		return
	}
	for rows.Next() {
		err := rows.Scan(&u.id, &u.name, &u.age)
		if err != nil {
			fmt.Println("数据读取失败. err:", err)
			return
		}
		// 输出数据
		fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age)
	}
}

// 更新数据
func prepareUpdateData(age, id int) {
	sqlStr := "update user set age=? where id = ?"
	stmt, err := db.Prepare(sqlStr)
	if err != nil {
		fmt.Println("预处理失败. err:", err)
		return
	}
	_, err = stmt.Exec(age, id)
	if err != nil {
		fmt.Println("数据更新失败. err:", err)
		return
	}

	fmt.Println("修改成功")
}

// 删除数据
func prepareDeleteData(id int) {
	sqlStr := "delete from user where id = ?"
	stmt, err := db.Prepare(sqlStr)
	if err != nil {
		fmt.Println("预处理失败. err:", err)
		return
	}
	_, err = stmt.Exec(id)
	if err != nil {
		fmt.Printf("数据删除失败, err:%v\n", err)
		return
	}
	fmt.Println("删除成功")
}

func main() {
	dsn := "demo:pass@tcp(127.0.0.1:3306)/demo"
	err := initDB(dsn)
	if err != nil {
		fmt.Println("数据库初始化失败. err:", err)
		return
	}
	prepareInsertData()
}
```

### MySQL事务

#### 什么是事务？

事务：一个最小的不可再分的工作单元；通常一个事务对应一个完整的业务(例如银行账户转账业务，该业务就是一个最小的工作单元)，同时这个完整的业务需要执行多次的DML(insert、update、delete)语句共同联合完成。A转账给B，这里面就需要执行两次update操作。 在MySQL中只有使用了`Innodb`数据库引擎的数据库或表才支持事务。事务处理可以用来维护数据库的完整性，保证成批的SQL语句要么全部执行，要么全部不执行。

#### 事务的ACID

通常事务必须满足4个条件（ACID）：原子性（Atomicity，或称不可分割性）、一致性（Consistency）、隔离性（Isolation，又称独立性）、持久性（Durability）。

| 条件   | 解释                                                         |
| ------ | ------------------------------------------------------------ |
| 原子性 | 一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。 |
| 一致性 | 在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。 |
| 隔离性 | 数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。 |
| 持久性 | 事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。 |

#### 事务相关方法

Go语言中使用以下三个方法实现MySQL中的事务操作。 

开始事务

```go
func (db *DB) Begin() (*Tx, error)
```

提交事务

```go
func (tx *Tx) Commit() error
```

回滚事务

```go
func (tx *Tx) Rollback() error
```

#### 示例代码

```go
// 事务操作
func transaction() {
	// 开启事务
	tx, err := db.Begin()
	if err != nil {
		fmt.Println("开启事务失败。err:", err)
		return
	}
	sqlStr1 := "update user set age=age-? where id =?"
	_, err = tx.Exec(sqlStr1, 3, 4)
	if err != nil {
		fmt.Println("SQL 1 执行失败，准备回滚")
		tx.Rollback()
		return
	}
	sqlStr2 := "update user set age=age+? where id =?"
	_, err = tx.Exec(sqlStr2, 3, 6)
	if err != nil {
		fmt.Println("SQL 2 执行失败，准备回滚")
		tx.Rollback()
		return
	}
	// 如果上面都没错。则提交事务
	err = tx.Commit()
	if err != nil {
		fmt.Println("事务提交失败，准备回滚")
		tx.Rollback()
		return
	}
	// 如果代码走到了这里，则代表事务处理成功
	fmt.Println("执行事务成功")
}
```


### SQL中的占位符

不同的数据库中，SQL语句使用的占位符语法不尽相同。

| 数据库     | 占位符语法   |
| ---------- | ------------ |
| MySQL      | `?`          |
| PostgreSQL | `$1`, `$2`等 |
| SQLite     | `?` 和`$1`   |
| Oracle     | `:name`      |

### 多数据库连接池管理

因业务场景比较特殊，系统中有多个数据库，要根据不同参数去连不同数据库，使用map实现连接池动态管理

```go
var dbMap map[string]*sql.DB

func GetDbContext(dbName string) *sql.DB {
	if dbMap == nil {
		dbMap = make(map[string]*sql.DB)
	}

	if db, ok := dbMap[dbName]; ok {
		return db
	} else {
		dsn := "demo:pass@tcp(127.0.0.1:3306)/" + dbName
		db, _ := sql.Open("mysql", dsn)
		dbMap[dbName] = db
		return db
	}
}
```