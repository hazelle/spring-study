# Spring Framework

## 1. 스프링 IoC 컨테이너
스프링 IoC 컨테이너의 IoC는 Inversion of Control

### **IoC란?**
어떤 클래스 타입의 객체가 사용할 다른 객체(의존 객체)를 직접 만들어서 사용하는 것이 아니라 어떤 장치(생성자 등등)를 통해 주입받아서 사용하는 방법.   
Spring이 없어도 이런 장치만 마련되어 있다면 얼마든지 사용할 수 있는 방식.

스프링 IoC 컨테이너는 애플리케이션 컴포넌트의 중앙 저장소.   
Bean 설정 소스로부터 Bean 정의를 읽고, 구성하고, 제공한다.

Spring이 제공하는 IoC 컨테이너를 사용하는 걸 권장하는 이유는 여러 Dependency Injection 방법들을 고려해서 개발되었기 때문.

컨테이너라고 부르는 이유는 IoC 기능을 제공하는 여러 객체들을 가지고 있기 때문.

Annotation 기반의 Dependency 지원을 하고 있음
(`@Service`, `@Repository`, `@Autowired` 등등)

Annotation을 통해 일반적인 클래스를 Bean으로 쉽게 등록하고, 쉽게 주입해 사용할 수 있음

Spring IoC Container의 가장 최상단에 있는 인터페이스(가장 핵심)는 **BeanFactory**.  
가장 많이 사용하게 될 BeanFactory는 **ApplicationContext**.

ApplicationContext는 BeanFactory를 상속받았음.
BeanFactory의 기능 외에도
* ApplicationEventPublisher
* EnvironmentCapable
* HierarchicalBeanFactory
* ListableBeanFactory
* MessageSource : i18n이라고 부르는 internationalization(다국어화)를 지원. 메시지 소스를 사용할 수 있는 기능
* ResourceLoader : Class path에 있는 특정 파일, File System에 있는 파일, Web에 있는 파일 등 resource를 읽어오는 기능
* ResourcePatternResolver   
  
등등 다양한 기능이 있음.

### **Bean이란?**
스프링 IoC 컨테이너가 관리하는 객체.

`@Repository`, `@Serivce` 등등의 어노테이션이 붙으면 Auto-scan을 통해 해당 클래스가 Bean으로 등록된다.

왜 Bean으로 등록해서 사용해야할까?
 1. 의존성 주입(Dependency Injection. DI)을 받기 위해서
 2. Scope 때문. 애플리케이션 전체에서 Singleton으로 관리하고 싶은 객체(인스턴스)인 경우 
(Spring IoC 컨테이너에 등록되는 Bean들은 기본적으로 Singleton Scope으로 등록됨.)
 3. Lifecycle Interface를 지원해준다.

> #### 싱글톤 vs 프로토타입
> * 싱글톤: 1개만 만들어서 사용하는 객체
> * 프로토 타입: 매번 다른 객체를 만들어서 사용;/하는 객체

Bean으로 등록할 때 아무런 Annotation을 붙이지 않았다면 기본적으로 싱글톤 scope으로 등록된다.   
싱글톤은 프로토타입에 비해서 메모리 면에서 효율적이고, 단 하나의 객체만을 사용하기 때문에 Runtime 시 성능 최적화에도 유리하다.

DB와 통신하는 Repository 객체들의 경우 만드는 데에 비용이 비쌈 (메모리를 많이 씀)   
이런 객체들의 경우 싱글톤으로 사용했을 때 특히나 효율적이겠지~

Spring IoC 컨테이너에 등록된 Bean들은 Lifecycle interface를 지원해준다.   
어떤 Bean이 만들어졌을 때 추가적인 작업을 하고싶은 경우 사용 가능   
> ex) @PostConstruct : 객체가 생성되기 전에 실행될 메소드 지정!

### 1-1. ApplicationContext

스프링 IoC 컨테이너는 **Bean 설정 파일**이 필요하다.

application.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.sporingframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans">

    <bean id="bookService"
          class="com.ddd.ddddd.BookService">
        <property name="bookRepository" ref="bookRepository" />
        <!-- ref: reference라는 말로 다른 bean의 id 값이 옴 -->
    
    </bean>

    <bean id="bookRepository"
          class="com.ddd.ddddd.BookRepository">

    <!-- 
        ** bean 설정값들에 대한 내용
        id: camel case로 쓰는 것이 naming convention
        scope:
            prototype: 매번 새로운 객체를 생성해서 사용
            request: request당 새로운 객체를 생성해서 사용
            session: http sessiong당 새로운 객체를 생성해서 사용
            singleton (default): Application 내에서 객체 1개만 생성 (나머지는 모두 prototype)
        autowire:
            default
            byName: 이름에 따라
            byType: 타입에 따라
            constructor: 생성자
            no: 안 한다
     -->
</beans>
```

BookService.java
``` java
package com.ddd.ddddd;

public class BookService {
    
    BookRepository bookRepository;

    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

}

```

BookRepository.java
``` java
package com.ddd.ddddd;

public class BookRepository {  }
```

DemoApplication.java
``` java
package com.ddd.ddddd;

public class DemoApplication {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        String[] beanDefinitionNames = context.getBeanDefinitionNames();

        BookService bookService = (BookService) context.getBean("bookService");
        System.out.println(bookService.bookRepository != null);     // true

        // 아까 application.xml에서 설정한대로 BookSerivce 내의 BookRepository가 null이 아님을 확인할 수 있음
    }
}
```

하지만 이런 식으로 bean을 이용하면 너무 불편!   
아래의 방법으로 application.xml을 변경해보자.

application.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.sporingframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans">

    <context:component-scan base-package="com.ddd.ddddd" />
    <!--
        나는 base-package(com.ddd.ddddd) 패키지부터 그 이하에 있는 bean을 스캔해서 등록하겠다!
        이 때 스캔하는 bean의 기준은 `@Component`라는 어노테이션.

        Annotaion 기반으로 bean을 등록하고 설정하는 방법

        Spring 2.5부터 가능한 기능~!
    -->

</beans>
```

`@Component`를 확장받은 어노테이션들: `@Serivce`, `@Repository` 등등   

(앞의 BookService에는 `@Serivce`를, BookRepository에는 `@Repository`를 붙여준다.)

위의 어노테이션이 등록되면 해당 객체가 bean으로 등록된다.   
Dependency Injection은 안 됨! 그건 따로 해주어야 함.
> `@Autowired`, `@Inject`라는 어노테이션들을 통해 주입할 수 있음   
> 하지만 `@Inject`는 다른 의존성을 필요로하기 때문에 `@Autowired`를 사용!

---

bean 설정 파일을 xml이 아니고 java로도 만들 수 있다!

ApplicationConfig.java
``` java
package com.ddd.dddd;

import org.springframework.context.annotation.Configuration;

@Configuration // 이 자바 파일이 bean 설정 파일이라는 의미
public class ApplicationConfig {

    @Bean
    public BookRepository bookRepository() {
        return new BookRepository();
    }

    @Bean
    public BookService bookService() {
        // 혹은 bookRepository를 이 메소드의 파라미터로 받아서 그걸 set해주어도 됨
        BookService bookService new BookService();
        bookSerivce.setBookRepository(bookRepository());
        return bookService;
    }

    /*
        혹은 BookService에서 지금 BookRepository가 setter를 통해 주입되고 있기 때문에

        @Autowired
        BookRepository bookRepository;

        처럼 써서 Bean 등록할 때 BookRepository를 아예 세팅해주지 않아도 됨
        BookRepository가 bean으로 등록되어있기 때문에 주입됨

        BookRepository가 생성자(constructor)를 통해 주입되는 구조였다면 이 방법은 불가능.
    */

}
```

DemoApplication.java
``` java
package com.ddd.ddddd;

public class DemoApplication {

    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
        String[] beanDefinitionNames = context.getBeanDefinitionNames();

        BookService bookService = (BookService) context.getBean("bookService");
        System.out.println(bookService.bookRepository != null);     // true

        // 아까 application.xml에서 설정한대로 BookSerivce 내의 BookRepository가 null이 아님을 확인할 수 있음
    }
}
```

하지만 이 방법 또한 일일이 bean으로 등록해야 해서 번거로움.   
아까 xml에서의 component-scan처럼 패키지 단위로 등록하는 방법이 있음

ApplicationConfig.java
``` java
package com.ddd.dddd;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration // 이 자바 파일이 bean 설정 파일이라는 의미
@ComponentScan(basePackageClasses = DemoApplication.class)
/*
    basePackages: 직접 패키지명을 써주어야 함
        ex) @ComponentScan(basePackages = "com.ddd.ddddd")
    basePackageClasses: 지금 쓰는 class가 위치한 곳부터 그 이하에서 Component Scanning을 해라
        ex) @ComponentScan(basePackageClasses = DemoApplication.class)
*/
public class ApplicationConfig {

    @Bean
    public BookRepository bookRepository() {
        return new BookRepository();
    }

    @Bean
    public BookService bookService() {
        // 혹은 bookRepository를 이 메소드의 파라미터로 받아서 그걸 set해주어도 됨
        BookService bookService new BookService();
        bookSerivce.setBookRepository(bookRepository());
        return bookService;
    }

}
```

하지만... Spring Boot를 쓴다면...

DemoApplication.java
``` java
package com.ddd.ddddd;

import org.springframework.boot.autoconfigure.SpringBootApplication;

/*
    이 @SpringBootApplication 어노테이션 내에 이미
    @ComponentScan, @Configuration(@SpringBootConfiguration)이 포함되어 있음

    따라서 이 클래스 자체가 Bean 설정 파일임!
*/
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        
    }

}
```