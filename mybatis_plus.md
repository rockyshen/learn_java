![](https://raw.githubusercontent.com/rockyshen/blog-img/master/images.png)
## 基本概念
Mybatis-plus是半自动[[Mybatis]]（手动写sql语句）向全自动（常规查询操作不用自己写sql语句）迈进的体现。

mybatis_plus 可以在mybatis基础上提供增强，==单表查询==不用去[[Mybatis#XxxMapper.xml]]中写sql语句了。

现实中，单表查询太少了，一般都是多表+多条件+分页！

[MyBatis-Plus官方文档](https://baomidou.com/)

```
<dependency>  
	<groupId>com.baomidou</groupId>  
	<artifactId>mybatis-plus-boot-starter</artifactId>  
	<version>3.5.3.1</version>
<!--            不用到mybatis,已经集成了-->  
</dependency>
```

总结：
- ibatis没有代理对象，简陋
- mybatis有代理对象，加入ioc容器，但自己还要在xml中写sql语句。
- mybatis_plus：
	在Mapper接口上继承了`BaseMapper<T>`，就可以调用一些方法来查询
    在Service接口上继承了`Iservice<T>` + ServiceImpl，可以在Service层调用一些方法来见大查询

## 核心功能
简单总结一句话：

| XX          | XX                                                                                                                    |
| ----------- | --------------------------------------------------------------------------------------------------------------------- |
| Mapper层的增强  | 在Mapper接口上， `extends BaseMapper<T>`                                                                                   |
| Service层的增强 | 在Service接口上，`extends IService<T>`<br>在Service接口实现类上，`extends ServiveImpl<XxxMapper, XxxEntity> implements XxxService` |

### Service CRUD接口
复杂逻辑，走mapper层
简单逻辑，走service层

[Service CRUD接口：常用方法](https://baomidou.com/pages/49cc81/#service-crud-%E6%8E%A5%E5%8F%A3)

Mybatis-plus提供的Service层的CRUD，一半放在IService接口中，一半放在ServiceImpl实现类中！   奇葩！！！
```
Service层的接口中
public interface UserService extends IService<User>{}

Service层的impl/实现类中
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper,User> implements UserService

// ServiceImple<实体类对应的BaseMapper,实体类>
```

Service的CRUD都==以行为动作给方法命名==：

| 描述   | Mapper           | Service |
| ---- | ---------------- | ------- |
| 插入   | insert           | save    |
| 删除   | delete           | remove  |
| 查询单行 | selectById       | get     |
| 查询集合 | selectBatchByIds | list    |
| 分页   |                  | page    |

#### `Iservice<T> + ServiceImpl<T>` 常用方法

@雷丰阳：不用在Service层进行依赖注入userMapper，使用baseMapper泛型指代的就是userMapper，baseMapper就是userMapper，直接baseMapper.selectList()，不用 userMapper.selectList()
原理是：ServiceImpl已经在背后执行了将Dao层Autowired注入到Service层。

```
public class ServiceImpl<M extends BaseMapper<T>, T> implements IService<T> {   
    @Autowired  
    protected M baseMapper;
```

**经验：Service层直接调用baseMapper，不用注入Mapper层！**

##### 增
###### save(T entity)
保存一条数据，一条数据 对应 一个实体类对象实例！

###### saveBatch(集合)
根据集合，批量保存，上限1000条。
返回布尔类型，表示是否成功。

| 类型              | 参数名        | 描述     |
| --------------- | ---------- | ------ |
| `Collection<T>` | entityList | 实体对象集合 |

```
List<User> list = new ArrayList<>();
# 省略构造10个User的过程
boolean b = userService.saveBatch(list);
```

###### saveOrUpdate(user)
如果有ID就修改，否则插入。（可以发现，业务层的方法名，就更偏向业务）
```
User user = new User();
user.setId(1);
user.setName("驴蛋蛋")；
user.setAge(18);

userService.saveOrUpdate(user);
```

##### 删


###### removeById(id)
按ID删除，返回值是布尔类型！
```
boolean b = userService.removeById(1);
```

##### 改
###### update(updateWrapper)
必须携带查询条件（[[#`UpdateWrapper<T>`]]），才能进行更新
要不就用updateById(实体类)

###### updateById(T entity) 
只要传需要更新的实体类就行了，不用传ID，会自动去匹配！
明确知道有就用update
```
User user = new User();
user.setId(1);
user.setName("驴蛋蛋")；
user.setAge(18);

userService.updateById(user);

```

不清楚的情况下用saveOrUpdate

```
// 根据 ID 选择修改
boolean updateById(T entity);
```

###### updateBatchById(T entity)
根据传递的实体类集合，按id去更新实体类中有的字段

一个小技巧：通过创建一个新的PurchaseDetailEntity对象（myDetailEntity），并将原始对象的部分属性复制给新对象。然后，对新对象的status属性进行设置。最后，将新对象添加到collect1列表中。这种方式实现了对原始对象的==浅拷贝==，并对新对象进行修改，不会影响原始对象。
```
List<PurchaseDetailEntity> collect1 = purchaseDetailEntities.stream().map(purchaseDetailEntity -> {  
    PurchaseDetailEntity myDetailEntity = new PurchaseDetailEntity();  
    myDetailEntity.setId(purchaseDetailEntity.getId());  
    myDetailEntity.setStatus(WareConstant.PurchaseDetailStatusEnum.BUYING.getCode());  
    return myDetailEntity;  
}).collect(Collectors.toList());  
purchaseDetailService.updateBatchById(collect1);
```
##### 查
###### getById(id)
查询单个
###### getOne(queryWrapper)
带上查询条件，查询一个
###### getMap(queryMapper)
根据查询条件，查询，并封装进Map
###### list(queryWrapper)
==queryWrapper = null  就是查询全部！==
查询全部，带wrapper条件,返回集合
	联想到：Mapper层的 selectList(null)，参数传递null，就是查询全部！
###### page(Ipage`<T>`page，queryWrapper)
**步骤1**：配置类中引入mybatis_plus的分页插件：[[#配置类引入分页插件]]

**步骤2**：传递参数
- 参数1：page对象，封装了current、limit等参数
	Ipage是一个接口，实际传入的是实现类Page。
	Ipage是mybatis-plus约定的接口，Page是Ipage接口的实现类！
```
public interface IPage<T> extends Serializable
```
- 参数2：wrapper

步骤3：从先前的page对象中获取结果。
```
@ApiOperation(value = "条件查询分页")  
@GetMapping("findQueryPage/{current}/{limit}")  
public Result findPage(@PathVariable Long current,  
                       @PathVariable Long limit,  
                       TeacherQueryVo teacherQueryVo){  
                       
    IPage<Teacher> page = new Page<>(current,limit);  
  
    if(teacherQueryVo == null){  
        // wrapper=null，查询全部
        // 分页结果还在原先的page对象中
        IPage<Teacher> page = teacherService.page(page,null);  
        return Result.ok(page);
```

| 方法名              | 备注      | 别名        |
| ---------------- | ------- | --------- |
| page.getCurrent; | 当前页码    | pageNum   |
| page.getSize     | 每页显示数据量 | pageSize  |
| page.getRecord   | 数据集     | data      |
| page.getTotal    | 总记录数    | totalSize |
| page.getPages    | 总页数     | totalPage |

### Mapper CRUD接口
继承`BaseMapper<T>`   默认将实体类名当做表名去查找！

[Mapper CRUD接口：常用方法介绍](https://baomidou.com/pages/49cc81/#mapper-crud-%E6%8E%A5%E5%8F%A3)
#### `BaseMapper<T>` 常用方法
==mapper层的方法，以数据库动作命名！==

思考：之所以`BaseMapper<T>`能推断出具体什么类型，源码层面使用的是反射机制！
![[Java_03_高级特性#^445344]]

**说明**：
BaseMapper插入数据，ID自增，默认使用雪花算法，ID为Long类型
![[#^aacf1b]]

增、删、改所有方法的返回值都是int rows，也就是受影响了几行！
##### 增

###### insert
插入一条记录

| 类型  | 参数名    | 描述   |
| --- | ------ | ---- |
| T   | entity | 实体对象 |

```
User user = new User();  
user.setName("Rocky");  
user.setAge(29);  
user.setEmail("rockyshenjunjie@gmail.com");  
int rows = userMapper.insert(user);     // 受影响行数！
```

注意：
Mapper层，不提供批量添加的方法；Service层，提供了批量添加的方法  [[#saveBatch(集合)]]
##### 删

###### deleteById
根据Id删除

| 类型           | 参数名 | 描述    |
| ------------ | --- | ----- |
| Serializable | id  | 主键 ID |
```
// 根据id删除  
int rows = userMapper.deleteById(1768163869118865410L);
```

###### deleteByMap
根据Map删除

| 类型                  | 参数名       | 描述       |
| ------------------- | --------- | -------- |
| Map<String, Object> | columnMap | 表字段， 表的值 |

```
// 名字是张三，年龄为20的行，删除  
Map param = new HashMap();
param.put("name","zhangsan");
param.put("age","20");  
int rows = userMapper.deleteByMap(param);
```

###### delete(`wrapper<T>`)
根据条件构造器删除

###### deleteBatchIds(集合)
根据传入的id集合，批量删除行

| 类型                                   | 参数名    | 描述                          |
| ------------------------------------ | ------ | --------------------------- |
| `Collection<? extends Serializable>` | idList | 主键 ID 列表(不能为 null 以及 empty) |
```
List<Long> list = Arrays.asList(1L,2L,3L);
int rows = userMapper.deleteBatchIds(list);
```

##### 改

###### updateById(entity)
传入一个实体对象，更新这个对象！

==**注意点**==：
1、传递一个实体，会自动读实体的ID，去根据ID更新对应的条目！
2、实体类的属性要设置成包装类型，不能设置成基元类型，因为int的默认值是0，会把所有int的列改为默认值0。包装类型的默认值是null，就不会改！

| 类型  | 参数名    | 描述                            |
| --- | ------ | ----------------------------- |
| T   | entity | 传递一个实体，会自动读实体的ID，去根据ID更新对应的条目 |

```
//根据ID修改，主键属性必须有值  
//将User id = 1的age改为30  
User user = new User();  
user.setId(1L);  
user.setAge(30);        // 当属性没有被设置的时候，是不会修改的
int i = userMapper.updateById(user);
```


###### update(entity, updateWrapper)
根据条件构造器，更新记录。

| 类型           | 参数名           | 描述                     |
| ------------ | ------------- | ---------------------- |
| T            | entity        | 实体对象 (set 条件值,可为 null) |
| `Wrapper<T>` | updateWrapper | 条件构造器                  |

```
userMapper.update
```

实体对象 (set 条件值,可为 null)
加入我不想更新实体类对象的全部字段。我们就new一个对象，然后set需要更新的Field。
传到Update方法中，就只会更新实体指定的Filed/Column

例如下例：
更新字段：BrandId 和 BrandName，其他字段都不更新
更新条件：传入的brand_id 等于 数据库中brand_id时，更新
```
@Override  
public void updateBrand(Long brandId, String name) {  
    /**来自：BrandServiceImpl的调用  
     * 更新pms_brand这个表，且修改了brandName时：  
     * 在pms_brand保存brandName的同时，pms_category_brand_relation中的brandName同步保存；  
     */  
  
    CategoryBrandRelationEntity relationEntity = new CategoryBrandRelationEntity();  
    relationEntity.setBrandId(brandId);  
    relationEntity.setBrandName(name);  
  
    UpdateWrapper<CategoryBrandRelationEntity> updateWrapper = new UpdateWrapper<>();  
    updateWrapper.eq("brand_id",brandId);  
  
    this.update(relationEntity,updateWrapper);  
}
```

##### 查
###### selectById
根据ID查询当前项目的用户信息

| 类型           | 参数名 | 描述   |
| ------------ | --- | ---- |
| Serializable | id  | 主键ID |

```
User user = userMapper.selectById(1);
```


###### selectBatchIds(集合)
根据传入的id集合，批量查询

```
List<Long> ids = Arrays.asList(1L,2L,3L);
List<User> users = userMapper.selectBatchIds(ids);
users.forEach(System.out::println);
```


###### selectByMap
根据Map查询

| 类型                  | 参数名       | 描述     |
| ------------------- | --------- | ------ |
| Map<String, Object> | columnMap | 列名，对应值 |
```
Map<String,Object> map = new HashMap<>();
map.put("name","jack");
map.put("age",20);
List<User> users = userMapper.selectByMap(map);
```

###### selectList(queryWrapper)
根据wrapper条件，查询记录；==设为null，则为查询全部==

| 类型           | 参数名          | 描述                  |
| ------------ | ------------ | ------------------- |
| `Wrapper<T>` | queryWrapper | 实体对象封装操作类（可以为 null） |

`List<User> users = userMapper.selectList(null);`         无条件，查询全部

###### selectPage(Ipage`<T>`page，queryWrapper)
**步骤1**：配置类中引入mybatis_plus的分页插件：[[#配置类引入分页插件]]

**步骤2**：传递参数
参数1：page对象，封装了current、limit等参数
	Ipage是一个接口，实际传入的是实现类Page。
	Ipage是mybatis-plus约定的接口，Page是Ipage接口的实现类！
```
public interface IPage<T> extends Serializable
```
参数2：wrapper

**步骤3**：从先前的page对象中获取结果。

示例
```
test测试类中

@Autowired
private UserMapper userMapper;

@Test
public void test_page(){

	// 构造page对象时，提供current、limit参数
	Page<User> page = new Page<>(1,5);

	// wrapper为null，查询全部
	userMapper.selectPage(page, null);

	// 查询完的分页结果，继续封装会先前的page对象中
	page.getCurrent;         当前页码
	page.getSize;            每页显示数据量
	page.getRecord();        数据集
	page.getTotal;           总记录数
}
```

| 方法名              | 备注      | 别名        |
| ---------------- | ------- | --------- |
| page.getCurrent; | 当前页码    | pageNum   |
| page.getSize     | 每页显示数据量 | pageSize  |
| page.getRecord   | 数据集     | data      |
| page.getTotal    | 总记录数    | totalSize |
| page.getPages    | 总页数     | totalPage |

********************************
### 分页查询
适用于Mapper和Service中
#### 基本步骤
![](https://raw.githubusercontent.com/rockyshen/blog-img/master/Scannable%E6%96%87%E6%A1%A3%E5%88%9B%E5%BB%BA%E4%BA%8E2024%E5%B9%B45%E6%9C%8816%E6%97%A5%2010_55_37.PNG)

1、配置类中，加入分页插件
2、在业务类中，先new一个Page对象，提供参数一：current，参数二：size（或limit）
4、在业务类中
	 Service层，调page(page对象，wrapper)
	Mapper层，调selectPage()方法。参数一：Page；参数2:Wrapper
5、结果封装回page对象中。
	或`IPage<T> pageModel = teacherService.page(page,null);`

##### Page对象
待补充！

#### 配置类引入分页插件
联想到mybatis使用的[[Mybatis#PageHelper插件]]：
![[Mybatis#^d06214]]

mybatis_plus  3.4之前，用旧版
mybatis_plus  3.4之后，用新版
##### 旧版：PaginationInterceptor()
ref：[详解MybatisPlus3.4版本之后分页插件的使用](https://www.finclip.com/news/f/36331.html)
3.4.0之前的旧版，使用分页插件的方法，PaginationInterceptor已经废弃了。雷神的谷粒商城在用！

配置类中，加入分页插件：PaginationInterceptor
```
@Configuration  
@MapperScan("com.atguigu.gulimall.product.dao")  
public class MybatisConfig {  
    @Bean  
    public PaginationInterceptor paginationInterceptor(){  
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();  
        //设置请求的页面大于最大页后的操作，true回到首页；false继续请求  
        paginationInterceptor.setOverflow(true);  
          
        // 设置最大单页限制数量，默认500条，-1不受限制  
        paginationInterceptor.setLimit(500);  
        return paginationInterceptor;  
    }  
}
```

new + return + @Bean -> mybatisPlusInterceptor加入ioc容器。
	mybatisPlusInterceptor调用addInnerInterceptor方法，new一个分页器：PaginationInnerInterceptor，传入数据库类型

##### 新版：MybatisPlusInterceptor()
MybatisPlusInterceptor -> addInnerInterceptor(new PaginationInnerInterceptor(Dbtype.MYSQL))

1、mybatis-plus提供了一个插件集合（mybatisPlusInterceptor），放到一个类中，@Bean加入ioc容器， 加入分页的拦截器：PaginationInnerInterceptor
```
Main主类中

@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor(){
	MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
	interceptor.addInnerInterceptor(new PaginationInnerInterceptor(Dbtype.MYSQL))
	return interceptor;
}
```

#### Service层的分页方法 -> page()
[[#page(Ipage`<T>`page，queryWrapper)]]

| 方法名              | 备注      | 别名        |
| ---------------- | ------- | --------- |
| page.getCurrent; | 当前页码    | pageNum   |
| page.getSize     | 每页显示数据量 | pageSize  |
| page.getRecord   | 数据集     | data      |
| page.getTotal    | 总记录数    | totalSize |
| page.getPages    | 总页数     | totalPage |

#### Mapper层的分页方法 -> selectPage()
[[#selectPage(Ipage`<T>`page，queryWrapper)]]

#### 自定义
selectMyPage(Ipage, XXX)
我们自定义的分页方法，但满足要求：
	1、返回数据类型写`Ipage<T>`
	2、第一个参数传Ipage的实现类即可。
	3、建议第二个参数加上@Param注解，帮助mybaits定位

1、在XxxMapper接口中，声明一个抽象方法
```
public interface HeadlineMapper extends BaseMapper<Headline> {  
	// XxxMapper接口中，声明抽象方法
	// 返回类型写Ipage
	// 第一个参数传Ipage
    IPage<Map> selectMyPage(IPage page, PortalVo portalVo);
```

2、在mapper.xml中写自定义查询语句
```
<!--    自定义了一个分页查询方法-->  
    <select id="selectMyPage" resultType="map">  
        select hid,title,type,page_views pageViews,TIMESTAMPDIFF(HOUR, create_time,NOW()) pastHours,  
               publisher from news_headline where is_deleted=0        <if test="portalVo.keyWords != null and portalVo.keyWords.length()>0">  
            and title like concat('%', #{portalVo.keyWords}, '%')  
        </if>  
        <if test="portalVo.type != 0">  
            and type = #{portalVo.type}  
        </if>  
    </select>
```

3、业务方法中使用即可，结果在page中
```
IPage<Map> page = new Page<>(当前页，页容量);  
headlineMapper.selectMyPage(page,portalVo);  
List<Map> records =  page.getRecords();
```


### 条件构造器 wrapper
**==这个wrapper很实用的！一定要牢牢掌握==**
复杂的条件，全部封装在wrapper对象中
[条件构造器-参考文档](https://baomidou.com/guides/wrapper/)
补图
#### 普通条件构造器
Mapper和Service提供的CRUD方法，很多时候可以传wrapper对象，自定义查询条件。
```
1. new一个条件构造器，把复杂条件写进去
QueryWrapper<Xxx> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("brand_id",key).or().like("brand_name",key);

2. 在方法中传入wrapper，就会根据复杂条件查出结果，结果封装在list集合中
List<Xxx> list = userMapper.selectList(queryWrapper);

3.调用集合的forEach方法，方法引用，遍历每一个
list.forEach(System.out::println)
```

##### `QueryWrapper<T>`
拼接“查询、修改、删除”条件的时候用
```
查询用户名包含a，年龄在20-30之间，并且邮箱不为空的用户信息

QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.like("name","a");
queryWrapper.between("age","20","30");
queryWrapper.isNotNull("email");

List<User> users = userMapper.selectList(queryWrapper);
```

| 常用方法   | 参数             | 备注                          |
| ------ | -------------- | --------------------------- |
| eq()   | ("column", 变量) | 等于，equal                    |
| gt()   |                | 大于，greate than              |
| ge()   |                | 大于等于，greate equal           |
| lt()   |                | 小于，less than                |
| le()   |                | 小于等于，less equal             |
| like() |                |                             |
| or()   |                | 条件的链式调用默认是and。想要或，调用`.or()` |
| and()  |                |                             |

默认查询是返回全部列，除非调用`queryWrapper.select("column")`方法，指定返回列

条件判断，当条件满足时，才拼接SQL！condition
```
//前端传入两个参数，name age
//当name不为空时，作为条件查询
//当age>18时，作为条件查询

QueryWrapper<User> queryWrapper = new QueryWrapper<>();

实现1：
if (StringUtils.isNotBlank(name)) {
	queryWrapper.eq("name",name);
}
if (age!=null && age >18) {
	queryWrapper.eq("age",age);
}

List<User> users = userMapper.selectList(queryWrapper);

实现2：
gt(boolean condition, R column, Object value)
只有当参数1condition表达式为true时，才会执行后面的！

queryWrapper.eq(StringUtils.isNotBlank(name),"name",name);
queryWrapper.eq(age!=null && age >18,"age",age);

```
###### eq()
mysql中catelog_id列，等于变量catelogId
```
QueryWrapper<AttrEntity> queryWrapper = new QueryWrapper<>();  

// 如果没有传catelogId,就是查全部  
if(catelogId != 0){  
	queryWrapper.eq("catelog_id",catelogId);  
}  
```

###### and(Consumer函数式接口)
在原先queryWrapper基础上，追加条件！
```
queryWrapper...         // 之前已经声明的一个queryWrapper

// 在原先queryWrapper基础上，追加条件！
queryWrapper.and( w -> {  
	w.eq("attr_id",key).or().like("attr_name",key);  
});  
```



##### `UpdateWrapper<T>`
基于条件，更新数据库中的数据！

将构建好的updateWrapper对象，传递给[[#update(updateWrapper)]]

queryWrapper里面只放条件，且列名不能设置为null；
updateWrapper里面可以放条件，也可以用set方法修改列名对应的值

```
1.找到年龄大于20，名字包含“张”，或者email为空的数据库行；
2.将满足上述条件的行，执行更新：将email设置为null，年龄统一改为99.

UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();

updateWrapper.gt("age",20).like("name","张").or().isNull("email")
			.set("email",null)
			.set("age",99)

int row = userMapper.update(null,updateWrapper);   //执行按条件更新！,返回值是int类型的影响行数

null表示，不涉及具体的某一个实体对象，而是所有满足查询条件的全部实体对象
```

|     | 常用方法   | 参数             | 备注               |
| --- | ------ | -------------- | ---------------- |
| 条件  | eq     | ("column", 变量) |                  |
|     | gt     |                |                  |
|     | ge     |                |                  |
|     | lt     |                |                  |
|     | le     |                |                  |
|     | like   |                |                  |
|     | or     |                |                  |
| 更新  | setSql | (String sql)   | 设置更新语句中SET部分的SQL |
setSql方法示例
```
UpdateWrapper<UserInterfaceInfo>updateWrapper = new UpdateWrapper<>();  
updateWrapper.eq("interfaceInfoId",interfaceInfoId);  
updateWrapper.eq("userId",userId);  
updateWrapper.setSql("leftNum = leftNum -1, totalNum = totalNum + 1");


根据上述代码，转为SQL语句
UPDATE table_name 
SET leftNum = leftNum - 1, totalNum = totalNum + 1 
WHERE interfaceInfoId = 'interfaceInfoId' AND userId = 'userId';
```

#### Lambda条件构造器
避免列名写错的方式   User::getUserName，如果没有这个列就会报错！
##### `LambdaUpdateWrapper<T>`
拼接“修改”条件的时候用
##### `LambdaQueryWrapper<T>`
拼接“查询、修改、删除”条件的时候用

注意：new的时候一定要加泛型！
```
LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.eq(User::getUsername,user.getUsername());
```

原先手动写列名，现在换成`User::getName`引用属性值，避免写错
```
LambdaQueryWrapper.eq(User::getName,"John");
```

### 核心注解
全局属性，下文所有注解都可以在yaml中指定全局生效，不用一个个去注解
```
mybatis-plus:
	global-config:
		db-config:
			table-prefix: t_     # 默认表名都自动加上前缀
			id-type: auto        # 全局主键都设置自增长
		logic-deleted-field: deleted     # 设置逻辑删除的约定
		logic-delete-value: 1            # 默认
		logic-not-delete-value: 0        # 默认
```

#### @TableName()
如果不指定的话，mybatis_plus会把实体类名当作表名（忽略大小写）去查找（加入设置了prefix前缀就拼接上） User实体类 --> user表

当数据库表名与实体类命名不同时（忽略大小写），需要使用@TableName指定表名！

全局追加前缀
```
mybatis-plus:
	global-config:
		db-config:
			table-prefix: t_     # 默认表名都自动加上前缀

```

#### @TableId()
@TableId("主键列名", type=组件策略)
```
@TableName("t_user")
public class User{
	@TableId(value="t_id"， type = IdType.AUTO)
	private Long id;
	private String name;
	private Integer age;
	private String email;
}
```

IdType属性值
- AUTO             mybaits-plus自动给ID自长，前提是数据库设置了ID自增长 Auto Increment
- ASSIGN_ID    ==默认==，mybatis_plus帮我们分配一个ID，基于雪花算法，随机生成Long类型的数字
	前提是mysql设置主键为 BIG INT或 VARCHAR(64)   pojo用Long 或 String ^aacf1b
- INPUT          表示主键的值需要手动自己输入。例如由前端传入，而不是由数据库自动生成（@雷神的谷粒商城）
	![[mysql.DQL#^b5bb51]]![[mysql.DQL#^214725]]
	类似，UUID：随机生成不重复的字符串

雪花算法，适合巨量数据，平行分表，ID不用自增长。大型分布式
##### @TableField
非主键的Filed列。

@TableField(value, exist)
	value     数据库中的列名
	exist      设置为false，表示这个Field不是mysql的一个列，不会执行查询

雷丰阳的谷粒商城例子中，如果我们需要三级目录，需要额外字段进行辅助，我们可以声明一个Field，并注解为exist=False，就不会参与mybatis的映射！

```

//这个字段不为空的时候才包含到json中，级联菜单有用！  
@JsonInclude(JsonInclude.Include.NON_EMPTY)  
// 声明这个不用映射mysql的表单，是我们代码需要用的  
@TableField(exist = false)  
private List<CategoryEntity> children;
```

##### @JsonInclude(JsonInclude.Include.NON_EMPTY)
加上这个注解，当这个字段不为空时，才会被加进去。

```
CategoryEntity

@JsonInclude(JsonInclude.Include.NON_EMPTY)  //这个字段不为空的时候才包含到json中，级联菜单有用！  
@TableField(exist = false)                  // 声明这个字段数据库表不存在，不用映射mysql的表单，是我们代码需要用的  
private List<CategoryEntity> children;
```

![](https://picgo-rockyshen.oss-cn-shanghai.aliyuncs.com/picgo/202409061303964.png)

##### @JsonIgnore
加上这个注解，这个字段就不会被传递到页面上去（适用于加到：逻辑删除、乐观锁的字段上）

## 高级拓展
### 逻辑删除  @TableLogic
物理删除，就是真的从mysql中删除数据
逻辑删除，mybatis-plus将这个数据标注为“已删除”状态，后续都差不到了，但是mysql中还有。


#### 全局配置
配置全局逻辑删除属性
全局指定：logic-delete-field: isDeleted
```
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted    # 全局逻辑删除字段名
      logic-delete-value: 1          # 逻辑已删除值
      logic-not-delete-value: 0      # 逻辑未删除值
```

#### 单个实体类
步骤 2: 在单个实体类中指定。使用 `@TableLogic` 注解
mybatis-plus中约定，已删除为1，未删除为0
```
@TableName("t_user")
public class User{
	@Tabled(value="t_id"， type = IdType.AUTO)
	private Long id;
	private String name;
	private Integer age;
	private String email;

	@TableLogic
	private Integer deleted;     # 数据库的字段也是deleted
}
```

实体类中的任何==**布尔类型的变量**==，都不要加is前缀（例如isDeleted)，否则部分框架解析时会引起序列化错误。
另外，在Mysql建表规约中，表达“是否的值”采用is_xxx的命名方式。所以需要在`<resultMap>`设置从is_xxx到xxx的映射关系。 #Java编程规约  ^6a170c


### 乐观锁 @Version   OptimistickerInnerInterceptor
[[并发与并行#乐观锁]]

mybatis_plus使用乐观锁机制 -> 版本号机制
实现流程：
1、每条数据添加一个版本号字段 version；
2、取出数据的时候，获取当前version；
3、更新时，检查版本号和数据库中是否一致； ^111199
- 如果一致，证明没有人修改过数据，可以执行更新
- 如果不一致，证明有人已经修改了，更新失败！
联想到：[[mysql.DML#MVCC 多版本并发控制]]

第一步：添加拦截器
```
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor(){
	MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
	
	//分页插件
	mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor(Dbtype.MYSQL))
	
	//乐观锁
	mybatisPlusInterceptor.addInnerInterceptor(new OptimistickerInnerInterceptor());
	
	return interceptor;
}
```

第二步：乐观锁字段添加@Version注解，数据库中也要加列名：`ALTER TABLE user ADD version INT DEFAULT 1;`
```
@Version
private Integer version;
```

第三步：正常使用即可
mybatis_plus自动

乐观锁没有全局设置，只能加@Version注解！

### 防止全表删除和更新  BlockAttackInterceptor
一种防御机制
```
    //配置mybatis-plus拦截器插件  
@Bean  
public MybatisPlusInterceptor mybatisPlusInterceptor(){  
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();  
  
    //分页插件  
    interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));  
  
    //乐观锁  
    interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());  
  
    //防止全表删除或更新  
    interceptor.addInnerInterceptor(new BlockAttackInnerInterceptor());  
  
    return interceptor;  
    }
```

## [[SpringBoot#mybatis-plus的SpringBoot配置]]


## 代码生成器 
### ~~CodeGet~~
[Mybatis_Plus提供的代码生成器](https://baomidou.com/guides/code-generator/)
@尚硅谷-王泽老师进行了修改，简化！拿来即用
放在第一个模块的Test里面就行了，放哪里无所谓，能运行即可。
1、修改输出路径，也即IDEA项目路径
2、数据库修改路径、表名、账号密码
3、包配置->模块名
4、运行即可生成到对应位置。

```
public class CodeGet {  
    /*  
    代码生成器！  
     */  
     
    public static void main(String[] args) {  
        // 1、创建代码生成器  
        AutoGenerator mpg = new AutoGenerator();  
  
        // 2、全局配置  
        // 全局配置  
        GlobalConfig gc = new GlobalConfig();  
        String projectPath = System.getProperty("user.dir");  
        //gc.setOutputDir(projectPath + "/src/main/java");  
        gc.setOutputDir("/Users/shen/Nutstore Files/Learn_Java/IdeaProjects/ggkt_parent/service/service_wechat"+"/src/main/java");  
        gc.setServiceName("%sService"); //去掉Service接口的首字母I  
        gc.setAuthor("atguigu");  
        gc.setOpen(false);  
        mpg.setGlobalConfig(gc);  
  
        // 3、数据源配置  
        DataSourceConfig dsc = new DataSourceConfig();  
        dsc.setUrl("jdbc:mysql://118.25.46.207:3306/glkt_wechat?connectionCollation=utf8mb4_general_ci&useSSL=false");  
        dsc.setDriverName("com.mysql.jdbc.Driver");  
        dsc.setUsername("root");  
        dsc.setPassword("AmRI3@n2");  
        dsc.setDbType(DbType.MYSQL);  
        mpg.setDataSource(dsc);  
  
        // 4、包配置  
        PackageConfig pc = new PackageConfig();  
        pc.setParent("com.atguigu.ggkt");  
        pc.setModuleName("wechat"); //模块名  
  
        pc.setController("controller");  
        pc.setEntity("entity");  
        pc.setService("service");  
        pc.setMapper("mapper");  
        mpg.setPackageInfo(pc);  
  
        // 5、策略配置  
        StrategyConfig strategy = new StrategyConfig();  
        strategy.setInclude("menu");  
        strategy.setNaming(NamingStrategy.underline_to_camel);//数据库表映射到实体的命名策略  
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);//数据库表字段映射到实体的命名策略  
        strategy.setEntityLombokModel(true); // lombok 模型 @Accessors(chain = true) setter链式操作  
        strategy.setRestControllerStyle(true); //restful api风格控制器  
        strategy.setControllerMappingHyphenStyle(true); //url中驼峰转连字符  
        mpg.setStrategy(strategy);  
  
        // 6、执行  
        mpg.execute();  
    }  
}
```

### [[Mybatis#MybatisX插件]]

**********************
**这玩意，用不太上！**

Mapper 4

依赖！
```
<dependency>
   <groupId>tk.mybatis</groupId>
   <artifactId>mapper</artifactId>
   <version>4.2.3</version>
</dependency>
```

[Mapper使用指南](https://mapper.mybatis.io/docs/1.getting-started.html#%E4%BB%8B%E7%BB%8D)

1、编制`config.properties`（含mysql连接账号密码）；`generatorConfig.properties` 基于连接mysql的信息，自动生成PO（entities）和Mapper

2、Maven-Plugin-双击：mybatis-generator:generate，就可以自动生成实体类和Mapper

3、XxxMapper接口去继承`Mapper<T>`   联想mybatis_plus继承的是：[[#`BaseMapper<T>` 常用方法]]
```
public interface PayMapper extends Mapper<Pay>
```

4、主启动上加注解@MapperScan("com.atguigu.cloud.mapper")将接口扫描到ioc容器中。
注意：千万别导错包：`import tk.mybatis.spring.annotation.MapperScan`

易混淆：`import org.mybatis.spring.annotation.MapperScan;`这个包是mybatis的包扫描注解
常用接口
 `Mapper<T>`

常用方法

insertSelective(T)
保存一个实体，null的属性不会保存，会使用数据库默认值。
例如数据库中有逻辑删除、乐观锁字段，增加的时候就会存默认值，而不是存null！有用👍

insert(T)
null的属性也会保存，不会使用数据库默认值！

deleteByPrimaryKey(id)
按主键删除

deleteByExample
按Example封装的条件是删除，类似Wrapper

updateByPrimaryKeySelective(T)
根据id更新，注意传一个实体，会自动读主键！（mybatis_plus也是这样！）
null的属性不会保存，会使用数据库默认值。

常用注解
逆向工程  代码生成器
配合：mybatis-generator插件实现！
分页插件，借助pageHelper


**另一款轻量级的依赖：tk.mybatis**
对比mybatis_plus：都是对mybatis的增强，简化单表CRUD；
mybatis_plus，提供更加复杂的查询操作。

|      | mybatis_plus               | tk.mybatis          |
| ---- | -------------------------- | ------------------- |
| 通用接口 | BaseMapper                 | `Mapper<T>`         |
| 逆向工程 | MybatisX                   | mybatis-generator插件 |
| 分页   | selectPage(Ipage, wrapper) | pageHelper          |

