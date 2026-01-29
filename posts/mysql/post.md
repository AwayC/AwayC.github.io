## MySQL 使用

> [!TIP]
>
> MacOS 环境

### 1. 安装 MySQL 官网，或者 Homebrew

### 2. 安装 C++ connector

这里安装的不是`JDBC`的版本，是新的 `X DevAPI`, Homebrew 只能下载后者

``` bash
brew install mysql-connector-c++
```

### 3. CMake 导入 connector

```cmake
set(MYSQL_CONNECTOR_DIR "/opt/homebrew/opt/mysql-connector-c++") # 路径

find_path(MYSQL_CONNECTOR_INCLUDE_DIR mysqlx/xdevapi.h
    HINTS ${MYSQL_CONNECTOR_DIR}/include
)

find_library(MYSQL_CONNECTOR_LIBRARY NAMES mysqlcppconnx
    HINTS ${MYSQL_CONNECTOR_DIR}/lib
)

target_link_libraries(main
    ${MYSQL_CONNECTOR_LIBRARY}
)
target_include_directories(main PRIVATE
    ${MYSQL_CONNECTOR_INCLUDE_DIR}
)
```

### 4. X DevAPI 文档

[官方文档](https://dev.mysql.com/doc/x-devapi-userguide/en/database-connection-example.html)

### 5. 示例

池连接

```c++
const std::string url = "mysqlx://usr@password:localhost:33060/usr_db"; 
mysqlx::Clinet cli(url, mysqlx::ClientOption::POOL_MAX_SIZE, 7); // client 管理连接池

// 获取连接
mysqlx::Session sess = cli.getSession(); 

// 请求 api
try { 
  auto result = sess.sql(R"(SELECT uid, email FROM users WHERE username = ?)")
                    .bind("away")
                    .execute(); 

  mysqlx::Row row; 
  while(row = result.fetchOne()) { 
    std::cout << row << ", uid: " << row[0].get<int>() << 
      ", email" << row[1].get<std::string>() << std::endl;
  }

  sess.sql(R"(UPDATE users SET password = ? , email = ? WHERE uid = ?)")
      .bind("password", "email")
      .excute();
  
} catch(const mysqlx::Error& e) { 
	std::cerr << e << std::endl; 
}


```



