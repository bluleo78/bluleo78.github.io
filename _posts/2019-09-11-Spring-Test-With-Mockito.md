---
title: Spring Unit Test with Mockito
categories: 
  - Spring
---

스프링 프로젝트에서 유닛 테스트를 구성하는 경우, 대상 테스트 대상 빈(Bean)의 로직만 검증하고자 할 때, 모키토(Mockito)를 이용한다.

모킹은 대상 테스트 빈에서 사용하는 빈에 대해서 처리를 한다.
예를 들어 테스트 대상 빈 A가 내부에서 빈 B를 사용하고 있을 때, B 에 대해서 모킹 처리를 하여 빈 B' 를 만들고, 이를 빈 A 에 강제 주입하는 방식이다.

그리고 테스트 케이스에서는 모킹된 빈에서 제공하는 메소드 호출에 대한 가상의 결과를 기술해 주고, 테스트 대상 빈 메소드를 호출하면 된다.


스프링 빈 목킹
---
스프링부트에서는 @MockBean 이라는 어노테이션을 지원하여 아래처럼 매우 쉽게 스프링 빈을 모킹할 수 있다.
```
public class MockTest {

	@MockBean
	B mockB;

	@Autowired
	A a;
	
	@Test
  public void testA()
  {
    ...
  }
}
```

스프링부트에서는 아닌 경우에는 몇가지 방법이 제공된다.
* ReflectionTestUtils 사용하여 의존하는 빈을 모킹하고, 이를 테스트 대상 빈의 필드에 강제 설정하는 방법
* @InjectMocks 사용하여 의존하는 빈을 강제 주입하는 방법
* @Configuration 과 @Profile 을 사용하여 특정 프로파일에서 의존하는 빈을 모킹하여 만든 빈의 우선 순위를 높여 강제 주입되게 하는 방법

이 중 첫번째 ReflectionTestUtils 을 제외한 나머지 방법들은 다른 테스트 클래스와 같이 테스트를 수행하면 문제가 발생한다.

다음은 ReflectionTestUtils 을 사용하여 스프링 빈을 모킹하는 방법이다.
```
public class MockTest {

	@Mock
	B mockB;

	@Autowired
	A a;

	B backupB;
	
	@Before
	public void setup() {
		MockitoAnnotations.initMocks(this);
		backupDao = (RenewalRequestDao) ReflectionTestUtils.getField(a, "b", mockB);
 		ReflectionTestUtils.setField(a, "b", dao);
	}

	
	@After
	public void tearDown() {
		ReflectionTestUtils.setField(a, "b", backupB);
	}

	
	@Test
  public void testA()
  {
    ...
  }
}
	
```



참고
---
* <>