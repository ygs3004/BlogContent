

@Test 어노테이션의 패키지를 확인해주자



``` java
import org.junit.jupiter.api.Test; // jupiter package, JUnit 5

@SpringBootTest
```

-----

```java
import org.junit.Test; // JUnit 4

@SpringBootTest
@RunWith(SpringRunner.class)
```




org.junit.jupiter.api.Test 패키지는 JUnit 5 를 사용하며 해당 어노테이션을 쓰지 않아도 동작하지만  
org.junit.Test 패키지는 JUnit 4 를 사용하며 @RunWith(SpringRunner.class) 가 필수이다.

사이드 프로젝트 중 @Test 어노테이션 만으로도 동작하는 것을 확인하고 새로운 테스트 클래스를 만든후에 스프링 빈이 주입되지 않아 당황하였다. 스프링 버젼 자체를 최신으로 해서 RunWith 어노테이션이 필요하지 않아진줄 알았지만 Test 어노테이션의 패키지가 영향을 준다는 사실을 확인했다.