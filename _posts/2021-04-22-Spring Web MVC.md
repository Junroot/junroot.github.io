---
title:  "Spring Web MVC"
categories: programming
tags: Java Spring
---

Spring Web MVC는 Servlet API를 기반으로 구축된 웹 프레임워크다. 아래는 Java 코드를 이용하여 사용하는 방법을 정리했다.

## 컨트롤러 선언

`@Controller`와 `@RestController`를 어노테이션을 통해 컨트롤러를 선언할 수 있다. @Controller는 전통적인 Spring MVC의 컨트롤러에 해당하며, `@RestController`는 Restful 웹서비스의 컨트롤러다. Restful은 현재 문서의 주제 밖의 내용이라 생략한다. 간단하게 `@Controller`는 View를 반환하기 위해 사용, `@RestController`는 Data를 반환하기 위해 사용한다고 생각하자.

## 요청 매핑

`@RequestMapping`을 사용하여 컨트롤러 메서드에 매핑할 수 있다. URL, HTTP 메서드, request parameters, headers, media type 등 다양한 속성으로 매핑할 수 있다.

### HTTP 메서드 별 매핑

HTTP 메서드에 대한 매핑은 @RequestMapping의 단축된 표현이 존재한다.

- `@GetMapping` == `@RequestMapping(method=HttpMethod.GET)`
- `@PostMapping` == `@RequestMapping(method=HttpMethod.POST)`
- `@PutMapping` == `@RequestMapping(method=HttpMethod.PUT)`
- `@DeleteMapping` == `@RequestMapping(method=HttpMethod.DELETE)`
- `@PatchMapping` == `@RequestMapping(method=HttpMethod.PATCH)`

```java
@RestController
@RequestMapping("/persons")
class PersonController {

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        // ...
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void add(@RequestBody Person person) {
        // ...
    }
}
```

### URI 패턴

매핑시킬 URI의 패턴을 지정할 수 있다.

- `"/resources/ima?e.png"` - 경로 세그먼트에서 한 문자 일치
- `"/resources/*.png"` - 경로 세그먼트에서 0 개 이상의 문자와 일치
- `"/resources/**"` - 여러 경로 세그먼트 일치
- `"/projects/{project}/versions"` - 경로 세그먼트를 일치시키고 변수로 캡처
- `"/projects/{project:[a-z]+}/versions"` - 정규식과 일치하고 변수를 캡처

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class OwnerController {

    @GetMapping("/pets/{petId}")
    public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
        // ...
    }
}
```

위와 같이 클래스 수준에서 URI 변수를 선언할 수도 있다.

### 소비 가능한 미디어 타입

`consumes`속성을 사용하여 요청으로 받을 수 있는 Content-Type의 종류를 매핑할 수 있다.

```java
@PostMapping(path = "/pets", consumes = "application/json") 
public void addPet(@RequestBody Pet pet) {
    // ...
}
```

- `!text/plain` 과 같이 부정의 표현이 가능하다.

### 생산 가능한 미디어 타입

`produces` 속성을 사용하여 요청으로 받을 수 있는 `Accept`의 종류를 매핑할 수 있다.

```java
@GetMapping(path = "/pets/{petId}", produces = "application/json") 
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```

- `!text/plain` 과 같이 부정의 표현이 가능하다.

### 매개 변수, 헤더

요청의 매개 변수 조건에 따라서도 매핑할 수 있다. 매개 변수가 있는지 (`myParam`), 없는지 (`!myParam`), 특정 값이 있는지(`myParam=myValue`)에 따라 매핑할 수 있다.

```java
@GetMapping(path = "/pets/{petId}", params = "myParam=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

같은 방법으로 요청의 헤더 조건에 따라서도 매핑할 수 있다.

```java
@GetMapping(path = "/pets", headers = "myHeader=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

## 핸들러 매서드

`@RequestMapping` 핸들러 메서드는 컨트롤러 메서드의 인자 및 반환 값의 종류를 선택할 수 있다. 받은 인자는 `int`, `long`, `Date` 등으로 자동으로 타입 변환이 가능하다.

### Matrix Varaibles - @MatrixVariable

```java
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11
}
```

아래와 같이 경로 세그먼트 별 matrix varable을 인자로 받을 수 있다.

```java
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22
}
```

아래와 같이 기본값을 지정할 수도 있다.

```java
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1
}
```

모든 matrix varaible을 얻으려면 `MultiValueMap`을 사용할 수 있다.

```java
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable MultiValueMap<String, String> matrixVars,
        @MatrixVariable(pathVar="petId") MultiValueMap<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 22, "s" : 23]
}
```

### 쿼리 매개 변수 - @RequestParam

```java
@Controller
@RequestMapping("/pets")
public class EditPetForm {

    // ...

    @GetMapping
    public String setupForm(@RequestParam("petId") int petId, Model model) { 
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...
```

- `Map<String, String>` 또는 `MultiValueMap<String, String>`를 통해 매개 변수 값을 한꺼번에 받아올 수 있다.
- `@RequestParam` 를 생략하면 메서드 인자와 같은 이름을 가진 매개 변수를 받는다.

### 요청 헤더

아래와 같은 헤더가 있다고 가정한다.

```
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```

```java
@GetMapping("/demo")
public void handle(
        @RequestHeader("Accept-Encoding") String encoding, 
        @RequestHeader("Keep-Alive") long keepAlive) { 
    //...
}
```

- `Map<String, String>` 또는 `MultiValueMap<String, String>`를 통해 매개 변수 값을 한꺼번에 받아올 수 있다.

### 쿠키

아래와 같은 쿠키 정보가 있다고 가정한다.

```
JSESSIONID = 415A4AC178C59DACE0B2C9CA727CDD84
```

```java
@GetMapping("/demo")
public void handle(@CookieValue("JSESSIONID") String cookie) { 
    //...
}
```

### Model

메서드 인자에 `@ModelAttribute` 붙이면 모델의 attribute에 접근하거나 존재하지 않는 경우 인스턴스화 되도록한다. 

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) { }
```

위의 pet은 다음과 같이 처리된다.

- 모델에 이미 추가되어 있는경우 모델에서
- `@SessionAttributes`를 사용하면  HTTP session에서
- `Converter`를 통해 전달된 URI 경로 변수에서
- 기본 생성자 호출에서
- 서블릿 요청 파라미터와 일치하는 인자를 가진 주 생성자에서. 이 때 인자 이름은 자바빈의 `@ConstructorProperties` 또는 바이트 코드의 런타임 보유 매개변수 이름을 통해 결정된다.

### Session

컨트롤러에서 사용하는 세션속성을 선언할 때 `@SessionAttributes`를 사용할 수 있다.

```java
@Controller
@SessionAttributes("pet") 
public class EditPetForm {
    // ...
}
```

위는 `pet` 이라는 모델의 속성이 자동으로 추가되어 HTTP 서블릿 세션에 저장된다.

```java
@Controller
@SessionAttributes("pet") 
public class EditPetForm {

    // ...

    @PostMapping("/pets/{id}")
    public String handle(Pet pet, BindingResult errors, SessionStatus status) {
        if (errors.hasErrors) {
            // ...
        }
            status.setComplete(); 
            // ...
        }
    }
}
```

- 위의 pet은 `@ModelAttribute` 가 생략되었다.
- `SessionStatus`의 `setComplete` 를 통해 세션을 지울 수 있다.

```java
@RequestMapping("/")
public String handle(@SessionAttribute User user) { 
    // ...
}
```

- `@SessionAttribute` 로 기존 세션 속성에 접근할 수 있다.

### 기존 요청 속성

`@RequestAttribute` 를 이용하여 기존에 설정된 속성을 사용할 수있다.

```java
@GetMapping("/")
public String handle(@RequestAttribute Client client) { 
    // ...
}
```

### 리다이렉션

아래와 같이 리다이렉션을 할 경로를 지정할 수 있다.

```java
@PostMapping("/files/{path}")
public String upload(...) {
    // ...
    return "redirect:files/{path}";
}
```

리다이랙션 대상에 데이터를 전송하는 방법으로는 Flash 속성이 있다. Flash 속성은 HTTP 세션에 저장되므로 URL에 표시되지 않는다.

### Request body

요청의 body를 역직렬화하여 사용할 수 있다.

```java
@PostMapping("/accounts")
public void handle(@RequestBody Account account) {
    // ...
}
```

`@HttpEntity` 는 `@RequestBody`와 비슷하지만 헤더도 접근할 수 있다.

### Response body

응답의 body를 직렬화 시킬 수 있다. 직렬화 할때는 Jackson JSON을 사용한다.

```java
@GetMapping("/accounts/{id}")
@ResponseBody
public Account handle() {
    // ...
}
```

참고로 `@RestController` 는 `@Controller @ResponseBody` 를 함께 사용한 것과 같다.

`@ResponseEntity`는 `@ResponseBody`와 비슷하지만 헤더도 설정할 수 있다.

```java
@GetMapping("/something")
public ResponseEntity<String> handle() {
    String body = ... ;
    String etag = ... ;
    return ResponseEntity.ok().eTag(etag).build(body);
}
```

### Jackson JSON

`@JsonView`를 이용하여 직렬화 또는 역직렬화시 사용할 필드를 선택할 수 있다. 자세한 내용은 [https://www.baeldung.com/jackson-json-view-annotation](https://www.baeldung.com/jackson-json-view-annotation) 를 참고한다.

```java
@RestController
public class UserController {

    @GetMapping("/user")
    @JsonView(User.WithoutPasswordView.class)
    public User getUser() {
        return new User("eric", "7!jd#h23");
    }
}

public class User {

    public interface WithoutPasswordView {};
    public interface WithPasswordView extends WithoutPasswordView {};

    private String username;
    private String password;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @JsonView(WithoutPasswordView.class)
    public String getUsername() {
        return this.username;
    }

    @JsonView(WithPasswordView.class)
    public String getPassword() {
        return this.password;
    }
}
```

## 참고 자료

[https://mangkyu.tistory.com/49](https://mangkyu.tistory.com/49)

[https://docs.spring.io/spring-framework/docs/current/reference/html/web.html](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html)