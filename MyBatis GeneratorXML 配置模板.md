#  MyBatis GeneratorXML 配置模板

------------------------------------------------------------------------------

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    
      <!--   <properties resource="application.properties"/>   引入配置文件 -->  
    
  <classPathEntry location="-----------------------此处应为jdbc驱动jar包的位置-------------------------" />
    <!--指定特定数据库的jdbc驱动jar包的位置-->

  <context id="default" targetRuntime="MyBatis3">
          <!--一个数据库对应一个context ,可选属性如下-->
          <!-- id                             此上下文的唯一标识符,该值将用于某些错误消息中-->
      
          <!-- targetRuntime   此属性用于指定生成的代码的运行时目标 
可选值有:
				 MyBatis3: 默认的值，生成基于MyBatis3.x以上版本的内容，包括XXXBySample；
        	     MyBatis3Simple：类似MyBatis3，只是不生成XXXBySample;-->
      
          <!--defaultModelType   指定生成对象的样式
可选值有：
				conditional :与hierarchical模型相似,除了如果一个实体类只包含一个字段,则不会单独生成此实体类。因此,如果一个表的主键只有一个字段,那么不会为该字段生成单独的实体类,会将该字段合并到基本实体类中。
                flat：该模型为每一张表只生成一个实体类。这个实体类包含表中的所有字段。一般使用这个模型就够了；
                hierarchical：如果表有主键,那么该模型会产生一个单独的主键实体类,如果表还有BLOB字段，则会为表生成一个包含所有BLOB字段的单独的实体类,然后为所有其他的字段生成一个单独的实体类。MBG会在所有生成的实体类之间维护一个继承关系。（看不懂）


    introspectedColumnImpl：类全限定名，用于扩展MBG-->
        <commentGenerator>
            <property name="suppressDate" value="true"/><!-- 是否生成注释代时间戳 -->
            <property name="suppressAllComments" value="true"/><!-- 是否取消注释 -->
        </commentGenerator>
         <!-- optional ,旨在创建class时，对注释进行控制 -->
      
    <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
        connectionURL="----------------------数据对应的url------------------------"
        userId="----------------------------------数据库用户名-------------------------"
        password="------------------------------数据库密码---------------------------">
    </jdbcConnection>

      <!-- 类型转换 -->
    <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>
      <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer true，把JDBC DECIMAL 
              和 NUMERIC 类型解析为java.math.BigDecimal -->

    <javaModelGenerator targetPackage="test.model" targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
      <property name="trimStrings" value="true" />
    </javaModelGenerator>
      
    <sqlMapGenerator targetPackage="test.xml"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>

    <javaClientGenerator type="XMLMAPPER" targetPackage="test.dao"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>

    <table schema="DB2ADMIN" tableName="ALLTYPES" domainObjectName="Customer" >
      <property name="useActualColumnNames" value="true"/>
      <generatedKey column="ID" sqlStatement="DB2" identity="true" />
      <columnOverride column="DATE_FIELD" property="startDate" />
      <ignoreColumn column="FRED" />
      <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />
    </table>

  </context>
</generatorConfiguration>


```



###  <javaModelGenerator> <sqlMapGenerator><javaClientGenerator>查了一下具体的含义，看不太懂。。。暂且先按默认值来吧

------------------------------------------------------------------------------

### <table>元素

该元素至少要配置一个，可以配置多个。

该元素用来配置要通过内省的表。只有配置的才会生成实体类和其他文件。

该元素有一个必选属性：

- `tableName`：指定要生成的表名，可以使用[SQL通配符](http://www.w3school.com.cn/sql/sql_wildcards.asp)匹配多个表。

例如要生成全部的表，可以按如下配置：

```
<table tableName="%" />
```

该元素包含多个可选属性：

- `schema`:数据库的schema,可以使用[SQL通配符](http://www.w3school.com.cn/sql/sql_wildcards.asp)匹配。如果设置了该值，生成SQL的表名会变成如`schema.tableName`的形式。
- `catalog`:数据库的catalog，如果设置了该值，生成SQL的表名会变成如`catalog.tableName`的形式。
- `alias`:如果指定，这个值会用在生成的select查询SQL的表的别名和列名上。 列名会被别名为 alias_actualColumnName(别名_实际列名) 这种模式。
- `domainObjectName`:生成对象的基本名称。如果没有指定，MBG会自动根据表名来生成名称。
- `enableXXX`:XXX代表多种SQL方法，该属性用来指定是否生成对应的XXX语句。
- `selectByPrimaryKeyQueryId`:DBA跟踪工具会用到，具体请看详细文档。
- `selectByExampleQueryId`:DBA跟踪工具会用到，具体请看详细文档。
- `modelType`:和`<context>`的`defaultModelType`含义一样，这里可以针对表进行配置，这里的配置会覆盖`<context>`的`defaultModelType`配置。
- `escapeWildcards`:这个属性表示当查询列，是否对schema和表名中的SQL通配符 (‘_’ and ‘%’) 进行转义。 对于某些驱动当schema或表名中包含SQL通配符时（例如，一个表名是MY_TABLE，有一些驱动需要将下划线进行转义）是必须的。默认值是`false`。
- `delimitIdentifiers`:是否给标识符增加**分隔符**。默认`false`。当`catalog`,`schema`或`tableName`中包含空白时，默认为`true`。
- `delimitAllColumns`:是否对所有列添加**分隔符**。默认`false`。

该元素包含多个可用的`<property>`子元素，可选属性为：

- `ignoreQualifiersAtRuntime`:生成的SQL中的表名将不会包含`schema`和`catalog`前缀。
- `modelOnly`:此属性用于配置是否为表只生成实体类。如果设置为`true`就不会有Mapper接口。如果配置了`<sqlMapGenerator>`，并且`modelOnly`为`true`，那么XML映射文件中只有实体对象的映射元素(`<resultMap>`)。如果为`true`还会覆盖属性中的`enableXXX`方法，将不会生成任何CRUD方法。
- `runtimeCatalog`:运行时的`catalog`，当生成表和运行环境的表的`catalog`不一样的时候可以使用该属性进行配置。
- `runtimeSchema`:运行时的`schema`，当生成表和运行环境的表的`schema`不一样的时候可以使用该属性进行配置。
- `runtimeTableName`:运行时的`tableName`，当生成表和运行环境的表的`tableName`不一样的时候可以使用该属性进行配置。
- `selectAllOrderByClause`:该属性值会追加到`selectAll`方法后的SQL中，会直接跟`order by`拼接后添加到SQL末尾。
- `useActualColumnNames`:如果设置为true,那么MBG会使用从数据库元数据获取的列名作为生成的实体对象的属性。 如果为false(默认值)，MGB将会尝试将返回的名称转换为驼峰形式。 在这两种情况下，可以通过 元素显示指定，在这种情况下将会忽略这个（useActualColumnNames）属性。
- `useColumnIndexes`:如果是true,MBG生成resultMaps的时候会使用列的索引,而不是结果中列名的顺序。
- `useCompoundPropertyNames`:如果是true,那么MBG生成属性名的时候会将列名和列备注接起来. 这对于那些通过第四代语言自动生成列(例如:FLD22237),但是备注包含有用信息(例如:”customer id”)的数据库来说很有用. 在这种情况下,MBG会生成属性名FLD2237_CustomerId。

----------------------------------------------------------------------------



