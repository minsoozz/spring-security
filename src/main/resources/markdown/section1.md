# 스프링 시큐리티 기본 API 및 Filter 이해
## 프로젝트 구성 및 의존성 추가

~~~
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
~~~

***스프링 시큐리티의 의존성 추가 시 일어나는 일들***

1. 서버가 기동되면 스프링 시큐리티의 초기화 작업 및 보안 설정이 이루어진다
2. 별도의 설정이나 구현을 하지 않아도 기본적인 웹 보안 기능이 현재 시스템에 연동되어 작동함

- 모든 요청은 인증이 되어야 자원에 접근이 가능하다
- 인증 방식은 폼 로그인 방식과 httpBasic 로그인 방식을 제공한다
- 기본 로그인 페이지를 제공한다
- 기본 계정을 한개 제공한다 - username : user / password : 랜덤 문자열

***문제점***

1. 계정 추가, 권한 추가, DB연동 등
2. 기본적인 보안 기능 외에 시스템에서 필요로 하는 더 세부적이고 추가적인 보안 기능이 필요

## 사용자 정의 보안 기능 구현
- - - 
![img.png](../md-images/img.png)
### SecurityConfig 설정

~~~
  @Configuration
  @EnableWebSecurity
  public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception { 
      http
          .authorizeRequests()
          .anyRequest().authenticated()
          .and()
          .formLogin();
    }
~~~

#### WebSecurityConfigurerAdapter를 상속받은 SecurityConfig 클래스에서는 사용자가 직접 보안 설정을 정의 할 수 있다
## Form Login 인증
- - -
![img_1.png](../md-images/img_1.png)
#### 클라이언트와 서버간의 관계속에서 스프링 시큐리티가 인증처리 프로세스
1. 사용자가 GET방식으로 /home 자원에 접근한다
2. 사용자가 인증을 받지 않은 경우 로그인 페이지로 리다이렉트 (실패)
3. POST 방식으로 인증을 시도 (성공)
4. 서버에서는 스프링 시큐리티가 세션을 생성하게 되고 그 세션에 Authorization 객체를 생성해서 저장한다 (SecurityContext에 저장한다)
5. 인증을 받은 이후 /home 자원에 접근을 시도하면 스프링 시큐리티는 사용자가 가진 세션으로부터 토큰의 존재여부를 판단 후 자원에 접근한다, 스프링 시큐리티는 세션에 저장된
   인증토큰이 있다면 사용자가 인증된 사용자라고 판단하고 인증을 유지한다

### Form Login 인증 API

~~~
protected void configure(HttpSecurity http) throws Exception {
         http
         formLogin()
            .loginPage(“/login.html") // 사용자 정의 로그인 페이지
            .defaultSuccessUrl("/home) // 로그인 성공 후 이동 페이지
            .failureUrl(＂/login.html?error=true“) // 로그인 실패 후 이동 페이지
            .usernameParameter("username") // 아이디 파라미터명 설정
            .passwordParameter(“password") // 패스워드 파라미터명 설정
            .loginProcessingUrl(“/login") // 로그인 Form Action url
            .successHandler(loginSuccessHandler()) // 로그인 성공 후 핸들러
            .failureHandler(loginFailureHandler()) // 로그인 실패 후 핸들러
}
~~~

## UsernamePasswordAuthenticationFilter
- - - 
![img_2.png](../md-images/img_2.png)

#### 인증처리를 담당하고 인증처리에 요청을 처리하는 필터가 UsernamePasswordAuthenticationFilter이다 내부적으로 각각의 인증처리의 역할을 통해 인증처리를 하게 된다
1. 사용자가 Request
2. 사용자의 요청정보를 UsernamePasswordAuthenticationFilter가 확인한다
3. AntPathRequestMatcher가 요청 정보가 매칭되는지 확인한다
4. 매칭이 되면 인증처리, 매칭이 되지 않으면 필터로 이동한다
5. Authentication 객체를 생성해서 사용자가 로그인 할 때 입력한 정보를 저장한다
6. AuthenticationManager (인증관리자) 가  필터로부터 인증 객체를 전달 받고 인증을 하게 된다
7. AuthenticationProvider에게 인증을 위임한다 (실질적으로 인증을 하는 클래스)
8. 인증이 실패하면 AuthenticationException (인증예외) 를 발생시켜 다시 필터가 받아 예외 후속작업을 실행
9. 인증이 성공하면 Authentication 객체를 생성해서 AuthenticationManager에게 리턴한다
10. AuthenticationManager는 전달받은 인증 객체를 다시 Filter 에게 리턴한다
11. Filter는 인증을 처리한 이후에 Authentication 객체를 전달받고 인증 객체를 SecurityContext에 저장한다(인증 객체 저장소, 전역으로 Authentication 객체를 참조 가능)
12. SuccessHandler에서 인증 성공 이후 작업를 진행

####크게 인증을 하기 전 작업 , 인증 후 작업으로 나뉘는데 그 분기점은 AuthenticationManager이다.

## Logout 처리, LogoutFilter
- - - 
![logout.png](../md-images/logout.png)
~~~
protected void configure(HttpSecurity http) throws Exception {
	 http.logout()						// 로그아웃 처리
             .logoutUrl(＂/logout＂)				// 로그아웃 처리 URL
	         .logoutSuccessUrl(＂/login＂)			// 로그아웃 성공 후 이동페이지
             .deleteCookies(＂JSESSIONID“, ＂Remember-Me＂) 	// 로그아웃 후 쿠키 삭제
	         .addLogoutHandler(logoutHandler())		 // 로그아웃 핸들러
             .logoutSuccessHandler(logoutSuccessHandler()) 	// 로그아웃 성공 후 핸들러
}
~~~
1. 클라이언트가 POST 방식으로 로그아웃 요청
2. AntPathRequestMatcher가 로그아웃 요청을 확인한다
3. 일치하면 인증객체를 담고 있는 SecurityContext를 Authentication로 꺼내온다
4. SecurityContextLogoutHandler가 세션 무효화, 쿠키 삭제, 인증 객체 NULL 처리, securityContext.clearContext() 컨텍스트에서 정보를 삭제
5. 로그아웃이 성공하면 SimpleUrlLogoutSuccessHandler를 호출에 로그인 페이지로 이동하도록 한다

## Remember Me 인증
- - - 
~~~
protected void configure(HttpSecurity http) throws Exception {
	http.rememberMe()
		.rememberMeParameter("remember") // 기본 파라미터명은 Remember-Me
		.tokenValiditySeconds(3600) // Default 는 14일
		.alwaysRemember(true) // 리멤버 미 기능이 활성화되지 않아도 항상 실행
		.userDetailsService(userDetailsService)
}
~~~
1. 세션이 만료되고 웹 브라우저가 종료된 후에도 어플리케이션이 사용자를 기억하는 기능
2. Remember-Me 쿠키에 대한 Http 요청을 확인한 후 토큰 기반 인증을 사용해 유효성을 검사하고 토큰이 검증되면 사용자는 로그인 된다
3. 사용자 라이프 사이클
- 인증 성공 (Remember-Me 쿠키 설정)
- 인증 실패 (쿠키가 존재하면 쿠키 무효화)
- 로그아웃 (쿠키가 존재하면 쿠키 무효화)

#### JSESSIONID가 없더라도 스프링 시큐리티에서는 Remember-Me 쿠키가 있다면 다시 인증을 시도하고 JSESSIONID를 다시 생성한다

## RememberMeAuthenticationFilter
- - -
![Remember-Me.png](../md-images/Remember-Me.png)
### RememberMeAuthenticationFilter가 정상적으로 작동하는 조건
1. 인증객체가 없는 경우
2. 사용자가 Remember-Me 쿠키를 가지고 오는 경우
####RememberMeAuthenticationFilter가 요청을 처리하는 조건은 Authentication이 NULL일 경우이다
####ex) 사용자의 세션이 만료되었거나 브라우저가 종료되어 시큐리티 컨텍스트를 찾지 못하는 경우

### Remember-Me 인증 절차
1. Remember-Me를 가지고 있는 세션이 만료된 사용자가 요청
2. RememberMeAuthenticationFilter가 동작한다 
3. RememberMeService가 인증을 시도 (각 구현체가 Remember-me 인증 처리 역할을 한다)
- TokenBasedRememberMeServices 토큰을 비교해서 인증처리
- PersistentTokenBasedRememberMeServices DB에 저장한 토큰을 비교해서 인증처리
4. RememberMeService가 토큰을 추출한다
5. Decode Token(정상 유무 판단) 토큰의 일치여부를 확인한다
- 정상이 아닐 경우 예외 발생
6. 토큰에 포함된 User 정보를 통해 현재 DB에 저장된 User를 조회하여 해당 쿠키에 포함된 유저에 해당하는지를 확인한다
- User 계정이 존재하지 않을 경우 예외 발생
7. 새로운 Authentication 객체를 생성한다
8. Authentication 인증 객체를 AuthenticationManager 에게 전달하여 인증 처리를 하게 된다

