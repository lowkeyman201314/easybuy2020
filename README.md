## 易买网开发日志

### 搭建web框架（已完成）

### 建库、建表（已完成）

### easycode自动生成实体类（已完成）

### 配置Alibaba数据源——Druid（已完成）
#### 下载druid-1.1.9.jar并引入项目
#### 

### 编写数据库操作辅助类——DataSourceUtil.java（已完成）
```java
package com.buy.utils;

import com.alibaba.druid.pool.DruidDataSource;

import java.sql.Connection;
import java.sql.SQLException;

/**
 * @Author: laoyu
 * @Date: 2020/2/19 8:53
 * @Description: 数据库操作辅助类
 */
public class DataSourceUtil {

    private static final String URL = "jdbc:mysql://127.0.0.1:3306/easybuy?useUnicode=true&characterEncoding=UTF-8";
    private static final String USER = "root";
    private static final String PASSWORD = "root";
    private static final String DRIVER = "com.mysql.jdbc.Driver";

    //创建druid数据源对象
    private static DruidDataSource druidDataSource =null;

    static{
        try {
            init();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     * 配置阿里巴巴数据源
     */

    private static void init() throws SQLException {
        //实例化DruidDataSource
        druidDataSource = new DruidDataSource();
        //设置属性的值
        druidDataSource.setDriverClassName(DRIVER);
        druidDataSource.setUrl(URL);
        //设置连接池相关属性
        druidDataSource.setInitialSize(5);//初始化连接池数量
        druidDataSource.setMaxActive(100);//最大连接数
        druidDataSource.setMinIdle(1);//最大空闲连接数
        druidDataSource.setMaxWait(1000);//连接等待时长，单位：毫秒
        druidDataSource.setFilters("stat");//设置监控
    }

    /**
     * 连接数据源的方法
     * @return 连接对象
     */
    public static Connection getConn(){
        Connection conn=null;
        //加载mysql驱动（开启服务）
        try {
            Class.forName(DRIVER);
            //如果数据库处于没有连接状态，则创建一个连接
            if (conn == null) {
                conn = druidDataSource.getConnection(USER,PASSWORD);
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return conn;
    }

}

```

#### 连接数据库的方法
#### 关闭数据库的方法


### 首页商品分类信息（已完成）

### 配置log日志

### 编写数据库连接配置文件——database.properties


