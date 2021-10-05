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

> #### **cf.**
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

    > #### **cf.**
    > 사실 지금까지 진행한 것과 같이 자바 코드로 스프링 빈을 등록하게 되면 생성자를 호출하면서 의존 관계 주입까지 한번에 처리<br>
    > 그러나 온전히 스프링을 통해 스프링 빈을 등록하는 경우, 빈을 생성하고 의존관계를 주입하는 단계가 나누어져 있음<br>
    > 자세한 내용은 의존관계 자동 주입 참고
