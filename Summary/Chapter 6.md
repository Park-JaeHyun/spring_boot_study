
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
