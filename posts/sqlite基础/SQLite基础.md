# SQLite 基础

## 1.下载

就是一库调用，和命令行工具，下载zip，解压即可。

配置到环境变量

```bash
vim ~/.zshrc

#加入
export SQLITE=/sqlite_path/...
export PATH=$SQLITE:$PATH

source ~/.zshrc
```



## 2.基础命令

打开SQLite CLI

```bash
> SQLite3
```

1. 当前文件夹中创建db文件

   ```sqlite
   .open mydb.db
   ```

2. 退出SQLite CLI

   ```sqlite
   .quit
   ```

3. 进入数据库

   ```bash
   sqlite3 mydb.db
   ```

4. **创建表**

   ```sql
   CREATE TABLE users (
   	id INTEGER PRIMARY KEY, 
     name TEXT NOT NULL, 
     email TEXT UNIQUE,
     age INTEGER
   )
   ```

   `INTEGER`: 整型

   `PRIMARY`: 主键， 唯一标识每一行，自动增长

   `TEXT NOT NULL`: 文本不为空

   `TEXT UNIQUE`: 文本，唯一
   
   
   
    - `.tables`: 查看所有表
   
   
    - `.schema users`: 查看表结构
   
5. **插入数据**

    ```sql
    INSERT INTO users (name, email, age) VALUES ('Alice', 'alice@hdu.com', 30); 
    INSERT INTO users (name, email, age) VALUES ('Bob', 'bob@hdu.com', 25); 
    INSERT INTO users (name, email, age) VALUES ('Charlie', 'charlie@hdu.com', 35); 
    ```

6. **查询数据**

    ```sql
    // 查询所有信息
    SELECT * FROM users; 
    //查询特定列
    SELECT name, age FROM users; 
    //按条件查询（WHERE）
    SELECT * FROM users WHERE age > 30; 
    SELECT * FROM users WHERE name = 'Bob'; 
    
    ```

7. **美化输出**

    ```sql
    .headers on 
    .mode column
    SELECT * FROM users; 
    ```

8. **修改数据**

    ```sql
    UPDATE users SET age = 26 WHERE name = 'Bob'; 
    ```

    Warning: `UPDATE`一定要加`WHERE`否则会修改整个表

9. **删除数据**

    ```sql
    DELETE FROM users WHERE id = 3; 
    ```

10. 创建索引

    为email创建列表索引，加快查询

    ```sql
    CREATE INDEX index_email ON users(email); 
    ```

## 3.关联查询（JOIN）

关系型数据库精髓

假设有一个posts（文章）表

```sql
CREATE TABLE posts ( 
	post_id INTEGER PRIMARY KEY, 
  user_id INTEGER, 
  title TEXT, 
  content TEXT, 
  FOREIGN KEY (user_id) REFERENCES user(id)
); 

INSERT INTO posts (user_id, title, content) VALUES (1, 'Alice\' post', '...'); 
INSERT INTO posts (user_id, title, content) VALUES (2, 'Bob\' post', '...'); 
INSERT INTO posts (user_id, title, content) VALUES (1, 'Alice', '...'); 
```

查询所有文章作者的名字， 只需要把 posts表 和 users表连接起来

```sql
SELECT
	users.name, 
	posts.title
FROM 
	posts
JOIN
	users ON posts.user_id = users.id
```



## 4. C 接口

mac 安装

```bash
brew install sqlite3
```

源码整合

下载 `sqlite-amalgamation-3500400.zip`

[官方api文档](https://sqlite.org/cintro.html)

**样例**

```cpp
#include <cstdio>
#include <sqlite3.h>

static int callback(void *NotUsed, int argc, char **argv, char **azColName)
{
    for (int i = 0;i < argc;i ++)
    {
        printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
    }
    printf("\n");
    return 0;
}

int main(int argc, char* argv[])
{
    sqlite3 *db;
    char *zErrMsg = nullptr;
    int rc;

    if (argc != 3)
    {
        fprintf(stderr, "Usage: %s DATABASE SQL-STATEMENT\n", argv[0]);
        return 1;
    }
    rc = sqlite3_open(argv[1], &db);
    if (rc)
    {
        fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
        return 1;
    }
    rc = sqlite3_exec(db, argv[2], callback, nullptr, &zErrMsg);
    if (rc != SQLITE_OK)
    {
        fprintf(stderr, "SQL error: %s\n", zErrMsg);
        sqlite3_free(zErrMsg);
    }
    sqlite3_close(db);
    return 0;
}
```



## 6. C++ 封装

[SQLiteCPP](https://github.com/SRombauts/SQLiteCpp) 提供了c++的封装。

**Example1**

```c++
#include <SQLiteCpp/SQLiteCpp.h>
#include <SQLiteCpp/VariadicBind.h>

try
{
    SQLite::Database db("expable.db3");
  	// '?' 为占位符
    SQLite::Statement query(db, "SELECT * FROM test WHERE size > ?");
		
  	//将第1个占位符绑定为6
    query.bind(1, 6);
		
 		//取出查询结果直到空
    while (query.executeStep())
    {
        int id = query.getColumn(0);
        const char* value = query.getColumn(1);
        int size = query.getColumn(2);
    }
} catch (std::exception& e)
{
    std::cerr << "exception: " << e.what() << std::endl;
}
```



**Example2**

```c++
try
{
    SQLite::Database    db("transaction.db3", SQLite::OPEN_READWRITE|SQLite::OPEN_CREATE);

  	//删除 test
    db.exec("DROP TABLE IF EXISTS test");

    // 把下列操作统一提交，保证原子性
    SQLite::Transaction transaction(db);

    db.exec("CREATE TABLE test (id INTEGER PRIMARY KEY, value TEXT)");

    int nb = db.exec("INSERT INTO test VALUES (NULL, \"test\")");
    std::cout << "INSERT INTO test VALUES (NULL, \"test\")\", returned " << nb << std::endl;
		
  	//提交
    transaction.commit();
}
catch (std::exception& e)
{
    std::cout << "exception: " << e.what() << std::endl;
}
```



