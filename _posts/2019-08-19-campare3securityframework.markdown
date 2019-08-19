---
layout: post
title: 比较 apache shiro、spring security和spring reactive scurity
date: 2019-08-19 12:00:00 +0800
tags: 技术 java
---

## 序

apache shiro 和 spring security 都是常用的java下的安全框架，其实在以前的项目中也都用过这两个框架。
但是由于之前的项目，大多数处于一个修改者和功能扩展者的角度，所以其实用的比较多，对这两个安全框架的设计，构件都不太熟。
而且也没有从头配过一个安全框架，再加上，发现基于webflux的reactive scurity 虽然在设计思路上和spring security上比较像，
但是细节上还是有很大的不同，于是就基于这三种（其实是两种）安全框架，写了三个不同的项目来对比三种安全框架下，配置的异同。

以下是三个项目在GitHub下的代码

[spring reactive scurity](w)

[spring scurity](s)

[apache shiro](sh)

## apache shiro

### 基本概念

#### shiro中的四个重点（keypooint）

- Authentication 认证

- Authorization 授权

- Cryptography 加解密

- Session Management 会话管理

#### shiro的核心概念

- Subject 主体

- SecurityManager

- Realms 界限

### 项目介绍

项目除了使用了apache shiro 做安全框架之外还使用了以下技术：

- H2 内嵌数据库
- spring data jpa ORM框架
- Lombok java简化框架
- spring mvc MVC框架
- spring boot

### 项目结构

```bash
.
|-- README.md
|-- build.gradle
|-- gradlew
|-- gradlew.bat
|-- settings.gradle
`-- src
    |-- main
    |   |-- java
    |   |   `-- group
    |   |       `-- gamelife
    |   |           `-- shiro
    |   |               `-- demo
    |   |                   |-- AppApplication.java
    |   |                   |-- config
    |   |                   |   |-- CustomRealm.java
    |   |                   |   `-- ShiroConfig.java
    |   |                   |-- entity
    |   |                   |   |-- Operator.java
    |   |                   |   `-- Role.java
    |   |                   |-- repository
    |   |                   |   |-- OperatorRepository.java
    |   |                   |   `-- RoleRepository.java
    |   |                   |-- service
    |   |                   |   `-- OperatorService.java
    |   |                   `-- web
    |   |                       |-- AuthController.java
    |   |                       `-- WebResource.java
    |   `-- resources
    |       |-- application.yml
    |       |-- import.sql
    |       |-- static
    |       `-- templates
    `-- test
        `-- java
            |-- group
            |   `-- gamelife
            |       `-- shiro
            |           `-- demo
            |               |-- AppApplicationTests.java
            |               `-- MethodTest.java
            `-- resources
```

### 关键步骤

1. 继承抽象类AuthenticatingRealm，重写相应的函数（doGetAuthenticationInfo）

    ```java
    public class CustomRealm extends AuthenticatingRealm {

    CustomRealm(OperatorService operatorService) {
        this.operatorService = operatorService;
    }

    private OperatorService operatorService;

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token)
        throws AuthenticationException {

        log.info("----------authentication beginning----------");
        boolean result =
            operatorService.userAuth((String) token.getPrincipal(), (char[]) token.getCredentials());
        log.info("----------authentication check finish----------");
        if (result) {
        return new SimpleAuthenticationInfo(
            token.getPrincipal(), token.getCredentials(), (String) token.getPrincipal());
        }
        // return new
        // SimpleAuthenticationInfo(token.getPrincipal(),token.getCredentials(),"withoutAuth");
        return null;
        }
    }
    ```

2. 编写ORM层相关的类和对映的查询函数

    - entity
    - repository

3. 编写服务层

4. 编写配置类ShiroConfig

    ```java
    @Configuration
    public class ShiroConfig {

    private final OperatorService operatorService;

    @Autowired
    public ShiroConfig(OperatorService operatorService) {
        this.operatorService = operatorService;
    }

    @Bean
    public AuthenticatingRealm customRealm() {
        return new CustomRealm(operatorService);
    }

    @Bean
    public DefaultWebSecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(customRealm());
        return securityManager;
    }

    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        shiroFilterFactoryBean.setLoginUrl("/login");
        shiroFilterFactoryBean.setSuccessUrl("/index");
        shiroFilterFactoryBean.setUnauthorizedUrl("/login");

        Map<String, String> map = new LinkedHashMap<>();
        map.put("/static/**", "anon");
        map.put("/login", "anon");
        map.put("/auth/**", "anon");
        map.put("/favicon.ico", "anon");
        map.put("/h2", "anon");
        map.put("/h2/**", "anon");
        map.put("/**", "authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(map);

        return shiroFilterFactoryBean;
    }
    }
    ```

5. 编写相应的web层接口或者页面
    - AuthController
    - WebResource

### 总结

关键就是实现realm的抽象类，并且通过配置类使用自定义的realm，用自定义的realm配置securityManager实例，把securityManager注入到ShiroFilter实例中，并且配置ShiroFilter，来在webapp中使用shiro框架。

ShiroFilterFactoryBean是ShiroFilterFactory工厂类的配置类，通过修改这个bean来实现修改ShiroFilter的配置。

核心的思路就是，自定义相关的realm，自定义相关的securityManager，自定义相关的ShiroFilterFactory工厂类的配置，来实现shiro框架的各层面的自定义。

## spring scurity

### 概念

spring scurity在web项目上，也是通过配置相关的 web filter 来进行权限相关的一些操作。
不过spring scurity是使用了servlet本身的一些特征来实现相应的功能。

#### spring scurity 的主要一些主要概念

- subject 这里的和apache shiro的subject有区别，shiro中subject是相当是认证的主体，但是这个不是这个概念，spring scurity中，认证的主体是Principal
- Credentials 这个相当于apache shiro中 Roles和Permissions的集合，在Credentials中，role和Permission也统称为Authority。Authority相当于一个集合，可以存其它的扩展的任何的权限概念
- Principal 认证的用户主体的概念，等同于shiro 中的subject

### 介绍

除了安全框架进行了替换以外，其它的技术上基本保持了一致。

ORM，Lombok，MVC等都是一样的，spring boot也是使用了和apache shiro 相同的版本。

### 目录结构

和apache shiro 的结构类似，security框架下，没有realm相关的抽象类，但是多继承了一个UserDetailService接口

```bash
.
|-- HELP.md
|-- README.md
|-- build.gradle
|-- gradlew
|-- gradlew.bat
|-- settings.gradle
`-- src
    |-- main
    |   |-- java
    |   |   `-- group
    |   |       `-- gamelife
    |   |           `-- security
    |   |               `-- demo
    |   |                   |-- AppApplication.java
    |   |                   |-- config
    |   |                   |   `-- SecurityConfig.java
    |   |                   |-- entity
    |   |                   |   |-- Operator.java
    |   |                   |   `-- Role.java
    |   |                   |-- repository
    |   |                   |   |-- OperatorRepository.java
    |   |                   |   `-- RoleRepository.java
    |   |                   |-- service
    |   |                   |   `-- IUserDetailService.java
    |   |                   `-- web
    |   |                       `-- WebResource.java
    |   `-- resources
    |       |-- application.properties
    |       |-- application.yml
    |       |-- import.sql
    |       |-- static
    |       `-- templates
    `-- test
        `-- java
            `-- group
                `-- gamelife
                    `-- security
                        `-- demo
                            `-- AppApplicationTests.java

21 directories, 18 files
```

### spring security 项目的关键步骤

1. 编写ORM层相关的类和对映的查询函数
2. 实现UserDetailsService接口
    这个步骤挺关键的，除了实现这个接口，并注入到配置中去，其实还可以使用继承Users的方法实现自定义用户在数据库中的查询，但是现在这个接口的方式，耦合性会更小，也是官方推荐的方式，具体的使用方式其实还是看项目需要。

    ```java
    @Service("userDetailService")
    @Log
    public class IUserDetailService implements UserDetailsService {

        @Autowired
        private OperatorRepository operatorRepository;

        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

            Optional<Operator> result = operatorRepository.findOperatorByName(username);
            if(result.isPresent()){
                Operator account = result.get();
                Set<Role> roles = account.getRoles();

            // Collection<GrantedAuthority> authorities = new HashSet<>();

                Collection<GrantedAuthority> authorities = roles.stream().collect(HashSet::new,(temp,role)-> temp.add(new SimpleGrantedAuthority("ROLE_"+role.getRoleName().toUpperCase())),HashSet::addAll);
                log.info("username:"+account.getName());
                log.info("roles:"+authorities);
                return new User(account.getName(),account.getPassword(), authorities);
            }
            else {
                throw new UsernameNotFoundException("can not find this user / 查找用户失败");
            }
        }
    }
    ```

3. 编写spring security配置类

    编写spring security配置类是通过WebSecurityConfigurerAdapter这个适配器里相应的方法实现的。如果不改写这个适配器，框架回实例化默认的适配器。

    ```java
    @Configuration
    @EnableGlobalMethodSecurity(securedEnabled = true,prePostEnabled = true)
    public class SecurityConfig extends WebSecurityConfigurerAdapter {

        private final UserDetailsService userDetailService;

        @Autowired
        public SecurityConfig(UserDetailsService userDetailService) {
            this.userDetailService = userDetailService;
        }

        @Override
        protected void configure(AuthenticationManagerBuilder auth) throws Exception {
            auth.userDetailsService(userDetailService).passwordEncoder(new PasswordEncoder() {
                @Override
                public String encode(CharSequence rawPassword) {
                    return rawPassword.toString();
                }

                @Override
                public boolean matches(CharSequence rawPassword, String encodedPassword) {
                    return encodedPassword.equals(rawPassword.toString());
                }
            });
        }

        @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
            .loginPage("/login")
                .successHandler((request, response, authentication) -> response.sendRedirect("/index"))
            .and()
            .logout()
                .logoutUrl("/auth/logout")
            .permitAll()
            .logoutSuccessUrl("/login")
            .and()
            .authorizeRequests()
            .antMatchers("/static/**", "/h2/**", "/login", "/favicon.ico")
            .permitAll()
            .anyRequest()
            .authenticated()
            .and()
            .csrf()
            .disable()
            .headers()
            .frameOptions()
            .sameOrigin()
        .and()
        .sessionManagement();
    }
    }
    ```

4. 编写web接口和页面

## spring reactive scurity

### 概念（）

scurity层面的概念其实和spring scurity是一样的，但是在具体的实现层面有很大的不同。
所以概念这块就不讲了。

### 项目目录结构

```bash
.
|-- HELP.md
|-- README.md
|-- build.gradle
|-- gradlew
|-- gradlew.bat
|-- mvnw
|-- mvnw.cmd
|-- pom.xml
|-- settings.gradle
`-- src
    |-- main
    |   |-- java
    |   |   `-- group
    |   |       `-- gamelife
    |   |           `-- security
    |   |               `-- reactor
    |   |                   `-- demo
    |   |                       |-- AppApplication.java
    |   |                       |-- config
    |   |                       |   `-- WebFluxSecurityConfig.java
    |   |                       |-- entity
    |   |                       |   |-- Operator.java
    |   |                       |   `-- Role.java
    |   |                       |-- repository
    |   |                       |   |-- OperatorRepository.java
    |   |                       |   `-- RoleRepository.java
    |   |                       |-- service
    |   |                       |   `-- IUserDetailService.java
    |   |                       `-- web
    |   |                           `-- WebResource.java
    |   `-- resources
    |       |-- application.properties
    |       |-- application.yml
    |       `-- import.sql
    `-- test
        `-- java
            `-- group
                `-- gamelife
                    `-- security
                        `-- reactor
                            `-- demo
                                `-- AppApplicationTests.java

21 directories, 21 files
```

### spring reactive security 关键步骤

1. 编写ORM层相关的类和对映的查询函数
    这部分内容和 spring security是一摸一样的
2. 实现ReactiveUserDetailsService接口
    和spring security比，这里要实现的接口改变了。在reactive中，大部分的接口都是有变化的。虽然框架的整体设计是类似的，但是返回类型都变成了反应式编程需要的类型。
    所以配置类和相关接口都有变化。使用细节也有很大的变化。

    ```java
    @Service("userDetailsService")
    @Log
    public class IUserDetailService implements ReactiveUserDetailsService {

        @Autowired
        private OperatorRepository operatorRepository;

        @Override
        public Mono<UserDetails> findByUsername(String username) throws UsernameNotFoundException {

            Optional<Operator> result = operatorRepository.findOperatorByName(username);
            if(result.isPresent()){
                Operator account = result.get();
                Set<Role> roles = account.getRoles();

            // Collection<GrantedAuthority> authorities = new HashSet<>();

                Collection<GrantedAuthority> authorities = roles.stream().collect(HashSet::new,(temp, role)-> temp.add(new SimpleGrantedAuthority("ROLE_"+role.getRoleName().toUpperCase())),HashSet::addAll);
                log.info("username:"+account.getName());
                log.info("roles:"+authorities);
                //return new User(account.getName(),account.getPassword(), authorities);
                return Mono.just(new User(account.getName(),account.getPassword(), authorities));
            }
            else {
                throw new UsernameNotFoundException("can not find this user / 查找用户失败");
            }
        }

    }
    ```

3. 编写spring webflux security配置类

    spring webflux security并不需要像security那样去继承一个适配器，只需要写一个配置类来注入必要的bean就行。
    注意：spring reactive security 的配置链和spring security 是有区别的，而且改变还不小，请好好看文档。
    并且，由于webflux并不基于servlet，所以servlet的相关功能都无法使用。request和reponse的修改都有webflux下的写法。

    ```java
    @EnableWebFluxSecurity
    @EnableReactiveMethodSecurity
    @Configuration
    @Log
    public class WebFluxSecurityConfig {
        /**
        * Password Encoder
        * @return
        */
        @Bean
        public PasswordEncoder passwordEncoder() {
            return new PasswordEncoder() {
                @Override
                public String encode(CharSequence rawPassword) {
                    return rawPassword.toString();
                }

                @Override
                public boolean matches(CharSequence rawPassword, String encodedPassword) {
                    return encodedPassword.equals(rawPassword.toString());
                }
            };
        }

        /**
        * config spring security web filter chain
        * @param http
        * @return
        */
        @Bean
        public SecurityWebFilterChain springSecurityWebFilterChain(ServerHttpSecurity http) {

            http.formLogin()
                    .loginPage("/login")
                    .authenticationSuccessHandler((exchange, authentication) -> {
                        log.info("login success！！！！！！！！！！！！！！");
                        ServerHttpResponse res = new ServerHttpResponseDecorator(exchange.getExchange().getResponse());
                        res.getHeaders().add("Location", "/index");
                        res.setStatusCode(HttpStatus.FOUND);
                        return exchange.getChain().filter(exchange.getExchange().mutate().request(req -> req.method(HttpMethod.GET).build()).response(res).build());
                    })
                    .authenticationFailureHandler((exchange, exception) -> {
                        log.info("something wrong！！！！！！！");
                        ServerHttpResponse res = new ServerHttpResponseDecorator(exchange.getExchange().getResponse());
                        res.getHeaders().add("Location", "/login");
                        res.setStatusCode(HttpStatus.FOUND);
                        return exchange.getChain().filter(exchange.getExchange().mutate().request(req -> req.method(HttpMethod.GET).build()).response(res).build());
                    })
                    .and()
                    .logout()
                    .logoutUrl("/logout")
                    .logoutSuccessHandler((webFilterExchange, authentication) -> {
                        log.info("logout success！！！！！！！！");
                        ServerHttpResponse res = webFilterExchange.getExchange().getResponse();
                        res.getHeaders().add("Location", "/login");
                        res.setStatusCode(HttpStatus.FOUND);
                        return webFilterExchange.getChain().filter(webFilterExchange.getExchange().mutate().response(res).build());
                    })
                    .and()
                    .authorizeExchange()
                    .pathMatchers("/static/**", "/h2/**", "/login", "/favicon.ico")
                    .permitAll()
                    .anyExchange()
                    .authenticated()
                    .and()
                    .csrf()
                    .disable()
                    .headers()
                    .frameOptions()
                    .mode(XFrameOptionsServerHttpHeadersWriter.Mode.SAMEORIGIN);
            return http.build();
        }

    }
    ```

4. 编写反应式的web接口和页面

### 响应式和阻塞式的一些需要注意的地方

1. 基于webflux的MVC编程，虽然大部分的注解都是可以使用的，但是控制器函数接受的某些参数还是有区别的。包括可以接收的返回也是有区别的。
2. 基于webflux的编程，Flux和Mono不能使用block类方法把值阻塞出来，这个会抛出exception，不允许这样使用。
3. 使用和@EnableReactiveMethodSecurity相关的注解的控制器，其返回内容必须用Mono或者Flux类型进行包裹。
4. 注意区别各种exchange，有些exchange的类型不一定一样。
5. ReactiveSecurityContextHolder的使用方式和SecurityContextHolder有很大的区别，具体可以见样例

## 3个框架的对比

1. 概念上的不同：apache shiro 的 subject 是认证的主体，认证主体下有 role 和 permission 两个属性用来控制资源访问；spring security 的 认证主体是 Principal ，role 和 permission 也是存在一个集合中的，并且spring security并没有realm的概念。

2. 使用的项目范围不同，spring security 一般来说只用于web相关的项目，apache shiro 可以用于非web项目。 shiro的使用范围其实要更大，而且更灵活。

3. 在 csrf、cros、oauth、jwt等方面，spring security 有默认的集成功能，而这些方面，apache shiro要更加原始，要实现这些功能，得shiro没有集成，要引入相应得包，或者自己做相对应的实现。

4. spring security 和 spring webflux security 的区别主要还是 基于webflux和基于servlet编程的不同，要使用 spring webflux security，首先要掌握 webflux。而且，webflux的相关资料还是太少，学习曲线也比较高。

5. 所有框架的可扩展性都比较高，shiro主要的优势是轻量化，spring security的主要优势是和spring的紧密结合以及开箱即用性。
