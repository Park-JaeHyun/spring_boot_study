

# 스프링 부트 웹

## 4.1 커뮤니티 게시판 설계하기

- MVC 패턴으로 사용자의 요청에 따른 데이터 처리


<img width="526" alt="스크린샷 2019-03-12 오후 9 33 54" src="https://user-images.githubusercontent.com/34764544/54200758-6ef6ba00-450f-11e9-90e3-cbca1e0e83c9.png">





- 기본 제공 테스트 클래스명은 프로젝트명에 Tests를 붙인 형태로 자동 생성

    ####  장점
    - 실제 구동되는  애플리케이션과 동일하게 컨텍스트를 로드하여 하고 싶은 테스트 모두 수행 가능

    #### 단점
     - 애플리케이션에 설정된 빈을 모두 로드하기 때문에, 애플리케이션 규모가 클수록 느림
<br>

### 기본 테스트
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootTestApplicationTests {

	    @Test
	    public void contextLoads() {
	    }
}
```
<br>

### @RunWith

- JUnit에 내장된 Runner를 사용하는 대신 어노테이션에 정의된 Runner를 사용

- @SpringBootTest 어노테이션을 사용하려면 @RunWith(SpringRunner.class)를 꼭 붙여야함
<br>


### SpringBootTest 어노테이션 프로퍼티
