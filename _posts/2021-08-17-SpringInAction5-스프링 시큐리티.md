---
title:  "Spring-웹 애플리케이션 개발"
excerpt: "스프링 인 액션 5 공부"
categories:
  - Study
tag:
  - SpringInAction5
toc: true
---

### 스프링 시큐리티

##### 스프링 시큐리티 활성화 하기
> 스프링 애플리케이션의 보안에서 맨 먼저 할 일은스프링 부트 보안 스타터 의존성을 빌드 명세에 추가하는것이다.

보안 스터타를 프로젝트 빌드 파일에 추가만 했을때는 다음의 보안 구성이 제공된다.
- 모든 HTTP 요청 겨로는 인증 되어야 한다.
- 어떤 특정 역할이나 권환이 없다.
- 로그인 페이지가 따로 없다.
- 스프링 시큐리티의 HTTP 기본 인증을 사용해서 인증된다.
- 사용자는 하나만 있으며, 이름은 user다. 비밀번호는 암호화해 준다.

##### 스프링 시큐리티 구성하기

SecurityCOnfig 클래스
- 사용자의 HTTP 요청 경로에 대해 접근 제한과 같은 보안 관련 처리를 우리가 원하는 대로 할 수 있게 해준다.

``` java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
		.authorizeRequests()
		.antMatchers("/design", "/orders")
		.access("hasRole('ROLE_USER')")
		.antMatchers("/", "/**").access("permitAll")
		.and()
		.httpBasic();

	}
	@Override
	public void configure(AuthenticationManagerBuilder auth) throws Exception{
		auth.inMemoryAuthentication()
		.withUser("user1")
		.password("{noop}password1")
		.authorities("ROLE_USER")
		.and()
		.withUser("user2")
		.password("{noop}password2")
		.authorities("ROLE_USER");
	}
}
```

한 명 이상의 사용자를 처리할 수 있도록 사용자 정보를 유지, 관리하는사용자 스토어를 구성해야 한다. 
스프링 시큐리티에서는 여러가지 사용자 스토어 구성 방법을 제공한다.
- 인메머리 사용자 스토어
- JDBC 기반 사용자 스토어
- LDAP 기반 사용자 스토어
- 커스텀 사용자 명세 서비스

SecurityConfig 클래스는 보안 구성 클래스인 WebSecurity ConfigurerAdapter의 서브 클래스다.
두개의 configure() 메서드를 오버라라이딩을하고 있다.
- configure(HttpSecurity) : HTTP 보안을 구성하는 메서드
- configure(AuthenticationManagerbuilder) : 사용자 인증 정보를 구성하는 메서드

###### 인메모리 사용자 스토어

> 사용자의 정보를 유지.관리할 수 잇는 곳중 하난가 메모리다. 만일 변경이 필요 없는 사용자만 미리 정해 놓고 애플리케이션을 사용한다면 아예 보안 구성 코드 내부에 정의할 수 있다.

user1과 user2라는 사용자를 인메모리 사용자 스토어에 구성하는 방법이다.

``` java
	@Override
	public void configure(AuthenticationManagerBuilder auth) throws Exception{
		auth.inMemoryAuthentication()
		.withUser("user1")
		.password("{noop}password1")
		.authorities("ROLE_USER")
		.and()
		.withUser("user2")
		.password("{noop}password2")
		.authorities("ROLE_USER");
	}
```
withUser()를 호출하면 해당 사용자의 구성이 시작되며, 사용자 이름을 인자로 전달한다. 
반면에 비밀번호와 부여 권한은 각각 password()와 authorities() 메서드의 인자로 전달하여 호출한다.
- 스프링 5 부터는 반드시 비밀번호를 암호화 해야 하므로  password() 메서드를 호출하여 암호화 하지 않으면 403, 500 이 발생한다.
- 위 예제는 {noop}를 지정하여 비밀번호를 암호화 하지 않았다.

##### JDBC 기반의 사용자 스토어

> 사용자 정보눈 관계형 데이터베이스로 유지.관리되는 경우가 많으므로 JDBC 기반의 사용자 스토어가 적합해 보인다.

**JDBC 기반의사용자 스토어로 인증하기** 
@Autowired로 DataSource가 자동으로 주입된다.

``` java
@Autowired
DataSource dataSource;

@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception{    
	auth
	.jdbcAuthentication()
	.dataSource(dataSource);
}
```

스프링 시큐리티의 것과 다른 데이터베이스(테이블이나 열의 이름이 다를때)를 사용한다면, 스프링 시큐리티의 SQL 쿼리를 우리의 SQL 쿼리로 대체할 수 있다.

``` java
@Autowired
DataSource dataSource;

@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception{    
	auth
	.jdbcAuthentication()
	.dataSource(dataSource)usersByUsernameQuery(
			"select username, password, enabled from users " +
			"where username=?")
		.authoritiesByUsernameQuery(
			"select username, authority from authorities " +
			"where username=?");
}
```
위 쿼리에서 사용하는 테이블의 이름은 스프링 시큐리티의 기본 데이터베이스 테이블과 달라도 된다(위에는 동일함).
그러나 테이블이 갖는 열의 데이터 타입과 길이는 일치해야하고 다음 사항을 지켜야 한다.
- 매개변수(where절에 사용)는 하나이며, username이어야 한다.
- 사용자 정보 인증 쿼리에서는 username, password, enabled 열의 값을 반환해야 한다.

**암호화된 비밀번호 사용**  
비밀번호를암호화 할때는 passwordencoder() 메서드를호출하여 비밀번호 인코더를 지정한다.


``` java
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception{    
	auth
	.jdbcAuthentication()
	.dataSource(dataSource)usersByUsernameQuery(
			"select username, password, enabled from users " +
			"where username=?")
		.authoritiesByUsernameQuery(
			"select username, authority from authorities " +
			"where username=?")
		.passwordEncoder(new BCryptPasswordEncoder());
}
```
passwordencoder() 메서드는 스플이 시큐리티의 PasswordEncoder 인터페이스를 구현하는 어떤 객체도 인자로 받을 수있다.
암호화 알고리즘을 구현한 스프링 시큐리티의 모듈에는 다음과 같은 구현 클래스가 포함되어 있다.
- BCryptPasswordEncoder : bcrypt를 해싱 암호화 한다.
- NoOpPasswordEncoder: 암호화 하지 않는다.
- Pbkdf2PasswordEncoder : PBKDF2를 암호화 한다.
- SCryptPasswordEncoder : scrypt를 해싱 암호화 한다.
- standardPasswordEncoder : SHA-256을 해싱 암호화한다.
