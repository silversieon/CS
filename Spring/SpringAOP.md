## ➡️ Spring Aop 용어 정리

- `Pointcut`: 어디에 부가 기능을 적용할지, 어디에 부가 기능을 적용하지 않을지 판단하는 필터링 로직. 주로 **클래스**와 **메서드 이름으로 필터링**한다. (말 그대로 어떤 Point에 기능을 적용할지 하지 않을지 cut 하여 구분하는 것.)
- `Advice`: **프록시가 호출하는 부가 기능**. (간단하게 프록시 로직임)
- `Advisor`: 단순하게 하나의 **Pointcut**과 하나의 **Advice**를 가지고 있는 것. **(Pointcut 1 + Advice 1)**

## ➡️ Spring이 제공하는 PointCut

- `NameMatchMethodPointcut`: **메서드 이름을 기반**으로 매칭. 내부적으로 **PatternMatchUtils**를 사용
- `JdkRegexMethodPointcut`: **JDK 정규 표현식을 기반**으로 포인트컷을 매칭.
- `TruePointcut`: 항상 참을 반환한다.
- `AnnotationMatchingPointcut`: 어노테이션으로 매칭.
- ⭐`AspectJExpressionPointcut`: aspectJ 표현식으로 매칭.
    - 사용하기도 편리하고 기능도 가장 많은 표현식

## ➡️ BeanPostProcessor의 두 메서드

```sql
BeanDefinition 등록
        ↓
객체 생성
        ↓
의존성 주입
		↓
*postProcessBeforeInitialization* // 초기화 전 개입(검증 등)
        ↓
@PostConstruct
		↓
*postProcessAfterInitialization* // 프록시로 변경
        ↓
컨테이너 등록 (사용 가능)
```

- `postProcessBeforeInitialization`
    - PostConstructor 실행(초기화)
- `postProcessAfterInitialization`

두 단계를 나눈 이유는:

> **“초기화 과정에 개입할 기회”와
“초기화가 끝난 객체를 확장할 기회”를 분리하기 위해서**
>

---

## 📌Spring Container 핵심 Advisor

1️⃣ TransactionAttributeSourceAdvisor (트랜잭션)

- 대상: `@Transactional`
- 인터셉터: `TransactionInterceptor`
- 역할: 트랜잭션 시작 / 커밋 / 롤백

```
@Transactional → TransactionAttributeSourceAdvisor → TransactionInterceptor
```

가장 대표적인 인프라 Advisor입니다.

---

2️⃣ AsyncAnnotationAdvisor (비동기)

- 대상: `@Async`
- 인터셉터: `AsyncExecutionInterceptor`
- 역할: 별도 스레드 풀로 메서드 실행

```
@Async → AsyncAnnotationAdvisor → AsyncExecutionInterceptor
```

---

3️⃣ CacheOperationSourceAdvisor (캐시)

- 대상: `@Cacheable`, `@CachePut`, `@CacheEvict`
- 인터셉터: `CacheInterceptor`
- 역할: 캐시 조회 / 저장 / 제거

---

4️⃣ MethodValidationPostProcessor → MethodValidationAdvisor

- 대상: `@Validated`
- 인터셉터: `MethodValidationInterceptor`
- 역할: 파라미터/리턴값 Bean Validation

```
@Validated
publicvoidcreate(@NotNullStringname)
```

---

5️⃣ PersistenceExceptionTranslationAdvisor (JPA 예외 변환)

- 대상: `@Repository`
- 인터셉터: `PersistenceExceptionTranslationInterceptor`
- 역할: JPA 예외 → Spring DataAccessException 변환

---

6️⃣ Secured / PreAuthorize Advisor (Spring Security)

- 대상:
    - `@PreAuthorize`
    - `@PostAuthorize`
    - `@Secured`
- 인터셉터:
    - `MethodSecurityInterceptor`
- 역할: 메서드 보안 검사

---

## 💡 AOP 적용 시점 3가지 (Weaving: 옷감을 짜다, 직조하다)

- **컴파일 타임 위빙**
    - .java 소스코드를 컴파일러를 사용해서 .class 파일로 만드는 시점에 부가 기능 로직 추가(AspectJ가 제공하는 특별한 컴파일러 필요)

- **로드 타임 위빙**
    - 자바를 실행하면 자바 언어는 .class 파일을 JVM 내부의 클래스 로더에 보관한다. 이 클래스 로더에 올리는 사이에서 .class 파일을 조작한 다음에 JVM에 올릴 수 있다(자바는 이런 기능을 제공함, 모니터링 툴들이 이 방식을 사용함)

- **런타임 위빙**
    - 컴파일이 다 끝나고, 클래스 로더에 클래스도 다 올라가서 자바가 실행되고 난 다음을 말한다. (Spring, DI, Bean PostProcessor 총 동원해서 부가 기능을 적용한다)
    - 생성자, 객체와 같은 곳에 AOP를 적용하기에 제약이 있다.(메서드 호출을 통해서만 가능) 하지만 컴파일러, 클래스 로더 조작기 설정을 할 필요가 없다.

**⭐스프링 AOP는 런타임 위빙 방식**을 사용하므로 **Join Point가 메서드 실행으로 제한**된다.

⭐또한 **스프링 빈에만 AOP를 적용**할 수 있다.

---

## ✅ AOP 용어 정리

- Join point
    - 어드바이스가 적용될 수 있는 위치(메서드 실행, 생성자 호출, 필드 값 접근 등)
    - AOP를 적용할 수 있는 모든 지점의 추상적인 개념
    - Spring AOP는 프록시 방식을 사용하므로 Join point가 메서드 실행 지점으로 제한

- Pointcut
    - Join point 중에 Advice가 적용될 위치를 선별(필터)하는 기능
    - 실무에서는 주로 AspectJ 표현식을 사용하여 지정
    - Spring AOP는 메서드 실행 지점만 Pointcut으로 선별 가능

- Target
    - Advice(부가기능)를 받는 객체, Pointcut에 의해서 결정

- Advice
    - 부가 기능
    - 특정 Join point에서 Aspect에 의해 취해지는 조치

- Aspect
    - Advice + Pointcut을 모듈화 한 것
    - @Aspect와 같음
    - 여러 Advice와 Pointcut이 함께 존재

- Advisor
    - 하나의 Advice와 하나의 Pointcut으로 구성
    - Spring AOP만 사용하는 특별한 용어

- Weaving
    - Pointcut으로 결정한 Target의 Joinpoint에 Advice를 적용하는 것
    - Weaving을 통해 핵심 기능 코드에 영향을 주지 않고 부가 기능 추가 가능
    - Spring AOP는 런타임 위빙, 프록시 방식 사용

- AOP 프록시
    - AOP 기능을 구현하기 위해 만든 프록시 객체
    - Spring AOP에서는 JDK Dynamic Proxy, CGLIB Proxy 사용

---

## AspectJ exprssion

```java
execution(접근제어자? 반환타입 선언타입? 메서드이름(파라미터) 예외?)
```