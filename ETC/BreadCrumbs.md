# :bread: 빵 부스러기

>개발 관련 학습 중 하나의 글로 작성하기엔 짧고,  
버리기엔 아까운 부스러기 정보들을 모아두는 곳.
-----
## 목차
1. 클래스라는 개념이 등장한 기본적인 이유
2. Primitive Type과 Class의 차이점
3. URL 유효성 체크시, 해당 리소스에서 HTTP HEAD 메소드를 막아 둔 경우
4. 다형성(Polymorphism)과 동적 바인딩(Dynamic binding)
5. Type Parameter와 new 연산자
6. Spring에서 생성자 주입 방식이 권장되는 이유
7. Annotation을 이용한 간단한 Spring AOP 사용 예시
8. [오늘은 더 이상 보지 않기] 버튼 구현
9. 리버스 프록시
10. 몽키 패치
11. SpringBoot DevTools의 동작 원리
12. Getter/Setter 메소드명 차이 (Lombok vs IDE자동생성)
13. 프로젝트 생성시 groupId, artifactId 설정 방법
14. Spring Boot CommandLineRunner, ApplicationRunner
15. 프로젝트 외부에서 application.yml 설정하기
16. ~~ActiveMQ의 Virtual Destinations를 활용한 로드밸런싱~~ -> :bread: 되서 나감
17. @InitBinder 사용하기
18. 기본 생성자의 접근 레벨을 Protected/Private로 설정해서 Setter 사용하지 않기
19. OneToMany 관계에서 fetch join 적용할 경우 limit 쿼리가 동작하지 않는 문제
-----
</br>

## 1. 클래스라는 개념이 등장한 기본적인 이유
예를 들어, 전화번호부에서 한 사람의 이름과 전화번호는 항상 붙어다녀야 하는 데이터이다.  
그런데 이 데이터를 별개의 변수에 저장하면, 이름 데이터를 옮길 때 마다 전화번호 데이터도 따로 옮겨줘야 한다.  
만약 이름과 전화번호 외에 더 많은 데이터를 저장해야 한다면, 더 불편해질 것이다.  
그래서 **서로 관련있는 데이터들을 하나의 단위로 묶어두기 위해** 등장한 것이 클래스라는 개념이다.
- Reference - [Java로 배우는 자료구조 - 권오흠](https://www.inflearn.com/course/java-%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/%EB%AC%B8%EC%9E%90%EC%97%B4-%EB%8B%A4%EB%A3%A8%EA%B8%B0-3/)

-----
</br>

## 2. Primitive Type과 Class의 차이점
Class도 하나의 Type인데, int 혹은 double 형 변수를 선언하고 사용하는 것 처럼 User 변수를 선언하고 사용한다.
  - int count = 0;
  - User user = new User();
  
이 때, count라는 이름의 변수에는 그 안에 정수값 0이 저장되지만,  
user라는 이름의 변수는 그 안에 사용자(User)의 정보가 저장되지 않는다.  
사용자의 정보를 저장할 User객체(object)는 new 명령으로 따로 만들고,  
변수 user에는 따로 만든 User 객체의 주소(참조)를 저장하는 것이다.

Primitive 타입이 아닌 모든 변수는 참조 변수다.
- Reference - [Java로 배우는 자료구조 - 권오흠](https://www.inflearn.com/course/java-%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/%EB%AC%B8%EC%9E%90%EC%97%B4-%EB%8B%A4%EB%A3%A8%EA%B8%B0-3/)

-----
</br>

## 3. URL 유효성 체크시, 해당 리소스에서 HTTP HEAD 메소드를 막아 둔 경우

URL을 입력하면 해당 컨텐츠의 이미지와 제목을 파싱하는 프로그램 개발 중,  
아래와 같이 사용자가 입력한 URL의 유효성을 체크하는 코드가 있었다.

~~~java
...

URL url = new URL(inputUrl);

HttpURLConnection huc = (HttpURLConnection) url.openConnection();
huc.setRequestMethod("HEAD");
huc.setRequestProperty("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) " +
        "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36");
huc.connect();

if(huc.getResponseCode() == HttpURLConnection.HTTP_OK) {
  // 통과
}else
  // 유효하지 않은 URL
~~~ 

위의 코드는 URL이 해당 resource로 제대로 접속되는지 체크하기 위해 HttpURLConnection를 사용했고,  
Http 응답코드가 200 OK를 반환하는지만 빠르게 확인하기 위해 HEAD 메소드로 접속을 시도했다.  

여러 테스트 케이스에서 잘 동작하는듯 보였지만,  
정상적인 URL인 https://meetup.toast.com/posts/86 로 테스트 한 결과,  
접속을 시도해도 403 Forbidden 을 반환하며 접속이 실패했다. 그리고 모든 meetup.toast의 게시물들이 403 Forbidden을 반환했다.

알아 본 결과, **보안상의 이유로 GET메소드나 POST 메소드만 오픈해두는 경우가 있다**는 사실을 알게 되었다.  
그래서 HEAD 메소드로 접속을 시도했을 때, 응답코드로 403 Forbidden을 반환하는 경우  
GET 메소드로 다시 접속을 시도하게끔 코드를 수정했고,  정상적인 결과를 얻을 수 있었다.

-----
</br>

## 4. 다형성(Polymorphism)과 동적 바인딩(Dynamic binding)
다형성을 한마디로 정의하면 **'수퍼클래스 타입의 변수가 서브클래스 타입의 객체를 참조할 수 있다.'** 이다.

~~~java
Computer theComputer = new Notebook(...)
~~~

위의 코드에서 처럼 수퍼클래스인 Computer 타입의 참조변수가 서브클래스인 Notebook 객체를 참조할 수 있는 것이다.

그런데 만약 Computer 클래스의 toString() 메소드가 있고,  
그것을 오버라이딩한 toString() 메소드가 서브클래스인 Notebook 클래스에 있다면,  
아래 코드에서 test.toString()의 toString() 메소드는 어느 메소드일까?

~~~java
public class Notebook extends Computer  {
    
    ...
    
    public String toString() {
        ...
    }
    
    public static void main(String[] args) {
        Computer test = new Notebook(...);
        System.out.println( test.toString() );
    }
~~~

***Static binding*** 의 경우, Computer 객체의 toString() 메소드이고,  
***Dynamic binding*** 의 경우, Notebook 객체의 toString() 메소드이다.  

Static binding은 컴파일러가 어떤 메소드일지 결정하는 것이고,  
Dynamic binding은 런타임에 해당 코드를 실행할 때 어떤 메소드인지 결정하는 방식이다.  

>그리고, **Java는 항상 동적 바인딩(Dynamic binding)을 한다.**  

즉, 위의 코드에서 `test.toString()` 은 Notebook 객체의 toString() 메소드가 호출된다.

- Reference - [Java로 배우는 자료구조 - 권오흠](https://www.inflearn.com/course/java-%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/lecture/7458)

-----
</br>

## 5. Type Parameter와 new 연산자

~~~java
public void addFirst( T item) {

    Node<T> newNode = new Node<T>(item); // possible.
                                         // T = Type Parameter
                                         // T를 Type Parameter로 가지는 객체를 new 연산자로 생성할 수 있다. 하지만,

    T t = new T(); // impossible.
                   // Type Parameter는 가상 클래스이므로 T 타입의 객체를 new 연산자로 생성할 수 없고,
                   
    T[] array = new T[100]; // impossible.
                            // T타입의 배열도 생성할 수 없으며,
                            
    Node<T>[] arr = new Node<T>[100]; // impossible.
                                      // Type Parameter T를 가지는 배열을 new 연산자로 생성할 수도 없다.
}
~~~
    
-----
</br>

## 6. Spring에서 생성자 주입 방식이 권장되는 이유
: ***필수적으로 사용해야하는 레퍼런스 없이는 인스턴스를 만들지 못하도록 강제할 수 있다.***  

예를 들어, 아래 코드에서 OwnerController 클래스는 ownerRepository 없이는 제대로 동작할 수 없다.

~~~java
@Controller
class OwnerController {

    private OwnerRepository ownerRepository;
    
    public OwnerController(OwnerRepository ownerRepository) {
        this.ownerRepository = ownerRepository;
    }
~~~
}

하지만 @Autowired를 사용하는 Field injection 이나 Setter injection은  
ownerRepository 없이도 OwnerController 인스턴스를 만들 수 있다.  

이는 단점이 될 수도 있고, 장점이 될 수 있다.  
예를 들어 순환참조가 일어나면 생성자 주입방식은 양쪽 다 인스턴스를 생성할 수 없게 되지만,  
Field injection이나 Setter injection은 순환참조와 상관 없이 동작할 수 있다.  
하지만 순환참조가 일어나지 않게끔 코드를 작성하는 것이 바람직하다.

- Reference - [예제로 배우는 스프링 입문 8. 의존성 주입](https://www.youtube.com/watch?v=IVzYerodIyg&list=PLfI752FpVCS8_5t29DWnsrL9NudvKDAKY&index=8)

-----
</br>

## 7. Annotation을 이용한 간단한 Spring AOP 사용 예시

메서드의 실행시간을 측정하는 기능이 여러 메서드에 필요하다고 가정해보자.  
이 기능이 필요한 모든 메서드에 같은 코드를 붙이는 것은 문제가 많으므로 AOP를 적용할 수 있다.

우선, AOP를 적용할 메서드들에 Annotation을 붙인다. (아직 만들어지지 않은 Annotation)

~~~java
@LogExecutionTime
public String someMethod1() {
    ...
    return something1;
}

@LogExecutionTime
public String someMethod2() {
    ...
    return something2;
}
~~~

그리고 Annotation(@LogExecutionTime)을 아래와 같이 생성한다.  

~~~java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
}
~~~

`@Target` 으로 `어디에 사용할 지` 설정.  
`@Retention` 으로 어노테이션 정보를 `언제까지 유지할 것인지` 설정.

이제 어노테이션이 붙은 메서드에 어떤 기능을 넣을지 작성한다.

~~~java
@Component
@Aspect
public class LogAspect {
    
    Logger logger = LoggerFactory.getLogger(LogAspect.class);
    
    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        
        Object proceed = joinPoint.proceed();
        
        stopWatch.stop();
        logger.info(stopWatch.prettyPrint());
        
        return proceed;
    }
}
~~~

위의 코드에서 `joinPoint`가 우리가 생성한 @LogExecutionTime 어노테이션이 붙은 메서드이다.  
그러므로 `joinPoint.proceed();` 부분에서 메서드를 실행하고 그 결과를 반환하되,  
메서드 실행 시간을 측정하는 코드를 메서드 위아래로 붙여주는 것이다.

위 코드의 경우, @LogExecutionTime 이 붙은 메서드들이 실행되면 각 실행시간이 콘솔에 찍힌다.  
Spring 내부에서 프록시 패턴을 사용하는 방식인데, 어떻게 적용되는 것인지는 토비의 스프링3에 자세히 나와있다.

- Reference - [예제로 배우는 스프링 입문, 11 스프링 @AOP](https://www.youtube.com/watch?v=3750wh1wNuY&list=PLfI752FpVCS8_5t29DWnsrL9NudvKDAKY&index=11)

-----
</br>

## 8. [오늘은 더 이상 보지 않기] 버튼 구현

공고를 팝업으로 띄워야하는 업무가 있었다.  
마크업을 보니 `하루 동안 열지 않습니다. [ ]` 버튼을 누르면 오늘 하루는 더 이상 팝업이 나타나지 않도록 구현해야 했다.

기존의 팝업창 로직을 보니
`$.cookie`라는 함수를 사용하여 쿠키값을 변경하는 방법으로 구현했다.  
하지만 `$.cookie is not a function` 이라는 오류를 맞이했는데,  
[이 답변](https://stackoverflow.com/questions/18024539/jquery-cookie-is-not-a-function)에서 알 수 있듯이 $.cookie는 일반적인 jQuery의 함수가 아니라 다운받아야 하는 플러그인이었다.

`jquery.cookie.js` 파일을 추가하고,  
팝업창의 스타일 속성에 `display:none` 을 추가한 후에, 아래와 같은 코드로 팝업창을 제어했다.  

~~~js
<script type="text/javascript">
   
    (function() {
        if($.cookie('merge_notice_popup') !== "true") {
            $('#home_layer_wrap').show();
        }

	var bCheckedNeverOpen = false;
	$('#check').click(function(){
		bCheckedNeverOpen = !bCheckedNeverOpen;
	});
	
	$('.btn_close').click(function(){
            if (bCheckedNeverOpen) {
                $.cookie('merge_notice_popup', 'true', { expires: 1, path: '/' });
                $.cookie('merge_notice_popup', 'true', { expires: 1, path: '/index' });
                $.cookie('merge_notice_popup', 'true', { expires: 1, path: '/Index' });
            }
            $('#home_layer_wrap').hide();
			return false;
	});
    })();	
</script>
~~~

우선, 즉시 실행 함수 `(function(){...})();`로 구현하여 해당 페이지가 호출되면,  
즉시 실행되어 쿠키 유무를 확인하여 팝업창을 보이거나 숨긴다.

그리고 `하루 동안 열지 않습니다.[]` 버튼에 체크하면 boolean 변수를 true로 바꾸고,  
체크된 상태로 닫기 버튼이 클릭되었을 때 `merge_notice_popup`이라는 key와 `true`라는 값, 그리고 만료 시간과 경로를 설정하여 쿠키를 생성한다.

즉, `merge_notice_popup`이라는 쿠키가 생성되면,  
이후 쿠키가 생성된 해당 경로에 접근했을 때 쿠키의 유무에 따라 팝업창이 보이거나 보이지 않는 것이다.


-----
</br>

## 9. 리버스 프록시
작성중

-----
</br>

## 10. 몽키 패치
몽키 패치란 런타임 중에 프로그램의 메모리를 직접 건드려 소스를 바꾸는 것이다.

- Reference - https://mingrammer.com/writing-unit-test-for-time-dependent-code/


-----
</br>

## 11. SpringBoot DevTools의 동작 원리
>DevTools를 사용하면서 어떤 원리로 동작하는 것인지 궁금해서 찾아보았다.

### DevTools가 하는 일
1. 코드가 바뀌면 자동으로 어플리케이션을 재시작한다
2. Resources(템플릿, 자바스크립트, 스타일시트 등등)가 바뀌면 브라우저를 리프레쉬한다.
3. 템플릿 캐시들을 자동으로 막는다
4. H2 데이터베이스가 사용될 경우, 빌트인 H2 콘솔을 사용할 수 있다.

### 1. 코드가 바뀌면 자동으로 어플리케이션을 재시작하는 원리
1. DevTools가 사용되면 어플리케이션은 JVM의 2개의 분리된 클래스 로더(Class Loader)에 로드된다.
2. 첫번째 클래스 로더에는 자주 변동이 발생하는 Java 코드, 설정 파일들, 그리고 `src/main/` 경로의 모든 것이 로드되고,
3. 나머지 하나의 클래스 로더에는 변동이 거의 없는 의존성 라이브러리(Dependency libraries)들이 로드된다.
4. 변화가 감지되면, DevTools는 첫번째 클래스 로더만 다시 로드하고, Spring Application Context를 재시작한다.
5. 하지만 나머지 하나의 클래스 로더와 JVM은 가만히 둔다.
6. 단점은 의존성 라이브러리가 변화하면 자동으로 재시작할 수 없다. 

### 2. 템플릿 캐시들을 자동으로 막는다.
1. Thymeleaf나 FreeMarker는 템플릿 파싱의 결과를 캐시하는 것이 기본 설정이다. (매 요청마다 다시 파싱하지 않기 위해)
2. 이는 서비스 운영 시, 약간의 성능상 이점을 가진다.
3. 하지만 개발중일 경우, 캐시된 템플릿은 어플리케이션이 동작하는 동안 브라우저를 새로고침해도 변경된 사항을 볼 수 없게한다.
4. 즉, 캐시된 템플릿 때문에 어플리케이션을 재시작해야만 변화를 확인할 수 있는데, DevTools는 템플릿 캐싱을 자동으로 막아서 변화를 재시작 없이 확인할 수 있게 한다.


- Reference - Spring in Action (5th Edition)

-----
</br>

## 12. Getter/Setter 메소드명 차이 (Lombok vs IDE자동생성)
>레거시 프로젝트의 어느 클래스에서 Getter 메서드들을 지우고,  
>아무런 고민 없이 Lombok의 @Getter 어노테이션을 붙여놨는데 여기저기서 문제가 발생했다.  
>Lombok이 생성해주는 메서드명과 IDE에서 자동생성한 메서드명이 달랐기 때문이다. (Setter도 마찬가지)

~~~java
@Getter
public class Test {

    private Boolean hasMovie1;
    private boolean hasMovie2;
    private Boolean isMovie3;
    private boolean isMovie4;

    // IDE에서 자동 생성한 Getter 메소드명
    public Boolean getHasMovie1()
    public boolean isHasMovie2()
    public Boolean getMovie3()
    public boolean isMovie4()

    // Lombok이 생성해주는 Getter 메소드명
    public Boolean getHasMovie1()
    public boolean isHasMovie2()
    public Boolean getIsMovie3()  // 요놈 주목.
    public boolean isMovie4()
}
~~~

### 공통점
- Primitive 타입인 `boolean hasMovie2`와 `boolean isMovie4`는 필드명 앞에 is만 붙는다. (is가 붙어있으면 더 붙이진 않는다.)

### 차이점
- Reference 타입인 `Boolean hasMovie1`와 `Boolean isMovie3`의 경우
  - IDE 자동생성
    - 필드명 앞에 is가 붙어있으면 제거하고 get을 붙인다.
  - Lombok
    - is의 존재여부와 상관없이 필드명 그대로 앞에 get을 붙인다.
    
-----
</br>

## 13. 프로젝트 생성시 groupId, artifactId 설정 방법

### GroupId
- 프로젝트를 구분짓는 명칭이다.
- GroupId는 [Java 패키지 네이밍 규칙](https://docs.oracle.com/javase/specs/jls/se6/html/packages.html#7.7)을 따라야 하므로, 도메인명을 거꾸로 한다.
- 예를 들어, `org.apache.maven` `org.apache.maven.plugins` `org.apache.maven.reporting` `com.sun.sunsoft.DOE`

### ArtifactId
- 버전을 제외한 jar파일 명칭이다.
- 특수문자가 없는 소문자면 어떤 명칭도 상관없다.
- 만약 third party jar 라면, 배포된 jar파일 이름을 사용해야 한다.
- 예를 들어, `maven` `commons-math` 


### Reference
- [Guide to naming conventions on groupId, artifactId, and version](https://maven.apache.org/guides/mini/guide-naming-conventions.html)


-----
</br>

## 14. Spring Boot CommandLineRunner, ApplicationRunner
>Spring Boot로 AmazonMQ를 테스트 할 때, 멀티모듈로 구성된 Publisher와 다수의 Subscriber를 실행시키기 위해 알아보던 중 알게 된 CommandLineRunner와 ApplicationRunner에 대해 정리해본다.

### 특징
- 스프링부트는 두 가지 인터페이스(CommandLineRunner, ApplicationRunner)를 제공해서 어플리케이션이 완전히 시작되는 시점에 특정 코드를 실행할 수 있게 한다.
- 두 Runner를 사용하면 main()메서드가 있는 클래스가 아닌, @Component로 등록된 어떠한 클래스에서도 사용할 수 있기 때문에 구현 위치가 자유롭다.
- 또한, static 메서드인 main() 메서드가 아닌 일반 메서드이기 때문에 static에 오염되지 않는다.
- 여러 개의 Runner를 사용할 수 있는데, 순서가 필요한 경우 각 Runner 클래스에 `@Order(순서)`를 붙여서 순서를 관리할 수 있다.

### CommandLineRunner
- 어플리케이션 인자들을 String 배열로 제공한다.

~~~java
@Component
public class CommandLineAppStartupRunner implements CommandLineRunner {
    private static final Logger logger = LoggerFactory.getLogger(CommandLineAppStartupRunner.class);
    
    @Override
    public void run(String...args) throws Exception {
    	// 구현
        logger.info("Application started with command-line arguments: {}", Arrays.toString(args));
    }
}
~~~

### ApplicationRunner
- CommandLineRunner와 달리 raw한 어플리케이션 인자들을 래핑해서 ApplicationArguments 인터페이스로 제공한다.
- ApplicationArguments 인터페이스는 편리한 메서드들을 제공한다.
  - `getOptionNames()` : 인자들의 이름 반환
  - `getOptionValues()` : 인자들의 값 반환
  - `getSourceArgs()` : raw source 인자들 반환
  
~~~java
@Component
public class AppStartupRunner implements ApplicationRunner {
    private static final Logger logger = LoggerFactory.getLogger(AppStartupRunner.class);
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
    	// 구현
        logger.info("Your application started with option names : {}", args.getOptionNames());
    }
}
~~~

### Reference
- [Spring Boot: ApplicationRunner and CommandLineRunner](https://dzone.com/articles/spring-boot-applicationrunner-and-commandlinerunne)
- [Spring Boot의 실행과 종료 시 특정 동작을 실행하도록 해보기](https://zepinos.tistory.com/41)


-----
</br>

## 15. 프로젝트 외부에서 application.yml 설정하기
>개발할 때 자주 변경하게 되는 설정들(스케쥴러, 외부 연동 등)을 프로젝트에서 직접 수정해서 배포하지 않고,  
>프로젝트 외부에 설정 파일(application.yml)을 두고 외부 설정파일로 덮어씌우게끔 하는 방법

~~~java
@SpringBootApplication
public class SchedulerApplication {

    private static final String EXTERNAL_PROPERTIES = "file:/data/etc/example.project/application.yml";

    public static void main(String[] args) {
        new SpringApplicationBuilder(SchedulerApplication.class)
                .properties("spring.config.additional-location="
                        + EXTERNAL_PROPERTIES
                )
                .run(args);
    }
}
~~~
- 서버에 `/data/etc/example.project/application.yml` 파일을 생성하고, 이 곳에서 설정값을 변경한다.
- 이 방식을 사용하면, 자주 변경되는 설정 때문에 매 번 프로젝트를 새로 배포하지 않아도 된다.
- 서버에 있는 설정파일을 수정하고 jar 파일을 다시 시작하면, 어플리케이션 시작 시점에 서버에 있는 외부 설정파일을 읽어서 프로젝트 내부의 설정파일을 덮어씌운다.
- 테스트를 위해서 자주 설정 값 변경이 필요한 경우 유용하다.


-----
</br>

## 17. @InitBinder 사용하기
>작성중

~~~java
    /**
     * PathVariable을 String 타입으로 요청해도 Enum 타입으로 받을 수 있도록
     * string -> Enum 변환
     */
    @InitBinder
    public void initBinder(WebDataBinder binder) {
        binder.registerCustomEditor(HubCategory.class, new PropertyValuesEditor() {
            @Override
            public void setAsText(String text) {
                setValue(HubCategory.valueOf(text.toUpperCase()));
            }
        });
    }
~~~

-----
</br>

## 18. 기본 생성자의 접근 레벨을 Protected/Private로 설정해서 Setter 사용하지 않기
>작성중

~~~java
/**
 * 기본 생성자를 protected/private 레벨로 막아두면,
 * 매개변수 있는 생성자로 객체를 생성하기 때문에 Setter 없이도 Get메서드로 으로 넘어오는 RequestParam을 객체에 담을 수 있다.
 * 즉, @Setter를 사용하지 않아도 된다. (Post 메서드의 경우, Jackson2HttpMessageConverter의
 * ObjectMapper가 Json으로 넘어온 RequestParam을 객체로 변환해주므로 setter가 필요없다.)
 */
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
public class MediaIdxByHubUrlRequest {

    private String hubUrl;
}
~~~

### Reference
- [@Request Body에서는 Setter가 필요없다?](https://jojoldu.tistory.com/407)

-----
</br>

## 19. OneToMany 관계에서 fetch join 적용할 경우 limit 쿼리가 동작하지 않는 문제
>N+1 문제를 예방하기 위해 무자비하게 사용한 fetch join으로 인해,  
OneToMany 관계에서는 fetch join을 사용할 경우 limit 쿼리가 동작하지 않는 문제를 알게되었다.

QueryDsl로 작성된 아래의 쿼리에서 `문제가 된 부분 1`, `문제가 된 부분 2`의 fetchJoin()은  
N+1 문제를 예방하고자 습관적으로 작성한 것이었다.

~~~java
public List<HubContent> getBeforeAfterArticleList(Integer categoryIdx, LocalDateTime baseAt, Integer size) {
        return jpaQueryFactory.selectFrom(hubContent)
                .innerJoin(hubContent.media, media).fetchJoin()
                .leftJoin(hubContent.hubContentHasCategories, hubContentHasCategory).fetchJoin() // 문제가 된 부분 1
                .leftJoin(hubContentHasCategory.category, category).fetchJoin() // 문제가 된 부분 2
                .where(category.categoryIdx.eq(categoryIdx.longValue())
                    .and(hubContent.createdAt.loe(baseAt))
                    .and(hubContent.state.eq(HubContentState.ACTIVE))
                    .and(media.state.eq(MediaState.ACTIVE)))
                .orderBy(hubContent.createdAt.desc())
                .orderBy(hubContent.hubContentIdx.asc())
                .limit(size)
                .fetch();
    }
~~~

그런데 HubContent와 HubContentHasCategory는 OneToMany 관계였고,  
fetch join으로 인해 아래와 같이 limit 쿼리가 무시되어 모든 데이터를 스캔하는 쿼리가 되었다.  

정확하게는,  
OneToMany 관계에서 fetch join시, limit 쿼리를 사용할 경우 모든 데이터를 조회 후 데이터를 limit 개수만큼 반환하고,  
WHERE절이 존재할경우 WHERE절에 따라 데이터가 먼저 추출되고,  
추출된 데이터 기반으로 fetch join이 이루어지며, 그 후에 limit 개수만큼 데이터를 반환한다.

~~~sql
...(생략)
FROM   hub_content hubcontent0_ 
       INNER JOIN media media1_ 
               ON hubcontent0_.media_idx = media1_.media_idx 
       LEFT OUTER JOIN hub_content_has_category hubcontent2_ 
                    ON hubcontent0_.hub_content_idx = 
                       hubcontent2_.hub_content_idx 
       LEFT OUTER JOIN category category3_ 
                    ON hubcontent2_.category_idx = category3_.category_idx 
WHERE  category3_.category_idx = 5 
       AND hubcontent0_.created_at <= '06/01/2020 10:40:00.222' 
       AND hubcontent0_.state = 'ACTIVE' 
       AND media1_.state = 'ACTIVE' 
ORDER  BY hubcontent0_.created_at DESC, 
          hubcontent0_.hub_content_idx ASC
	  
## limit 쿼리가 나가지 않았다.
~~~

그런데 이 쿼리를 스케쥴러가 1분 마다 요청하고 있었고,  
AWS로 프로젝트를 이전하는 과정에서 인스턴스가 죽어버리는 문제를 파악하다가 이 문제를 알게되었다.

fetchJoin()을 제거하니 데이터 풀스캔이 발생하지 않았고, 아래와 같이 limit 쿼리가 정상적으로 동작했다.    

>물론 N+1 문제는 완전히 예방되지 않았지만, 기본적으로 fetch 전략을 LazyLoading으로 사용중이고,  
추가적인 요청이 발생하는 로직이 아니기 때문에 데이터 풀스캔을 발생시키는 fetch join을 제거하는 것으로 마무리되었다.

~~~sql
...(생략)
FROM   hub_content hubcontent0_ 
       INNER JOIN media media1_ 
               ON hubcontent0_.media_idx = media1_.media_idx 
       LEFT OUTER JOIN hub_content_has_category hubcontent2_ 
                    ON hubcontent0_.hub_content_idx = 
                       hubcontent2_.hub_content_idx 
       LEFT OUTER JOIN category category3_ 
                    ON hubcontent2_.category_idx = category3_.category_idx 
WHERE  category3_.category_idx = 5 
       AND hubcontent0_.created_at <= '06/01/2020 10:40:00.222' 
       AND hubcontent0_.state = 'ACTIVE' 
       AND media1_.state = 'ACTIVE' 
ORDER  BY hubcontent0_.created_at DESC, 
          hubcontent0_.hub_content_idx ASC 
LIMIT  10  ## limit 쿼리가 나갔다.
~~~

### Reference
- [QueryDsl 삽질기 2부](https://dotoridev.tistory.com/3)
- [JPA Paging 처리 Fetch Join 적용시 limit 동작하지 않는 이슈](https://github.com/cheese10yun/blog-sample/tree/master/jpa-fetch-join)
