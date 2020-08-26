# 스프링 인 액션

## 1장 스프링 시작하기

### 프로젝트 빌드 주요 항목
- mvnw, mvnw.cmd : local에 maven이 없어도 빌드할 수 있도록 해주는 파일
- pom.xml : 빌드 명세

Q: pom.xml의 <parent><version>의 용도는?

A: 주로 사용되는 라이브러리에 대한 의존성 관리

### starter 의존성의 장점

- 선언할 빌드 파일의 수가 적어진다
- 라이브러리 이름 대신 기능의 관점으로 작성할 수 있다
- 버전관리를 하지 않아도 된다

### pom.xml plugin

- maven 사용하는 앱 실행
- 의존성에 지정된 라이브러리가 JAR에 포함되어 있는지 런타임에 classpath에 존재하는지 확인
- JAR 메인 클래스로 부트스트랩 클래스인 매니페스트 파일을 JAR에 생성

### @SpringBootApplication의 구성

- @SpringBootConfiguration
- @EnableAutoconfiguration
- @ComponentScan

## 2장 웹 애플리케이션 개발하기

> pom.xml에 starter 의존성을 쉽게 추가하는 [IntelliJ edit-starter plugin](https://github.com/hdzitao/idea-editstarters-plugin)

view만 반환하는 컨트롤러는 별도 클래스를 생성하지 않고 WebMvcConfigurer의 addViewControllers를 통해 등록할 수 있다.
