## 验证流程

![alt text](pic/SpringSecurity-AuthorProcess.png)

## 快速流程
### 1. 创建Spring Boot项目并导入依赖
```
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>4.4.0</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.5</version>
        </dependency>

        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>8.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
```

### 2. 修改用户查找功能

2.1. 创建Service实现UserDetailsService接口

```
@Service
public class UserSecurityService implements UserDetailsService {

    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        // 查询用户信息
        QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
        userQueryWrapper.eq("firstname",s);
        User user = userMapper.selectOne(userQueryWrapper);
        // 如果用户不存在，抛出异常
        if(user==null){
            throw new RuntimeException("用户不存在~");
        }
        // TODO: 权限认证

        // 把查询结果封装到UserLogin中
        return new UserLogin(user);

    }
}
```

2.2 由于重载的方法需要返回UserDetails类型，创建UserLogin类来实现UserDetails接口:

```
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UserLogin implements UserDetails {
    User user;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getFirstname();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

此时Spring Security filter chain中用户查找的功能便被覆盖了。

### 3. 密码加密

上述方式返回的UserLogin（UserDetails的实现）类里的密码是明文的，也没有指明加密方式，所以其它filter进行密码匹配的时候会不知道怎么做，所以我们需要创建加密器。

#### 3.1 创建配置类，添加加密器为Bean
```
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public BCryptPasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```
这样Spring会将以BCryptPasswordEncoder为加密器，用它比较：从UserDetails拿到的密码（加密后的）和用户输入的密码（明文的）。


