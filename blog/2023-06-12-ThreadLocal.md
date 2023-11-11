---
layout:       post
title:        "ThreadLocal使用笔记"
author:       "Young Sheep"
header-style: text
catalog:      true
tags:
    - ThreadLocal
    - Token
    - Redis
    - SpringBoot
    - Java开发笔记
---
# ThreadLocal

## ThreadLocal实现流程

-   首先创建ThreadLocal类，在其中设置相关的添加、获取以及删除方法。
-   创建登录拦截器，重写其中的`preHandle()`和`afterCompletion()`方法。
-   注册拦截器

在日常开发中我们可以根据自己需要去获取用户信息，比如从token解析等

## ThreadLocal类

```java
public class UserThreadLocal {
 
    private UserThreadLocal(){}
 
    private static final ThreadLocal<SysUser> LOCAL = new ThreadLocal<>();
 
    public static void put(SysUser sysUser){
        LOCAL.set(sysUser);
    }
 
    public static SysUser get(){
        return LOCAL.get();
    }
 
    public static void remove(){
        LOCAL.remove();
    }
}
```

## 拦截器

在`preHandle()`方法中根据自己的需要将用户的登录信息存放之ThreadLocal中即可。
然后最后记得清除相关数据以避免内存泄漏

```java
@Component
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {
 
    @Autowired
    private LoginService loginService;
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        /**
         * 1. 需要判断 请求的接口路径 是否为 HandlerMethod (controller方法)
         * 2. 判断 token是否为空，如果为空 未登录
         * 3. 如果token 不为空，登录验证 loginService checkToken
         * 4. 如果认证成功 放行即可
         */
        if(!(handler instanceof HandlerMethod)){
            return true;
        }
        //从请求头中获取token
        String token = request.getHeader("Authorization");

        log.info("=================request start===========================");
        log.info("request url :{} ",request.getRequestURI());
        log.info("request method :{}",request.getMethod());
        log.info("request token :{}",token);
        log.info("=================request end===========================");

        if (StringUtils.isBlank(token)){
            Result result = Result.fail(ErrorCode.NO_LOGIN.getCode(), "未登录");
            response.setContentType("application/json;charset=utf-8");
            response.getWriter().print(JSON.toJSONString(result));
            return false;
        }
        //验证token并获取用户对象
        SysUser sysUser = loginService.checkToken(token);
        if(sysUser == null){
            System.out.println("user is null");
            Result result = Result.fail(ErrorCode.NO_LOGIN.getCode(), "未登录");
            response.setContentType("application/json;charset=utf-8");
            response.getWriter().print(JSON.toJSONString(result));
            return false;
        }
        //登录验证成功，放行
        UserThreadLocal.put(sysUser);
        return true;
    }
 
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        //如果不删 ThreadLocal中用完的信息会有内存泄漏的风险
        UserThreadLocal.remove();
    }
}
```

## 注册自定义拦截器

```java
@Configuration
public class WebMVCConfig implements WebMvcConfigurer {
 
    @Autowired
    private LoginInterceptor loginInterceptor;
 
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //假设拦截test接口 后续实际遇到拦截的接口是时，再配置真正的拦截接口
//        registry.addInterceptor(loginInterceptor).addPathPatterns("/**").excludePathPatterns("/login").excludePathPatterns("/register");    //拦截所有，排除登录注册接口
        registry.addInterceptor(loginInterceptor)
                .addPathPatterns("/test")
    }
}
```
