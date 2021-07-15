#스프링 시큐리티 주요 아키텍처 이해
## DelegatingFilterProxy
![delegating_filter_proxy_filter_chain_proxy.png](../images/delegating_filter_proxy_filter_chain_proxy.png)
- 서블릿 필터는 스프링에서 정의된 빈을 주입해서 사용할 수 없음
- 특정한 이름을 가진 스프링 빈을 찾아 그 빈에게 요청을 위임
    - springSecurityFilterChain 이름으로 생성된 빈을 ApplicationContext 에서 찾아 요청을 위임
    - 실제 보안처리를 하지 않음
    
## FilterChainProxy
- springSecurityFilterChain의 이름으로 생성되는 필터 빈
- DelegatingFilterProxy으로 부터 요청을 위임 받고 실제 보안 처리
- 스프링 시큐리티 초기화 시 생성되는 필터들을 관리하고 제어
    - 스프링 시큐리티가 기본적으로 생성하는 필터
    - 설정 클래스에서 API 추가 시 생성되는 필터
- 사용자의 요청을 필터 순서대로 호출하여 전달
- 사용자 정의 필터를 생성해서 기존의 필터 전,후로 추가 가능
    - 필터의 순서를 잘 정의
- 마지막 필터까지 인증 및 인가 예외가 발생하지 않으면 보안 통과

## 필터 초기화와 다중 설정 클래스
![multi_config_class.png](../images/multi_config_class.png)
![multi_config_class2.png](../images/multi_config_class2.png)
~~~
  protected void configure(HttpSecurity http) throws Exception {
    http
        .antMatcher("/admin/**")
        .authorizeRequests()
        .anyRequest().authenticated()
        .and()
        .httpBasic();
  }
}

@Configuration
class SecurityConfig2 extends WebSecurityConfigurerAdapter {

  protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
        .anyRequest().permitAll()
        .and()
        .formLogin();
  }
~~~
- 설정 클래스 별로 보안 기능이 각각 작동
- 설정 클래스 별로 RequestMatcher 설정
    - http.antMatcher("/admin")
    
- 설정 클래스 별로 필터가 생성
- FilterChainProxy가 각 필터를 가지고 있음
- 요청에 따라 RequestMatch와 매칭되는 필터가 작동하도록 함

## Authentication
![authentication.png](../images/authentication.png)
>당신이 누구인지 증명하는 것
- 사용자의 인증 정보를 저장하는 토큰 개념
- 인증 시 아이디와 패스워드를 담고 인증 검증을 위해 전달되어 사용된다
- 인증 후 최종 인증 결과 (User 객체, 권한정보)를 담고 SecurityContext에 저장되어 전역적으로 참조가 가능하다
  - Authentication authentication = SecurityContextHolder.getContext().getAuthentication()
  
- 구조
  - principal : 사용자 아이디 혹은 User 객체를 저장
  - credentials : 사용자 비밀번호
  - authorities : 인증된 사용자의 권한 목록
  - details : 인증 부가 정보
  - Authenticated : 인증 여부
  
## SecurityContext
- Authentication 객체가 저장되는 보관소로 필요 시 언제든지 Authentication 객체를 꺼내어 쓸 수 있도록 제공되는 클래스
- ThreadLocal에 저장되어 아무 곳에서나 참조가 가능하도록 설계함
- 인증이 완료되면 HttpSession 에 저장되어 어플리케이션 전반에 걸쳐 전역적인 참조가 가능하다

## SecurityContextHolder
![security_context_holder.png](../images/security_context_holder.png)
- SecurityContext 객체 저장 방식
  - MODE_THREADLOCAL : 스레드당 SecurityContext 객체를 할당, 기본값
  - MODE_INHERITABLETHREADLOCAL : 메인 스레드와 자식 스레드에 관하여 동일한 SecurityContext 를 유지
  - MODE_GLOBAL : 응용 프로그램에서 단 하나의 SecurityContext 를 저장한다.
- SecurityContextHolder.clearContext() : SecurityContext 기존 정보 초기화
- Authentication authentication = SecurityContextHolder.getContext().getAuthentication()