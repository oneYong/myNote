## RestApi Test

### pom.xml 추가
	<dependency>
		<groupId>com.jayway.restassured</groupId>
		<artifactId>rest-assured</artifactId>
		<version>2.4.1</version>
		<scope>test</scope>
	</dependency>
	
**선행작업은 생략!!**

	@RunWith(SpringJUnit4ClassRunner.class)
	@SpringApplicationConfiguration(classes = App.class)
	@WebAppConfiguration
	@IntegrationTest({"server.port:0",
 				  	"spring.datasource.url:jdbc:h2:mem:bookmark;DB_CLOSE_ON_EXIT=FALSE"})
 	public class Test {
 		@Autowired
		CustomerRepository customerRepository;

		@Value("${local.server.port}")
		int port;
		Customer customer1;
		Customer customer2;
		
		@Before
		public void setUp(){
			customerRepository.deleteAll();
			customer1 = new Customer();
			customer1.setFirstName("Taro");
			customer1.setLastName("Yamada");
			customer2 = new Customer();
			customer2.setFirstName("Ichiro");
			customer2.setLastName("Suzuki");
	
			customerRepository.save(Arrays.asList(customer1, customer2));
			RestAssured.port = port;
	
		}
		
		@Test
		public void testGetCustomers() throws Exception {
			when().get("/api/customers")
					.then()
					.statusCode(HttpStatus.OK.value())
					.body("numberOfElements", is(2))
					.body("content[0].id", is(customer2.getId()))
					.body("content[0].firstName", is(customer2.getFirstName()))
					.body("content[0].lastName", is(customer2.getLastName()))
					.body("content[1].id", is(customer1.getId()))
					.body("content[1].firstName", is(customer1.getFirstName()))
					.body("content[1].lastName", is(customer1.getLastName()));
		}

		@Test
		public void testPostCustomers() throws Exception {
			Customer customer3 = new Customer();
			customer3.setFirstName("Nobita");
			customer3.setLastName("Nobi");
	
			given().body(customer3)
					.contentType(ContentType.JSON)
					.and()
					.when().post("/api/customers")
					.then()
					.statusCode(HttpStatus.CREATED.value())
					.body("id", is(notNullValue()))
					.body("firstName", is(customer3.getFirstName()))
					.body("lastName", is(customer3.getLastName()));
	
			when().get("/api/customers")
					.then()
					.statusCode(HttpStatus.OK.value())
					.body("numberOfElements", is(3));
		}

		@Test
		public void testDeleteCustomers() throws Exception {
			when().delete("/api/customers/{id}", customer1.getId())
					.then()
					.statusCode(HttpStatus.NO_CONTENT.value());
	
			when().get("/api/customers")
					.then()
					.statusCode(HttpStatus.OK.value())
					.body("numberOfElements", is(1));
		}
		
 	}
 	
 	
### Comment

	
	@RunWith(SpringJUnit4ClassRunner.class)
: JUnit 애너테이션인 @RunWith에  SpringJUnit4ClassRunner.class를 지정하면 JUnit 테스트에서 스프링의 기능을 사용할 수 있다.

	@SpringApplicationConfiguration(classes = App.class)
: @SpringAllicationConfiguration으로 테스트용 ApplicationContext를 만듭니다(@EnableAutoConfiguration을 붙인 클래스를 지정합니다).

	@WebAppConfiguration
: @WebAppConfiguration으로 웹 애플리케이션 테스티임을 알립니다.
  이 애너테이션과 @SpringApplicationConfiguration을 조합하면 내장 서버를 실행할 수 있습니다.
  
  
	@IntegrationTest({"server.port:0",
 		"spring.datasource.url:jdbc:h2:mem:bookmark;DB_CLOSE_ON_EXIT=FALSE"})
: @IntegrationTest로 통합 테스트 기능을 활성화합니다. value 속성으로 테스트할 때 사용할 속성을 덮어 쓸 수 있습니다. server.port 속성에 테스트용 서버 포트를 지정합니다.
  0 을 지정하면 현재 비어 있는 포트를 사용할 수 있습니다. 이 기능은 다른 프로세스가 사용하고 있는 포트를 중복 사용해서 테스트가 실패하는 일이 일어나지 않게 하므로 아주 편리합니다.
  또한 실제 애플리케이션에서 데이터가 지속되는 데이터베이스를 사용하고 있다면 테스트에서만 인 메모리 데이터베이스를 선택적으로 사용할 수 있습니다.
  
  	@Value("${local.server.port}")
: 위에서 사용한 포트 번호를 주입합니다. 속성 값을 주입하려면 @Value("${속성 이름}") 형식으로 씁니다.


[ 가장빨리 만나는 스프링부트 -마키 토시아키 - 중에서 ]