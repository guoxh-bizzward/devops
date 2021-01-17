# Spring Security

* 认证
* 授权

## 认证

### 常用认证方式

- HTTP BASIC authentication headers：基于IETF RFC 标准。
- HTTP Digest authentication headers：基于IETF RFC 标准。
- HTTP X.509 client certificate exchange：基于IETF RFC 标准。
- LDAP：跨平台身份验证。
- Form-based authentication：基于表单的身份验证。
- Run-as authentication：用户用户临时以某一个身份登录。
- OpenID authentication：去中心化认证。
- Jasig Central Authentication Service：单点登录。
- Automatic "remember-me" authentication：记住我登录（允许一些非敏感操作）。
- Anonymous authentication：匿名登录。
- ......

## 授权

### 授权模式

* 基于URL
* 基于方法
* 基于对象



## 源码解析

Spring Security默认配置的密码会输出到控制台中,这块的代码是在`org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration#getOrDeducePassword`

```
	private String getOrDeducePassword(SecurityProperties.User user, PasswordEncoder encoder) {
		String password = user.getPassword();
		if (user.isPasswordGenerated()) {
			logger.info(String.format("%n%nUsing generated security password: %s%n", user.getPassword()));
		}
		if (encoder != null || PASSWORD_ALGORITHM_PATTERN.matcher(password).matches()) {
			return password;
		}
		return NOOP_PASSWORD_PREFIX + password;
	}
```

`org.springframework.boot.autoconfigure.security.SecurityProperties.User`

```
	public static class User {

		/**
		 * Default user name.
		 */
		private String name = "user";

		/**
		 * Password for the default user name.
		 */
		private String password = UUID.randomUUID().toString();
```

### Spring Security配置密码的方式

* 配置文件

```
spring.security.user.name=
spring.security.user.password=
spring.security.user.roles=
```

* Java配置类

```
@Configuration(proxyBeanMethods = false)
public class WebSecurityDevConfig extends WebSecurityConfigurerAdapter {

        @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("javabody")
                .password("123")
                .roles("admin");
    }
 }
```

### 加密方案

密码加密我们一般会用到散列函数，又称散列算法、哈希函数，这是一种从任何数据中创建数字“指纹”的方法。散列函数把消息或数据压缩成摘要，使得数据量变小，将数据的格式固定下来，然后将数据打乱混合，重新创建一个散列值。散列值通常用一个短的随机字母和数字组成的字符串来代表。好的散列函数在输入域中很少出现散列冲突。在散列表和数据处理中，不抑制冲突来区别数据，会使得数据库记录更难找到。我们常用的散列函数有 MD5 消息摘要算法、安全散列算法（Secure Hash Algorithm）。

但是仅仅使用散列函数还不够，为了增加密码的安全性，一般在密码加密过程中还需要加盐，所谓的盐可以是一个随机数也可以是用户名，加盐之后，即使密码明文相同的用户生成的密码密文也不相同，这可以极大的提高密码的安全性。但是传统的加盐方式需要在数据库中有专门的字段来记录盐值，这个字段可能是用户名字段（因为用户名唯一），也可能是一个专门记录盐值的字段，这样的配置比较繁琐。

Spring Security 提供了多种密码加密方案，官方推荐使用 BCryptPasswordEncoder，BCryptPasswordEncoder 使用 BCrypt 强哈希函数，开发者在使用时可以选择提供 strength 和 SecureRandom 实例。strength 越大，密钥的迭代次数越多，密钥迭代次数为 2^strength。strength 取值在 4~31 之间，默认为 10。

不同于 Shiro 中需要自己处理密码加盐，在 Spring Security 中，BCryptPasswordEncoder 就自带了盐，处理起来非常方便。

而 BCryptPasswordEncoder 就是 PasswordEncoder 接口的实现类

`org.springframework.security.crypto.password.PasswordEncoder`

```
public interface PasswordEncoder {
	//对明文密码加密返回加密后的密文
   String encode(CharSequence rawPassword);
   //密码校对方法,判断输入的密码和用户密码是否一致
   boolean matches(CharSequence rawPassword, String encodedPassword);
   //是否还要进行再次加密,一般来说不用 
   default boolean upgradeEncoding(String encodedPassword) {
      return false;
   }
}
```

### 自定义登录页

```
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/js/**","/css/**","/images/**");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors().and().csrf().disable()
                .authorizeRequests()
                .antMatchers(HttpMethod.OPTIONS,"/**").permitAll()
                .antMatchers("/oauth/*").permitAll() //生产token的uri允许任何登录的用户访问
                .antMatchers("/swagger/**","/webjars/**","/v2/**","/v2/api-docs-ext/**","/swagger-resources/**","/doc.html").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin().loginPage("/login.html")
                .permitAll()
                .and()
                .logout().logoutSuccessHandler(new HttpStatusReturningLogoutSuccessHandler())
                .permitAll();
    }
```

### 登录接口

在SpringSecurity中,如果不做任何配置,默认的登录页面和登录接口的地址都是/login

`get http://localhost:8080/login`

`post http://localhost:8080/login`

get请求表示你想访问登录页面;post请求表示你想提交登录数据;

```
                .and()
                .formLogin().loginPage("/login.html")
                .permitAll()
                .and()
```

当我们配置了loginPage为 login.html之后,还有一个隐藏的操作就是登录接口地址也设置成了/login.html,也就是说,新的登录页和登录接口地址都是 /login.html.

如果要分开配置,则要参考下面这段代码

```
                .and()
                .formLogin()
                .loginPage("/login.html")
                .loginProcessingUrl("/doLogin")
                .permitAll()
                .and()
```

form表单相关的配置在`org.springframework.security.config.annotation.web.configurers.FormLoginConfigurer`

该类继承了`AbstractAuthenticationFilterConfigurer`.所以当`FormLoginConfigurer`初始化的时候,`AbstractAuthenticationFilterConfigurer`也会初始化.`AbstractAuthenticationFilterConfigurer`的构造方法如下:

```
	protected AbstractAuthenticationFilterConfigurer() {
		setLoginPage("/login");
	}
```

另一方面,`FormLoginConfigurer``的初始化方法init方法也调用了父类的init方法

```
	@Override
	public void init(H http) throws Exception {
		super.init(http);
		initDefaultLoginFilter(http);
	}
```

而父类的init方法中,又调用了`updateAuthenticationDefaults`

```
	@Override
	public void init(B http) throws Exception {
		updateAuthenticationDefaults();
		updateAccessDefaults(http);
		registerDefaultAuthenticationEntryPoint(http);
	}
```

```
	protected final void updateAuthenticationDefaults() {
		if (loginProcessingUrl == null) {
			loginProcessingUrl(loginPage);
		}
		if (failureHandler == null) {
			failureUrl(loginPage + "?error");
		}

		final LogoutConfigurer<B> logoutConfigurer = getBuilder().getConfigurer(
				LogoutConfigurer.class);
		if (logoutConfigurer != null && !logoutConfigurer.isCustomLogoutSuccess()) {
			logoutConfigurer.logoutSuccessUrl(loginPage + "?logout");
		}
	}
```

从这个方法的逻辑中我们可以看到,如果用户没有给loginProcessingUrl设置值的话,默认就使用了loginPage作为loginProcessingUrl

而如果配置了loginPage,在配置完loginPage之后,`updateAuthenticationDefaults`方法还是会被调用,此时如果没有配置loginProcessingUrl,则使用新配置的loginPage作为loginProcessUrl.

### 登录参数

在`FormLoginConfigurer`类的构造方法中,可以看到有两个配置用户名密码的方法

```
	public FormLoginConfigurer() {
		super(new UsernamePasswordAuthenticationFilter(), null);
		usernameParameter("username");
		passwordParameter("password");
	}
```

首先调用父类的构造方法,传入`UsernamePasswordAuthenticationFilter`实例,该实例将被赋值给父类的`authFilter`属性.

```
	public FormLoginConfigurer<H> usernameParameter(String usernameParameter) {
		getAuthenticationFilter().setUsernameParameter(usernameParameter);
		return this;
	}
		public FormLoginConfigurer<H> passwordParameter(String passwordParameter) {
		getAuthenticationFilter().setPasswordParameter(passwordParameter);
		return this;
	}
```

username 和password 来自于`org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter#attemptAuthentication`

```
 public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        if (this.postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        } else {
            String username = this.obtainUsername(request);
            String password = this.obtainPassword(request);
            if (username == null) {
                username = "";
            }

            if (password == null) {
                password = "";
            }

            username = username.trim();
            UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
            this.setDetails(request, authRequest);
            return this.getAuthenticationManager().authenticate(authRequest);
        }
    }
```

当然,这个参数我们也可以自己设置

```
				.and()
                .formLogin()
                .loginPage("/login.html")
                .loginProcessingUrl("/doLogin")
                .usernameParameter("name")
                .passwordParameter("passwd")
                .permitAll()
                .and()
```



### 登录回调

登录成功后,就需要分情况处理了,大概有两种情况

* 前后端分离登录
* 前后端不分登录

Spring Security中,和登录成功重定向URL相关的方法有两个

* defaultSuccessUrl
* successForwardUrl