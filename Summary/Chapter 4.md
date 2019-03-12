
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

### 4.3.2 스프링 부트 웹 스타터 살펴보기
