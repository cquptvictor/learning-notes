# Mybatis

## JDBC的缺陷

1. 创建连接存在硬编码
2. 执行statement时存在硬编码
3. 频繁的开闭数据库连接，会影响数据库的性能。

## Mybaits的配置文件

### Hello Mybatis

1. 创建全局配置文件
2. 根据配置文件创建SqlSessionFactory
3. 通过SqlSessionFactory创建SqlSession（非线程安全）来执行映射的SQL语句

### 全局配置文件

- `<environments default="" />` :设置多个环境，通过default来切换环境
- `<enviroment>:`配置连接哪个数据库，是否使用连接池，事务配置等；需要id来标识
- 使用 `<properties url = "" or resource="" />` 标签引入外部properties文件，${}来取值，url是绝对路径或网络路径，resource的类相对路径

#### settings标签

配置一些参数

```
<settings>

<setting name = "" value=""/>

</settings>
```

#### Mybatis配置别名

- `<typeAliases>`: 单独配置类的别名，或者@Alias注解配置别名
- 配置的别名对namespace无效，namespace任然要使用全类名
- 通过package标签批量配置别名，但是要注意子包下相同类名的冲突
- 无论是批量配置还是单独配置，别名都不区分大小写；且默认为类名

#### transactionManager

JDBC或Managed事务管理器;或实现transactionFactory接口来完成自定义事务管理器

#### dataSource数据源

- type有UNPOOLED,POOLED,JNDI
- 如果要使用自定义的数据源，实现DataSource接口，方法返回自定义的数据源,type是全类名

#### mappers

- 引入映射文件
- resource、class、url三种注册方式
- `<package name=""/>`:进行批量注册
- class方式注册非注解接口时，接口与对应的XML需在同一个包下

### 映射配置文件

- `<resultMap>`:配置实体与数据库的表的映射;<id>映射主键;<result>映射非主键属性； #{}为占位符
- 写SQL语句
- `<mapper namespace="">`:需要在namespace中声明唯一命名空间
- ParameterType为Boolean时，受影响的行数超过0就返回True，0行返回false



## CRUD

默认需要手动提交

### 增

id：根据id来调用的SQL语句



```
<insert id="add" parameterType="domain.Student">
    INSERT INTO STUDENT (ID, NAME, SAL) VALUES (#{id},#{name},#{sal});
</insert>
```

### 查

- parameterType:传入参数的数据类型，可以不写，自动判断
- resultMap:返回值类型，应该为resultMap映射相应对象时设置的id
- 返回的是List，返回值类型写为集合中元素的对象
- 如果返回值`Map<String,User>`，只能有一条返回值,{key=value,key=value}；`List<Map>`可以返回多条
- @MapKey指定返回值的key是pojo的哪个属性，key重复会发生覆盖；{key = User{ }};加上注解后，相当于map中的map
- 如果查询出来的对象不止一个，例如`List<Student>`，那么只用写一个就可以了，调用时应使用selectList




```
<select id="select" parameterType="int" resultMap="student">
    select * from student where id = #{id};
</select>
```

### 删

```
<delete id ="delete" parameterType="int">
    delete from student where id = #{id};
</delete>
```

### 改



```
<update id="update" parameterType="domain.Student">
    update student set id = #{id}, sal = #{sal}, name = #{name};
</update>
```

- 实体的属性不一定要全部应用到参数上，如果id设置为自增的，SQL插入语句去掉id，传入Student为参数也同样能够成功
- 增删改可以用相同的XML标签

## Mybatis接口式编程

- 定义一个接口；mappings文件的namespace是接口的全类名，方法的id是接口的方法名
- 这样传入的参数具有类型检查，而不是Object
- 基本类型使用包装类，来避免空指针错误

## #{}占位符

- 单个参数时括号中随便写一个名字，但不能为空
- 多个参数会被封装成一个Map，key为param1——paramN，可以通过@Param来指定key
- 如果多个参数中有pojo，也有基本数据类型,paramN.Property来获得pojo中的值
- 如果参数为List，Set，数组，也会被封装成Map，通过collection[i],array[i],list[i]来获得值，也可以通过@Param来指定key
- 在参数是Map时，括号中为Map的key值
- 在参数是pojo时，括号中为pojo的属性名

## #{}和${}的区别与联系

- #{}和${}的使用方法没有什么区别
- #{}采用JDBC的预编译形式
- ${}是字符串拼接，并且会转义危险的输入值
- ${}通常用于对表名，字段名，排序字段等不能预编译的字段进行占位处理

## 获取自增主键

```
<insert id="addMember" parameterType="User" useGeneratedKeys="true" keyProperty="id" >
```

keyProperty代表将自增主键赋值给pojo的哪个属性

## 自动映射

1. 全局设置autoMappingBehavior,默认为Partial，开启自动映射，null为关闭自动映射
2. a_cloumn -> aColumn符合驼峰命名法，mapUnderscoreToCamelCase=true,开启驼峰映射
3. resultMap进行自定义映射

### 延迟加载

  Mybatis的延迟加载是针对嵌套查询而言的，是指在进行查询的时候先只查询最外层的SQL，对于内层SQL将在需要使用的时候才查询出来

## resultMap

- `<id>:`映射主键;
- `<result>:`映射非主键属性；
- 可以只自定义一部分，剩下的按照规则1,2进行匹配

#### 应用

##### 返回的pojo对象的某些属性类型是另一个pojo

1.使用级联属性

`<result column = "columnName" property = "innerPojo.innerPojoPropertry">`

 columnName是查询出来的列名，innerPojo是outerPojo的属性名，不是类型

2.使用`<association>`标签

- property是outerPojo中innerPojo的属性名

- javaType指定innerPojo对应的类，可以用别名
- 子标签的配置和resultMap一样

```
<association property="dep" javaType="department">
    <id column="did" property="id"/>
    <result column="department" property="department"/>
    <result column="id" property="uid"/>
</association>
```

3.使用association来进行分步查询

1. 先按照员工id查询员工信息
2. 根据查询员工信息中的d_id的值去部门表查出部门信息
3. 部门设置到员工中z
   	   

```
<association property="dep" select="selectDep" column="id"/>
```

- property:查询出来的结果封装到哪个属性

- select:指定分步查询的语句

- column：第一次查询出来的列作为参数传入分步查询

- 传多个参数的时候封装成map,`column = "{key = value}"`

  

##### collection标签定义集合封装规则

```
<collection property="userAll" ofType="user"  fetchType="lazy">
    <id column="id" property="id"/>
    <result column="mail" property="mail"/>
    <result column="password" property="password"/>
</collection>
```

- property:查询出来的结果封装到哪一个属性
- ofType：集合中的元素的类型
- 外层的属性需要是一样的，内层属性不同，才能把内层封装为一个List，外层一样的话会报selectOne错误
- 可以通过select 和 cloumn属性来进行分步查询

## Collection和association总结

- association用于一对一的情况
- 三种使用方式：分步查询，嵌套resultMap（复用版与非复用版）
- collection适用于一对多的情况
- 三种使用方式：分步查询，嵌套resultMap（复用版与非复用版）

视频看到39了