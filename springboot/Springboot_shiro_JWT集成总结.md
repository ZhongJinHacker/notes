### SpringBoot Shiro JWT



#### 1.建表

**DDL.sql**

```mysql
CREATE TABLE `t_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(30) NOT NULL DEFAULT '',
  `password` varchar(32) NOT NULL DEFAULT '',
  `salt` varchar(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=14 DEFAULT CHARSET=utf8mb4 COMMENT='用户表';

CREATE TABLE `t_sys_role` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `role_name` varchar(20) DEFAULT NULL COMMENT '角色名',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='角色表';

CREATE TABLE `t_sys_permission` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL COMMENT '权限名称解释',
  `url` varchar(60) DEFAULT NULL COMMENT '权限',
  `permission` varchar(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='权限表';

CREATE TABLE `t_user_role` (
  `id` bigint(11) unsigned NOT NULL AUTO_INCREMENT,
  `user_id` bigint(11) DEFAULT NULL COMMENT '用户id',
  `role_id` bigint(11) DEFAULT NULL COMMENT '角色id',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户角色对应表';

CREATE TABLE `t_sys_permission` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL COMMENT '权限名称解释',
  `permission` varchar(30) DEFAULT NULL,
  `url` varchar(60) DEFAULT NULL COMMENT '权限',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='权限表';
```

**DML.sql**

```mysql
-- 创建角色
INSERT INTO `t_sys_role` VALUES (null, '管理员');
INSERT INTO `t_sys_role` VALUES (null, '普通用户');

-- 创建用户
INSERT INTO `t_user` VALUES (null, 'admin', '856aea89ad509f163284abb75579dcfc', 'admin');
INSERT INTO `t_user` VALUES (null, 'user', 'ea6151b14a3b64331d6c33e645fccef6', 'user');

-- 创建权限
INSERT INTO `t_sys_permission` VALUE(null, '超级用户权限', 'root:all', '/**');
INSERT INTO `t_sys_permission` VALUE(null, '查看用户列表', 'user:list', '/admin/userList');

-- 给用户配置角色
INSERT INTO `t_user_role` VALUE(null, 14, 1);
INSERT INTO `t_user_role` VALUE(null, 15, 2);

-- 给角色配置权限
INSERT INTO `t_role_permission` VALUE(null, 1, 1);
INSERT INTO `t_role_permission` VALUE(null, 2, 2);
```



#### 4. springboot 项目加入shiro依赖

```xml
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-spring-boot-starter</artifactId>
			<version>1.4.0</version>
		</dependency>

		<!-- 这个用于开启缓存功能 -->
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-ehcache</artifactId>
			<version>1.4.0</version>
		</dependency>

		<!-- jwt jar-->
		<dependency>
			<groupId>com.auth0</groupId>
			<artifactId>java-jwt</artifactId>
			<version>3.2.0</version>
		</dependency>
```



#### 5. 配置shiro

```java
package com.grady.shirodemo.config;

import com.grady.shirodemo.filter.JWTFilter;
import com.grady.shirodemo.realm.JWTRealm;
import com.grady.shirodemo.realm.UserRealm;
import org.apache.shiro.authc.credential.HashedCredentialsMatcher;
import org.apache.shiro.authc.pam.AtLeastOneSuccessfulStrategy;
import org.apache.shiro.authc.pam.ModularRealmAuthenticator;
import org.apache.shiro.cache.ehcache.EhCacheManager;
import org.apache.shiro.mgt.DefaultSessionStorageEvaluator;
import org.apache.shiro.mgt.DefaultSubjectDAO;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.realm.Realm;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.handler.SimpleMappingExceptionResolver;

import javax.servlet.Filter;
import java.util.*;

@Configuration
public class ShiroConfig {

  	/**
  	* 最重要的一个bean
  	**/
    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();

      	// 加入自定义的拦截器，会拦截（所有被“jwt”标记的方法）请求到url访问的方法
        Map<String, Filter> filterMap = new HashMap<>();
        filterMap.put("jwt", new JWTFilter());
        shiroFilterFactoryBean.setFilters(filterMap);

        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 设置未登录跳转到的url
        shiroFilterFactoryBean.setLoginUrl("/toLogin");
        shiroFilterFactoryBean.setUnauthorizedUrl("/unAuthorize");

        // 设置拦截器与url映射关系map
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        filterChainDefinitionMap.put("/login", "anon");
        filterChainDefinitionMap.put("/unAuthorize", "anon");
        // 所有请求通过我们自己的JWT Filter
      	// 注意越先设置的url会越先生效，原因你懂的（LinkedHashMap 有先后顺序）
        filterChainDefinitionMap.put("/**", "jwt");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);

        return shiroFilterFactoryBean;
    }

  	// 这里是配置密码转换逻辑matcher（此处会把传进来的密码以MD5的算法序列化5次）
    @Bean
    public HashedCredentialsMatcher hashedCredentialsMatcher() {
        HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
        matcher.setHashAlgorithmName("MD5");
        matcher.setHashIterations(5);
        matcher.setStoredCredentialsHexEncoded(true);
        return matcher;
    }

    // 核心bean，所有的realm都要在这里加进去
    @Bean
    public DefaultWebSecurityManager securityManager(UserRealm userRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setAuthenticator(modularRealmAuthenticator());
        // realm只能在你setAuthenticator下设置入securityManager，否则不生效
        List<Realm> list = new ArrayList<>();
        list.add(userRealm());
        list.add(jwtRealm());
        securityManager.setRealms(list);
        // 这里虽是方法调用，但是引用地址都是一样的，所以是同一个对象
        securityManager.setCacheManager(ehCacheManager());

        // 关闭Shiro自带的session
        DefaultSubjectDAO subjectDAO = new DefaultSubjectDAO();
        DefaultSessionStorageEvaluator defaultSessionStorageEvaluator = new DefaultSessionStorageEvaluator();
        defaultSessionStorageEvaluator.setSessionStorageEnabled(false);
        subjectDAO.setSessionStorageEvaluator(defaultSessionStorageEvaluator);
        securityManager.setSubjectDAO(subjectDAO);

        return securityManager;
    }

  	// 这是用账号密码来校验的realm
    @Bean
    public UserRealm userRealm() {
        UserRealm realm = new UserRealm();
        //启用授权缓存
        realm.setAuthorizationCachingEnabled(true);
        realm.setCredentialsMatcher(hashedCredentialsMatcher());
        return realm;
    }

  	// 这是通过token来校验的realm
    @Bean
    public JWTRealm jwtRealm() {
        return new JWTRealm();
    }

    /**
     *  开启shiro aop注解支持.
     *  使用代理方式;所以需要开启代码支持;
     * @param securityManager
     * @return
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager){
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }

    /**
     * 配置异常处理器，无权限时会走这里
     * @return
     */
    @Bean
    public SimpleMappingExceptionResolver resolver() {
        SimpleMappingExceptionResolver resolver = new SimpleMappingExceptionResolver();
        Properties properties = new Properties();
        properties.setProperty("org.apache.shiro.authz.UnauthorizedException", "/unAuthorize");
        resolver.setExceptionMappings(properties);
        return resolver;
    }

    /**
     * shiro缓存管理器;
     * 需要添加到securityManager中
     * @return
     */
    @Bean
    public EhCacheManager ehCacheManager(){
        EhCacheManager cacheManager = new EhCacheManager();
        cacheManager.setCacheManagerConfigFile("classpath:ehcache-shiro.xml");
        return cacheManager;
    }

    /**
     * 系统自带的Realm管理，主要针对多realm
     * */
    @Bean
    public ModularRealmAuthenticator modularRealmAuthenticator(){
        ModularRealmAuthenticator modularRealmAuthenticator = new ModularRealmAuthenticator();
        // 这里配置多个realm只要一个通过就算通过（会串型执行）
        modularRealmAuthenticator.setAuthenticationStrategy(new AtLeastOneSuccessfulStrategy());
        return modularRealmAuthenticator;
    }
}

```



#### 6. 完成两个realm的编写

**UserRealm.java**

```java
package com.grady.shirodemo.realm;

import com.grady.shirodemo.entity.model.User;
import com.grady.shirodemo.entity.model.UserPermission;
import com.grady.shirodemo.entity.model.UserRole;
import com.grady.shirodemo.service.PermissionService;
import com.grady.shirodemo.service.UserService;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.util.ByteSource;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

public class UserRealm extends AuthorizingRealm {

    @Autowired
    UserService userService;

    @Autowired
    PermissionService permissionService;

    @Override
    public boolean supports(AuthenticationToken token) {
        return token instanceof UsernamePasswordToken;
    }

    /**
     * 用于授权，进行权限校验
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("UserRealm : doGetAuthorizationInfo 鉴权");
        // 这里即使抛出runtime异常也会被shiro Advicefilter 捕获
        User user = (User) principalCollection.getPrimaryPrincipal();
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        List<UserRole> roleList = permissionService.findRolesByUserId(user.getId());
        Optional.ofNullable(roleList).ifPresent(roles -> {
            authorizationInfo.addRoles(roles.stream().map(UserRole::getRoleName).collect(Collectors.toList()));
            List<String> permList = new ArrayList<>();
            roles.forEach(role -> {
                // 这里即使抛出runtime异常也会被shiro Advicefilter 捕获
                List<UserPermission> permissions = permissionService.findPermissionByRoleId(role.getId());
                Optional.ofNullable(permissions).ifPresent(perms -> {
                    permList.addAll(perms.stream().map(UserPermission::getPermission).collect(Collectors.toList()));
                });
            });
            authorizationInfo.addStringPermissions(permList);
        });
        return authorizationInfo;
    }

    /**
     * 进行身份认证，即登录
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("UserRealm : doGetAuthenticationInfo 认证");
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        User user = Optional.ofNullable(userService.findUserByUserName(token.getUsername()))
                .orElseThrow(() -> new UnknownAccountException("无此用户"));
        //用用户名作为唯一盐值，确保序列化后的MD5值唯一
        ByteSource salt = ByteSource.Util.bytes(user.getUserName());
        AuthenticationInfo info = new SimpleAuthenticationInfo(user, user.getPassword(), salt, getName());
        return info;
    }
}
```



**JWTRealm.java**

```java
package com.grady.shirodemo.realm;

import com.grady.shirodemo.entity.model.User;
import com.grady.shirodemo.jwt.JWTToken;
import com.grady.shirodemo.service.PermissionService;
import com.grady.shirodemo.service.UserService;
import com.grady.shirodemo.utils.JWTUtil;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.Optional;

public class JWTRealm extends AuthorizingRealm {

    @Autowired
    UserService userService;

    @Autowired
    PermissionService permissionService;

    @Override
    public boolean supports(AuthenticationToken token) {
        return token instanceof JWTToken;
    }

    /**
     * 用于授权，进行权限校验
     * @param principals
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        System.out.println("JWTRealm : doGetAuthorizationInfo 鉴权");
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        return authorizationInfo;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken auth) throws AuthenticationException {
        System.out.println("JWTRealm : doGetAuthenticationInfo 认证");
        String token = (String) auth.getCredentials();
        String username = JWTUtil.getUsername(token);
        if (username == null) {
            throw new AuthenticationException("token invalid");
        }
        User user = Optional.ofNullable(userService.findUserByUserName(username))
                .orElseThrow(() -> new UnknownAccountException("无此用户"));
        if (!JWTUtil.verify(token, username, user.getPassword())) {
            throw new AuthenticationException("Username or password error");
        }
        return new SimpleAuthenticationInfo(user, token, getName());
    }
}

```



#### 7. 配置Filter

```java
package com.grady.shirodemo.filter;

import com.grady.shirodemo.jwt.JWTToken;
import org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter;

import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class JWTFilter extends BasicHttpAuthenticationFilter {

    private static final String TOKEN = "token";

    /**
     * 判断是否允许访问
     * @param request
     * @param response
     * @param mappedValue
     * @return
     */
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        if (isLoginAttempt(request, response)) {
            try {
                executeLogin(request, response);
                return true;
            } catch (Exception e) {
                e.printStackTrace();
              	// 如果responseUnAuthorize 重定向的url也是要被拦截的话会陷入死循话
                responseUnAuthorize(request, response);
                return false;
            }
        }
        //如果请求头不存在 Token，则可能是执行登陆操作或者是游客状态访问，无需检查 token，直接返回 true
        return true;
    }

    @Override
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
        return true;
    }

    @Override
    protected boolean executeLogin(ServletRequest request, ServletResponse response) throws Exception {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        String authorization = httpServletRequest.getHeader(TOKEN);
        JWTToken token = new JWTToken(authorization);
        // 提交给realm进行登入，如果错误他会抛出异常并被捕获
        getSubject(request, response).login(token);
        // 如果没有抛出异常则代表登入成功，返回true
        return true;
    }

    @Override
    protected boolean isLoginAttempt(ServletRequest request, ServletResponse response) {
        HttpServletRequest req = (HttpServletRequest) request;
        String token = req.getHeader(TOKEN);
        return token != null;
    }

    private void responseUnAuthorize(ServletRequest request, ServletResponse resp) {
        try {
            HttpServletResponse httpServletResponse = (HttpServletResponse) resp;
            httpServletResponse.sendRedirect("/unAuthorize");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```



#### 8. token 相关

**JWToken**

```java
package com.grady.shirodemo.jwt;

import org.apache.shiro.authc.AuthenticationToken;

public class JWTToken implements AuthenticationToken {

    /**
     * 密钥
     */
    private String token;

    public JWTToken(String token) {
        this.token = token;
    }

    @Override
    public Object getPrincipal() {
        return token;
    }

    @Override
    public Object getCredentials() {
        return token;
    }
}

```



**JWTUtil**

```java
package com.grady.shirodemo.utils;

import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.exceptions.JWTDecodeException;
import com.auth0.jwt.interfaces.DecodedJWT;

import java.io.UnsupportedEncodingException;
import java.util.Date;

public class JWTUtil {

    // 过期时间30天
    private static final long EXPIRE_TIME = 24 * 60 * 30 * 1000;

    private static final String CLAIM_STR = "username";
    /**
     * 校验token是否正确
     *
     * @param token    密钥
     * @param username 登录名
     * @param password 密码
     * @return
     */
    public static boolean verify(String token, String username, String password) {
        try {
            // 这里与密码无关，是生成（或确认）token的算法
            Algorithm algorithm = Algorithm.HMAC256(password);
            JWTVerifier verifier = JWT.require(algorithm).withClaim(CLAIM_STR, username).build();
            DecodedJWT jwt = verifier.verify(token);
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    /**
     * 获取登录名
     *
     * @param token
     * @return
     */
    public static String getUsername(String token) {
        try {
            DecodedJWT jwt = JWT.decode(token);
            return jwt.getClaim(CLAIM_STR).asString();
        } catch (JWTDecodeException e) {
            return null;
        }
    }

    /**
     * 生成token签名,指定时间后过期,一经生成不可修改，令牌在指定时间内一直有效
     * @param userNo 用户编号
     * @param secret 用户的密码
     * @return 加密的token
     */
    public static String sign(String userNo, String secret) {
        try {
            Date date = new Date(System.currentTimeMillis()+EXPIRE_TIME);
            // 这里与密码无关，是生成token的算法
            Algorithm algorithm = Algorithm.HMAC256(secret);
            // 附带username信息
            return JWT.create()
                    .withClaim(CLAIM_STR, userNo)
                    .withExpiresAt(date)
                    .sign(algorithm);
        } catch (UnsupportedEncodingException e) {
            return null;
        }
    }
}

```

