---
layout: post
title: Spring boot로 JWT 사용하기
date: 2022-04-05
categories: [Spring boot]
tag: [spring boot, JWT]
---

## JWT 프로젝트 세팅

JWT를 사용하기 위해서 JWT 라이브러리를 먼저 추가한다. JWT 토큰을 직접 만들 수도 있지만 라이브러리를 활용해서 편하게 사용할 수 있다. 

#### Maven

```java
    <dependency>
        <groupId>com.auth0</groupId>
        <artifactId>java-jwt</artifactId>
        <version>3.19.1</version>
    </dependency>
```

#### Gradle

```java
    implementation 'com.auth0:java-jwt:3.19.1'
```

## JWT Security 세팅

Security에서 제공하는 formLogin은 x-www-form-urlencoded의 Context-type으로 데이터를 받는다. JSON을 받아서 로그인 처리를 하려면 해당 기능을 비활성화 해줘야 하고 JWT는 세션을 사용하지 않기 때문에 서버를 stateless로 설정해야 한다.

```java:SecurityConfig.java

    @Configuration
    @EnableWebSecurity

    public class SecurityConfig extends WebSecurityConfigurerAdapter {

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.csrf().disable();
            http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // 세션을 사용하지 않는다
                .and()
                .formLogin().disable()                                                      
                .httpBasic().disable()                                                       
                .authorizeRequests()
                .antMatchers("/api/v1/user/**")
                .access("hasRole('ROLE_USER') or hasRole('ROLE_MANAGER') or hasRole('ROLE_ADMIN')")
                .antMatchers("/api/v1/manager/**")
                .access("hasRole('ROLE_MANAGER') or hasRole('ROLE_ADMIN')")
                .antMatchers("/api/v1/admin/**")
                .access("hasRole('ROLE_ADMIN')")
                .anyRequest().permitAll();
        }
    }

```

httpBasic 방식은 header에 Authorization 키 값에 유저를 식별할 수 있는 정보를 담아서 서버에 요청을 하는 방식이다. 이 경우 유저를 매 요청 시 확인하므로 쿠키/세션이 필요하지 않다. 다만 http 방식은 header 정보를 암호화가 안되서 https 방식으로 사용해야 한다.
JWT를 사용할 경우 httpBasic을 disable로 처리하고 Authorization 키 값에 JWT 토큰을 담아서 보낸다. 이 방식을 Bearer 인증 방식이라 한다.

이제 Security에서 CORS 문제를 해결해 줘야 한다. CORS Filter를 만들어서 Security Filter Chain에 추가한다.

```java:CorsConfig.java

    @Configuration
    public class CorsConfig {

        @Bean
        public CorsFilter corsFilter() {
            UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
            CorsConfiguration config = new CorsConfiguration();
            config.setAllowCredentials(true);   // 내 서버가 응답할 때 josn을 자바스크립트에서 처리할 수 있게 할지를 설정하는 것
            config.addAllowedOrigin("*");       // 모든 ip에 응답을 허용하겠다
            config.addAllowedHeader("*");       // 모든 header에 응답을 허용하겠다
            config.addAllowedMethod("*");       // 모든 post, get, put, delete, patch 요청을 허용하겠다
            source.registerCorsConfiguration("/api/**", config);
            return new CorsFilter(source);
        }
    }

```

```java:SecurityConfig.java

    @Configuration
    @EnableWebSecurity
    @RequiredArgsConstructor
    public class SecurityConfig extends WebSecurityConfigurerAdapter {

        private final CorsConfig corsConfig;
        private final UserRepository userRepository;

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.csrf().disable();
            http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // 세션을 사용하지 않는다
                .and()
                .addFilter(corsFilter)  // @CrossOrigin(인증X), 시큐리티 필터에 등록 인증(O)
                .formLogin().disable()
                ...   
        }
    }

```

## JWT Filter 등록

#### Spring Filter 등록

Spring Boot로 서버를 사용하면 클라이언트의 요청이 왔을때 Filter Chain으로 요청이 넘어가서 각 Filter에서 요청에 맞는 작업을 진행해서 요청을 처리한다. Filter를 만들때는 ***Filter*** 를 구현화해서 사용한다.

```java:MyFilter1.java

    public class MyFilter1 implements Filter {

        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            System.out.println("MyFilter1");
            chain.doFilter(request, response);
        }
    }

```

***chain.doFilter(reauset, response)*** 를 하지 않으면 해당 필터에서 작업이 종료된다. 해당 라인을 통해 다음 필터로 요청을 넘길 수 있다.
필터를 등록하려면 Filter Config파일을 설정해서 커스텀 필터를 등록할 수 있다.

```java:FilterConfig.java

    @Configuration
    public class FilterConfig {

        @Bean
        public FilterRegistrationBean<MyFilter1> filter1() {
            FilterRegistrationBean<Myfilter1> filter1() {
                FilterRegistrationBean<MyFilter1> bean = new FilterRegistrationBean<>(new MyFilter1());
                bean.addUrlPatterns("/*");  // 필터가 동작할 Url 패턴
                bean.setOrder(0);           // 필터 동작 순서
                return bean;
            }
        }
    }

```

Filter Config에 등록한 필터는 Security Filter Chain이 끝난 후 필터가 동작한다. Security Filter Chain 사이 또는 먼저 필터를 등록하기 위해서는 Security Filter Chain에 사용할 필터를 등록해야 한다. Security Filter Chain에 필터를 등록하는 방법은 아래와 같다.

```java:SecurityConfig.java

    @Configuration
    @EnableWebSecurity
    @RequiredArgsConstructor
    public class SecurityConfig extends WebSecurityConfigurerAdapter {

        private final CorsConfig corsConfig;
        private final UserRepository userRepository;

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.addFilterBefore(new MyFilter(), BasicAuthenticationFilter.class);
            http.csrf().disable();
            http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // 세션을 사용하지 않는다
            ...   
        }
    }

```

Security Filter에 등록할 경우 ***addFilterBefore() 또는 addFilterAfter()*** 를 사용한 한다. Security Filter Chain에서 어떤 필터 전후로 등록할지 명시해 줘야한다.

#### JWT를 위한 로그인 처리

JWT를 발행하기위해 먼저 로그인 처리를 해야한다. 현재 SecurityConfig에서 Security Login을 비활성화로 설정해뒀기 때문에 클라이언트가 Login요청을 보내면 직접 로그인 처리를 해야한다. 스프링 시큐리티에서는 Login요청이 오면 ***UsernamePasswordAuthenticationFilter*** 에서 로그인 처리를 한다. 따라서 직접 로그인 처리를 하려면 UsernamePasswordAuthenticationFilter를 상속받은 JWTAuthenticationFilter를 만들어서 처리해야 한다.

```java:JwtAuthenticationFilter.java

    // 클라이언트가 username, password를 담아 Login 요청을 함
    // UsernamePasswordAuthenticationFilter 동작
    
    @RequiredArgsConstructor
    public class JwtAuthenticationFilter extends UsernamePasswordAuthenticationFilter {
        private final AuthenticationManager authenticationManager;

        // /login 요청을 하면 로그인 시도를 위해서 실행되는 함수
        @Override
        public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
            // 1. username, password 받아서
            try {
                ObjectMapper om = new ObjectMapper();
                User user = om.readValue(request.getInputStream(), User.class);

                UsernamePasswordAuthenticationToken authenticationToken =
                    new UsernamePasswordAuthenticationToken(user.getUsername(), user.getPassword());    // Longin을 위해 토큰 생성
                
                // PrincipalDetailsService의 loadUserByUsername() 함수가 실행됨
                // authenticationManager를 통해 로그인 인증을 진행하고
                // authentication에 로그인한 정보가 담긴다. 
                Authentication authentication = authenticationManager.authenticate(authenticationToken);
                
                // 로그인 완료된 건지 확인 
                // PrincipalDetails principalDetails = (PrincipalDetails) authentication.getPrincipal();
                // System.out.println(principalDetails.getUser().getUsername());  

                // authentication 객체가 세션에 저장됨 => 로그인 되었다는 뜻
                // authentication 리턴을 하면 세션에 저장되고 security가 권한관리를 해주게 된다.
                return authentication;
            } catch (IOException e) {
                e.printStackTrace();
            }

            reutnr null;
        }

        // attemptAuthentication 실행 후 인증이 정상적으로 되었으면 successfulAuthentication 함수가 실행됨
        // JWT 토큰을 만들어서 request 요청한 사용자에게 JWT토큰을 respose 해주면 됨
        @Override
        protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
            System.out.println("successfulAuthentication 실행됨: 인증이 완료되었다는 뜻");
            PrincipalDetails principalDetails = (PrincipalDetails) authResult.getPrincipal();

            // RSA방식은 아니고 Hash 암호방식
            String jwtToken = JWT.create()
                    .withSubject("cos Token")
                    .withExpiresAt(new Date(System.currentTimeMillis() + (600000 * 10)))
                    .withClaim("id", principalDetails.getUser().getId())
                    .withClaim("username", principalDetails.getUser().getUsername())
                    .sign(Algorithm.HMAC512("cos"));

            response.addHeader("Authorization", "Bearer: " + jwtToken);
        }
    }

```

