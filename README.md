## 易买网开发日志

### 搭建web框架（已完成）

### 建库、建表（已完成）

### easycode自动生成实体类（已完成）

### 配置Alibaba数据源——Druid（已完成）
> 下载druid-1.1.9.jar并引入项目

### 编写数据库操作辅助类——DataSourceUtil.java（已完成）

> 配置Druid数据源

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

> 配置Druid监控

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
		  http://java.sun.com/xml/ns/javaee/web-app_3_1.xsd"
           version="3.1">

    <filter>
        <filter-name>DruidWebStatFilter</filter-name>
        <filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
        <init-param>
            <param-name>exclusions</param-name>
            <param-value>*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>DruidWebStatFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <servlet>
        <servlet-name>DruidStatView</servlet-name>
        <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>DruidStatView</servlet-name>
        <url-pattern>/druid/*</url-pattern>
    </servlet-mapping>
</web-app>

```

> 连接数据库的方法

```java
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
```

> 关闭数据库的方法

```java
/**
     * 关闭连接的方法
     * @param conn 数据库连接对象
     */
    public static void closeConnection(Connection conn){
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
```

### 首页商品分类信息dao层（已完成）
> 接口IProductCategory

```java
package com.buy.dao.product;

import com.buy.entity.EasybuyProductCategory;

import java.util.List;

/**
 * @Author: laoyu
 * @Date: 2020/2/19 9:59
 * @Description:
 */
public interface IProductCategory {
    //获取商品的一级分类
    List<EasybuyProductCategory> queryAllProductCategory(String parentId);
}

```

> 实现类ProductCategoryImpl

```java
package com.buy.dao.product;

import com.buy.entity.EasybuyProductCategory;
import com.buy.utils.DataSourceUtil;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.ArrayList;
import java.util.List;

/**
 * @Author: laoyu
 * @Date: 2020/2/19 10:00
 * @Description:
 */
public class ProductCategoryImpl implements IProductCategory {

    private Connection conn;
    private PreparedStatement pstmt;
    private ResultSet rs;

    @Override
    public List<EasybuyProductCategory> queryAllProductCategory(String parentId) {
        List<EasybuyProductCategory> productCategories = new ArrayList<EasybuyProductCategory>();
        EasybuyProductCategory productCategory = null;

        try {
            StringBuffer sql = new StringBuffer();
            sql.append("select * from easybuy_product_category where 1=1");

            //判断parentID的值，如果为0，显示的是一级分类
            if (!"".equals(parentId) && null != parentId) {
                sql.append(" and parentId = ?");
            }

            //获取连接
            conn = DataSourceUtil.getConn();
            pstmt = conn.prepareStatement(sql.toString());
            if (!"".equals(parentId) && null != parentId) {
                pstmt.setObject(1, parentId);
            }

            rs = pstmt.executeQuery();

            //处理结果集
            while (rs.next()) {
                //实例化对象
                productCategory = new EasybuyProductCategory();
                productCategory.setId(rs.getInt(1));
                productCategory.setName(rs.getString(2));
                productCategory.setParentid(rs.getInt(3));
                productCategory.setType(rs.getInt(4));
                productCategory.setIconclass(rs.getString(5));
                //将对象填充到集合中
                productCategories.add(productCategory);
            }
            System.out.println(sql.toString());

        } catch (Exception e) {
            e.printStackTrace();
        }

        return productCategories;
    }
}

```
### 首页商品分类信息service层
### 首页商品分类信息servlet层
### 首页商品分类信息jsp层
> 把首页的静态页面素材改为jsp文件

> 数据渲染

### 编写数据库连接配置文件——database.properties

### 配置log日志

### 用户登录与注销




