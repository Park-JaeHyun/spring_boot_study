
# 스프링 부트 데이터 레스트

#### REST란?
웹의 장점을 극대화하는 통신 네트워크 아키텍처
<br>

#### RESTful이란?
REST의 구현 원칙을 제대로 지키는 시스템
<br>


REST 서버 : 단일 서버로 데이터를 관리하며 유연하게 클라이언트 영역을 대응할 수 있는 방법
<br>

#### REST (Representational State Transfer)

- Representation : 어떤 리소스의 특정 시점의 상태를 반영하고 있는 정보

- State : 웹 애플리케이션의 상태

- Transfer : Representation의 전송

웹 애플리케이션의 상태가 representation의 전송으로 변경됨
<br>

[참고]
https://blog.npcode.com/2017/04/03/rest%EC%9D%98-representation%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80/
<br>

## 6.1.1 REST 소개

#### REST란
웹과 같은 분산 하이퍼미디어 시스템에서 사용하는 통신 네트워크 아키텍처의 원리 모음

- 전송 방식 : HTTP (웹에서 GET, POST, PUT, DELETE 등의 메서드를 사용하여 정보를 주고받는 프로토콜)

- 식별 방법 : URI
<br>

#### REST의 목적

- 구성요소 상호작용의 규모 확장성

- 인터페이스의 범용성

- 구성요소의 독립적인 배포

- 레거시 시스템 인캡슐레이션
<br>
<br>

## 6.1.2 RESTful 제약 조건

#### RESTful이란?
REST의 구현 원칙을 제대로 지키면서 REST 아키텍처를 만드는 것
<br>
1. 클라이언트-서버 (client-server)
: 명확한 분리, 서버의 구성요소 단순화, 확장성이 향상되어 여러 플랫폼 지원 가능

2. 무상태성 (stateless)
: 서버에 클라이언트 상태 정보를 저장하지 않음

3. 캐시 가능 (cacheable)
: 클라이언트의 응답을 캐시 가능

4. 계층화 시스템 (layered system)
: 서버는 중개 서버(게이트웨이, 프록시)나 로드 밸런싱, 공유 캐시 등의 기능을 사용하여 확장성 있는 시스템을 구성할 수 있음

5. 코드 온 디맨드 (code on demand)
: 클라이언트는 서버에서 자바 애플릿, 자바스크립트 실행 코드를 전송받아 기능을 일시적으로 확장시킬 수 있음

6. 인터페이스 일관성 (uniform interface)
: URI로 지정된 리소스에 균일하고 통일된 인터페이스 제공, 아키텍처를 단순하게 분리하여 독립적으로 만들 수 있음
<br>

#### 인터페이스 일관성
인터페이스 일관성에는 다음 4가지 프로퍼티가 존재

1. 자원 식별
: 웹 기반의 REST에서 리소스 접근은 주로 URI를 사용한다는 것을 나타냄

2. 메시지를 통한 리소스 조작
: content-type은 리소스가 어떤 형식인지 지정. HTML, XML, JSON 등 다양한 형식으로 리소스 제공

3. 자기 서술적 메시지
: 웹 기반 REST에서는 HTTP Method와 Header를 활용 (GET, POST, PUT, DELETE 메서드)

4. 애플리케이션 상태에 대한 엔진으로서의 하이퍼미디어 (HATEOAS)
: HATEOAS(애플리케이션 상태에 대한 엔진으로서의 하이퍼미디어)
: 클라이언트에 응답할 때 단순히 결과 데이터만 제공해주기보다는 URI를 함께 제공해야 함
<br>
<br>

## 6.1.3 REST API 설계하기

#### REST API 구성

1. 자원(resource) : URI

2. 행위(verb) : HTTP 메서드

3. 표현(representations) : 리소스에 대한 표현 (HTTP Mesaage Body)

#### URI 설계
URL : 웹상의 파일 위치를 표현함, 리소스를 가져오는 방법에 대한 위치
ex) http://localhost:8080/api/book.pdf

URI : 웹에 있는 자원의 이름과 위치를 식별, 문자열을 식별하기 위한 표준
ex) http://localhost:8080/api/book/1
<br>
<br>

## 6.2 설계하기

#### 6.2.1 MVC 패턴을 활용하는 방법
컨트롤러, 서비스(BO), 리포지토리(DAO)로 나누어 데이터 운반 처리를 세부적으로 조작할 수 있음

기존의 웹과 거의 차이가 없고 단지 데이터의 반환 형태가 HTML이냐 JSON, XML 등의 형태냐의 차이

### Client <-> Controller <-> Service <-> Repository <-> DB
<br>


#### 6.2.2 스프링 부트 데이터 레스트를 활용하는 방법
스프링 부트 데이터 레스트는 리포지토리 하나만 생성하면 됨

스프링 부트 데이터 레스트 라이브러리 내부에 미리 만들어져 있어 컨트롤러와 서비스 영역을 자동화할 수 있음

### Client <-> REST Repository <-> DB
<br>
<br>

## 6.3 스프링 부트 MVC 패턴으로 REST API 구현하기

#### 6.3.1 준비하기
커뮤니티 게시판과 연동하려면 클라이언트 쪽에 자바스크립트로 통신용 코드를 구현해야함
=> Ajax를 이용해 비동기로 서버와 통신할 수 있음

<img width="550" alt="스크린샷 2019-04-18 오전 12 00 19" src="https://user-images.githubusercontent.com/34764544/56298387-041f4b00-616d-11e9-8e00-92f1d1a69b15.png">
<br>
<br>

##### MySQL 세팅
application.yml

<img width="423" alt="스크린샷 2019-04-18 오전 12 02 09" src="https://user-images.githubusercontent.com/34764544/56298513-3c268e00-616d-11e9-8cae-37cd99864470.png">
<br>

##### 모듈 세팅
MVC 패턴의 'rest-web' 모듈, 레스트 방식의 'data-rest'로 설정
<br>

setting.gradle
```java
rootProject.name = 'boot-rest'

include 'data-rest'
include 'rest-web'
```
<br>

build.gradle
<img width="550" alt="스크린샷 2019-04-18 오전 12 05 27" src="https://user-images.githubusercontent.com/34764544/56298782-bd7e2080-616d-11e9-9ea5-7f5492d2fd32.png">

<img width="550" alt="스크린샷 2019-04-18 오전 12 05 34" src="https://user-images.githubusercontent.com/34764544/56298804-c7a01f00-616d-11e9-81dc-aee475927c9b.png">
<br>

##### REST API 프로젝트 디렉토리 구조
<img width="561" alt="스크린샷 2019-04-18 오전 12 05 43" src="https://user-images.githubusercontent.com/34764544/56298908-f6b69080-616d-11e9-8f85-d84fdb755bad.png">
<br>
<br>

#### 6.3.2 REST API구현하기
8080 포트 : 기존의 커뮤니티 게시판
8081 포토 : REST API 
<br>

##### DataSource 및 포트 설정
/rest-web/src/main/application.yml
```java
spring:
	datasource:
    	url: jdbc:mysql://127.0.0.1:3306 (DB명)
        username : {아이디}
        password : {패스워드}
        driver-class-name: com.mysql.jdbc.Driver
server:
	port: 8081
```
<br>
<br>

##### BoardRestController 생성
/rest-web/src/main/java/com/community/rest/controller/BoardRestController.java
```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PageableDefault;
import org.springframework.hateoas.PagedResources;
import org.springframework.hateoas.PagedResources.PageMetadata;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import static org.springframework.hateoas.mvc.ControllerLinkBuilder.linkTo;
import static org.springframework.hateoas.mvc.ControllerLinkBuilder.methodOn;

@RestController
@RequestMapping("/api/boards")
public class BoardRestController {
    private BoardRepository boardRepository;

    public BoardRestController(BoardRepository boardRepository) {
        this.boardRepository = boardRepository;
    }

    @GetMapping(produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<?> getBoards(@PageableDefault Pageable pageable) {
        Page<Board> boards = boardRepository.findAll(pageable);
        PageMetadata pageMetadata = new PageMetadata(pageable.getPageSize(), boards.getNumber(), boards.getTotalElements());
        PagedResources<Board> resources = new PagedResources<>(boards.getContent(), pageMetadata);
        resources.add(linkTo(methodOn(BoardRestController.class).getBoards(pageable)).withSelfRel());
        return ResponseEntity.ok(resources);
    }

    @PostMapping
    public ResponseEntity<?> postBoard(@RequestBody Board board) {
        //valid 체크
        board.setCreatedDateNow();
        boardRepository.save(board);
        return new ResponseEntity<>("{}", HttpStatus.CREATED);
    }

    @PutMapping("/{idx}")
    public ResponseEntity<?> putBoard(@PathVariable("idx")Long idx, @RequestBody Board board) {
        //valid 체크
        Board persistBoard = boardRepository.getOne(idx) ;
        persistBoard.update(board);
        boardRepository.save(persistBoard);
        return new ResponseEntity<>("{}", HttpStatus.OK);
    }

    @DeleteMapping("/{idx}")
    public ResponseEntity<?> deleteBoard(@PathVariable("idx")Long idx) {
        //valid 체크
        boardRepository.deleteById(idx);
        return new ResponseEntity<>("{}", HttpStatus.OK);
    }
}
```
<br>

##### HATEOAS 링크 예제
<img width="559" alt="스크린샷 2019-04-18 오전 12 21 05" src="https://user-images.githubusercontent.com/34764544/56299880-de477580-616f-11e9-8bf4-4993808c88de.png">
<br>

##### Board 객체의 페이징 처리된 데이터
<img width="398" alt="스크린샷 2019-04-18 오전 12 21 59" src="https://user-images.githubusercontent.com/34764544/56299961-033be880-6170-11e9-9c12-f4b8438d2bee.png">

<img width="427" alt="스크린샷 2019-04-18 오전 12 23 15" src="https://user-images.githubusercontent.com/34764544/56300048-2b2b4c00-6170-11e9-92b4-cd79e9d6d477.png">


#### 6.3.3 CORS 허용 및 시큐리티 설정
Ajax 통신용 코드에서 <script></script> 태그로 둘러싸인 부분은 HTTP 요청 시 동일 출처 정책이 적용

따라서 8080포트(웹), 8081(REST API 프로젝트)은 호스트가 동일할지라도 포트가 상이하기 때문에 Ajax 요청은 실패

즉, 출처는 자원 + 도메인 + 포트번호("http://localhost:8080")로 결합된 문자열
<br>

##### 교차 출처 자원 공유
교차 출처 (cross-origin) HTTP 요청을 가능하게 해주는 매커니즘
<br>

##### 교차 출처 방식의 시퀀스 다이어그램
<img width="575" alt="스크린샷 2019-04-18 오전 12 28 00" src="https://user-images.githubusercontent.com/34764544/56300378-d50ad880-6170-11e9-94ba-7441cd946d1b.png">
<br>

##### CORS 적용
/rest-web/src/main/java/com/community/rest/RestWebApplication.java
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.ArrayList;
import java.util.List;

@SpringBootApplication
public class RestWebApplication {

	public static void main(String[] args) {
		SpringApplication.run(RestWebApplication.class, args);
	}

	@Configuration
	@EnableGlobalMethodSecurity(prePostEnabled = true)
	@EnableWebSecurity
	static class SecurityConfiguration extends WebSecurityConfigurerAdapter {

		@Override
		protected void configure(HttpSecurity http) throws Exception {
			CorsConfiguration configuration = new CorsConfiguration();
			configuration.addAllowedOrigin(CorsConfiguration.ALL);
			configuration.addAllowedMethod(CorsConfiguration.ALL);
			configuration.addAllowedHeader(CorsConfiguration.ALL);
			UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
			source.registerCorsConfiguration("/**", configuration);

			http.httpBasic()
					.and().authorizeRequests()
					.anyRequest().permitAll()
					.and().cors().configurationSource(source)
					.and().csrf().disable();
		}
	}
}
```
<br>
<br>

## 6.4 스프링 부트 데이터 레스트로 REST API 구현하기

MVC 패턴
- 컨트롤러와 서비스 영역을 두면 세부적인 처리가 가능하고 복잡한 서비스도 처리할 수 있음

- But 복잡한 로직 없이 단순 요청을 받아 데이터를 있는 그대로 반환할 때는 비용 낭비가 됨!
</br>

데이터 레스트
- MVC 패턴에서 VC를 생략

- 복잡한 로직 없이 단순 요청을 반아 데이터를 반환할 때 좋음

- Domain, Repository로만 Rest Api를 제공하기 때문에 빠르고 쉽게 프로젝트 진
</br>
</br>

#### 6.4.1 준비하기
/data-rest/src/main/resources/application.yml
```java
spring:
  datasource:
    url: jdbc:mysql://172.16.135.15:3306/autoconfiguration
    username: mns
    password: dltmxmdlsxjspt
    driver-class-name: com.mysql.jdbc.Driver
  data:
    rest:
      // API의 모든 요청의 기본 경로를 지정
      base-path: /api
      // 클라이언트가 따로 페이지 크기를 요청하지 않았을 때 적용할 기본 페이지 크기
      default-page-size: 10
      // 최대 페이지 수를 지정
      max-page-size: 10
server:
  port: 8081
```
</br>

##### 스프링 부트 데이터 레스트 프로퍼티
- page-param-name
페이지를 선택하는 쿼리 파라미터명을 변경

- limit-param-name
페이지 아이템 수를 나타내는 쿼리 파라미터명을 변경

- sort-param-name
페이지의 정렬값을 나타내는 쿼리 파라미터명을 변경

- default-media-type
미디어 타입을 지정하지 않았을 때 사용할 기본 미디어 타입

- return-body-on-update
엔티티를 수정한 이후응답 바디 반환 여부를 설정

- enable-enum-translation
'rest-message'라는 프로퍼티 파일을 만들어서 지정한 enum 값을 사용하게 해줌

- detection-strategy
리포지토리 노출 전략을 설정하는 프로퍼티
ALL : 모든 유형의 리포지토리를 노출
DEFAULT : pulbic으로 설정된 모든 리포지토리를 노출
ANNOTATION : @RestResouce가 설정된 리포지토리만 노출
VISIBILITY : public으로 설정된 인터페이스만 노출

</br>


#### 6.4.3 스프링 부트 데이터 레스트로 REST API 구현하기

@RepositoryRestResource
- 스프링 부트 데이터 레스트에서 지원하는 어노테이션

- 별도의 컨트롤러와 서비스 영역 없이 미리 내부적으로 정의
<br>

/data-rest/src/main/java/com/commnunity/rest/repository/BoardRepository.java
```java
@RepositoryRestResource
public interface BoardRepository extends JpaRepository<Board, Long> {
}
```
<br>

/data-rest/src/main/java/com/commnunity/rest/repository/UserRepository.java
```java
@RepositoryRestResource
public interface UserRepository extends JpaRepository<Board, Long> {
}
```
<br>
프로퍼티를 설정 후 도메인, 레포지토리만 구현하면 다른 영역은 스프링 부트 데이터 레스트가 알아서 처리해줌
<br>

##### 테스트
- $ curl http://localhost:8081/api/boards

<img width="550" alt="스크린샷 2019-05-01 오후 6 15 51" src="https://user-images.githubusercontent.com/34764544/57011588-59fbf480-6c3d-11e9-91d9-918000c4a92b.png">

<img width="550" alt="스크린샷 2019-05-01 오후 6 16 11" src="https://user-images.githubusercontent.com/34764544/57011621-90397400-6c3d-11e9-87bb-3f6f5f95557d.png">

<br>
내부 "_links" : 해당 Board와 관련된 링크 정보를 포함
외부 "_links" : Board의 페이징 처리와 관련된 링크 정보를 포함

MVC 패턴을 활용한 방법보다 더 많은 링크 정보를 제공

이러한 정보는 key:value 형식으로 구성되어 있음
=> 클라이언트가 키를 참조하도록 코드를 설정한다면 서버엣 요청된 데이터의 정보가 바뀌더라도 클라이언트 입장에서 코드를 수정할 필요가 없음
</br>

#### 6.4.4 @RepositoryRestController를 사용하여 REST API 구현하기

모든 사람이 위와 같은 데이터형을 원하지는 않음
- @RepositoryRestController를 사용하면 좋음
</br>

두 가지 주의사항
- 매핑하는 URL 형식이 스프링 부트 데이터 레스트에서 정의하는 REST API 형식에 맞아야 함

- 기존에 기본으로 제공하는 URL 형식과 같게 제공해야 해당 컨트롤러의 메서드가 기존의 기본 API를 오버라이드 함
</br>

```java
@RepositoryRestController
public class BoardRestController {

    private BoardRepository boardRepository;

    public BoardRestController(BoardRepository boardRepository) {
        this.boardRepository = boardRepository;
    }

	// 스프링 부트 데이터 레스트에서 기본적으로 제공해주는 URL 형식을 오버라이드
    @GetMapping("/boards")
    public @ResponseBody Resources<Board> simpleBoard(@PageableDefault Pageable pageable) {
        Page<Board> boardList = boardRepository.findAll(pageable);
		// 전체 페이지 수, 현재 페이지 번호, 총 게시판 수 등의 페이지 정보를 담는 PageMetadata 객체를 생성
        PageMetadata pageMetadata = new PageMetadata(pageable.getPageSize(), boardList.getNumber(), boardList.getTotalElements());
        // 컬렉션의 페이지 리소스 정보를 추가적으로 제공해주는 PagedResource 객체를 만들어 반환값으로 사용
        PagedResources<Board> resources = new PagedResources<>(boardList.getContent(), pageMetadata);
// 필요한 링크 추가        resources.add(linkTo(methodOn(BoardRestController.class).simpleBoard(pageable)).withSelfRel());
        return resources;
    }
}
```
</br>

##### 테스트
- $ curl http://localhost:8081/api/boards

<img width="550" alt="스크린샷 2019-05-01 오후 6 33 32" src="https://user-images.githubusercontent.com/34764544/57012078-a5170700-6c3f-11e9-90c0-0a43553cd204.png">

 스프링 부트 데이터 레스트에서 제공해주는 기본 URL은 @RepositoryRestContorller를 사용하여 오버라이드 가능
<br>
<br>

#### 6.4.6 프로젝션으로 노출 필드 제한하기

스프링 부트 데이터 레스트는 반환값을 제어하는 3가지 방법을 제공

- @JsonIgnore 추가

- @Projection 사용

- 프로젝션을 수동으로 등록
<br>

1. @JsonIgnore로 필드 반환값에서 제거
```java
@Column
@JsonIgnore
private String password;
```
<br>

2. @Projection로 필드 반환값에서 제거
프로젝션 인터페이스 생성 시 반드시 해당 도메인 클래스와 같은 패키지 경로 또는 하위 패키지 경로에 생성해야 함
```java
@Projection(name = "getOnlyName", types = {User.class})
	public interface UserOnlyContainName {
	String getName();
}
```
```java
@RepositoryRestResource(excerptProjection = UserOnlyContainName.class)
public interface UserRepository extends JpaRepository<User, Long> {
}
```
<br>

##### 테스트
 $ curl http://localhost:8081/api/boards
<img width="550" alt="스크린샷 2019-05-01 오후 6 46 09" src="https://user-images.githubusercontent.com/34764544/57012433-684c0f80-6c41-11e9-9adf-889b8f961c54.png">
<br>

3. 수동으로 프로젝션을 등록
수동 등록 시에는 반드시 프로젝션 타깃이 될 도메인과 동일하거나 하위에 있는 패키지 경로로 들어가야 함
```java
	@Configuration
	public class CustomizedRestMvcConfiguration extends RepositoryRestConfigurerAdapter {

	@Override
	public void confiugreRepositoryRestConfiguration(RepositoryRestConfiguration config) {
	config.getProjectionConfiguration().addProjection(UserOnlyContainName.class);
	}
}
```
<br>

#### 6.4.7 각 메서드 권한 제한

사용자에 따라 다른 권한을 부여해야 함

스프링 부트 데이터 레스트 프로젝트는 스프링 시큐리티와의 호환을 통해 이 문제를 해결

@Secured, @PreAuthorize를 사용

- @Secured : 순수하게 롤 기반으로 접근을 제한

- @PreAuthorize : @Secured보다 더 효율적으로 권한 지정을 할 수 있음
<br>

ROLE_ADMIN 권한을 갖고 있어야만 Board를 저장할 수 있는 예제
```java
@Projection(name = "getOnlyTitle", types = { Board.class })
public interface BoardOnlyContainTitle {

    String getTitle();
}
```

```java
@RepositoryRestResource(excerptProjection = BoardOnlyContainTitle.class)
public interface BoardRepository extends JpaRepository<Board, Long> {

	// ROLE_ADMIN 권한을 가진 사용자만 Board를 저장할 수 있음
    @Override
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    <S extends Board> S save(S entity);
}
```
<br>
<br>

#### 6.4.8 이벤트 바인딩

스프링 부트 데이터 레스트에서는 여러 메서드의 이벤트 발생 시점을 가로채서 원하는 데이터를 추가하거나 검사하는 이벤트 어노테이션을 제공

##### 이벤트 어노테이션
- BeforeCreateEvent
생성하기 전의 이벤트

- AfterCreateEvent
생성한 후의 이벤트

- BeforeSaveEvent
수정하기 전의 이벤트

- AfterSaveEvent
수정한 후의 이벤트

- BeforeDeleteEvent
삭제하기 전의 이벤트

- AfterDeleteEvent
삭제한 후의 이벤트

- BeforeLinkSaveEvent
관계를 가진 (1:1, M:M) 링크를 수정하기 전의 이벤트

- AfterLinkSaveEvent
관계를 가진 (1:1, M:M) 링크를 수정한 후의 이벤트

- BeforeLinkDeleteEvent
관계를 가진 (1:1, M:M) 링크를 삭제하기 전의 이벤트

- AfterLinkDeleteEvent
관계를 가진 (1:1, M:M) 링크를 삭제한 후의 이벤트
<br>

각 이벤트는 @Hanlde + 이벤트명 형태로 지원
<br>


게시글 생성 시 생성한 날짜와 수정한 날짜를 서버에서 설정하도록 하는 BoardEventHandler라는 클래스를 생성
```java
@RepositoryEventHandler
public class BoardEventHandler {

	// 게시글의 생성 날짜를 현재 시간으로 할당
    @HandleBeforeCreate
    public void beforeCreateBoard(Board board) {
        board.setCreatedDateNow();
    }

	// 게시글 수정 시 수정 날짜를 현재 시간으로 할당
    @HandleBeforeSave
    public void beforeSaveBoard(Board board) {
        board.setUpdatedDateNow();
    }
}
```
<br>

선언한 이벤트를 어떻게 적용?

1. 수동으로 이벤트를 적용하는 ApplicationListener를 사용하는 방법
-> AbstractRepositoryEventListsener를 상속받고 관련 메서드를 오버라이드하여 원하는 이벤트만 등록할 수 있음

2. 어노테이션을 기반으로 하는 이벤트 처리 방법
-> 생성한 이벤트 핸들러 클래스에는 @RepositoryEventHandler 어노테이션이 선언되어 있어야 함
-> 이벤트 핸들러를 등록하려면 @Component를 사용하거나 직접 ApplicationContext에 빈으로 등록해야 함
<br>

DtataRestApplication 클래스에 직접 빈으로 등록
```java
@SpringBootApplication
public class DataRestApplication {

	public static void main(String[] args) {
		SpringApplication.run(DataRestApplication.class, args);
	}

	@Bean
	BoardEventHandler boardEventHandler() {
		return new BoardEventHandler();
	}
```
<br>

##### 테스트
```java
@RunWith(SpringRunner.class)
// 스프링 부트 데이터 레스트를 테스트하기 위해 시큐리티 설정이 들어 있는 DataRestApplication 클래스 주입, 포트도 정의되어 있는 8081을 동일하게 사용하기 위해 DEFINED_PORT로 지정하여 사용
@SpringBootTest(classes = DataRestApplication.class, webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)

// @AutoConfigureTestDatabase : H2가 build.gradle의 클래스 경로에 포함되어 있으면 H2를 테스트 데이터 베이스로 지정, 만약 이 어노테이션을 사용하지 않는다면 테스트 Board를 저장할 때마다 실제 데이터베이스에 반영
@AutoConfigureTestDatabase
public class BoardEventTest {

	// TestRestTemplate : RestTemplate을 래핑한 객체로서 GET, POST, PUT, DELETE와 같은 HttpRequest를 편하게 테스트하도록 도와줌
    private TestRestTemplate testRestTemplate = new TestRestTemplate("havi", "test");

	// 테스트
    @Test
    public void 저장할때_이벤트가_적용되어_생성날짜가_생성되는가() {
        Board createdBoard = createBoard();
        assertNotNull(createdBoard.getCreatedDate());
    }

	// 테스트
    @Test
    public void 수정할때_이벤트가_적용되어_수정날짜가_생성되는가() {
        Board createdBoard = createBoard();
        Board updatedBoard = updateBoard(createdBoard);
        assertNotNull(updatedBoard.getUpdatedDate());
    }

    private Board createBoard() {
        Board board = Board.builder().title("저장 이벤트 테스트").build();
        return testRestTemplate.postForObject("http://127.0.0.1:8081/api/boards", board, Board.class);
    }

    private Board updateBoard(Board createdBoard) {
        String updateUri = "http://127.0.0.1:8081/api/boards/1";
        testRestTemplate.put(updateUri, createdBoard);
        return testRestTemplate.getForObject(updateUri, Board.class);
    }
}
```
<br>

#### 6.4.9 URI 처리

basePath만 설정하면 게시판의 기본 접속 URI는 아래와 같음
http://localhost:8081/api/boards

URI로 요청하는 모든 검색 쿼리 메서드는 search 하위로 표현됨
다음과 같은 기본 설정 URI를 갖고 있음
http://localhost:8081/api/boards/search
<br>

제목을 찾는 쿼리 메서드 생성
```java
@RepositoryRestResource(excerptProjection = BoardOnlyContainTitle.class)
public interface BoardRepository extends JpaRepository<Board, Long> {

    @Override
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    <S extends Board> S save(S entity);

    @RestResource
    List<Board> findByTitle(@Param("title") String title);
}
```

추가된 제목을 찾는 쿼리를 호출하는 URI를 다음과 같이 표현

http://localhost:8081/api/boards/search/findByTitle?title=게시글1

@RestResource의 path를 설정하지 않으면 기본값에 해당 메서드명이 적용됨

다음과 같이 'query'로 값을 변경하여 path 값을 다르게 줄 수 있음
```java
@RepositoryRestResource(excerptProjection = BoardOnlyContainTitle.class)
public interface BoardRepository extends JpaRepository<Board, Long> {

    @Override
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    <S extends Board> S save(S entity);

    @RestResource(path = "query")
    List<Board> findByTitle(@Param("title") String title);
}
```

http://localhost:8081/api/boards/search/query?title=게시글1
<br>
<br>

특정 리포지토리, 쿼리 메서드, 필드를 노출하고 싶지 않은 상황에는?

```java
@RepositoryRestResource(exported = false)
@RestResource(exported = false)
```
