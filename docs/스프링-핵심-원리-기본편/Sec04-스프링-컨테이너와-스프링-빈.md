# 4. 스프링 컨테이너와 스프링 빈

## 4.1 스프링 컨테이너 생성

### `ApplicationContext`
- **스프링 컨테이너**
- 인터페이스 -> 여러 구현체가 존재한다.
    - XML 기반으로 생성하는 구현체
    - 애노테이션 기반의 자바 설정 클래스로 생성하는 구현체
    - 기타 여러 방식의 구현체가 존재
- 요즘은 애노테이션 기반의 자바 설정 클래스로 생성하는 방식이 많이 쓰임 -> 스프링 부트가 사용하는 방식
    ```Java
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
    ```
    - `AnnotationConfigApplicationContext` <- `ApplicationContext` 인터페이스의 구현체
    - 직전에 AppConfig에 @Configuration, @Bean 등을 붙여 설정 클래스로 사용해 스프링 컨테이너를 만든 방식이 이 방식

> #### **참고**
> 정확히는 스프링 컨테이너를 부를 때 BeanFactory와 ApplicationContext로 구분한다.<br>
> BeanFactory를 직접 사용하는 경우는 거의 없어서 보통 ApplicationContext를 스프링 컨테이너라고 한다.<br>
> BeanFactory <- ApplicationContext <- 기타 유틸성 기능

### 스프링 컨테이너의 생성 과정

1. **스프링 컨테이너 생성**
    - 지정된 설정 정보를 활용해 스프링 컨테이너를 생성
    - `new AnnotationConfigApplicationContext(AppConfig.class)`
    <p align="center"><img src="../../images/스프링-컨테이너-생성.png" width="80%"></p>

2. **스프링 빈 등록**
    - 파라미터로 넘어온 설정 클래스 정보를 활용해 스프링 빈의 대상을 특정하고 등록
    - 애노테이션 기반 자바 설정 클래스를 이용하는 경우에는 @Bean 이 붙어 있는 메소드가 스프링 빈 등록 대상
    - 스프링 빈 저장소에 빈 이름을 key, 빈 객체를 value와 같은 형태로 저장
    <p align="center"><img src="../../images/스프링-빈-등록.png" width="80%"></p>

    - 빈 이름
        - 빈 이름은 메소드 이름을 사용 - (Default)
        - 빈 이름을 직접 부여하는 것도 가능
            - `@Bean(name="memberService2")`
        - 빈 이름은 항상 서로 다른 이름을 부여해야 한다.
            - 같은 이름을 부여하게 되면 다른 빈이 무시되거나 덮어씌워질 수 있음

3. **스프링 빈 의존관계 설정 - 준비**
    <!-- - 스프링 빈 등록 -->
    <p align="center"><img src="../../images/스프링-빈-의존관계-설정-준비.png" width="80%"></p>

4. **스프링 빈 의존관계 설정 - 완료**
    - 설정 정보를 참고하여 객체 간의 의존 관계 주입 (DI)
    <p align="center"><img src="../../images/스프링-빈-의존관계-설정-완료.png" width="80%"></p>

    > #### **참고**
    > 사실 지금까지 진행한 것과 같이 자바 코드로 스프링 빈을 등록하게 되면 생성자를 호출하면서 의존 관계 주입까지 한번에 처리<br>
    > 그러나 온전히 스프링을 통해 스프링 빈을 등록하는 경우, 빈을 생성하고 의존관계를 주입하는 단계가 나누어져 있음<br>
    > 자세한 내용은 의존관계 자동 주입 참고

## 4.2 컨테이너에 등록된 모든 빈 조회

### 모든 빈 출력하기
```Java
@Test
@DisplayName("모든 빈 출력하기")
void findAllBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for(String beanDefinitionName : beanDefinitionNames) {
        Object bean = ac.getBean(beanDefinitionName);
        System.out.println("name= " + beanDefinitionName + " object= " + bean);
    }
}
```
- 스프링에 등록된 모든 빈 정보 출력
- `ac.getBeanDefinitionNames()` : 스프링에 등록된 모든 빈 이름 조회
- `ac.getBean()`: 빈 이름으로 빈 객체 조회

### 애플리케이션 빈 출력하기
```Java
@Test
@DisplayName("애플리케이션 빈 출력하기")
void findApplicationBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for(String beanDefinitionName : beanDefinitionNames) {
        BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);
        // Role ROLE_APPLICATION : 직접 등록한 애플리케이션 빈
        // Role ROLE_INFRASTRUCTURE : 스프링이 내부에서 사용하는 빈
        if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
        System.out.println("name= " + beanDefinitionName + " object= " + bean);
        }
    }
}
```
- 스프링이 내부에서 사용하는 빈은 제외, 개발자가 등록한 빈만 출력
- `beanDefinition.getRole()` 로 구분 가능
    - `ROLE_APPLICATION` : 일반적으로 사용자가 정의한 빈
        - 설정 클래스인 AppConfig 도 사용자가 정의한 빈에 포함
    - `ROLE_INFRASTRUCTURE` : 스프링이 내부에서 사용하는 빈
- `getBeanDefinition()`
    - 빈에 대한 메타데이터 정보는 `ApplicatinoContext` 의 구현체인 `GenericApplicationContext` 에 구현
    - `AnnotationConfigApplicationContext` 는 `GenericApplicationContext` 를 상속받아 `getBeanDefinition()` 메소드를 가짐
    - `ApplicationContext` 에는 없음

> #### **참고**
> JUnit5부터는 테스트 클래스와 메소드에 public 붙이지 않아도 됨

## 4.3 스프링 빈 조회 - 기본

스프링 컨테이너에서 스프링 빈을 찾는 가장 기본적인 조회 방법

- `ac.getBean(빈이름, 타입)`
    ```Java
    MemberService memberService = ac.getBean("memberService", MemberService.class);
    ```
    ```Java
    MemberService memberService = ac.getBean("memberService", MemberServiceImpl.class);
    ```
    - 구체 타입으로 조회하는 것도 가능
        - AppConfig의 memberService 메소드의 반환 타입이 MemberService 이긴 하지만<br>
        스프링 빈 조회 시 스프링 빈에 등록된 객체 인스턴스 타입을 보고 가져오는 것이기 때문에 상관없음
    - 단, 구체 타입으로 조회 시 유연성이 떨어지기 때문에 좋은 방법은 아님
- `ac.getBean(타입)`
    ```Java
    MemberService memberService = ac.getBean(MemberService.class);
    ```
    - 마찬가지로 구체 타입으로 조회 가능
- `ac.getBean(빈이름)`
    ```Java
    MemberService memberService = ac.getBean("memberService");
    ```
    - 빈 이름만 가지고 스프링 빈을 조회할 경우, 클래스를 지정하지 않아 Object 타입으로 반환<br>
    -> 사용 전 사용하고자 하는 타입으로 반드시 캐스팅

<br>

- 조회 대상 스프링 빈이 없으면 예외 발생
    - `NoSuchBeanDefinitionException: No bean named 'xxxxx' available`
    - 테스트 방법
        ```Java
        // import static org.assertj.core.api.Assertions.*;
        Assertions.assertThrows(NoSuchBeanDefinitionException.class, () -> ac.getBean("xxxx", MemberService.class));
        ```
        - assertThrows(예외클래스, 예외발생코드)
            - 예외 발생 코드를 실행했을 때, 예외 클래스의 예외가 발생하면 테스트 통과