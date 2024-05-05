---
layout: default
title: Spring Security Components
grand_parent: Springs
parent: Spring Security
nav_order: 4
---


**스프링 시큐리티는 필터와 인터셉터들로 이뤄져 있다.**

---
# 참고 문서

* [Spring Security Authentication Process : Authentication Flow Behind the Scenes](https://medium.com/geekculture/spring-security-authentication-process-authentication-flow-behind-the-scenes-d56da63f04fa)
* [Implementing JSON Web Token (JWT) Authentication using Spring Security | A Detailed Walkthrough](https://medium.com/geekculture/implementing-json-web-token-jwt-authentication-using-spring-security-detailed-walkthrough-1ac480a8d970)





---
# 1. Authentication Filter

Its a Filter in the FilterChain which detects an authentication attempt and forwards it to the AuthenticationManager.
The Authentication object is stored in the SecurityContext object by the filter for future use and the response will be returned to the end user.



### AbstractSecurityInterceptor
Identify that the user is not logged in.


### DefaultLoginPageGeneratorFilter
redirect the user to login page.

인증을 위한 로그인폼 URL을 감시한다.


### BasicAuthenticationFilter
HTTP 기본 인증 헤더를 감시하여 처리한다.


### AbstractAuthenticationProcessingFilter
UsernamePasswordAuthenticationFilter 의 상위 클래스. AbstractAuthenticationProcessingFilter 를 통해 시큐리티의 필터 목록들 확인할 수 있다.

### UsernamePasswordAuthenticationFilter
(아이디와 비밀번호를 사용하는 form 기반 인증) 설정된 로그인 URL로 오는 요청을 감시하며, 유저 인증 처리

AuthenticationManager를 통한 인증 실행

인증 성공 시, 얻은 Authentication 객체를 SecurityContext에 저장 후 AuthenticationSuccessHandler 실행

인증 실패 시, AuthenticationFailureHandler 실행



로그인 양식 인증 요청은 UsernamePasswordAuthenticationFilter에 도달할 때까지 Filter chain을 통과한다.

UsernamePasswordAuthenticationFilter는 username, password를 이용해 UsernamePasswordAuthenticationToken을 만듭니다. 이 클래스는 Authentication 인터페이스의 구현체입니다. 이것을 AuthenticationToken이라고 통칭하겠습니다.

그리고 AuthenticationToken에 있는 username, password가 유효한 계정인지 판단하기 위해 AuthenticationManager의 구현체인 ProviderManager의 authenticate()메소드를 호출한다.

계정 정보가 유효하다면 AuthenticationFilter는 AuthenticationSuccessHandler에 따라 요청을 redirect시킵니다.

AuthenticationFilter가 해당 요청의 JSESSIONID(SessionID)와 매핑되는 AuthenticationToken이 있는지 판단합니다. 존재하면 해당하는AuthenticationToken을 반환받아 이후 요청 흐름에 포함시킵니다. 따라서 Controller, Service, jsp에서 이 AuthenticationToken 정보를 사용할 수 있습니다.

요청은 Filter Chain을 통과한 후에 DispatcherServlet 에 전달됨.


### RememberMeAuthenticationFilter

RememberMeAuthenticationFilter 필터가 실행될 조건

1. 스프링 시큐리티에서 사용하는 인증객체(Authentication)가 Security Context에 없어야 함
   + 세션 만료(time out), 브라우저 종료, 세션id 자체를 모르는 등의 요인이 있다. 인증객체가 있다라는 소리는 로그인이 정상적으로 되었고, 회원 정보도 정상적으로 세션에서 찾을 수 있다라는 이야기이다. 따라서 이 필터가 실행될 필요가 없다.
2. 사용자 request header에 remember-me 쿠키 토큰이 존재해야 한다.

위의 실행 조건이 부합하다면 RememberMeService(인터페이스)가 실행.


### SecurityContextPersistenceFilter
SecurityContextRepository에서 SecurityContext를 가져오거나 저장하는 역할을 한다. (SecurityContext는 밑에)

### LogoutFilter
설정된 로그아웃 URL로 오는 요청을 감시하며, 해당 유저를 로그아웃 처리

### RequestCacheAwareFilter
로그인 성공 후, 원래 요청 정보를 재구성하기 위해 사용된다.

### SecurityContextHolderAwareRequestFilter
HttpServletRequestWrapper를 상속한 SecurityContextHolderAwareRequestWapper 클래스로 HttpServletRequest 정보를 감싼다. SecurityContextHolderAwareRequestWrapper 클래스는 필터 체인상의 다음 필터들에게 부가정보를 제공한다.

### OAuth2ClientAuthenticationProcessingFilter
OAuth2 인증용 필터

사용예(spring-security-OAuth2-ssofilter 프로젝트)
```java
private Filter ssoFilter(ClientResources client, String path) {
		OAuth2ClientAuthenticationProcessingFilter filter = new OAuth2ClientAuthenticationProcessingFilter(path);
		OAuth2RestTemplate template = new OAuth2RestTemplate(client.getClient(), oauth2ClientContext);
		filter.setRestTemplate(template);
		UserInfoTokenServices tokenServices = new UserInfoTokenServices(client.getResource().getUserInfoUri(),
				client.getClient().getClientId());
		tokenServices.setRestTemplate(template);
		filter.setTokenServices(tokenServices);
		return filter;
	}
```

* OAuth2ClientAuthenticationProcessingFilter 필터의 구현내용
![](./imgs/OAuth2ClientAuthenticationProcessingFilter.png)

=> 필터 구현에서 restTemplate.getAccessToken()에서 accessToken과 refreshToken을 얻는다.

### AnonymousAuthenticationFilter
이 필터가 호출되는 시점까지 사용자 정보가 인증되지 않았다면 인증토큰에 사용자가 익명 사용자로 나타난다.

### SessionManagementFilter
이 필터는 인증된 사용자와 관련된 모든 세션을 추적한다.

### ExceptionTranslationFilter
이 필터는 보호된 요청을 처리하는 중에 발생할 수 있는 예외를 위임하거나 전달하는 역할을 한다.

FilterSecurityInterceptor(맨 마지막에 위치)가 이를 호출한다.

ExceptionTranslationFilter는 아래 2가지 예외를 처리한다.

 * AuthenticationException (인증 예외 처리)
   + AuthenticationEntryPoint 호출
     + 로그인 페이지 이동
     + 401 오류 코드 전달
   + 인증 예외가 발생하기 전의 요청정보를 저장
     + **RequestCache** : 사용자의 이전 요청 정보를 세션에 저장하고 이를 꺼내오는 캐시 메커니즘을 제공
     + **SavedRequest** : 사용자가 요청했던 request의 파라미터 값들, 그 당시의 헤더값 들이 저장
     + 위의 2개는 모두 인터페이스이며 세부 기능을 하는 구현체가 존재한다.
 * AccessDeniedException (인가 예외 처리)
   + AccessDeniedHandler에서 예외 처리하도록 제공
   + 얘도 인터페이스이며 세부 기능을 하는 구현체가 존재한다.



### FilterSecurityInterceptor
이 필터는 AccessDecisionManager 로 권한부여 처리를 위임함으로써 접근 제어 결정을 쉽게해준다.

자동으로 설정된 Spring Security 필터 체인의 마지막 서블릿 필터는 FilterSecurityInterceptor 이다. 이 필터는 해당 요청의 수락 여부를 결정한다.

FilterSecurityInterceptor 가 실행되는 시점에는 이미 사용자가 인증되어 있을 것이므로 유효한 사용자인지도 알 수 있다. Authentication 인터페이스에는 List<GrantedAuthority> getAuthorities() 라는 메소드가 있다는 것을 상기해 보자. 이 메소드는 사용자 아이디에 대한 권한 목록을 리턴한다. 권한처리시에 이 메소드가 제공하는 권한정보를 참조해서 해당 요청의 승인 여부를 결정하게 된다.

Access Decision Manager 라는 컴포넌트가 인증 확인을 처리한다. AccessDecisionManager 인터페이스는 인증 확인을 위해 두 가지 메소드를 제공한다.

 * supports : AccessDecisionManager 구현체는 현재 요청을 지원하는지의 여부를 판단하는 두개의 메소드를 제공한다. 하나는 java.lang.Class 타입을 파라미터로 받고 다른 하나는 ConfigAttribute 타입을 파라미터로 받는다.
 * decide : 이 메소드는 요청 컨텍스트와 보안 설정을 참조하여 접근 승인여부를 결정한다. 이 메소드에는 리턴값이 없지만 접근 거부를 의미하는 예외를 던져 요청이 거부되었음을 알려준다.

인증과정에서 발생할 수 있는 예상 가능한 에러를 처리하는 AuthenticationException 과 하위 클래스를 사용했던 것처럼 특정 타입의 예외 클래스들을 사용하면 권한처리를 하는 애플리케이션의 동작을 좀더 세밀하게 제어할 수 있다.

AccessDecisionManager 는 표준 스프링 빈 바인딩과 레퍼런스로 완벽히 설정할 수 있다.디폴트 AccessDecisionManager 구현체는 AccessDecisionVoter 와 Vote 취합기반 접근 승인 방식을 제공한다.

Voter 는 권한처리 과정에서 다음 중 하나 또는 전체를 평가한다.

 * 보호된 리소스에 대한 요청 컨텍스트 (URL 을 접근하는 IP 주소)
 * 사용자가 입력한 비밀번호
 * 접근하려는 리소스
 * 시스템에 설정된 파라미터와 리소스

AccessDecisionManager 는 요청된 리소스에 대한 access 어트리뷰트 설정을 보터에게 전달하는 역할도 하므로 보터는 웹 URL 관련 access 어트리뷰트 설정 정보를 가지게 된다.

Voter는 사용할 수 있는 정보를 사용해서 사용자의 리소스에 대한 접근 허가 여부를 판단한다. 보터는 접근 허가 여부에 대해서 세 가지 중 한 가지로 결정하는데, 각 결정은 AccessDecisionVoter 인터페이스에 다음과 같이 상수로 정의되어 있다.

 * Grant(ACCESS_GRANTED) : Voter 가 리소스에 대한 접근 권한을 허가하도록 권장한다.
 * Deny(ACCESS_DENIED) : Voter 가 리소스에 대한 접근 권한을 거부하도록 권장한다.
 * Abstain(ACCESS_ABSTAIN) : Voter 는 리소스에 대한 접근권한 결정을 보류한다. 이 결정 보류는 다음과 같은 경우에 발생할 수 있다.
   1. Voter 가 접근권한 판단에 필요한 결정적인 정보를 가지고 있지 않은 경우
   2. Voter 가 해당 타입의 요청에 대해 결정을 내릴 수 없는 경우







---
# 2. Authentication
This component specifies the type of authentication to be conducted. Its is an interface. Its implementation specifies the type of Authentication.


### UsernamePasswordAuthenticationToken
UsernamePasswordAuthenticationToken is an implementation of the Authentication interface which specifies thnat the user wants to authenticate using a userame and password.



### OpenIDAuthenticationToken



### RememberMeAuthenticationToken





---
# 3. AuthenticationManager
The main job of this component is to delegate the **authenticate()** call to the correct AuthenticationProvider.
The Authentication Manager decides which Authentication Provider to delegate the call to by calling the supports() method on every available AuthenticationProvider.
If the supports() method returns true then that AuthenticationProvider supports the Authentication type and is used to perform authentication.



디폴트 동작으로는 DaoAuthenticationProvider가 호출된다.

AuthenticationManager는 기 등록한 AuthenticationProvider들을 연쇄적으로 실행시킵니다. AuthenticationProvider의 구현체에서는 다음과 같은 작업이 필요합니다.

  1. AuthenticationToken에 있는 계정 정보가 유효한지 판단하는 로직 (DB로부터 조회)

  2. 계정 정보가 유효하다면 유저의 상세 정보(이름, 나이 등 필요한 정보)를 이용해 새로운 UserPasswordAuthenticationToken을 발급  


| ![](imgs/springsecuritycomponents-authentication-manager.png) |
|:--:|
| *AuthenticationManager* |

In this diagram, we can see there are three Authentication Providers. Out of the three, Authentication Provider 2 supports the type of incoming Authentication as its supports() method returns true.
It then performs authentication and on success, returns an Authentication object of the same type with the authenticated property set to true along with some other relevant properties.


### ProviderManager
ProviderManager는 AuthenticationManager를 구현한다.

ProviderManager는 Spring-security의 내장 객체.


### AuthenticationManagerBuilder
빌더를 이용해서 AuthenticationManager를 생성

AuthenticationManagerBuilder is used to create an AuthenticationManager instance which is the main Spring Security interface for authenticating a user.



 * 사용 예1
```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    //  In Memory Authentication
    auth
            .inMemoryAuthentication()
            .withUser("admin").password("12345").authorities("admin")
            .and()
            .withUser("user").password("12345").authorities("read")
            .and()
            .passwordEncoder(NoOpPasswordEncoder.getInstance());

}
```


 * 사용 예2
```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    InMemoryUserDetailsManager userDetailsService = new InMemoryUserDetailsManager();
    UserDetails user = User.withUsername("admin").password("12345").authorities("admin").build();
    UserDetails user1 = User.withUsername("user").password("12345").authorities("read").build();
    userDetailsService.createUser(user);
    userDetailsService.createUser(user1);
    auth.userDetailsService(userDetailsService);
}
```



---
# 4. AuthenticationProvider
실제로 인증을 수행하기 위해서 AuthenticationProvider인터페이스를 구현


It is an interface whose implementation processes a certain type of authentication . An AuthenticaionProvider has an authenticate method which takes in the Authentication type and performs authentication on it.
On successful authentication, the AuthenticationProvider returns back an Authentication object of the same type that was taken as input with the authenticated property set to true.
If authentication fails, then it throws an Authentication Exception. Following figure shows a generalized AuthenticationProvider :


| ![](imgs/springsecuritycomponents-AuthenticationProvider.png) |
|:--:|
| *AuthenticationProvider* |

In most apps, we perform username and password authentication and therefore, before performing authentication we have to fetch the UserDetails (username / email, password, roles, etc..)
from a “datasource” such as a database with the help of an UserService, and then authenticate the provided data against the actual data.
The following figure demonstrates the process of a DaoAuthenticationProvider ( this is an implementation of the AuthenticationProvider interface which mainly deals with username password authentication ):


| ![DaoAuthenticationProvider](imgs/springsecuritycomponents-DaoAuthenticationProvider.png) |
|:--:|
| *DaoAuthenticationProvider* |




```java
@Override
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
  String userId = (String) authentication.getPrincipal();
  String password = (String) authentication.getCredentials();

  User user = userService.findByUserId(userId);
  if(user == null) {
      throw new UsernameNotFoundException(userId);
  }

  if (!matchPassword(password, user.getPassword())) {
      throw new BadCredentialsException(userId);
  }

  if (!user.isEnabled()) {
      throw new BadCredentialsException(userId);
  }

  ArrayList<GrantedAuthority> auth = new ArrayList<GrantedAuthority>();
  auth.add(new SimpleGrantedAuthority(user.getAuthority()));
  return new UsernamePasswordAuthenticationToken(user, password, auth);

}
```

파라미터로 전달받은 Authentication은 AuthenticationFilter에 의해 생성되고, 로그인 폼으로부터 입력받은 loginId/password로 생성한 AuthenticationToken

loginId는 AuthenticationToken의 principal에 저장되어있음.

password는 AuthenticationToken의 credentials에 저장되어있음.



유효성을 판단하여 유효하지 않으면 Exception을 발생시킨다. 여기서 사용된 UsernameNotFoundException과 BadCredentialsException은 모두 AuthenticationException.

AuthenticationManager가 등록된 AuthenticationProvider들을 chain으로 실행하지만, AuthenticationException이 발생하면 Exception을 전파하지 않고 chain에 엮여있는 다음 AuthenticationProvider를 실행한다.


맨마지막에 새로운 AuthenticationToken을 생성하는데, 파라미터로 받은 AuthenticationToken과의 차이점은 다음과 같다.
  1. principal의 정보가 확장되었습니다. (loginId -> Mongodb로부터 조회한 User 정보)
  2. 권한 목록을 3번째 파라미터에 추가합니다. 이는 권한 체크를 할 때 사용됩니다.



### DaoAuthenticationProvider
DaoAuthenticationProvider invokes the method loadUserByUsername() of InMemoryUserDetailsManager to load the user details from memory.
Once the user details loaded, it takes help from the default password encoder implementation to compare the password and validate if the user is authentic or not.

At last it returns the Authentication object with the details of authentication success or not to ProviderManager.

### LdapAuthenticationProvider


### OpenIDAuthenticationProvider


### OAuth2LoginAuthenticationProvider




---
# 5. Multi Provider


>Each Authentication provider is tested in order. If one passes, then its following Authentication providers are skipped

[Spring Security Multiple Authentication Providers](https://www.javadevjournal.com/spring-security/spring-security-multiple-authentication-providers/)

[Multiple Authentication Providers in Spring Security | Baeldung](https://www.baeldung.com/spring-security-multiple-auth-providers)



 * 설정 예1 (WebSecurityConfigurerAdapter)
 ```java
 @EnableWebSecurity public class AppSecurityConfig extends WebSecurityConfigurerAdapter {

   ...

   /**
     * Authentication manager which will be invoked by Spring security filter chain. This authentication
     * manager will delegate the work to the Authentication provider to
     * authenticate the user. Look out for the @DaoAuth provider in the above section to see
     * how it works with this.
     * @param auth
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) {
        auth.authenticationProvider(authProvider())
            .authenticationProvider(customAuthenticationProvider);
    }
 }
 ```


 * 설정 예2 (신규버전방식; Spring Security 5.4 부터?)
 ```java
 @EnableWebSecurity
public class MultipleAuthProvidersSecurityConfig {

    @Autowired
    CustomAuthenticationProvider customAuthProvider;

    @Bean
    public AuthenticationManager authManager(HttpSecurity http) throws Exception {
        AuthenticationManagerBuilder authenticationManagerBuilder =
          http.getSharedObject(AuthenticationManagerBuilder.class);
        authenticationManagerBuilder.authenticationProvider(customAuthProvider);
        authenticationManagerBuilder.inMemoryAuthentication()
            .withUser("memuser")
            .password(passwordEncoder().encode("pass"))
            .roles("USER");
        return authenticationManagerBuilder.build();
    }

	...
 ```



---
# 6. UserDetailsService
**유저정보를 불러오는 핵심 인터페이스**

UserDetails 구현체를 리턴하는 역할을 한다. UserDetails 인터페이스는 이전에 설명한 Authentication 인터페이스와 상당히 유사하지만 서로 다른 목적을 가진 인터페이스이므로 헷갈리면 안된다.

 * Authentication : 사용자 ID, 패스워드와 인증 요청 컨텍스트에 대한 정보를 가지고 있다. 인증 이후의 사용자 상세정보와 같은 UserDetails 타입 오브젝트를 포함할 수도 있다.
 * UserDetails : 이름, 이메일, 전화번호와 같은 사용자 프로파일 정보를 저장하기 위한 용도로 사용한다.


 * 소스분석 Tip!
   + UserDetailsService와 UserDetails interface를 구현한 클래스를 확인한다.
   + 이 이소스에서 테이블 구조와 관리하는 유저 데이터를 확인할 수 있다.
   + filter chain순서를 확인한다.
     + JWT토큰을 이용하는 경우라면, 리퀘스트헤더에서 JWT토큰을 얻어서 유효성 체크를 하는 부분을 확인하면, 파악이 쉽다.



This is a service which is responsible for fetching the details of the user from a “datasource”, most likely a database using the loadUserByUsername(String username) methods which takes in the username as a parameter.
It then returns a UserDetails object populated with the user data fetched from the datasource ( database ). The three main fields of an UserDetails object are username, password and the roles/authorities.

| ![](imgs/springsecuritycomponent-UserDetailsService.png) |
|:--:|
| *UserDetailsService* |



 * User Management 중요 클래스와 인터페이스들

![](imgs/springsecuritycomponent-UserManagement.png)



---
# 7. UserDetailManager
**UserDetailService를 확장하며, 유저정보 생성/수정기능 제공한다.**

### InMemoryUserDetailsManager
메모리에서 인증정보를 저장함

* InMemoryUserDetailsManager 예1
```java
@Bean
public InMemoryUserDetailManager userDetailsService() {
 UserDetails admin = User.withDefaultPasswordEncoder()
     .username(“admin”)
     .password(“12345”)
     .authorities(“admin”)
     .build();

 UserDetails user = User.withDefaultPasswordEncoder()
     .username(“user”)
     .password(“12345”)
     .authorities(“read”)
     .build();

 return new InMemoryUserDetailsManager(admin, user);
}
```

* InMemoryUserDetailsManager 예2
  + 예2는 PasswordEncoder를 따로 생성한다.
```java
@Bean
public InMemoryUserDetailManager userDetailsService() {
  InMemoryUserDetailsManager inMemoryUserDetailsManager = new InManagerUserDetailsManager();
  UserDetails admin = User.withUsername("admin").password("12345").authorities("admin").build();
  UserDetails user = User.withUsername("user").password("12345").authorities("read").build();
  inMemoryUserDetailsManager.createUser(admin);
  inMemoryUserDetailsManager.createUser(user);
  return inMemoryUserDetailsManager;
}

/* Not for Production */
@Bean
public PasswordEncoder passwordEncoder() {
  return NoOpPasswordEncoder.getInstance();
}
```

---
# GrantedAuthority
SpringSecurity에서 Authorities/Roles 정보는 GrantedAuthority 에 저장된다.

GrantedAuthority 인터페이스에는 authority나 role의 이름을 리턴하는 함수 하나만있다.

```java
public interface GrantedAuthority {
  String getAuthority();
}
```


### SimpleGrantedAuthority

```java
public final class SimpleGrantedAuthority implements GrantedAuthority {
  private final String role;

  public SimpleGrantedAuthority(String role) {
    this.role = role;
  }

  @Override
  String getAuthority() {
    return this.role;
  }
}
```

---
# 9. Authorities / Roles
Authority is like an individual privilege or an action(fine-grained manner).

Role is a group of privileges/actions(coarse-grained manner).

 * Authority의 예
   + VIEWACCOUNT
   + VIEWCARDS
 * Role의 예
   + ROLE_ADMIN
   + ROLE_USER

authorities/role의 이름은 필요의의해서 만들면 되지만, Role은 Authority와 구분을위해서 반드시 ROLE_로 시작해야 한다(코드에서는 ROLE_을 넣을 필요는 없다.)





# 10. **핸들러들**
---
# AuthenticationSuccessHandler
인증 성공 후에 Redirect시킬 URL을 설정하기 위해 AuthenticationSuccessHandler를 구현


### SavedRequestAwareAuthenticationSuccessHandler
Spring-security는 AuthenticationSuccessHandler의 구현체인 SavedRequestAwareAuthenticationSuccessHandler를 내장하고 있음.

config에서 별도의 successHandler를 지정해주지 않으면 SavedRequestAwareAuthenticationSuccessHandler를 실행한다. 이를 사용하면 인증 성공 후 원래 요청했던 URL로 redirect시켜준다. 그리고 원래 요청했던 URL이 없다면, "/"(defaultTargetUrl)로 redirect한다.

defaultTargetUrl을 변경하려고 하면 아래와 같은 코드를 작성하면 된다.
```java
public class LoginSuccessHandler extends SavedRequestAwareAuthenticationSuccessHandler {

    public LoginSuccessHandler(String defaultTargetUrl) {
        setDefaultTargetUrl(defaultTargetUrl);
    }
}
```


 * SavedRequestAwareAuthenticationSuccessHandler  관련한 참조 내용
   + [Spring Boot - (7) Security 적용 및 로그인 설정](https://4urdev.tistory.com/53?category=761752)
   + [Spring Boot - (7-1) Login 성공 시점에 처리 로직 구현](https://4urdev.tistory.com/54)


### AjaxAuthenticationSuccessHandler


---
# AuthenticationFailureHandler

### AjaxAuthenticationFailureHandler


---
# LogoutSuccessHandler

### AjaxLogoutSuccessHandler


---
# AccessDeniedHandler, AjaxAccessDeniedHandler
인증에 성공했으나 권한이 적합하지 않을 경우, 403에러 화면이 노출되는데,

표시되는 화면을 변경하기 위해서, 요청을 redirect시키기 위해 AccessDeniedHandler를 구현할 수 있다.이 Handler의 handle method는 AccessDeniedException이 발생했을 때 실행된다.


```java
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException exc)
            throws IOException, ServletException {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth != null) {
            LOGGER.info("User: " + auth.getName() + " attempted to access the protected URL: " + request.getRequestURI());
        }

        response.sendRedirect(request.getContextPath() + "/accessDenied");
    }
}
```


---
# 11. RememberMeServices
[RememberMe Filter 개념 & 사용법](https://velog.io/@seongwon97/Spring-Security-Remember-Me)


[Remember Me 기능](https://velog.io/@gmtmoney2357/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-Remember-Me-%EA%B8%B0%EB%8A%A5)
=> 유용하다.


[Spring Security – Persistent Remember Me](https://www.baeldung.com/spring-security-persistent-remember-me)

[Spring Security Remember Me](https://www.baeldung.com/spring-security-remember-me)



RememberMeAuthenticationFilter 필터 실행 조건이 부합하다면 RememberMeService(인터페이스)가 실행된다. 실제 구현체 2가지 있는데, 그 차이는 다음과 같다.

 * TokenBasedRememberMeServices: 서버 메모리에 있는 쿠키와 사용자가 보내온 remember-me 쿠키를 비교(기본적으로 14일간 존재)
 * PersistentTokenBasedRememberMeServices: DB에 저장되어 있는 쿠키와 사용자가 보내온 remember-me 쿠키를 비교(이름 그대로 persistent)


RememberMe 서비스의 처리 로직
 * Token Cookie이 존재하면 추출
 * Decode Token하여 토큰이 정상인지 판단
 * 사용자가 들고온 토큰과 서버에 저장된 토큰이 서로 일치하는지 판단
 * 토큰에 저장된 정보를 이용해 DB에 해당 User 계정이 존재하는지 판단
 * 위 조건을 모두 통과하면 새로운 인증객체(Authentication)을 생성 후 AuthenticationManager에게 인증 처리를 넘긴다. (물론 Security Context에도 인증 객체를 저장한다.)
 * 이후 response 될 때 JSESSIONID를 다시 보내준다.

remember-me 쿠키 사이클
 * 로그인(인증) 성공 = remember-me 쿠키 발급
 * 로그인(인증) 실패 = remember-me 쿠키가 있다면 무효화
   + 즉, 로그인이 성공했어도 사용자가 임의로 로그인 페이지로 돌아간 후 인증에 실패하면, 있는 쿠키도 무효화 시킨다.
 * 로그아웃 = remember-me 쿠키가 있다면 무효화
 * 만료시간 = 지날 겨웅 무효화


remember-me 토큰 초기화 시점
 * rembmer-me 토큰은 새롭게 로그인 인증을 받는 경우 초기화 된다.
 * 그럼 JSESSIONID이 만료해서 remember-me 토큰으로 로그인 해도 초기화 되지는 않는데...
 * remember-me 쿠키로 인증을 진행해서 로그인 될 경우에는 JSESSIONID은 초기화되지만, remember-me 토큰은 초기화가 안되고,
 * 새로 userid, password를 입력해서 로그인하는 경우에는 변경된다.


### AbstractRememberMeServices


### TokenBasedRememberMeServices
RememberMeServices의 구현체

TokenBasedRememberMeServices는 메모리에 있는 토큰과 사용자가 request header에 담아서 보낸 토큰을 비교하여 인증을 한다. (기본적으로 14일만 토큰을 유지한다.)


### PersistentTokenBasedRememberMeServices
RememberMeServices의 구현체

DB에 저장된 토큰과 사용자가 request header에 담아서 보낸 토큰을 비교하여 인증을 한다. (영구적인 방법)




---
# 12. Configuration

 * [Spring Security without the WebSecurityConfigurerAdapter](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)

### WebSecurityConfigurerAdapter
**In Spring Security 5.7.0-M2 we deprecated the WebSecurityConfigurerAdapter**

WebSecurityConfigurerAdapter을 확장해서 Spring Security설정을 할 수 있다.

* 어떤 요청에 대해서 인증을 요구할 것인지
* 특정 요청에 대해서 어떤 권한을 요구할 것인지
* 인증되지 않은 요청을 어떤 url로 redirect시킬지
* 로그인 성공 후 어느 화면으로 이동시킬지
* 로그아웃시 어떤 작업을 수행시킬지
* 403에러에 대해 어떻게 처리할지
* Authentication 유효성을 어떻게 판단할지


### SecurityFilterChain
Spring Security 5.4 에서 등장


### 설정관련 유용한 글들
[Difference between registerGlobal(), configure(), configureGlobal(),configureGlobalSecurity in Spring security](https://stackoverflow.com/questions/35218354/difference-between-registerglobal-configure-configureglobal-configureglo)


---
# 14. AuthenticationException
인증과 관련된 모든 예외는 AuthenticationException 을 상속한다. AuthenticationException 은 개발자에게 상세한 디버깅 정보를 제공하기위한 두개의 멤버 필드를 가지고 있다.

 * authentication : 인증 요청관련 Authentication 객체를 저장하고 있다.
 * extraInformation : 인증 예외 관련 부가 정보를 저장한다. 예를 들어 UsernameNotFoundException 예외는 인증에 실패한 유저의 id 정보를 저장하고 있다.

**많이 발생하는 예외들.**
 * BadCredentialsException : 사용자 아이디가 전달되지 않았거나 인증 저장소의 사용자 id 에 해당하는 패스워드가 일치하지 않을 경우 발생한다.
 * LockedException : 사용자 계정이 잠긴경우 발생한다.
 * UsernameNotFoundException : 인증 저장소에서 사용자 ID를 찾을 수 없거나 사용자 ID에 부여된 권한이 없을 경우 발생한다.


예외가 발생하면 ExceptionFitler가 동작해서 해당 예외를 캐치하게 된다.(AccessDeindException)

관련한 상세한 내부 동작(https://zgundam.tistory.com/60)

이때 인증 문제냐 인가 문제냐에 따라서 if 조건 처리가 달라지는데 동작은 아래와 같다.

 * 인증(Authentication) 예외는 EntryPoint
 * 인가(Authorization)  예외는 Handler


---
# 15. AuthenticationEntryPoint

인증(authenticate) 처리 과정에서 예외가 발생한 경우 예외를 핸들링하는 인터페이스

AuthenticationException(인증되지 않은 요청)인 경우 AuthenticationEntryPoint를 사용하여 처리한다.

ExceptionTranslationFilter가 사용한다.




### LoginUrlAuthenticationEntryPoint, AjaxLoginUrlAuthenticationEntryPoint
Holds the location of the login form in the loginFormUrl property, and uses that to build a redirect URL to the login page. Alternatively, an absolute URL can be set in this property and that will be used exclusively.



---
# Sequence Flow


| ![](imgs/SpringSecuritySequenceFlow-DefaultBehaviour.png) |
|:--:|
| *Default Behaviour* |


| ![](imgs/SpringSecuritySequenceFlow-WithOurOwnUserDetailsService.png) |
|:--:|
| *With Our Own UserDetailService Implementation* |


| ![](imgs/SpringSecuritySequenceFlow-WithOurOwnAuthenticationProvider.png) |
|:--:|
| *With Our Own AuthenticationProvider Implementation* |




---
# 전체흐름 정리
We are going to look at the entire flow of a username and password authentication with the help of a diagram.


| ![](imgs/springsecuritycomponents-AuthenticationFlow-1.png) |
|:--:|
| *Authentication Flow ( Pt. 1 )* |

### Step 1
When the server receives a request for authentication, such as a login request, it is first intercepted by the Authentication Filter in the Filter Chain.

### Step 2
A UsernamePasswordAuthenticationToken is created using the username and password provided by the user. As discussed above, the UsernamePasswordAuthenticationToken is an implementation of the Authentication interface and used when a user wants to authenticate using a username and password.

### Step 3
The UsernamePasswordAuthenticationToken is passed to the AuthenticationManager so that the token can be authenticated.




| ![](imgs/springsecuritycomponents-AuthenticationFlow-2.png) |
|:--:|
| *Authentication Flow ( Pt. 2 )* |


### Step 4
The AuthenticationManager delegates the authentication to the appropriate AuthenticationProvider. As dicussed before this is accomplished by calling the supports() method on the AuthenticationProvider.

### Step 5
The AuthenticationProvider calls the loadUserByUsername(username) method of the UserDetailsService and gets back the UserDetails object containing all the data of the user. The most important data is the password becuase it will be used to check whether the provided password is correct. If no user is found with the given user name, a UsernameNotFoundException is thrown.

### Step 6
The AuthenticationProvider after receiving the UserDetails checks the passwords and authenticates the user. FINALLY!!! . If the passwords do not match it throws a AuthenticationException. However, if the authentication is successful, a UsernamePasswordAuthenticationToken is created, and the fields principal, credentials, and authenticated are set to appropriate values . Here principal refers to your username or the UserDetails , credentials refers to password and the authenticated field is set to true. This token is returned back to the AuthenticationManager.

### Step 7
On successful authentication, the SecurityContext is updated with the details of the current authenticated user. SecurityContext can be used in several parts of the app to check whether any user is currently authenticated and if so, what are the user’s details.
