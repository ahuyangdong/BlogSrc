---
title: SpringMVC框架中配置多数据源（测试库与正式库分离）
date: 2017-03-31 15:13:18
id: 151318
tags: [SpringMVC,多数据源]
categories: Java Web
---
目的
--
为保护正式库数据相对安全，我们将系统中的数据库进行了分离。
开发者在windows环境开发时连接测试库，产品发布到Linux服务器上时连接正式库。

策略
--

鉴于开发环境与产品服务器存在系统差别，可以利用系统名称标识 os.name 来区分数据源。

步骤
--

 - 1、在jdbc.properties中添加以不同名称标识的路径参数，如：
 

``` java
driver=com.mysql.jdbc.Driver

#测试库
url-dev=jdbc:mysql://192.168.1.11:3306/db_xx?useUnicode=true&amp;characterEncoding=utf8&amp;autoReconnect=true
username-dev=user
password-dev=password

#正式库
url-pro=jdbc:mysql://192.168.1.201:3306/db_xx?useUnicode=true&amp;characterEncoding=utf8&amp;autoReconnect=true
username-pro=user
password-pro=password

#定义初始连接数  
initialSize=0
#定义最大连接数  
maxActive=20
#定义最大空闲  
maxIdle=20
#定义最小空闲  
minIdle=1
#定义最长等待时间  
maxWait=60000
```


 - 2、自定义AbstractRoutingDataSource，路径选择器
 

``` java
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

public class DynamicDataSource extends AbstractRoutingDataSource {
	private static String dataSourceRef = "dataSource-dev"; // 数据源配置
	static {
		try {
			String os = System.getProperties().getProperty("os.name");
			if (os == null || os.length() == 0) {
				os = "win";
			}
			if (os.toLowerCase().startsWith("win")) {
				// 测试库
				dataSourceRef = "dataSource-dev";
			} else {
				// 正式库
				dataSourceRef = "dataSource-pro";
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	@Override
	protected Object determineCurrentLookupKey() {
		return dataSourceRef;
	}

}
```

 - 3、配置spring-mabatis.xml数据源部分

``` xml
<!-- 本系统数据库配置 -->
<bean id="dataSource-base" class="org.apache.commons.dbcp.BasicDataSource"
    destroy-method="close">
    <property name="driverClassName" value="${driver}" />
    <!-- 初始化连接大小 -->
    <property name="initialSize" value="${initialSize}"></property>
    <!-- 连接池最大数量 -->
    <property name="maxActive" value="${maxActive}"></property>
    <!-- 连接池最大空闲 -->
    <property name="maxIdle" value="${maxIdle}"></property>
    <!-- 连接池最小空闲 -->
    <property name="minIdle" value="${minIdle}"></property>
    <!-- 获取连接最大等待时间 -->
    <property name="maxWait" value="${maxWait}"></property>
</bean>

<!-- 测试库 -->
<bean id="dataSource-dev" parent="dataSource-base">
    <property name="url" value="${url-dev}" />
    <property name="username" value="${username-dev}" />
    <property name="password" value="${password-dev}" />
</bean>

<!-- 正式库 -->
<bean id="dataSource-pro"  parent="dataSource-base">
    <property name="url" value="${url-pro}" />
    <property name="username" value="${username-pro}" />
    <property name="password" value="${password-pro}" />
</bean>

<!-- 使用自定义的路径选择器 -->
<bean id="dataSource" class="com.xxx.database.DynamicDataSource">
    <property name="defaultTargetDataSource" ref="dataSource-dev"/>
    <property name="targetDataSources">
        <map key-type="java.lang.String">
            <entry key="dataSource-dev" value-ref="dataSource-dev"></entry>
            <entry key="dataSource-pro" value-ref="dataSource-pro"></entry>
        </map>
    </property>
</bean>

<!-- spring和MyBatis整合 -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 数据源 -->
    <property name="dataSource" ref="dataSource" />
    <property name="configLocation" value="classpath:sqlMapConfig.xml"></property>
    <!-- 自动扫描mapping.xml文件 -->
    <property name="mapperLocations" value="classpath:com/xxx/mapper/*.xml"></property>
</bean>

<!-- DAO接口所在包名，Spring会自动查找其下的类 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.xxx.dao" />
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
</bean>

<!-- (事务管理)transaction manager -->
<bean id="transactionManager"
    class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
```

## 小结 ##
在当前业务特点下，我们通过增加AbstractRoutingDataSource实例实现了数据源的动态变更，巧妙的解决了多数据源的需求。

因为数据源在系统初始化时（static方法）就已经指定，因此不会出现切换数据源引发的异常。

我们看下org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource.determineCurrentLookupKey()方法源码注释。
``` java
/**
	 * Determine the current lookup key. This will typically be
	 * implemented to check a thread-bound transaction context.
	 * <p>Allows for arbitrary keys. The returned key needs
	 * to match the stored lookup key type, as resolved by the
	 * {@link #resolveSpecifiedLookupKey} method.
	 */
	protected abstract Object determineCurrentLookupKey();
```
大概意思是决定当前连接的数据源（lookupKey），通过本方法返回需要使用的数据源ID即可动态修改连接配置。