

# 스프링 부트 웹

## 4.1 커뮤니티 게시판 설계하기

- MVC 패턴으로 사용자의 요청에 따른 데이터 처리
<br>


<img width="550" alt="스크린샷 2019-03-12 오후 9 33 54" src="https://user-images.githubusercontent.com/34764544/54200758-6ef6ba00-450f-11e9-90e3-cbca1e0e83c9.png">


- 클라이언트로 데이터를 요청받은 서버는 타임리프(Thymeleaf)를 사용하여 뷰를 구성하여 보여줌

- 간단한 CRUD (Create, Read, Update, Delete) 기능만 제공
<br>

<img width="550" alt="스크린샷 2019-03-12 오후 9 43 56" src="https://user-images.githubusercontent.com/34764544/54200999-f47a6a00-450f-11e9-832b-1992c196a65a.png">

### 타임리프란?
웹 또는 독립적인 실행 환경에서 사용되는 자바 서버 사이드 템플릿 엔진
<br>


###




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
