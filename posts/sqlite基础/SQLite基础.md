# SQLite 基础

## 下载

就是一库调用，和命令行工具，下载zip，解压即可。

配置到环境变量

```bash
vim ~/.zshrc

#加入
export SQLITE=/sqlite_path/...
export PATH=$SQLITE:$PATH

source ~/.zshrc
```



## 基础命令

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

   ```sqlite
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

    ```sqlite
    INSERT INTO users (name, email, age) VALUES ('Alice', 'alice@hdu.com', 30); 
    INSERT INTO users (name, email, age) VALUES ('Bob', 'bob@hdu.com', 25); 
    INSERT INTO users (name, email, age) VALUES ('Charlie', 'charlie@hdu.com', 35); 
    ```

      
