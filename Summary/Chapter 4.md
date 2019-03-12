
# 스프링 부트 웹

## 4.1 커뮤니티 게시판 설계하기

MVC 패턴으로 사용자의 요청에 따른 데이터 처리
<br>


<img width="550" alt="스크린샷 2019-03-12 오후 9 33 54" src="https://user-images.githubusercontent.com/34764544/54200758-6ef6ba00-450f-11e9-90e3-cbca1e0e83c9.png">


- 클라이언트로 데이터를 요청받은 서버는 타임리프(Thymeleaf)를 사용하여 뷰를 구성하여 보여줌

- 간단한 CRUD (Create, Read, Update, Delete) 기능만 제공
<br>

<img width="550" alt="스크린샷 2019-03-12 오후 9 43 56" src="https://user-images.githubusercontent.com/34764544/54200999-f47a6a00-450f-11e9-832b-1992c196a65a.png">

### 타임리프란?
웹 또는 독립적인 실행 환경에서 사용되는 자바 서버 사이드 템플릿 엔진
<br>
<br>

### DB 테이블 설계

- Entity : 실체, 객체라는 의미 (유/무형의 객체로 서로 구별되는 것)

<img width="550" alt="스크린샷 2019-03-12 오후 9 51 50" src="https://user-images.githubusercontent.com/34764544/54201474-12949a00-4511-11e9-8231-17ee42109039.png">

- 게시판, 유저 1:1 관계
<br>
<br>
<br>
<br>

## 4.2 커뮤니티 프로젝트 준비하기

Web, 타임리프, JPA, DevTools, 롬복, H2 라이브러리(인메모리 DB) 사용
<br>
<br>
<br>
<br>

## 4.3 커뮤니티 게시판 구현하기

1. 프로젝트 의존성 구성
2. 스프링 부트 웹 스타터 살펴보기
3. 도메인 매핑하기
4. 도메인 테스트하기
5. CommandLineRunner를 사용하여 DB에 데이터 넣기
6. 게시글 리스트 기능 만들기
7. 타임리프 자바 8 날짜 포맷 라이브러리 추가하기
8. 페이징 처리하기
9. 작성 폼 만들기
<br>
<br>

### 4.3.1 프로젝트 의존성 구성

```java
buildscript {
	// 빌드 스크립트 내부의 버전, 의존 라이브러리, 저장소를 설정해 스프링 부트 플러그인을 사용할 수 있게 함
	ext {
		springBootVersion = '2.1.3.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

// 필요한 플러그인 적용
apply plugin: 'java'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	// spring
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'

	// lombok [컴파일 시점만 필요하고 런타임 시점에는 필요없을 때 compileOnly 사용]
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'

	// devtools
	runtime 'org.springframework.boot:spring-boot-devtools'

	// h2 [런타임 시점에만 H2 사용하도록 설정]
	runtime 'com.h2database:h2'

	// test
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```
<br>


#### 디렉토리 구조

<img width="550" alt="스크린샷 2019-03-12 오후 10 25 15" src="https://user-images.githubusercontent.com/34764544/54203500-baac6200-4515-11e9-81c9-bd9acb594a7d.png">
<br>

- domain : Model 또는 DTO(Data Transfer Oject) 클래스
- service : BO (Business Object) 클래스
- repository : DAO (Data Access Object) 클래스
<br>

### 4.3.2 스프링 부트 웹 스타터 살펴보기

<img width="550" alt="스크린샷 2019-03-12 오후 10 39 22" src="https://user-images.githubusercontent.com/34764544/54204515-b5e8ad80-4517-11e9-874d-fe2fe2ecb809.png">
<br>

1. spring-boot-starter : 스프링 부트를 시작하는 기본적인 설정이 담겨있는 스타터

2. spring-boot-starter-tomcat : 내장 톰캣을 사용하기 위한 스타터

3. hibernate-validator : 어노테이션 기반의 표준화된 제약 조건 및 유효성 검사 규칙을 표현하는 라이브러리

4. spring-boot-starter-json : jackson 라이브러리 지원 스타터

5. spring-web : HTTPIntegration, Servlet filters, Spirng HTTP invoikder 및 HTTP 코어를 포함시킨 라이브러리

6. spring-webmvc : request를 전달하는 MVC로 디자인된 DispatcherServlet 기반의 라이브러리
<br>

### 4.3.3 도메인 매핑하기

도메인 매핑 : JPA를 사용하여 DB와 도메인 클래스를 연결시켜주는 작업
<br>

<img width="550" alt="스크린샷 2019-03-12 오후 10 46 09" src="https://user-images.githubusercontent.com/34764544/54204943-a9b12000-4518-11e9-8bf0-5b2335a0c31f.png">
<br>

#### BoardType Enum 생성

```java
public enum BoardType {
    notice("공지사항"),
    free("자유게시판");

    private String value;

    BoardType(String value) {
        this.value = value;
    }

    public String getValue() {
        return this.value;
    }
}
```
<br>

#### Board 클래스 생성

```java
import com.example.SpringBootCommunityWeb.domain.enums.BoardType;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;
import java.io.Serializable;
import java.time.LocalDateTime;

@Getter
@NoArgsConstructor
@Entity
@Table
public class Board implements Serializable {

    @Id
    @Column
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long idx;

    @Column
    private String title;

    @Column
    private String subTitle;

    @Column
    private String content;

    @Column
    @Enumerated(EnumType.STRING)
    private BoardType boardType;

    @Column
    private LocalDateTime createDate;

    @Column
    private LocalDateTime updateDate;

    @OneToOne(fetch = FetchType.LAZY)
    private User user;

    @Builder
    public Board(String title, String subTitle, String content, BoardType boardType, LocalDateTime createDate, LocalDateTime updateDate, User user) {
        this.title = title;
        this.subTitle = subTitle;
        this.content = content;
        this.boardType = boardType;
        this.createDate = createDate;
        this.updateDate = updateDate;
        this.user = user;
    }
}
```
<br>
1. @GeneratedValue(strategy = GenerationType.IDENTITY)
: 기본 키가 자동으로 할당되도록 설정하는 어노테이션
: 스프링 1.X에서는 IDENTITY 전략이 Default였지만, 2.X부터는 명시적으로 TABLE로 변경되어 명시적으로 적어줘야함

2. @Enumerated(EnumType.STRING)
: Enum 타입 매핑용 어노테이션
: 실제로 자바 enum 형이지만 데이터베이스의 String형으로 변환하여 저장

3. @OneToOne(fetch=FetchType.Lazy)
: 1:1 관계로 설정하는 어노테이션
: DB에 저장될 때는 User 객체가 저장되는 것이 아니라 User의 PK인 user_idx 값이 저장됨
: FetchType eager/lazy가 있음, 전자는 Board 도메인을 조회할 때 즉시 관련 User 객체를 함께 조회, 후자는 User 객체를 조회하는 시점이 아닌 객체가 실제로 사용될 때 조회
