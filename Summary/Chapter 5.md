# 스프링 부트 시큐리티 + OAuth2


## 5.1.1 스프링 부트 시큐리티

#### Authentication(인증)
: 사용자가 애플리케이션의 특정 동작에 관하여 허락 된 사용자인지 허락된 사용자인지 확인하는 절차
<br>

#### Authorization(권한)
: 데이터나 프로그램 등의 특정 자원이나 서비스에 접근할 수 있는 권한을 허용하는 것
<br>
<br>

#### 인증 방법
1. Credential 인증
: Principal(사용자명) / Credential (비밀번호) 기반 인증 방식
<br>

2. 이중 인증 방식
: OTP와 같이 추가적인 인증 방식
<br>

3. OAuth2 인증
: 소셜 미디어를 사용한 편리한 인증 방식
<br>
<br>

## 5.1.2 OAuth2
토큰을 사용한 범용적인 방법의 인증을 제공하는 표준 인증 프로토콜

서드파티를 위한 범용적인 인증 표준
<br>

#### 인증 방식
1. Authorization Code Grant Type (권한 부여 코드 승인 타입)

 : 클라이언트가 다른 사용자 대신 특정 리소스에 접근을 요청할 때 사용

 : 리소스 접근을 위한 사용자명과 비밀번호, 권한 서버에 요청해서 받은 권한 코드를 함께 활용하여 리소스에 대한 엑세스 토큰을 받아서 이용
<br>

2. Implict Grant Type (암시적 승인 타입)

 : 권한 부여 코드 승인 타입과 다르게 권한 코드 교환 단계 없이 액세스 토큰을 즉시 반환받아 이를 인증에 이용하는 방식
<br>

3. Resource Owner Password Credentials Grant Type (리소스 소유자 암호 자격 증명 승인 타입)

 : 클라이언트가 암호를 사용하여 엑세스 토큰에 대한 자격 증명을 교환하는 방식
<br>

4. Client Credentials Grant Type (클라이언트 자격 증명 승인 타입)

 : 클라이언트가 컨텍스트 외부에서 액세스 토큰을 얻어 특정 리소스에 접근을 요청할 때 사용하는 방식
<br>

<img width="550" alt="스크린샷 2019-04-03 오후 9 32 59" src="https://user-images.githubusercontent.com/34764544/55479091-3eaac300-5658-11e9-949f-24ec3394dd81.png">
<br>

# 5.2 스프링 부트 시큐리티 + OAuth2 설계하기


#### 스프링 시큐리티 및 OAuth2 적용 흐름

<img width="571" alt="스크린샷 2019-04-03 오후 9 33 18" src="https://user-images.githubusercontent.com/34764544/55479126-54b88380-5658-11e9-91c6-6d7b93084c7b.png">
<br>

#### 커뮤니티 게시판 시큐리티/OAuth2 흐름

<img width="580" alt="스크린샷 2019-04-03 오후 9 33 35" src="https://user-images.githubusercontent.com/34764544/55479146-639f3600-5658-11e9-858f-a678f8ce41de.png">
<br>
<br>

```java
public enum SocialType {
    FACEBOOK("facebook"),
    GOOGLE("google"),
    KAKAO("kakao");

    private final String ROLE_PREFIX = "ROLE_";
    private String name;

    SocialType(String name) {
        this.name = name;
    }

    public String getRoleType() { return ROLE_PREFIX + name.toUpperCase(); }

    public String getValue() { return name; }

    public boolean isEquals(String authority) {
        return this.name.equals(authority);
    }
}
```
<br>

```java
@Getter
@NoArgsConstructor
@Entity
@Table
public class User implements Serializable {

    @Id
    @Column
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long idx;

    @Column
    private String name;

    @Column
    private String password;

    @Column
    private String email;

    // OAuth2 인증으로 제공받는 키 값
    @Column
    private String pincipal;

    // 인증 받은 소셜 미디어
    @Column
    @Enumerated(EnumType.STRING)
    private SocialType socialType;

    @Column
    private LocalDateTime createdDate;

    @Column
    private LocalDateTime updatedDate;

    @Builder
    public User(String name, String password, String email, String pincipal,
        SocialType socialType, LocalDateTime createdDate, LocalDateTime updatedDate) {
        this.name = name;
        this.password = password;
        this.email = email;
        this.pincipal = pincipal;
        this.socialType = socialType;
        this.createdDate = createdDate;
        this.updatedDate = updatedDate;
    }
}

```
<br>

# 5.3 스프링 부트 시큐리티 + OAuth2 의존성 설정
##### depency에 spring security oauth2를 추가
###### compile('org.springframework.security.oauth:spring-security-oauth2')
<br>

# 5.4 스프링 부트 시큐리티 + OAuth2 구현하기
<br>

## 5.4.1 SNS 프로퍼티 설정 및 바인딩

/resources/application.yml
```java
facebook :
  client :
    clientId : -------- CLIENT ID ---------
    clientSecret: -------- CLIENT SECRET ---------
    accessTokenUri: https://graph.facebook.com/oauth/access_token
    userAuthorizationUri: https://www.facebook.com/dialog/oauth?display=popup
    tokenName: oauth_token
    authenticationScheme: query
    clientAuthenticationScheme: form
    scope: email
  resource:
    userInfoUri: https://graph.facebook.com/me?fields=id,name,email,link

google :
  client :
    clientId : -------- CLIENT ID ---------
    clientSecret: -------- CLIENT SECRET ---------
    accessTokenUri: https://accounts.google.com/o/oauth2/token
    userAuthorizationUri: https://accounts.google.com/o/oauth2/auth
    scope: email, profile
  resource:
    userInfoUri: https://www.googleapis.com/oauth2/v2/userinfo

kakao :
  client :
    clientId : -------- CLIENT ID ---------
    accessTokenUri: https://kauth.kakao.com/oauth/token
    userAuthorizationUri: https://kauth.kakao.com/oauth/authorize
  resource:
    userInfoUri: https://kapi.kakao.com/v1/user/me
```

각 소셜 미디어 개발자 센터를 통해 client id / secret을 발급 받고 설정파일에 세팅
<br>
<br>

/com/web/oauth/ClientResources.java
```java
public class ClientResources {
    @NestedConfigurationProperty
    private AuthorizationCodeResourceDetails client = new AuthorizationCodeResourceDetails();

    @NestedConfigurationProperty
    private ResourceServerProperties resource = new ResourceServerProperties();

    public AuthorizationCodeResourceDetails getClient() {
        return client;
    }

    public ResourceServerProperties getResource() {
        return resource;
    }
}
```
@NestedConfigurationProperty
: 해당 필드가 단일 값이 아닌 중복으로 바인딩된다고 표시하는 어노테이
: 소셜 미디어 3 곳의 프로퍼티를 각각 바인딩하므로 사용

@AuthorizationCodeResourceDetails
: 설정한 각 소셜의 프로퍼티 값 중 'client'를 기준으로 하위의 키/값을 매핑해주는 대

@ResourceServerProperties
: 객체는 원래 OAuth2 리소스 값을 매핑하는데 사용
: 예제는 회원 정보를 얻는 userInfoUri 값을 받는데 사용
<br>

## 5.4.2 시큐리티 + OAuth2 설정하기

/com/web/config/SecurityConfig.java
```java
@Configuration
@EnableWebSecurity
@EnableOAuth2Client
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private OAuth2ClientContext oAuth2ClientContext;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        http
            .authorizeRequests()
                .antMatchers("/","/login/**","/css/**","/images/**","/js/**","/console/**")
                .permitAll()
                .antMatchers("/facebook").hasAuthority(FACEBOOK.getRoleType())
                .antMatchers("/google").hasAuthority(GOOGLE.getRoleType())
                .antMatchers("/kakao").hasAuthority(KAKAO.getRoleType())
                .anyRequest().authenticated()
            .and()
                .headers().frameOptions().disable()
            .and()
                .exceptionHandling()
                .authenticationEntryPoint(new LoginUrlAuthenticationEntryPoint("/login"))
            .and()
                .formLogin()
                .successForwardUrl("/board/list")
            .and()
                .logout()
                .logoutUrl("/logout")
                .logoutSuccessUrl("/")
                .deleteCookies("JSESSIONID")
                .invalidateHttpSession(true)
            .and()
                .addFilterBefore(filter, CsrfFilter.class)
                .addFilterBefore(oauth2Filter(), BasicAuthenticationFilter.class)
                .csrf().disable();
    }

    @Bean
    public FilterRegistrationBean oauth2ClientFilterRegistration(OAuth2ClientContextFilter 
    filter) {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(filter);
        registration.setOrder(-100);
        return registration;
    }

    private Filter oauth2Filter() {
        CompositeFilter filter = new CompositeFilter();
        List<Filter> filters = new ArrayList<>();
        filters.add(oauth2Filter(facebook(), "/login/facebook", FACEBOOK));
        filters.add(oauth2Filter(google(), "/login/google", GOOGLE));
        filters.add(oauth2Filter(kakao(), "/login/kakao", KAKAO));
        filter.setFilters(filters);
        return filter;
    }

    private Filter oauth2Filter(ClientResources client, String path, SocialType socialType) {
        OAuth2ClientAuthenticationProcessingFilter filter 
        		= new OAuth2ClientAuthenticationProcessingFilter(path);
        OAuth2RestTemplate template 
        		= new OAuth2RestTemplate(client.getClient(), oAuth2ClientContext);

        filter.setRestTemplate(template);
        filter.setTokenServices(new UserTokenService(client, socialType));
        filter.setAuthenticationSuccessHandler((request, response, authentication) 
        		-> response.sendRedirect("/" + socialType.getValue() + "/complete"));
        filter.setAuthenticationFailureHandler((request, response, exception) 
        		-> response.sendRedirect("/error"));
        return filter;
    }

	/** 
    * 소셜 미디어 리소스 정보는 시큐리티 설정에서 사용하기 때문에 빈으로 등록
    */
    @Bean
    @ConfigurationProperties("facebook")
    public ClientResources facebook() {
        return new ClientResources();
    }

    @Bean
    @ConfigurationProperties("google")
    public ClientResources google() {
        return new ClientResources();
    }

    @Bean
    @ConfigurationProperties("kakao")
    public ClientResources kakao() {
        return new ClientResources();
    }
}
```
<br>
<br>

/com/web/oauth/UserTokenService.java
```java
public class UserTokenService extends UserInfoTokenServices {
    public UserTokenService(ClientResources resources, SocialType socialType) {
        super(resources.getResource().getUserInfoUri(), resources.getClient().getClientId());
        setAuthoritiesExtractor(new OAuth2AuthoritiesExtractor(socialType));
    }

    public static class OAuth2AuthoritiesExtractor implements AuthoritiesExtractor {
        private String socialType;

        public OAuth2AuthoritiesExtractor(SocialType socialType) {
            this.socialType = socialType.getRoleType();
        }

        @Override
        public List<GrantedAuthority> extractAuthorities(Map<String, Object> map) {
            return AuthorityUtils.createAuthorityList(this.socialType);
        }
    }
}
```
<br>

UserTokenService

     1. User 정보를 비동기 통신으로 가져오는 REST Service인 UserInfoTokenServices를 커스터 마이징

     2. 소셜 미디어 원격 서버와 통신하여 User 정보를 가져오는 로직은 이미 UserInfoTokenServices에 구현되어 있음

     3. URI와 clientId 정보를 생성자에서 super()를 사용하여 각각의 소셜 미디어 정보를 주입할 수 있도록 함

	 4. OAuth2AuthoritiesExtractor 클래스에 SocialType을 넘겨주어 권한 네이밍을 일괄적으로 처리하도록 설정

<br>

## 5.4.3 어노테이션 기반으로 User 정보 불러오기
##### 인증 처리 후 User 정보 세션 처리

<img width="550" alt="스크린샷 2019-04-03 오후 10 48 24" src="https://user-images.githubusercontent.com/34764544/55483985-9ea66700-5662-11e9-964f-c4740e9637bb.png">
<br>

/com/web/controller/LoginController.java
```java
@Controller
public class LoginController {
    @GetMapping("/login")
    public String login() {
        return "login";
    }

    @GetMapping(value = "/{facebook|google|kakao}/complete")
    public String loginComplete(@SocialUser User user) {
        return "redirect:/board/list";
    }
}
```
<br>

/com/web/UserArgumentResolver.java
```java
@Component
public class UserArgumentResolver implements HandlerMethodArgumentResolver {
    private UserRepository userRepository;

    public UserArgumentResolver(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    /**
    * @SocialUser 어노테이션이 있고 타입이 User인 파라미터만 true를 반환
    * true가 반환되면 resolveArgument메서드 실행
    */
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.getParameterAnnotation(SocialUser.class) != null && parameter.getParameterType().equals(User.class);
    }

	/**
    * 검증이 완료된 파라미터 정보를 받음
    * 이미 검증이 되어 세션에 해당 User 객체가 있으면 User 객체를 구성하는 로직을 태우지 않음
    * 세션은 RequestContextHolder를 사용해서 가져올 수 있음
    */
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        HttpSession session = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest().getSession();
        User user = (User) session.getAttribute("user");
        return getUser(user, session);
    }

    private User getUser(User user, HttpSession session) {
        if(user == null) {
            try {
                OAuth2Authentication authentication = (OAuth2Authentication) SecurityContextHolder.getContext().getAuthentication();
                Map<String, String> map = (HashMap<String, String>) authentication.getUserAuthentication().getDetails();
                User convertUser = convertUser(String.valueOf(authentication.getAuthorities().toArray()[0]), map);
                user = userRepository.findByEmail(convertUser.getEmail());
                if (user == null) { user = userRepository.save(convertUser); }
                setRoleIfNotSame(user, authentication, map);
                session.setAttribute("user", user);
            } catch (ClassCastException e) {
                return user;
            }
        }
        return user;
    }

    private User convertUser(String authority, Map<String, String> map) {
        if(FACEBOOK.isEquals(authority)) return getModernUser(FACEBOOK, map);
        else if(GOOGLE.isEquals(authority)) return getModernUser(GOOGLE, map);
        else if(KAKAO.isEquals(authority)) return getKaKaoUser(map);
        return null;
    }

    private User getModernUser(SocialType socialType, Map<String, String> map) {
        return User.builder()
                .name(map.get("name"))
                .email(map.get("email"))
                .pincipal(map.get("id"))
                .socialType(socialType)
                .createdDate(LocalDateTime.now())
                .build();
    }

    private User getKaKaoUser(Map<String, String> map) {
        HashMap<String, String> propertyMap = (HashMap<String, String>)(Object) map.get("properties");
        return User.builder()
                .name(propertyMap.get("nickname"))
                .email(map.get("kaccount_email"))
                .pincipal(String.valueOf(map.get("id")))
                .socialType(KAKAO)
                .createdDate(LocalDateTime.now())
                .build();
    }

    private void setRoleIfNotSame(User user, OAuth2Authentication authentication, Map<String, String> map) {
        if(!authentication.getAuthorities().contains(new SimpleGrantedAuthority(user.getSocialType().getRoleType()))) {
            SecurityContextHolder.getContext().setAuthentication(new UsernamePasswordAuthenticationToken(map, "N/A", AuthorityUtils.createAuthorityList(user.getSocialType().getRoleType())));
        }
    }
}
```
<br>

HandlerMethodArgumentResolver

	1. supportsParameter() : 해당하는 파라미터를 지원할지 여부를 반환, true를 반환하면 resolveArgument 메서드가 수행

	2. resolveArgument() : 파라미터의 인잣값에 대한 정보를 바탕으로 실제 객체를 생성하여 해당 파라미터 객체에 바인딩
<br>

#### UserArgumentResolver 등록하기
/com/web/BootWebApplication.java
```java
@SpringBootApplication
public class BootWebApplication extends WebMvcConfigurerAdapter {

	@Autowired
	private UserArgumentResolver userArgumentResolver;

	public static void main(String[] args) {
		SpringApplication.run(BootWebApplication.class, args);
	}

	@Override
	public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
		argumentResolvers.add(userArgumentResolver);
	}
}
```
WebMvcConfigurerAdapter 내부에 구현된 addArgumentResolvers를 overide하여 userArgumentResolver를 추가
<br>
<br>

## 5.4.4 인증 동작 확인하기
/resources/templates/login.html
```java
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>login</title>
    <link rel="stylesheet" th:href="@{/css/base.css}" />
    <link rel="stylesheet" th:href="@{/css/bootstrap.min.css}" />
</head>
<body>
    <div th:replace="layout/header::header"></div>

    <div class="container" style="text-align: center;">
        <br/>
        <h2>로그인</h2><br/><br/>
        <a href="javascript:;" class="btn_social" data-social="facebook"><img th:src="@{/images/facebook.png}" width="40px" height="40px"/></a>
        <a href="javascript:;" class="btn_social" data-social="google"><img th:src="@{/images/google.png}" width="40px" height="40px"/></a>
        <a href="javascript:;" class="btn_social" data-social="kakao"><img th:src="@{/images/kakao.png}" width="40px" height="40px"/></a>
    </div>

    <div th:replace="layout/footer::footer"></div>

    <script th:src="@{/js/jquery.min.js}"></script>
    <script>
        $('.btn_social').click(function () {
            var socialType = $(this).data('social');
            location.href="/login/"+socialType;
        });
    </script>

</body>
</html>
```
<br>
<br>

## 5.4.5 페이지 권한 분리하기
/com/web/config/SecurityConfig.java
```java
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private OAuth2ClientContext oAuth2ClientContext;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        http
            .authorizeRequests()
                .antMatchers("/", "/login/**",  "/css/**", "/images/**", "/js/**", "/console/**").permitAll()
                .antMatchers("/facebook").hasAuthority(FACEBOOK.getRoleType())
                .antMatchers("/google").hasAuthority(GOOGLE.getRoleType())
                .antMatchers("/kakao").hasAuthority(KAKAO.getRoleType())
                .anyRequest().authenticated()
            .and()
                .headers().frameOptions().disable()
            .and()
                .exceptionHandling()
                .authenticationEntryPoint(new LoginUrlAuthenticationEntryPoint("/login"))
            .and()
                .formLogin()
                .successForwardUrl("/board/list")
            .and()
                .logout()
                .logoutUrl("/logout")
                .logoutSuccessUrl("/")
                .deleteCookies("JSESSIONID")
                .invalidateHttpSession(true)
            .and()
                .addFilterBefore(filter, CsrfFilter.class)
                .addFilterBefore(oauth2Filter(), BasicAuthenticationFilter.class)
                .csrf().disable();
    }
    ...
}
```
<br>

antMatchers()

: 각각의 소셜 미디어용 경로를 지정

hasAuthority()

: 메서드의 파라미터로 원하는 권한을 전달하여 해당 권한을 지닌 사용자만 경로를 사용할 수 있도록 통제
