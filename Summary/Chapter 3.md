
# 스프링 부트 테스트

## 3.1 @SpringBootTest

- 통합 테스트를 제공하는 기본적인 스프링 부트 테스트 어노테이션

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

아래 예제는 에러가 발생 => value, properties는 함께 사용하면 안되기 때문


```java
@RunWith(SpringRunner.class)
@SpringBootTest(value = "value=test", properties = {"property.value=propertyTest"}, classes = {SpringBootTestApplication.class},
        webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class SpringBootTestApplicationTests {

    @Value("${value}")
    private String value;

    @Value("${property.value}")
    private String propertyValue;

    @Test
    public void contextLoads() {
        assertThat(value, is("test"));
        assertThat(propertyValue, is("propertyTest"));
    }
}
```
- value : 테스트가 실행되기 전에 적용할 프로퍼티 주입 가능, 기존의 프로퍼티를 오버라이드

- properties : 테스트가 실행되기 전에 {key=value} 형식으로 프로퍼티를 추가할 수 있음

- classes : 애플리케이션 컨텍스트에 로드할 클래스 지정 가능, 따로 지정하지 않으면 @SpringBootConfiguration을 찾아서 로드

- webEnvironment : 애플리케이션이 실행될 때의 웹 환경을 설정 가능, 기본값은 Mock 서블릿을 로드
<br>

### @SpringBootTest 추가 팁

- @ActiveProfiles("local")과 같은 방식으로 원하는 프로파일 환경 값 부여 가능 (QA, STAGE, REAL에 따른 코드)

- 테스트에서 @Transactional을 사용하면 테스트를 마치고나서 수정된 데이터가 롤백, 다만 테스타가 서버의 다른 스레드에서 실행 중이면 WebEnvironment의 RANDOM_PORT나 DEFINE_PORT를 사용하여 테스트를 수행해도 트랙잭션이 롤백되지 않음!

- @SpringBoottest는 기본적으로 검색 알고리즘을 사용, @SpringBootApplication / @SpringBootConfiguration 어노테이션을 찾음 ( 스프링 부트 테스트이기 때문 둘 중 하나 어노테이션은 필수 )
<br>

### 참고 
- spring-boot-test-autoconfigure : 테스트 스타터에 포함된 자동 설정 패키지를 사용하면 주제에 따라 가볍게 테스트 가능

- @...Test : 주제에 관련된 빈만 애플리케이션 컨텍스트에 로드
<br>

## 3.2 @WebMvcTest

- 웹에서 테스트하기 힘든 컨트롤러 테스트, MVC를 위한 테스트

- 웹 상 요청 / 응답 테스트, 시큐리티 관련 필터 자동 / 수동 테스트
<br>

```java
### BookController
@RunWith(SpringRunner.class)
@WebMvcTest(BookController.class)
public class BookControllerTest {

    @Autowired
    private MockMvc mvc;

    @MockBean
    private BookService bookService;

    @Test
    public void Book_MVC_테스트() throws Exception {
        Book book = new Book("Spring Boot Book", LocalDateTime.now());
        given(bookService.getBookList()).willReturn(Collections.singletonList(book));

        mvc.perform(get("/books"))
                .andExpect(status().isOk())
                .andExpect(view().name("book"))
                .andExpect(model().attributeExists("bookList"))
                .andExpect(model().attribute("bookList", contains(book)));
    }
}
```

- given() : getBookList 메서드 실행에 대한 반환값에 해당하는 Mock 객체 정의

- andExpect(status().isOk()) : HTTP 상태값 200인지 테스트

- andExpect(view().name("book)) : 반환되는 뷰의 이름이 'book'인지 테스트

- andExpect(model().attributeExists("bookList")) : 모델의 프로퍼티 중 'bookList'가 존재하는지

- andExpect(model().attribute("bookList", contains(book))) : 모델의 프로퍼티 중 'bookList'라는 프로퍼티에 book 객체가 담겨져 있는지 테스트
<br>

## 3.3 @DataJpaTest

- JPA 관련 테스트 설정만 로드

- JPA 데이터 조회 / 생성 / 수정 / 삭제 테스트

- 내장형 DB를 사용하여 테스트 DB로 테스트 가능

- [중요] JPA 테스트가 끝날 때마다 자동으로 테스트에 사용한 데이터를 롤백
<br>

```java
@RunWith(SpringRunner.class)
@DataJpaTest
@ActiveProfiles("...")
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class JpaTest {
}
```
<br>

### @AutoConfigurationTestDatabase
- Replace.Any (default) : 기본적으로 내장된 데이터 소스 사용
- Replace.NONE : @ActiveProfiles에 설정한 프로파일 환경 값에 따라 데이터 소스가 적용

    [참고] 
    application.yml 파일 : spring.test.database.replace : NONE으로 사용 가능
<br>

### @AutoConfigureTestDatabase(connection = H2)
connection의 옵션으로 H2, Derby, HSQL 등의 테스트 데이터베이스 종류 선택 가능
<br>

### TestEntityManager
기본적인 JPA 테스트 persist, flush, find 등을 할 수 있음


```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class BookJpaTest {
    private final static String BOOT_TEST_TITLE = "Spring Boot Test Book";

    @Autowired
    private TestEntityManager testEntityManager;

    @Autowired
    private BookRepository bookRepository;

    @Test
    public void Book_저장하기_테스트() {
        Book book = Book.builder()
                .title(BOOT_TEST_TITLE)
                .publishedAt(LocalDateTime.now())
                .build();

        testEntityManager.persist(book);

        assertThat(bookRepository.getOne(book.getIdx()), is(book));
    }

    @Test
    public void BookList_저장하고_검색_테스트() {
        Book book1 = Book.builder()
                .title(BOOT_TEST_TITLE + "1")
                .publishedAt(LocalDateTime.now())
                .build();

        testEntityManager.persist(book1);

        Book book2 = Book.builder()
                .title(BOOT_TEST_TITLE + "2")
                .publishedAt(LocalDateTime.now())
                .build();

        testEntityManager.persist(book2);

        Book book3 = Book.builder()
                .title(BOOT_TEST_TITLE + "3")
                .publishedAt(LocalDateTime.now())
                .build();

        testEntityManager.persist(book3);

        List<Book> bookList = bookRepository.findAll();

        assertThat(bookList, hasSize(3));
        assertThat(bookList, contains(book1, book2, book3));
    }

    @Test
    public void BookList_저장하고_삭제_테스트() {
        Book book1 = Book.builder()
                .title(BOOT_TEST_TITLE + "1")
                .publishedAt(LocalDateTime.now())
                .build();

        testEntityManager.persist(book1);

        Book book2 = Book.builder()
                .title(BOOT_TEST_TITLE + "2")
                .publishedAt(LocalDateTime.now())
                .build();

        testEntityManager.persist(book2);

        bookRepository.deleteAll();

        assertThat(bookRepository.findAll(), IsEmptyCollection.empty());
    }
}
```

- Book_저장하기_테스트() : testEntityManager로 persist() 기능이 정상 동작하는지 테스트

- BookList_저장하고_검색_테스트() : Book 3개를 저장한 후 저장된 Book의 개수가 3개가 맞는지, 저장된 Book에 각 Book 객체가 모두 포함되어 있는지 테스트

- BookList_저장하고_삭제_테스트() : 저장된 Book 중에서 2개가 제대로 삭제되었는지 테스트
<br>

## 3.4 @RestClientTest

- REST 관련 테스트를 도와주는 어노테이션

- REST 통신의 데이터 형으로 사용되는 JSON 형식이 예상대로 응답을 반환하는지 등을 테스트

```java
@RestController
public class BookRestController {

    @Autowired
    private BookRestService bookRestService;

    @GetMapping(path = "/rest/test", produces = MediaType.APPLICATION_JSON_VALUE)
    public Book getRestBooks() {
        return bookRestService.getRestBook();
    }
}
```

```java
@Service
public class BookRestService {

    private final RestTemplate restTemplate;

    public BookRestService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.rootUri("/rest/test").build();
    }

    public Book getRestBook() {
        return this.restTemplate.getForObject("/rest/test", Book.class);
    }
}
```
- RestTemplateBuilder는 RestTemplate을 핸들링하는 빌더 객체
: connectionTimeout, ReadTimeOut 설정뿐만 아니라 여러 다른 설정을 간편하게 제공

- getForObject() 메서드를 사용
: URI 요청을 보내고 응답을 Book 객체 형식으로 받아옴

```java
@RunWith(SpringRunner.class)
@RestClientTest(BookRestService.class)
public class BookRestTest {

    @Rule
    public ExpectedException thrown = ExpectedException.none();

    @Autowired
    private BookRestService bookRestService;

    @Autowired
    private MockRestServiceServer server;

    @Test
    public void rest_테스트() {
        this.server.expect(requestTo("/rest/test"))
                .andRespond(withSuccess(new ClassPathResource("/test.json", getClass()), MediaType.APPLICATION_JSON));

        Book book = this.bookRestService.getRestBook();

        assertThat(book.getTitle()).isEqualTo("테스트");
    }

    @Test
    public void rest_error_테스트() {
        this.server.expect(requestTo("/rest/test"))
                .andRespond(withServerError());

        this.thrown.expect(HttpServerErrorException.class);
        this.bookRestService.getRestBook();
    }
}
```

- RestClientTest : 테스트 대상이 되는 빈을 주입

- @Rule : @Before / @After 상관없이 메서드 끝날 때마다 정의한 값으로 초기화

- MockRestServiceServer : 클라이언트 / 서버 사이의 REST 테스트를 위한 객체, 목 객체와 같이 실제로 통신이 이루어지지는 않지만 지정한 경로에 예상되는 반환값 혹은 에러를 반환하도록 명시하여 간단하게 테스트 진행 가능

- /rest/test 경로로 요청을 보내면 현재 리소스 폴더에 생성되어 있는 test.json 파일의 데이터로 응답을 줌, test.json 파일은 반드시 test 디렉토리 하위 경로로 생성해아 테스트 메소드에서 읽을 수 있음
<br>

## 3.5 @JsonTest

- JSON 테스트를 지원하는 어노테이션 @JsonTest

- Gson, Jackson API의 테스트를 제공
<br>

```java
@RunWith(SpringRunner.class)
@JsonTest
public class BookJsonTest {

    @Autowired
    private JacksonTester<Book> json;

    @Test
    public void json_테스트() throws Exception {
        Book book = Book.builder()
                .title("테스트")
                .build();

        String content = "{\"title\":\"테스트\"}";

        assertThat(this.json.parseObject(content).getTitle()).isEqualTo(book.getTitle());

        assertThat(this.json.parseObject(content).getPublishedAt()).isNull();

        assertThat(this.json.write(book)).isEqualToJson("/test.json");

        assertThat(this.json.write(book)).hasJsonPathStringValue("title");

        assertThat(this.json.write(book)).extractingJsonPathStringValue("title").isEqualTo("테스트");

    }
}
```

- JacksonTester의 parseObject 메서드를 사용하여 문자열인 content를 객체로 변환
