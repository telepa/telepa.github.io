# Spring Boot

> [Spring boot中文文档](https://springdoc.cn/spring-boot/)

## 1 分层

SpringBoot项目大概分为4层

1. Bean层（实体层）：数据库表的映射实体类，存放POJO对象
2. DAO层：包括xxxMapper.java（数据库访问接口类），xxxMapper.xml（数据库链接实现）
3. Service层（服务层）：包括xxxService.java(业务接口类)，xxxServiceImpl.java（业务实现类）（可以在service文件夹下新建impl文件放业务实现类，也可以把业务实现类单独放一个文件夹下，更清晰）
4. Web层（Controller层）：实现与web前端的交互

整个项目主要文件架构如下

```
--src
	--main
		--java
			--com.example.myapp
				--controller
				--pojo
				--DAO
				--service
					--impl
		--resources
			--mapper
			---templates
```

以一个简单的小项目为例，account表中有字段id，user_name与pwd。

（1）实体层

```java
public class account {
    //id,user_name,pwd
    public int id;
    public String user_name;
    public String pwd;
    //id
    public int getId(){return id;}
    public void setId(int myid){this.id=myid;}
    //user_name
    public String getUser_name(){return user_name;}
    public void setUser_name(String a){this.user_name=a;}
    //pwd
    public String getPwd(){return pwd;}
    public void setPwd(String mypwd){this.pwd=mypwd;}
}
```

（2）DAO层（映射层）

```java
@Component
public interface AccountMapper {
    account getInfo(String user_name,String pwd);
    void addAccount(String user_name,String pwd);
}
```

我们还需要添加与mysql数据库的依赖文件，与Mapper（DAO层对应）

```xml
<mapper namespace="com.example.myapp.DAO.AccountMapper">
    <select id="getInfo" parameterType="String" resultType="com.example.myapp.pojo.account">
        SELECT * FROM account where user_name= #{param1} and pwd= #{param2}
    </select>
    <insert id="addAccount" parameterType="String">
        insert into account(id,user_name,pwd) values(0,#{param1},#{param2})
    </insert>
</mapper>
```

（3）Service层（服务层）

AccountServiceImpl

```java
@Repository
@Service
@Component
public class AccountServiceImpl implements AccountService {
    //这里就是需要把这个实现方法具体的写出来了,然后这里就需要加入注解了
    //这就是所谓的自动装配的功能吧
    @Autowired
    private AccountMapper accountMapper;
    @Override
    public account loginIn(String user_name,String pwd){
        return accountMapper.getInfo(user_name,pwd);
    }
    public void Register(String user_name,String pwd){
        accountMapper.addAccount(user_name,pwd);
    }
}
```

AccountService

```java
public interface AccountService {
    public account loginIn(String user_name,String pwd);
    public void Register(String user_name,String pwd);
}
```

（4）Web层（Controller层）

这里的返回值指的是要显示的html页面文件名

```java
public class AccountController {
    @Autowired
    AccountService accountService;
    @RequestMapping("/login")
    public String show(){
        return "login";
    }
    @RequestMapping(value="/loginIn",method = RequestMethod.POST)
    public String login(String user_name,String pwd){
        account myaccount=accountService.loginIn(user_name,pwd);
        if(myaccount!=null){
            return "success";
        }else return "error";
    }
}
```

（5）顶层 MyappApplication.java

```java
@SpringBootApplication(scanBasePackages="com.example.myapp")
@MapperScan("com.example.myapp.DAO")
public class MyappApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyappApplication.class, args);
    }
}
```





!!! tip
	insert失败返回0，成功则返回插入的行数