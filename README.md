# auto-generate-test-maven-plugin 

## 介绍  
你还在为写大量单元测试而烦恼吗，你还在苦苦的构建包装类的值吗？这里有一款mock单元测试代码自动生成的Maven插件，解决开发人员消耗大量时间在单元测试的问题，全面优化开发人员的测试效率和测试时间。   

经过本人的亲自使用，外加我在团队的内部"推（qiang）广（tui）"使用，从统计来看，可以减少使用mock测试的小伙伴，至少30-50%的时间。没减少你的测试时间，请拿刀背砍我。      

注意，本插件目前无法完成所有的mock测试，让你不用修改一行代码，暂时需要在agt生成mock测试代码的基础上再进行一点修改，例如分支覆盖，断言使用。

本插件的目标是：实现让开发人员不再写一行mock测试代码，agt帮你全部实现    
愿景是：减少开发人员的测试负担，专注业务开发与迭代  

简称：AGT     
项目地址： https://github.com/chenhaoxiang/auto-generate-test-maven-plugin 

## 最新版本支持功能
- 支持包下所有类中公共非静态方法生成测试方法   
- 支持配置mock的包，将mock掉包下类的所有方法  
- 支持基础类型和包装类型自动赋值  
- 增加枚举参数的支持  
- 可进行配置需要跳过的参数类型，直接配置包名，会进行跳过包内所有类的构造（用于跳过接口的构造，直接赋值为null） 
- 优化全限定名称为简称，使用import导入包，名称存在重复的类，使用全限定名称  
- 支持配置选择是否自动mock掉父类&自身非测试的方法 - 默认true  
- 支持配置实体基础类型随机设置/使用默认值空值   
    a. 随机 String长度:10位数字与字母，使用JDK UUID进行生成，确保唯一  
    b. 随机 int:[0,1000)  
    c. 随机 byte:[0,1)  
    d. 随机 short:[0,127)   
    e. 随机 double:[0.00,10000.00)   
    f. 随机 float:[0.00f,10000.00f)   
    g. 随机 long:[0L,100000L)   
    g. 随机 char:数字/字母       
- mock注解的类，使用了全限定名称，优化为简称，类进行导入，重复类简称，第一个类使用简称，后面的类使用全限定名称   
- 不再支持配置其他包下的类进行mock，非测试类的所有方法均进行mock，测试类的私有方法也进行mock  
- 已生成测试类，不再进行覆盖生成   
- 支持非当前项目模块第三方类的加载和构造（需要在插件中引入第三方的依赖）   
- v1.1.0版本中对于一些没有setter方法的属性，也进行了set值；期望：对于没有setter的属性值，不进行设置  
- 支持字符串、int、long、布尔类型随机值的范围设置     
- 支持配置生成父类属性的set方法进行设置值，默认true，生成的set方法包含父类的属性(注意，父类如果不在当前项目中，需要在插件中引入包的依赖)     
- 测试类新增方法支持追加生成mock测试方法  
- 支持在不同包下的测试类同时进行生成   
- 初始化下载配置文件不再需要，直接运行生成，自动检测是否下载，未下载先进行下载配置文件再生成  
 

## 使用  

### 1. 首先依赖插件：

#### 版本1.0.0：（注意1.0.0以后的artifactId已经更换）
```xml
<plugin> 
  <groupId>com.uifuture.maven.plugins</groupId>
  <artifactId>auto-generate-test-maven-plugin</artifactId>
  <version>1.0.0</version> 
    <configuration>
        <testPackageName>com.jiaxuan.heaven.book.service.impl</testPackageName> 
    </configuration>
</plugin>
```   

(建议使用最新版本)：目前最新版本1.1.3      
#### 版本1.0.0+： 
（后续版本如无说明，groupId与artifactId不会再进行更换）  

最简单配置：
```xml

<plugin>
    <groupId>com.uifuture.maven.plugins</groupId>
    <artifactId>agt-maven-plugin</artifactId>
    <version>（版本号）</version>
    <configuration>
        <testPackageName>（待测试类的包名，v1.1.3+支持配置多个，英文分号进行隔开）</testPackageName>
    </configuration>
</plugin>
```
（使用最简单的配置，即可使用，感觉后面说明过多的不用往下看）  

示例： 
```xml
<plugin>
    <groupId>com.uifuture.maven.plugins</groupId>
    <artifactId>agt-maven-plugin</artifactId>
    <version>1.1.0</version>
    <configuration>
        <testPackageName>com.jiaxuan.heaven.book.service.impl</testPackageName>
    </configuration>
    <dependencies>
        <!-- 引入包，框架可以利用反射找到类，其他项目中的依赖请按照需要进行引入，本配置仅为示例 --> 
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>4.3.2.RELEASE</version>
        </dependency> 
    </dependencies>
</plugin>
``` 

### 2. 下载配置文件  
在引入插件的项目模块下运行maven插件的agt:init命令    
```
mvn agt:init
```   
（v1.1.3+起不再需要先运行agt:init命令，直接运行agt:test即可下载模板文件&生成测试类） 
agt:init仅仅作为单独下载模板的命令，agt:test包含了agt:init的所有功能  

相关配置：
```<configPath></configPath>```填写路径，相对路径为当前运行项目的根路径。(默认下载路径：/src/main/resources/test/template)   
运行插件的init命令，即可将配置文件下载到对应的路径。  
  
可设置配置文件的文件名，通过```<configFileName>magt.ftl</configFileName>```设置配置文件的文件名称。（默认文件名称为magt.ftl）  

### 3. 引入mock相关依赖  
自动测试代码生成插件   
插件生成的mock测试类方法依赖powermock&mockito，建议引入如下依赖（不依赖对于agt插件的运行没有影响）   
```xml
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <version>1.7.4</version> 
    <scope>test</scope> 
</dependency> 
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito2</artifactId>
    <version>1.7.4</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>2.8.47</version>
    <scope>test</scope> 
</dependency>

``` 

### 4. configuration中相关配置属性  
在引入插件的项目名模块下，运行插件的agt:test命令即可在对应的test路径下生成测试用例。
```
mvn agt:test
```   

#### 必填  
- ```<testPackageName>```:必填）配置需要生成单元测试用例的包名，不要填写接口所在包名，需要实现类所在包名,会遍历包下以及子包下所有类和类的方法（v1.1.2支持配置多个包名，英文分号隔开不同的包名）      

#### 选填 
- ```<configPath>```:下载配置文件的路径  
- ```<author>```:作者名称    
- ```<configFileName>```:下载下来的配置文件的名称   
- ```<isGetChildPackage>```:配置testPackageName的包是否递归获取子包下的类(默认true)    
- ```<isMockThisOtherMethod>```:配置是否mock掉父类以及自身测试类非测试的方法(默认true),父类/本类方法调用需要使用this进行调用，目前版本无法支持mock        
- ```<isSetBasicTypesRandomValue>```:配置是否设置基础类型的值随机生成(默认false)
    
- ```<setStringRandomRange>```:配置字符串随机值的位数（例如："10"，表示10位随机字母/数字字符）    
- ```<setIntRandomRange>```:配置int/Integer类型随机值的范围（例如："0,1000"，表示[0,1000)范围的int数值，配置固定的值可配置为"0",则int值固定为0 ）    
- ```<setLongRandomRange>```:配置long/Long类型随机值的范围(配置规则与setIntRandomRange类似)   
- ```<setBooleanRandomRange>```:配置boolean/Boolean类型随机值的范围（例如：配置为"true"/"false"表示为固定的值，其他任意值表示true和false随机）   

#### 已删除配置 
- ```<childPackage>```:已更名为 ```<isGetChildPackage>```   
- ```<mockPackage>```: ~~需要mock掉的类所在的包名~~ （V1.1.0已删除）   
- ```<skipPackages>```:~~在方法参数初始化的时候，需要进行赋值为null的类（例如，适合接口或者没有空参构造的类），在该包下的类不会进行赋值。可配置多个，使用英文分号隔开。~~（V1.1.0已删除）       
- ```<otherProjectName>```:~~适合多项目，配置其他的项目名称，可以使MAGT-plugin解析到该模块下的所有类。一个项目下多个项目模块。项目模块的路径名称。可配置多个，使用英文分号隔开。~~ （V1.1.0已删除）          

### 5. 配置第三方依赖  
待测试类有依赖第三方Jar包的，需要进行引入插件的依赖，否则agt无法解析到依赖的类信息   
配置位置如下所示：      
```xml
<plugin>
    <!-- ... -->   
    <configuration>
    <!-- ... -->
    </configuration>
    <dependencies>
        <!-- 引入包，框架可以利用反射找到类的所有信息 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus</artifactId>
            <version>3.1.0</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>4.3.2.RELEASE</version>
        </dependency>
        <!-- ... -->
    </dependencies>
</plugin>
```

# 版本功能

## V1.0.0    
### 本次新增/优化功能  
1. 支持包下所有类中公共非静态方法生成测试方法   
2. 支持配置mock的包，将mock掉包下类的所有方法  
3. 支持基础类型和包装类型自动赋值  
4. 增加枚举参数的支持 
5. 可进行配置需要跳过的参数类型，直接配置包名，会进行跳过包内所有类的构造（用于跳过接口的构造，直接赋值为null） 
6. 优化全限定名称为简称，使用import导入包，名称存在重复的类，使用全限定名称  

分支：version/1.0.0  
依赖：
```xml
<groupId>com.uifuture.maven.plugins</groupId>
<artifactId>auto-generate-test-maven-plugin</artifactId>
<version>1.0.0</version> 
```

## V1.1.0  

### 本次新增/优化功能  
1. 支持配置选择是否自动mock掉父类&自身非测试的方法 - 默认true  
2. 支持配置实体基础类型随机设置/使用默认值空值   
    a. 随机 String长度:10位数字与字母，使用JDK UUID进行生成，确保唯一  
    b. 随机 int:[0,1000)  
    c. 随机 byte:[0,1)  
    d. 随机 short:[0,127)   
    e. 随机 double:[0.00,10000.00)   
    f. 随机 float:[0.00f,10000.00f)   
    g. 随机 long:[0L,100000L)   
    g. 随机 char:数字/字母      
3. ~~每个测试类使用统一的before注解进行mock方法~~(考虑到后面每个分支的mock，如果同意进行mock的话，会导致分支无法全面覆盖)   
4. mock注解的类，使用了全限定名称，优化为简称，类进行导入，重复类简称，第一个类使用简称，后面的类使用全限定名称   
5. 不再支持配置其他包下的类进行mock，非测试类的所有方法均进行mock，测试类的私有方法也进行mock  
6. 已生成测试类，不再进行覆盖生成   
7. 支持第三方包类的加载和构造    
 
### 配置更改点  
```<childPackage>```配置属性修改为```<isGetChildPackage>```  

## V1.1.2  

### 本次新增/优化功能   
1. v1.1.0版本中对于一些没有setter方法的属性，也进行了set值；期望：对于没有setter的属性值，不进行设置  
2. 支持字符串、int、long、布尔类型随机值的范围设置     
3. 支持配置生成父类属性的set方法进行设置值，默认true，生成的set方法包含父类的属性(注意，父类如果不在当前项目中，需要在插件中引入包的依赖)     
4. 测试类新增方法支持追加生成mock测试方法  
 
# V1.1.3 
1. 支持在不同包下的测试类同时进行生成   
2. 初始化下载配置文件不再需要，直接运行生成，自动检测是否下载，未下载先进行下载配置文件再生成  



### 配置更改点  
```<testPackageName>``` 支持配置多个包，多个包使用英文分号";"隔开  


## 其他功能排期  
- 支持if-else生成多个mock分支方法   
- 支持简单集合构造赋值  
- 支持配置静态方法mock，需要进行配置静态类的全限定名称(静态方法不建议进行mock)         
- 测试类中的私有方法进行mock，私有方法专门开方法进行生成mock测试，默认不支持，需要配置(私有方法不建议进行mock)   
- 同一个测试方法中存在Mock方法名称重复（参数个数不同）无法进行区分，仅仅对于第一个方法进行mock，且会存在重复mock代码生成；期望：支持同名方法的mock  
- 无法支持重名方法（参数个数相同，参数类型不同），会出现生成的mock方法引用不明确；期望：mock方法引用明确（通过匹配参数类型解决） 
- 不支持嵌套自定义参数的构造；期望：支持多级参数的构造  
- 不支持集合的构造；期望：支持集合的构造  
- mock方法返回值不支持自定义，统一是返回null；期望：支持mock返回值的自定义/生成值   
- 不支持Spring自定义事务管理器DataSourceTransactionManager的mock；期望：支持自定义事务的mock   
- 支持多级参数构造赋值  
- 支持重载方法的mock    
- 支持断言配置    

### 注意 
配置mock静态方法：
默认使用@RunWith(MockitoJUnitRunner.class)，如果配置了mock静态方法，将使用@RunWith(PowerMockRunner.class)。

使用PowerMockRunner与MockitoJUnitRunner类，都无法支持父类中的属性（service的实现类中又同时注入了该类）自动注入的mock（例如mybatis中service层的泛型父类中的泛型baseMapper）。这是由于Mock类会将这两个类作为不同的实例来进行处理，只会mock掉你注入service实现类的基类，而无法注入service实现类的父类中的mapper。  
例如： 
service实现类  
```java

public class WorkFlowServiceImpl extends ServiceImpl<WorkFlowMapper, WorkFlowEntity> implements IWorkFlowService {
    @Autowired
    private WorkFlowMapper workFlowMapper;
    //...
}
```
父类ServiceImpl：
```java
public class ServiceImpl<M extends BaseMapper<T>, T> implements IService<T> {
    @Autowired
    protected M baseMapper;
    //...
}
```
在WorkFlowServiceImpl中使用时：
```java
baseMapper.deleteById("1");
```
在mock测试类中：
```java

@RunWith(PowerMockRunner.class)
@PowerMockIgnore("javax.management.*")
@PrepareForTest({BeanConvertUtil.class, FieldBaseDto.class})
public class WorkFlowServiceImplTest {
    @InjectMocks
    private WorkFlowServiceImpl workFlowServiceImpl;
    @Mock
    private WorkFlowMapper workFlowMapper;
    //...
}
```
无法进行mock掉baseMapper实例，这是由于baseMapper与workFlowMapper并不是同一个实例！这里仅仅只能mock掉workFlowMapper。  


目前可以在service层的实现类中将baseMapper再次注入，则使用PowerMockRunner.class也可以进行mock  

service层的实现类不推荐使用泛型基类service父类进行调用泛型mapper操作数据库层！可以选择注入Mapper再进行调用。  
   
   

# 使用体验  
199个测试方法，一共覆盖309个被测试方法，使用magt生成后，仅仅只使用了3个多小时进行mock优化（仅仅优化了运行报错的方法，没有进行完善分支测试）。     

按照我以前的经验，如果全部由自己写，一切顺利的情况下，199个方法的mock测试，至少要多出几倍的时间。  
（此测试项目为使用mybatis-plus的项目，service层的实现类非常多的方法直接使用了父类方法，导致mock很麻烦，耽搁了一些时间，其他项目相对而言会节省更多时间）      

## 测试类：  
![测试类](https://raw.githubusercontent.com/chenhaoxiang/auto-generate-test-maven-plugin/master/doc/images/20190619221913.jpg)
## 测试方法：
![测试方法](https://raw.githubusercontent.com/chenhaoxiang/auto-generate-test-maven-plugin/master/doc/images/20190619223834.jpg)   
## 单元覆盖数据：  
![总覆盖率](https://raw.githubusercontent.com/chenhaoxiang/auto-generate-test-maven-plugin/master/doc/images/20190620151913.jpg)   
--- 

![单元覆盖数据](https://raw.githubusercontent.com/chenhaoxiang/auto-generate-test-maven-plugin/master/doc/images/20190620151630.jpg)   

目前1.0.0版本不支持分支的覆盖，导致覆盖率不高，将在1.2.0版本后推出分支的多方法mock。大大提高分支覆盖率。      

有任何问题/建议欢迎提<a href="https://github.com/chenhaoxiang/auto-generate-test-maven-plugin/issues">Issues</a>

 
## 说明     

20190703拉了个团队小伙伴一块开发，项目从1.1.3版本开始，进入团队内部开发阶段。感谢您的使用。后期更新，本项目会进行同步更新/提醒。  

建议您可以先star本项目  

