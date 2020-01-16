스프링 IoC 컨테이너의 IoC는 Inversion of Control

#### IoC
어떤 클래스 타입의 객체가 사용할 다른 객체(의존 객체)를 직접 만들어서 사용하는 것이 아니라 어떤 장치(생성자 등등)를 통해 주입받아서 사용하는 방법.
Spring이 없어도 이런 장치만 마련되어 있다면 얼마든지 사용할 수 있는 방식.

애플리케이션 컴포넌트의 중앙 저장소.
Bean 설정 소스로부터 Bean 정의를 읽고, 구성하고, 제공한다.

Spring이 제공하는 IoC 컨테이너를 사용하는 이유는 여러 Dependency Injection 방법들을 고려해서 개발되었기 떄문.

컨테이너라고 부르는 이유는 IoC 기능을 제공하는 여러 객체들을 가지고있기 때문.

Annotation 기반의 Dependency 지원을 하고 있음
@Service, @Repository, @Autowired 등등

일반적인 클래스를 Annotation을 통해 Bean으로 쉽게 등록하고, 쉽게 주입해 사용할 수 있음

Spring IoC Container의 가장 최상단에 있는 인터페이스(가장 핵심)는 **BeanFactory**.
가장 많이 사용하게 될 BeanFactory는 **ApplicationContext**.

ApplicationContect는 BeanFactory를 상속받았음.
BeanFactory의 기능 외에도
* ApplicationEventPublisher
* EnvironmentCapable
* HierarchicalBeanFactory
* ListableBeanFactory
* MessageSource : i18n이라고 부르는 internationalization(다국어화)를 지원. 메시지 소스를 사용할 수 있는 기능
* ResourceLoader : Class path에 있는 특정 파일, File System에 있는 파일, Web에 있는 파일 등 resource를 읽어오는 기능
* ResourcePatternResolver
등등 다양한 기능이 있음.

#### Bean
스프링 IoC 컨테이너가 관리하는 객체.

@Repository, @Serivce 등등의 어노테이션이 붙으면 Auto-scan을 통해 해당 클래스가 Bean으로 등록된다.

왜 Bean으로 등록해서 사용해야할까?
> 1. 의존성 주입(Dependency Injection. DI)을 받기 위해서
> 2. Scope 때문. 애플리케이션 전체에서 Singleton으로 관리하고 싶은 객체(인스턴스)인 경우 
(Spring IoC 컨테이너에 등록되는 Bean들은 기본적으로 Singleton Scope으로 등록됨.)
> 3. Lifecycle Interface를 지원해준다.

> #### 싱글톤 vs 프로토타입
> * 싱글톤: 1개만 만들어서 사용하는 객체
> * 프로토 타입: 매번 다른 객체를 만들어서 사용하는 객체

Bean으로 등록할 때 아무런 Annotation을 붙이지 않았다면 기본적으로 싱글톤 scope으로 등록된다.
싱글톤은 프로토타입에 비해서 메모리 면에서 효율적이고, 단 하나의 객체만을 사용하기 때문에 Runtime 시 성능 최적화에도 유리하다.

DB와 통신하는 Repository 객체들의 경우 만드는 데에 비용이 비쌈 (메모리를 많이 씀)
이런 객체들의 경우 싱글톤으로 사용했을 때 특히나 효율적이겠지~

Spring IoC 컨테이너에 등록된 Bean들은 Lifecycle interface를 지원해준다.
어떤 Bean이 만들어졌을 때 추가적인 작업을 하고싶은 경우 사용 가능
ex) @PostConstruct : 객체가 생성되기 전에 실행될 메소드 지정!

