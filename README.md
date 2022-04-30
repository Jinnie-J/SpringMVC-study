
## 타임리프 - 스프링 통합과 폼

### 타임리프 스프링 통합
#### 스프링 통합으로 추가되는 기능들
- 스프링의 SpringEL 문법 통합
- ${@myBean.doSomething()} 처럼 스프링 빈 호출 지원
- 편리한 폼 관리를 위한 추가 속성
    - th:object(기능 강화, 폼 커멘드 객체 선택)
    - th:field, th:errors, th:errorclass
- 폼 컴포넌트 기능
    - checkbox, radio button, List 등을 편리하게 사용할 수 있는 기능 지원
- 스프링의 메시지, 국제화 기능의 편리한 통합
- 스프링의 검증, 오류 처리 통합
- 스프링의 변환 서비스 통합(ConversionService)

### 입력 폼 처리
- th:object: 커멘드 객체를 지정한다.
- *{...}: 선택 변수 식이라고 한다. th:object에서 선택한 객체에 접근한다.
- th:field: HTML 태그의 id, name, value 속성을 자동으로 처리해준다.

#### 렌더링 전
```javascript
<input type="text" th:field="*{itemName}"/>
```
#### 렌더링 후
```javascript
<input type="text" id="itemName" name"itemName" th:value="*{itemName}"/>
```

#### 등록 폼
- th:object를 적용하려면 먼저 해당 오브젝트 정보를 넘겨주어야 한다. 등록 폼이기 때문에 데이터가 비어있는 빈 오브젝트를 만들어서 뷰에 전달한다.
- th:object="${item}":< form >에서 사용할 객체를 지정한다. 선택 변수 식 *{...}을 적용할 수 있다.
- th:field="*{itemName}"
    - *{itemName}는 선택 변수 식을 사용했는데, ${item.itemName}과 같다. 앞서 th:object로 item을 선택했기 때문에 선택 변수 식을 적용할 수 있다.
    - th:field는 id, name, value 속성을 모두 자동으로 만들어준다.
    - id: th:field에서 지정한 변수 이름과 같다. id="itemName"
    - name: th:field에서 지정한 변수 이름과 같다. name="itemName"
    - value: th:field에서 지정한 변수의 값을 사용한다. value=""
  
### 체크 박스 - 단일1
- 체크 박스를 체크하면 HTML에서 open=on 이라는 값이 넘어간다. 스프링은 on이라는 문자를 true 타입으로 변환해준다.
- 주의: HTML에서 체크 박스를 선택하지 않고 폼을 전송하면 open이라는 필드 자체가 서버로 전송되지 않는다.
- HTML checkbox는 선택이 안되면 클라이언트에서 서버로 값 자체를 보내지 않는다. 수정의 경우에는 상황에 따라서 이 방식이 문제가 될 수 있다. 사용자가 의도적으로 체크되어 있던 값을 체크 해제해도 저장시 아무 값도 넘어가지 않기 때문에, 서버 구현에 따라서 값이 오지 않는 것으로 판단해서 값을 변경하지 않을 수도 있다.
- 이런 문제를 해결하기 위해서 스프링 MVC는 약간의 트릭을 사용하는데, 히든 필드를 하나 만들어서, _open처럼 기존 체크 박스 이름 앞에 언더스코어(_)를 붙여서 전송하면 체크를 해제했다고 인식할 수 있다. 히든 필드는 항상 전송된다.
- 따라서 체크를 해제한 경우 open은 전송되지 않고, _open만 전송되는데, 이 경우 스프링 MVC는 체크를 해제했다고 판단한다.
  ```javascript
  <input type="checkbox" id="open" name="open" class="form-check-input">
  <input type="hidden" name="_open" value="on"/>
  ```
#### 체크 박스 체크
- open=on&_open=on
  - 체크 박스를 체크하면 스프링 MVC가 open에 값이 있는 것을 확인하고 사용한다. 이때 _open은 무시한다.
- _open=on
  - 체크 박스를 체크하지 않으면 스프링 MVC가 _open만 있는 것을 확인하고, open의 값이 체크되지 않았다고 인식한다.
  - 이 경우 서버에서 Boolean 타입을 찍어보면 결과가 null이 아니라 false인 것을 확인할 수 있다. 
  
### 체크 박스 - 단일2
- 개발할 때 마다 히든 필드를 추가하는 것은 상당히 번거롭다. 타임리프가 제공하는 폼 기능을 사용하면 이런 부분을 자동으로 처리할 수 있다.
```javascript
<input type="checkbox" id="open" th:field="*{open}" class="form-check-input">
```
- 타임리프를 사용하면 체크 박스의 히든 필드와 관련된 부분도 함께 해결해준다. HTML 생성 결과를 보면 히든 필드 부분이 자동으로 생성되어 있다.

### 체크 박스 - 멀티
- 체크 박스를 멀티로 사용해서, 하나 이상을 체크할 수 있도록 한다.

#### @ModelAttribute의 특별한 사용법
- 등록 폼, 상세화면, 수정 폼에서 모두 서울, 부산, 제주라는 체크 박스를 반복해서 보여주어야 한다. 이렇게 하려면 각각의 컨트롤러에서 model.addAttribute(...)을 사용해서 체크 박스를 구성하는 데이터를 반복해서 넣어주어야 한다.
- @ModelAttribute는 이렇게 컨트롤러에 있는 별도의 메서드에 적용할 수 있다.
- 이렇게하면 해당 컨트롤러를 요청할 때 regions 에서 반환한 값이 자동으로 모델(model)에 담기게 된다.

- th:for="${#idx.prev('regions')}"
  - 멀티 체크박스는 같은 이름의 여러 체크박스를 만들 수 있다. 그런데 문제는 이렇게 반복해서 HTML 태그를 생성할 때, 생성된 HTML 태그 속성에서 name은 같아도 되지만, id는 모두 달라야 한다. 따라서 타임리프는 체크박스를 each 루프 안에서 반복해서 만들 때 임의로 1,2,3 숫자를 뒤에 붙여준다.

#### each로 체크박스가 반복 생성된 결과 - id 뒤에 숫자가 추가
```javascript
<input type="checkbox" value="SEOUL" class="form-check-input" id="regions1" name="regions">
<input type="checkbox" value="BUSAN" class="form-check-input" id="regions2" name="regions">
<input type="checkbox" value="JEJU" class="form-check-input" id="regions3" name="regions">
```
- HTML의 id가 타임리프에 의해 동적으로 만들어지기 때문에 < label for="id 값" >으로 label의 대상이 되는 id값을 임의로 지정하는 것은 곤란하다. 타임리프는 ids.prev(...), ids.next(...)을 제공해서 동적으로 생성되는 id 값을 사용할 수 있도록 한다.

#### 로그 출력
- 서울, 부산 선택
  - regions=SEOUL&_regions=on&regions=BUSAN_regions=on&_regions=on
  - 로그: item.regions=[SEOUL, BUSAN]
- 지역 선택 X
  - _regions=on&_regions=on&_regions=on
  - 로그: item.regions=[]
  
### 라디오 버튼
- 라디오 버튼은 여러 선택지 중에 하나를 선택해 사용할 수 있다.
- 체크 박스는 수정시 체크를 해제하면 아무 값도 넘어가지 않기 때문에, 별도의 히든 필드로 이런 문제를 해결했다. 라디오 버튼은 이미 선택이 되어 있다면, 수정시에도 항상 하나를 선택하도록 되어 있으므로 체크박스와 달리 별도의 히든 필드를 사용할 필요가 없다.

#### 타임리프에서 ENUM 직접 접근
```javascript
<div th:each="type: $(T(hello.itemservice.domain.item.ItemType.values()}">
```
- 스프링EL 문법으로 ENUM을 직접 사용할 수 있다. ENUM에 values()를 호출하면 해당 ENUM의 모든 정보가 배열로 반환된다.
- 그런데 이렇게 사용하면 ENUM의 패키지 위치가 변경되거나 할때 자바 컴파일러가 타임리프까지 컴파일 오류를 잡을 수 없으므로 추천하지는 않는다.

### 셀렉트 박스
- 셀렉트 박스는 여러가지 선택지 중에 하나를 선택해 사용할 수 있다.
- @ModelAttribute가 있는 deliveryCodes() 메서드는 컨트롤러가 호출될 때 마다 사용되므로 deliveryCodes 객체도 계속 생성된다. 이런 부분은 미리 생성해두고 재사용하는 것이 더 효율적이다.

## 메시지, 국제화

### 메시지, 국제화 소개
#### 메시지
- 여러 화면에 보이는 상품명, 가격, 수량 등 'label'에 있는 단어를 변경하려면 화면들을 다 찾아가면서 모두 변경해야 한다. 화면 수가 적으면 문제가 되지 않지만 화면이 수십개 이상이라면 수십개의 파일을 모두 고쳐야 한다.
- 왜냐하면 해당 HTML 파일에 메시지가 하드코딩 되어 있기 때문이다. 이런 다양한 메시지를 한 곳에서 관리하도록 하는 기능을 메시지 기능이라 한다.
  ```html
  'message.properties'라는 메시지 관리용 파일을 만들고
  item=상품
  item.id= 상품 ID
  item.itemName = 상품명
  item.price = 가격
  item.quantity = 수량
  
  각 HTML들은 해당 데이터를 key 값으로 불러서 사용한다.
  <label for="itemName" th:text="#{item.itemName}"></label>
  ```
  
#### 국제화
- 메시지에서 설정한 메시지 파일('message.properties')을 각 나라별로 별도로 관리하면 서비스를 국제화할 수 있다.
  ```html
  'message_en.properties'
  item = Item
  item.id = Item ID
  item.price = price
  item.quantity = quantity
  
  'message_ko.properties'
  item = 상품
  item.id = 상품 ID
  item.itemName = 상품명
  item.price = 가격
  item.quantity = 수량
  ```
- 영어를 사용하는 사람이면 'message_en.properties'를 사용하고, 한국어를 사용하는 사람이면 'message_ko.properties'를 사용하게 개발하면 된다.
- 한국에서 접근한 것인지 영어에서 접근한 것인지 인식하는 방법은 HTTP 'accept-language' 헤더 값을 사용하거나 사용자가 직접 언어를 선택하도록 하고, 쿠키 등을 사용해서 처리하면 된다.
- 스프링은 기본적인 메시지와 국제화 기능을 모두 제공한다. 그리고 타임리프도 스프링이 제공하는 메시지와 국제화 기능을 편리하게 통합해서 제공한다.

### 스프링 메시지 소스 설정
- 메시지 관리 기능을 활용하려면 스프링이 제공하는 'MessageSource'를 스프링 빈으로 등록하면 되는데, 'MessageSource'는 인터페이스이다. 따라서 구현체인 'ResourceBundleMessageSource' 를 스프링 빈으로 등록하면 된다.
- 'MessageSource'를 스프링 빈으로 등록하지 않고, 스프링 부트와 관련된 별도의 설정을 하지 않으면 'message'라는 이름으로 기본 등록된다. 따라서 'messages_en.properties','messages_ko.properties','messages.properties' 파일만 등록하면 자동으로 인식된다.
  ```html
  message.properties
  hello=안녕
  hello.name=안녕 {0}
  
  message_en.properties
  hello=hello
  hello.name=hello {0}
  ```
### 스프링 메시지 소스 사용
- 가장 단순한 테스트는 메시지 코드로 hello를 입력하고 나머지 값은 null을 입력했다. locale 정보가 없으면 basename에서 설정한 기본 이름 메시지 파일을 조회한다. basename으로 messages를 지정했으므로 messages.properties 파일에서 데이터 조회한다.
  ```html
   ms.getMessage("hello",null,null)
   - code: hello
   - args: null
   - locale: null
  ```
- 메시지가 없는 경우, 기본 메시지
  - 메시지가 없는 경우에는 NoSuchMessageException이 발생한다.
  - 메시지가 없어도 기본 메시지(defaultMessage)를 사용하면 기본 메시지가 반환된다.
- 매개변수 사용
  - 다음 메시지의 {0} 부분은 매개변수를 전달해서 치환할 수 있다.
  - hello.name 안녕 {0} -> Spring 단어를 매개변수로 전달 -> 안녕 Spring
- 국제화 파일 선택
  - locale 정보를 기반으로 국제화 파일을 선택한다.
  - Locale이 en_US의 경우 messages_en_US -> messages_en -> messages 순서로 찾는다.
  - Locale에 맞추어 구체적인 것이 있으면 구체적인 것을 찾고, 없으면 디폴트를 찾는다고 이해하면 된다.
  
### 웹 애플리케이션에 메시지 적용하기
- 타임리프의 메시지 표현식 #{...} 를 사용하면 스프링의 메시지를 편리하게 조회할 수 있다. 예) #{label.item}
  ```html
  렌더링 전
  <div th:text="#{label.item}"></h2>
  
  렌더링 후
  <div>상품</h2>
  ```
- 페이지 이름에 적용
  ```html
  <h2>상품 등록 폼</h2>
    <h2 th:text="#{page.addItem}">상품 등록</h2>
  ```
- 레이블에 적용
  ```html
  <label for="itemName">상품명</label>
    <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
    <label for="price" th:text="#{label.item.price}">가격</label> 
    <label for="quantity" th:text="#{label.item.quantity}">수량</label>
  ```
- 버튼에 적용
  ```html
  <button type="submit">상품 등록</button>
    <button type="submit" th:text="#{button.save}">저장</button>
    <button type="button" th:text="#{button.cancel}">취소</button>
  ```
  
### 웹 애플리케이션에 국제화 적용하기
- 웹 브라우저의 언어 설정 값을 변경하면 요청시 Accept-Language의 값이 변경된다.
- Accept-Language는 클라이언트가 서버에 기대하는 언어 정보를 담아서 요청하는 HTTP 요청 헤더이다.

#### 스프링의 국제화 메시지 선택
- 메시지 기능은 Locale 정보를 알아야 언어를 선택할 수 있다. 스프링은 언어 선택시 기본으로 Accept-Language 헤더의 값을 사용한다.
- LocaleResolver
  - 스프링은 Locale 선택 방식을 변경할 수 있도록 LocaleResolver라는 인터페이스를 제공하는데, 스프링 부트는 기본으로 Accept-Language를 활용하는 AcceptHeaderLocaleResolver를 사용한다.
  ```java
  public interface LocaleResolver{
    Locale resolveLocale(HttpServletRequest reauest);
    void setLocale(HttpServletRequest request, @Nullable HttpServletResponse response, @Nullable Locale locale);
  }
  ```
- LocaleResolver 변경
  - 만약 Locale 선택 방식을 변경하려면 LocaleResolver의 구현체를 변경해서 쿠키나 세션 기반의 Locale 선택 기능을 사용할 수 있다. 예를 들어서 고객이 직접 Locale을 선택하도록 하는 것이다.
  
## 검증 - Validation

### 검증 요구사항

#### 요구사항: 검증 로직 추가
- 타입 검증
  - 가격, 수량에 문자가 들어가면 검증 오류 처리
- 필드 검증
  - 상품명: 필수, 공백 x
  - 가격: 1000원 이상, 1백만원 이하
  - 수량: 최대 9999
- 특정 필드의 범위를 넘어서는 검증
  - 가격 * 수량의 합은 10,000원 이상
     

- 웹 서비스는 폼 입력시 오류가 발생하면, 고객이 입력한 데이터를 유지한 상태로 어떤 오류가 발생했는지 친절하게 알려주어야 한다. 
- 컨트롤러의 중요한 역할중 하나는 HTTP 요청이 정상인지 검증하는 것이다. 
- 클라이언트 검증, 서버 검증
  - 클라이언트 검증은 조작할 수 있으므로 보안에 취약하다.
  - 서버만으로 검증하면, 즉각적인 고객 사용성이 부족해진다.
  - 둘을 적절히 섞어서 사용하되, 최종적으로 서버 검증은 필수
  - API 방식을 사용하면 API 스펙을 잘 정의해서 검증 오류를 API 응답 결과에 잘 남겨주어야 함
  

### 검증 직접 처리 - 개발
- 검증 오류 보관
  ```java
  Map<String, String> errors = new HashMap<>();
  ```
  - 만약 검증 시 오류가 발생하면 어떤 검증에서 오류가 발생했는지 정보를 담아둔다.
  - 검증시 오류가 발생하면 errors에 담아둔다. 이때 어떤 필드에서 오류가 발생했는지 구분하기 위해 오류가 발생한 필드명을 key로 사용한다. 이후 뷰에서 이 데이터를 사용해서 고객에게 친절한 오류 메시지를 출력할 수 있다.
  - 특정 필드를 넘어서는 오류를 처리해야 할 수도 있다. 이때는 필드 이름을 넣을 수 없으므로 globalERror라는 key를 사용한다.
  - 만약 검증에서 오류 메시지가 하나라도 있으면 오류 메시지를 출력하기 위해 model에 errors를 담고, 입력 폼이 있는 뷰 템플릿으로 보낸다.
  

- 글로벌 오류 메시지
  ```html
  <div th:if="${errors?.containsKey('globalError')}">
  <p class="field-error" th:text="${errors['globalError']}">전체 오류 메시지</p>
  </div>
  ```
  - 오류 메시지는 errors에 내용이 있을 때만 출력하면 된다. 타임리프의 th:if를 사용하면 조건에 만족할때만 해당 HTML 태그를 출력할 수 있다.


- Safe Navigation Operater
  - 만약 'errors'가 'null'이라면 어떻게 될까? 등록폼에 진입한 시점에는 'errors'가 없다. 따라서 'errors.containsKey()'를 호출하는 순간 'NullPointerException'이 발생한다.
  - 'errors?.'은 'errors'가 'null'일때 'NullPointerException'이 발생하는 대신, 'null'을 반환하는 문법이다.
  - 'th:if'에서 'null'은 실패로 처리되므로 오류 메시지가 출력되지 않는다.
  

- 필드 오류 처리
  ```html
  <input type="text" th:classappend="${errors?.containsKey('itemName')} ? 'field-error' : _" class="form-control">
  ```
  - 'classappend'를 사용해서 해당 필드에 오류가 있으면 'field-error'라는 클래스 정보를 더해서 폼의 색깔을 빨간색으로 강조한다. 만약 값이 없으면 _(No-Operation)을 사용해서 아무것도 하지 않는다.
  
#### 정리
- 만약 검증 오류가 발생하면 입력 폼을 다시 보여준다.
- 검증 오류들을 고객에게 친절하게 안내해서 다시 입력할 수 있게 한다.
- 검증 오류가 발생해도 고객이 입력한 데이터가 유지된다.

#### 남은 문제점
- 뷰 템플릿에서 중복 처리가 많다. 뭔가 비슷하다.
- 타입 오류 처리가 안된다. item의 price, quantity 같은 숫자 필드는 타입이 Integer 이므로 문자 타입으로 설정하는 것이 불가능하다. 숫자 타입에 문자가 들어오면 오류가 발생한다. 그런데 이러한 오류는 스프링MVC에서 컨트롤러에 진입하기도 전에 예외가 발생하기 때문에, 컨트롤러가 호출되지도 않고, 400 예외가 발생하면서 오류 페이지를 띄워준다.
- Item의 price에 문자를 입력하는 것 처럼 타입 오류가 발생해도 고객이 입력한 문자를 화면에 남겨야 한다. 만약 컨트롤러가 호출된다고 가정해도 Item의 price는 Integer이므로 문자를 보관할 수가 없다. 결국 문자는 바인딩이 불가능하므로 고객이 입력한 문자가 사라지게 되고, 고객은 본인이 어떤 내용을 입력해서 오류가 발생했는지 이해하기 어렵다.
- 결국 고객이 입력한 값도 어딘가에 별도로 관리가 되어야 한다.

### BindingResult1
- 스프링이 제공하는 검증 오류 처리 방법

#### 필드오류 - FieldError
```java
if (!StringUtils.hasText(item.getItemName())){
    bindingResult.addError(new FieldError("item","itemName","상품 이름은 필수입니다."));
        }
```
#### 필드 생성자 요약
```java
public FieldError(String objectName, String field, String defaultMessage){}
```

- 필드에 오류가 있으면 FieldError 객체를 생성해서 bindingResult에 담아두면 된다.
  - objectName: @ModelAttribute이름
  - field: 오류가 발생한 필드 이름
  - defaultMessage: 오류 기본 메시지

#### 글로벌 오류 - ObjectError
```java
bindingResult.addError(new ObjectError("item","가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값="+ resultPrice));
```

#### ObjectError 생성자 요약
```java
public ObjectError(String objectName, String defaultMessage){}
```
- 특정 필드를 넘어서는 오류가 있으면 ObjectError 객체를 생성해서 bindingResult에 담아두면 된다.
- objectName: @ModelAttribute의 이름
- defaultMessage: 오류 기본 메시지

#### 타임리프 스프링 검증 오류 통합 기능
- 타임리프는 스프링의 BindingResult를 활용해서 편리하게 검증 오류를 표현하는 기능을 제공한다.
  - '#fields': #fields로 BindingResult가 제공하는 검증 오류에 접근할 수 있다.
  - th:errors: 해당 필드에 오류가 있는 경우에 태그를 출력한다. th:if의 편의 버전이다.
  - th:errorclass: th:field에서 지정한 필드에 오류가 있으면 class 정보를 추가한다.
  
### BindingResult2
- 스프링이 제공하는 검증 오류를 보관하는 객체이다. 검증 오류가 발생하면 여기에 보관하면 된다.
- BindingResult가 있으면 @ModelAttribute에 데이터 바인딩 시 오류가 발생해도 컨트롤러가 호출된다.
- @ModelAttribute에 바인딩 시 타입 오류가 발생하면?
  - BindingResult가 없으면 -> 400오류가 발생하면서 컨트롤러가 호출되지 않고, 오류 페이지로 이동한다.
  - BindingResult가 있으면 -> 오류 정보(fieldError)를 BindingResult에 담아서 컨트롤러를 정상 호출한다.
  

#### "BindingResult에 검증 오류를 적용하는 3가지 방법"
- @ModelAttribute의 객체에 타입 오류 등으로 바인딩이 실패하는 경우 스프링이 FieldError 생성해서 BindingResult에 넣어준다.
- 개발자가 직접 넣어준다.
- Validator 사용

#### 타입 오류 확인
- 숫자가 입력되어야 할 곳에 문자를 입력해서 타입을 다르게 해서 BindingResult를 호출하고 BindingResult의 값을 확인해보자

- 주의
  - BindingResult는 검증할 대상 바로 다음에 와야한다. 순서가 중요하다. '@ModelAttribute Item item' 바로 다음에 'BindingResult'는 Model에 자동으로 포함된다.
  
#### BindingResult와 Errors
- 'org.springframework.validation.Errors'
- 'org.springframework.validation.BindingResult'
- BindingResult는 인터페이스고, Errors 인터페이스를 상속받고 있다.
- 실제 넘어오는 구현체는 'BeanPropertyBindingResult'라는 것인데, 둘다 구현하고 있으므로 BindingResult 대신에 Errors를 사용해도 된다.
- Errors 인터페이스는 단순한 오류 저장과 조회 기능을 제공한다. BindingResult는 여기에 더해서 추가적인 기능을 제공한다. 
- addError()도 BindingResult가 제공한다. 주로 관례상 BindingResult를 많이 사용한다.

### FieldError, ObjectError
- 사용자 입력 오류 메시지가 화면에 남도록 하자
- FieldError 생성자
  ```java
  public FieldError(String objectName, String field, String defaultMessage);
  public FieldError(String objectName, String field, @Nullable Object rejectedValue, boolean bindingFailure, @Nullable String[] codes, @Nullable Object[] arguments, @Nullable String defaultMessage)
  ```
- 파라미터 목록
  - objectName: 오류가 발생한 객체 이름
  - field: 오류 필드
  - rejecedValue: 사용자가 이벽한 값(거절된 값)
  - bindingFailure: 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값
  - codes: 메시지 코드
  - arguments: 메시지에서 사용하는 인자
  - defaultMessage: 기본 오류 메시지
- 오류 발생시 사용자 입력 값 유지
  ```java
  new FieldError("item", "price", item.getPrice(), false, null, null, "메시지");
  ```
- 사용자의 입력 데이터가 컨트롤러의 @ModelAttribute에 바인딩되는 시점에 오류가 발생하면 모델 객체에 사용자 입력 값을 유지하기 어렵다. 오류가 발생한 경우 사용자 입력 값을 보관하는 별도의 방법이 필요하다. 이렇게 보관한 사용자 입력 값을 검증 오류 발생시 화면에 다시 출력하면 된다.
- FieldError는 오류 발생시 사용자 입력 값을 저장하는 기능을 제공한다. rejectValue가 오류 발생시 사용자 입력 값을 저장하는 필드다.

#### 타임 리프의 사용자 입력 값 유지
- th:field="*{price}"
- 정상 상황에는 모델 객체의 값을 사용하지만, 오류가 발생하면 FieldError에서 보관한 값을 사용해서 값을 출력한다.

#### 스프링의 바인딩 오류 처리
- 타입 오류로 바인딩에 실패하면 스프링은 FieldError를 생성하면서 사용자가 입력한 값을 넣어둔다. 그리고 해당 오류를 BindingResult에 담아서 컨트롤러를 호출한다. 따라서 타입 오류 같은 바인딩 실패시에도 사용자의 오류 메시지를 정상 출력할 수 있다. 

### 오류 코드와 메시지 처리1

#### FieldError 생성자
- FieldError는 두 가지 생성자를 제공한다
  ```java
  public FieldError(String objectName, String field, String defaultMessage);
  public FieldError(String objectName, String field, @Nullable Object rejectedValue, boolean bindingFailure, 
         @Nullable String[] codes, @Nullable Object[] arguments, @Nullable String defaultMessage);
  ```
- 파라미터 목록
  - objectName: 오류가 발생한 객체 이름
  - field: 오류 필드
  - rejectedValue: 사용자가 입력한 값(거절된 값)
  - bindingFailure: 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값
  - codes: 메시지 코드
  - arguments: 메시지에서 사용하는 인자
  - defaultMessage: 기본 오류 메시지
- FieldError, ObjectError의 생성자는 errorCode, arguments를 제공한다. 이것은 오류 발생시 오류 코드로 메시지를 찾기 위해 사용된다.

  ```java
  //range.item.price= 가격은 {0} ~ {1} 까지 허용합니다.
  new FiledError("item", "price", item.getPrice(), false, 
                 new String[]{"range.item.price"}, new Object[]{1000, 1000000}) 
  ```
- codes: required.item.itemName를 사용해서 메시지 코드를 지정한다. 메시지 코드는 하나가 아니라 배열로 여러 값을 전달할 수 있는데, 순서대로 매칭해서 처음 매칭되는 메시지가 사용된다.
- arguments: Object[]{1000, 1000000}를 사용해서 코드의 {0}, {1}로 치환할 값을 전달한다.

### 오류 코드와 메시지 처리2
- BindingResult가 제공하는 rejectValue(), reject()를 사용하면 FieldError, ObjectError를 직접 생성하지 않고, 깔끔하게 검증 오류를 다룰 수 있다.

#### rejectValue()
  ```java 
  void rejectValue(@Nullable String field, String errorCode,
          @Nullable Object[] errorArgs, @Nullable String defaultMessage);
  ```
- field: 오류 필드명
- errorCode: 오류 코드(이 오류 코드는 메시지에 등록된 코드가 아니다. 뒤에서 설명할 messageResolver를 위한 오류 코드이다.)
- errorArgs: 오류 메시지에서 {0}을 치환하기 위한 값
- defaultMessage: 오류 메시지를 찾을 수 없을 때 사용하는 기본 메시지
```java
bindingResult.rejectValue("price","range",new Object[]{1000, 1000000}, null)
```
- 앞에서 bindingResult는 어떤 객체를 대상으로 검증하는지 target을 이미 알고 있다고 했다. 따라서 target(item)에 대한 정보는 없어도 된다. 오류 필드명은 동일하게 price를 사용했다.

### 오류 코드와 메시지 처리3
- 오류코드를 단순하게 만들면 범용성이 좋아서 여러곳에서 사용할 수 있지만, 메시지를 세밀하게 작성하기 어렵다 .반대로 너무 자세하게 만들면 범용성이 떨어진다.
- 가장 좋은 방법은 범용성으로 사용하다가, 세밀하게 작성해야 하는 경우에는 세밀한 내용이 적용되도록 메시지에 단계를 두는 방법이다.

- 다음과 같이 required라는 메시지만 있으면 이 메시지를 선택해서 사용한다.
  ```properties
   required: 필수 값 입니다.
   ```
- 그런데 오류 메시지에 'required.item.itemName'와 같이 객체명과 필드를 조합한 세밀한 메시지 코드가 있으면 이 메시지를 높은 우선순위로 사용하는 것이다.
  ```properties
  #Level1
  required.item.itemName: 상품 이름은 필수 입니다.

  #Level2
  required: 필수 값 입니다.
  ```
- 스프링은 'MessageCodesResolver'라는 것으로 이러한 기능을 지원한다. 

### 오류 코드와 메시지 처리4
#### MessageCodesResolver
- 검증 오류 코드로 메시지 코드들을 생성한다.- MessageCodesResolver 인터페이스이고 DefaultMessageCodesResolver는 기본 구현체이다.- 주로 다음과 함꼐 사용 ObjectError, FieldError

#### DefaultMessageCodeResolver의 기본 메시지 생성 규칙
- 객체 오류
  1. code + "."+ object name
  2. code
- 필드 오류
  1. code + "." + object name + "." + field
  2. code + "." + field
  3. code + "." + field type
  4. code
  
#### FieldError - rejectValue("itemName","required")
- 다음 4가지 오류 코드를 자동으로 생성
- required.item.itemName
- required.itemName
- required.java.lang.String
- required

#### ObjectError - reject("totalPriceMin")
- 다음 2가지 오류 코드를 자동으로 생성
- totalPriceMin.item
- totalPriceMin

#### 오류 메시지 출력
- 타임리프 화면을 렌더링할 때 th:errors가 실행된다. 만약 이때 오류가 있다면 생성된 오류 메시지 코드를 순서대로 돌아가면서 메시지를 찾는다. 그리고 없으면 디폴트 메시지를 출력한다.

### 오류 코드와 메시지 처리5

#### 오류 코드 관리 전략
- 핵심은 구체적인 것에서 덜 구체적인 것으로
   - MessageCodeResolver는 required.item.itemName 처럼 구체적인 것을 먼저 만들어주고, required 처럼 덜 구체적인 것을 가장 나중에 만든다.
- 크게 중요하지 않은 메시지는 범용성 있는 require 같은 메시지로 끝내고, 정말 중요한 메시지는 꼭 필요할 때 구체적으로 적어서 사용하는 방식이 더 효과적이다.  

#### ValidationUtils
- ValidationUtils 사용 전
  ```java
  if (!StringUtils.hasText(item.getItemName())){
      bindingResult.rejectValue("itemName","required","상품 이름은 필수입니다.");
          }
  ```
  
- ValidationUtisl 사용 후
  
  - 다음과 같이 한줄로 가능, 제공하는 기능은 Empty, 공백 같은 단순한 기능만 제공
  ```java
  ValidationUtils.rejectEmptyOrWhitespace(bindingResult, "itemName","required");
  ```
  
#### 정리
- 1. rejectValue() 호출
- 2. MessageCodesResolver를 사용해서 검증 오류 코드로 메시지 코드들을 생성
- 3. new FieldError()를 생성하면서 메시지 코드들을 보관
- 4. th:errors에서 메시지 코드들로 메시지를 순서대로 메시지에서 찾고, 노출
  
### 오류 코드와 메시지 처리6
- 검증 오류 코드는 2가지로 나눌 수 있다.
  - 개발자가 직접 설정한 오류 코드 -> rejectValue()를 직접 호출
  - 스프링이 직접 검증 오류에 추가한 경우(주로 타입 정보가 맞지 않음)
- 스프링은 타입 오류가 발생하면 typeMismatch라는 오류 코드를 사용한다. 이 오류 코드가 MessageCodesResolver를 통하면서 4가지 메시지 코드가 생성된다.
  - typeMismatch.item.price
  - typeMismatch.price
  - typeMismatch.java.lang.Integer
  - typeMismatch
- 결과적으로 소스코드를 하나도 건들지 않고, 원하는 메시지를 단계별로 설정할 수 있다.  

### Validator 분리1
- 복잡한 검증 로직을 별도로 분리한다.
- 컨트롤러에서 검증 로직이 차지하는 부분은 매우 크다. 이런 경우 별도의 클래스로 역할을 분리하는 것이 좋다. 그리고 이렇게 분리한 검증로직을 재사용 할 수도 있다.

- 스프링은 검증을 체계적으로 제공하기 위해 다음 인터페이스를 제공한다.
  ```java
  public interface Validator{
    boolean supports(Class<?> clazz);
    void validate(Object target, Errors errors);
  }
  ```
- supports() {}: 해당 검증기를 지원하는 여부 확인
- validate(Object target, Errors errors): 검증 대상 객체와 BindingResult

### Validator 분리2
- 스프링이 Validator 인터페이스를 별도로 제공하는 이유는 체계적으로 검증 기능을 도입하기 위해서다. Validator 인터페이스를 사용해서 검증기를 만들면 스프링의 추가적인 도움을 받을 수 있다.
- WebDataBinder는 스프링의 파라미터 바인딩의 역할을 해주고 검증 기능도 내부에 포함한다.
  ```java
  @InitBinder
  public void init(WebDataBinder dataBinder){
    log.info("init binder {}", dataBinder);
    dataBinder.addValidators(itemValidator);
  }
  ```
- 이렇게 WebDataBinder에 검증기를 추가하면 해당 컨트롤러에서는 검증기를 자동으로 적용할 수 있다. 
- @InitBinder: 해당 컨트롤러에만 영향을 준다. 글로벌 설정은 별도로 해야한다.

- @Validated는 검증기를 실행하라는 애노테이션이다. 이 애노테이션이 붙으면 앞서 WebDataBinder에 등록한 검증기를 찾아서 실행한다. 그런데 여러 검증기를 등록한다면 그 중에 어떤 검증기가 실행되어야 할지 구분이 필요하다. 이때 supports()가 사용된다.


#### 참고
- 검증 시 @Validated, @Valid 둘다 사용가능하다.
- @Validated는 스프링 전용 검증 애노테이션이고, @Validate는 자바 표준 검증 애노테이션이다.