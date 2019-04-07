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
<br>
<br>
<br>

# 스프링 부트 2.0 기반의 OAuth2 설정하기

## 5.5.1 스프링 부트 2.0 버전으로 의존성 업그레이드

<img width="550" alt="스크린샷 2019-04-07 오후 8 20 35" src="https://user-images.githubusercontent.com/34764544/55682766-bb98ae00-5972-11e9-953d-9812266e478b.png">

- 2.0에서는 더 이상 dependency-management 플러그인을 자동으로 지원하지 않음
<br>

<img width="550" alt="스크린샷 2019-04-07 오후 8 23 54" src="https://user-images.githubusercontent.com/34764544/55682801-1b8f5480-5973-11e9-8102-2b5af392815a.png">

- 2.0버전 부터는 OAuth2 설정이 세분화 됨

- JWT와 관련한 권한을 안전하게 전송하기 위한 프레임워크 JOSE가 추가되었음

- JOSE는 JWT의 암호화/복호화 및 일정한 기능을 제공
<br>


## 5.5.2 스프링 부트 2.0 방식의 OAuth2 인증 재설정
<br>

스프링 부트 2.0 시큐리티 OAuth2의 스펙에는 여러 소셜 정보를 기본값으로 제공해주고 있음
<br>

/springframework/security/config/oauth2/client/CommonOAuth2Provider.java
```java
public enum CommonOAuth2Provider {

	GOOGLE {

		@Override
		public Builder getBuilder(String registrationId) {
			ClientRegistration.Builder builder = getBuilder(registrationId,
					ClientAuthenticationMethod.BASIC, DEFAULT_LOGIN_REDIRECT_URL);
			builder.scope("openid", "profile", "email");
			builder.authorizationUri("https://accounts.google.com/o/oauth2/v2/auth");
			builder.tokenUri("https://www.googleapis.com/oauth2/v4/token");
			builder.jwkSetUri("https://www.googleapis.com/oauth2/v3/certs");
			builder.userInfoUri("https://www.googleapis.com/oauth2/v3/userinfo");
			builder.userNameAttributeName(IdTokenClaimNames.SUB);
			builder.clientName("Google");
			return builder;
		}
	},

	GITHUB {

		@Override
		public Builder getBuilder(String registrationId) {
			ClientRegistration.Builder builder = getBuilder(registrationId,
					ClientAuthenticationMethod.BASIC, DEFAULT_LOGIN_REDIRECT_URL);
			builder.scope("read:user");
			builder.authorizationUri("https://github.com/login/oauth/authorize");
			builder.tokenUri("https://github.com/login/oauth/access_token");
			builder.userInfoUri("https://api.github.com/user");
			builder.userNameAttributeName("id");
			builder.clientName("GitHub");
			return builder;
		}
	},

	FACEBOOK {

		@Override
		public Builder getBuilder(String registrationId) {
			ClientRegistration.Builder builder = getBuilder(registrationId,
					ClientAuthenticationMethod.POST, DEFAULT_LOGIN_REDIRECT_URL);
			builder.scope("public_profile", "email");
			builder.authorizationUri("https://www.facebook.com/v2.8/dialog/oauth");
			builder.tokenUri("https://graph.facebook.com/v2.8/oauth/access_token");
			builder.userInfoUri("https://graph.facebook.com/me");
			builder.userNameAttributeName("id");
			builder.clientName("Facebook");
			return builder;
		}
	},

	OKTA {

		@Override
		public Builder getBuilder(String registrationId) {
			ClientRegistration.Builder builder = getBuilder(registrationId,
					ClientAuthenticationMethod.BASIC, DEFAULT_LOGIN_REDIRECT_URL);
			builder.scope("openid", "profile", "email", "address", "phone");
			builder.userNameAttributeName(IdTokenClaimNames.SUB);
			builder.clientName("Okta");
			return builder;
		}
	};

	private static final String DEFAULT_LOGIN_REDIRECT_URL = "{baseUrl}/login/oauth2/code/{registrationId}";

	protected final ClientRegistration.Builder getBuilder(String registrationId,
															ClientAuthenticationMethod method, String redirectUri) {
		ClientRegistration.Builder builder = ClientRegistration.withRegistrationId(registrationId);
		builder.clientAuthenticationMethod(method);
		builder.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE);
		builder.redirectUriTemplate(redirectUri);
		return builder;
	}

	/**
	 * Create a new
	 * {@link org.springframework.security.oauth2.client.registration.ClientRegistration.Builder
	 * ClientRegistration.Builder} pre-configured with provider defaults.
	 * @param registrationId the registration-id used with the new builder
	 * @return a builder instance
	 */
	public abstract ClientRegistration.Builder getBuilder(String registrationId);
}
```
<br>
<br>


/com/web/oauth2/CustomOAuth2Provider.java (카카오 정보를 담은 객체 생성)
```java
public enum CustomOAuth2Provider {

    KAKAO {
        @Override
        public ClientRegistration.Builder getBuilder(String registrationId) {
            ClientRegistration.Builder builder = getBuilder(registrationId, ClientAuthenticationMethod.POST, DEFAULT_LOGIN_REDIRECT_URL);
            builder.scope("profile");
            builder.authorizationUri("https://kauth.kakao.com/oauth/authorize");
            builder.tokenUri("https://kauth.kakao.com/oauth/token");
            builder.userInfoUri("https://kapi.kakao.com/v1/user/me");
            builder.userNameAttributeName("id");
            builder.clientName("Kakao");
            return builder;
        }
    };

    private static final String DEFAULT_LOGIN_REDIRECT_URL = "{baseUrl}/login/oauth2/code/{registrationId}";

    protected final ClientRegistration.Builder getBuilder(String registrationId, ClientAuthenticationMethod method, String redirectUri) {
        ClientRegistration.Builder builder = ClientRegistration.withRegistrationId(registrationId);
        builder.clientAuthenticationMethod(method);
        builder.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE);
        builder.redirectUriTemplate(redirectUri);
        return builder;
    }

    public abstract ClientRegistration.Builder getBuilder(String registrationId);

}
```
<br>
- 카카오 로그인 정보 등록
<br>
<br>
<br>

/resources/application.yml
```java
security:
    oauth2:
        client:
            registration:
                google:
                    client-id:
                    client-secret:
                facebook:
                    client-id:
                    client-secret:
custom:
    oauth2:
        kakao:
            client-id:
```
<br>

- 기본으로 등록되어 있는 정보를 수정하고 싶다면 프로퍼티에 새로 등록하는 방법으로 오버라이드하여 변경할 수 있음

- application.yml 파일에 ID와 Secret만 등록해주면 됨
<br>
<br>

/com/web/config/SecurityConfig.java

<img width="550" alt="스크린샷 2019-04-07 오후 9 06 43" src="https://user-images.githubusercontent.com/34764544/55683309-277e1500-5979-11e9-8aae-dc5afd43f5f3.png">

<img width="550" alt="스크린샷 2019-04-07 오후 9 07 02" src="https://user-images.githubusercontent.com/34764544/55683315-3cf33f00-5979-11e9-8cba-db088ca99910.png">

<img width="550" alt="스크린샷 2019-04-07 오후 9 07 07" src="https://user-images.githubusercontent.com/34764544/55683330-54322c80-5979-11e9-85f2-f83f74179845.png">

- 1.5 버전에서 작성한 OAuth2 관련 설정을 모두 삭제

- ouath2Login()만 추가로 설정하면 기본적으로 제공되는 구글과 페이스북에 대한 OAuth2 인증 방식이 적용됨
<br>
<br>

/com/web/config/SecurityConfig.java
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        http
            .authorizeRequests()
                .antMatchers("/", "/oauth2/**", "/login/**",  "/css/**", "/images/**", "/js/**", "/console/**").permitAll()
                .antMatchers("/facebook").hasAuthority(FACEBOOK.getRoleType())
                .antMatchers("/google").hasAuthority(GOOGLE.getRoleType())
                .antMatchers("/kakao").hasAuthority(KAKAO.getRoleType())
                .anyRequest().authenticated()
            .and()
                .oauth2Login()
                .defaultSuccessUrl("/loginSuccess")
                .failureUrl("/loginFailure")
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
                .csrf().disable();
    }
    
    /**
    * OAuth2ClientProerties와 설정했던 카카오 클라이언트 ID를 불러옴
    * @Configuration으로 등록되어 있는 클래스에서 @Bean으로 등록된 메서드의 파라미터로 지정된 객체들은 Autowiring 가능
    * OAuth2ClientProperties에는 구글과 페이스북의 정보가 들어 있고, 카카오는 따로 등록했기 때문에 @Value로 수동으로 불러옴
    */
    @Bean
    public ClientRegistrationRepository clientRegistrationRepository(OAuth2ClientProperties oAuth2ClientProperties, @Value("${custom.oauth2.kakao.client-id}") String kakaoClientId) {
        List<ClientRegistration> registrations = oAuth2ClientProperties.getRegistration().keySet().stream()
                .map(client -> getRegistration(oAuth2ClientProperties, client))
                .filter(Objects::nonNull)
                .collect(Collectors.toList());

        registrations.add(CustomOAuth2Provider.KAKAO.getBuilder("kakao")
                .clientId(kakaoClientId)
                .clientSecret("test") //필요없는 값인데 null이면 실행이 안되도록 설정되어 있음
                .jwkSetUri("test") //필요없는 값인데 null이면 실행이 안되도록 설정되어 있음
                .build());

        return new InMemoryClientRegistrationRepository(registrations);
    }
    
    /**
    * getRegistration() 메서드를 통해 구글과 페이스북 인증 정보를 빌드
    * registrations 리스트에 카카오 인증 정보를 추가
    * 실제 요청시 사용하는 정보는 클라이언트 ID뿐이지만 clientSecret()과 jwtSetUri()가 null이면 안되므로 값을 넣음
    * 페이스북의 그래프 API의 경우 scope()로는 필요한 필드를 반환해주기 않기 때문에 직접 id, name, email, link 등을 파리미터로 넣어 요청하도록 설정
    */
    private ClientRegistration getRegistration(OAuth2ClientProperties clientProperties, String client) {
        if ("google".equals(client)) {
            OAuth2ClientProperties.Registration registration = clientProperties.getRegistration().get("google");
            return CommonOAuth2Provider.GOOGLE.getBuilder(client)
                    .clientId(registration.getClientId())
                    .clientSecret(registration.getClientSecret())
                    .scope("email", "profile")
                    .build();
        }
        if ("facebook".equals(client)) {
            OAuth2ClientProperties.Registration registration = clientProperties.getRegistration().get("facebook");
            return CommonOAuth2Provider.FACEBOOK.getBuilder(client)
                    .clientId(registration.getClientId())
                    .clientSecret(registration.getClientSecret())
                    .userInfoUri("https://graph.facebook.com/me?fields=id,name,email,link")
                    .scope("email")
                    .build();
        }
        return null;
    }
}
```
<br>
<br>

/com/web/controller/LoginController.java
```java
@Controller
public class LoginController {

    @GetMapping("/login")
    public String login() {
        return "login";
    }

    @GetMapping("/loginSuccess")
    public String loginComplete() {
        return "redirect:/board/list";
    }
}
```
<br>
<br>


/com/web/resolver/UserArgumentResolver.java
```java
@Component
public class UserArgumentResolver implements HandlerMethodArgumentResolver {

    private UserRepository userRepository;

    public UserArgumentResolver(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.getParameterAnnotation(SocialUser.class) != null && parameter.getParameterType().equals(User.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) {
        HttpSession session = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest().getSession();
        User user = (User) session.getAttribute("user");
        return getUser(user, session);
    }

	/**
    * 2.0 버전에서는 액세스 토큰까지 제공한다는 의미로 OAuth2Authentication이 아닌 OAuth2AuthenticationToken을 지원
    * SecurityContextHolder에서 OAuth2AuthenticationToken을 가져옴
    */
    private User getUser(User user, HttpSession session) {
        if(user == null) {
            try {
                OAuth2AuthenticationToken authentication = (OAuth2AuthenticationToken) SecurityContextHolder.getContext().getAuthentication();
                Map<String, Object> map = authentication.getPrincipal().getAttributes();
                User convertUser = convertUser(authentication.getAuthorizedClientRegistrationId(), map);

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

    private User convertUser(String authority, Map<String, Object> map) {
        if(FACEBOOK.isEquals(authority)) return getModernUser(FACEBOOK, map);
        else if(GOOGLE.isEquals(authority)) return getModernUser(GOOGLE, map);
        else if(KAKAO.isEquals(authority)) return getKaKaoUser(map);
        return null;
    }

    private User getModernUser(SocialType socialType, Map<String, Object> map) {
        return User.builder()
                .name(String.valueOf(map.get("name")))
                .email(String.valueOf(map.get("email")))
                .pincipal(String.valueOf(map.get("id")))
                .socialType(socialType)
                .createdDate(LocalDateTime.now())
                .build();
    }

    private User getKaKaoUser(Map<String, Object> map) {
        Map<String, String> propertyMap = (HashMap<String, String>) map.get("properties");
        return User.builder()
                .name(propertyMap.get("nickname"))
                .email(String.valueOf(map.get("kaccount_email")))
                .pincipal(String.valueOf(map.get("id")))
                .socialType(KAKAO)
                .createdDate(LocalDateTime.now())
                .build();
    }

    private void setRoleIfNotSame(User user, OAuth2AuthenticationToken authentication, Map<String, Object> map) {
        if(!authentication.getAuthorities().contains(new SimpleGrantedAuthority(user.getSocialType().getRoleType()))) {
            SecurityContextHolder.getContext().setAuthentication(new UsernamePasswordAuthenticationToken(map, "N/A", AuthorityUtils.createAuthorityList(user.getSocialType().getRoleType())));
        }
    }
}
```
