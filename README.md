
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
  
## 검증1 - Validation

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

## 검증2 - Bean Validation

### Bean Validation 소개
  ```java
  public class Item{
    private Long id;
    
    @NotBalnk
    private String itemName;
    
    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;
}
  ```
- 이런 검증 로직을 모든 프로젝트에 적용할 수 있게 공동화하고, 표준화 한 것이 Bean Validation이다. Bean Validation을 잘 활용하면, 애노테이션 하나로 검증 로직을 매우 편리하게 적용할 수 있다.
- 특정한 구현체가 아니라 Bean Validation 2.0이라는 기술 표준이다. 쉽게 이야기해서 검증 애노테이션과 여러 인터페이스의 모음이다. 
- Bean Validation을 구현한 기술중에 일반적으로 사용하는 구현체는 하이버네이터 Validator이다. 

### Bean Validation 시작

#### 검증 애노테이션
- @NotBlank: 빈값 + 공백만 있는 경우를 허용하지 않는다.
- @NotNull: null을 허용하지 않는다.
- @Range(min=1000, max=1000000): 범위 안의 값이어야 한다.
- @Max(9999): 최대 9999까지만 허용한다.

#### 검증기 생성
- 이후 스프링과 통합하면 직접 이런 코드를 작성하지는 않는다.
  ```java
  ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
  Validator validator = factory.getValidator();
  ```
#### 검증 실행
- 검증 대상을 직접 검증기에 넣고 그 결과를 받는다. Set에는 ConstraintViolation이라는 검증 오류가 담긴다. 따라서 결과가 비어있으면 검증 오류가 없는 것이다.
  ```java
  Set<ConstraintViolation<Item>> violations = validator.validate(item);
  ```
- ConstraintViolation 출력 결과를 보면, 검증 오류가 발생한 객체, 필드, 메시지 정보등 다양한 정보를 확인할 수 있다.

### Bean Validation - 스프링 적용
- 스프링 부트가 'spring-boot-starter-validation' 라이브러리를 넣으면 자동으로 Bean Validator를 인지하고 스프링에 통합한다.
- 'LocalValidatorFactoryBean'을 글로벌 Validator로 등록한다. 이 Validator는 '@NotNull'같은 애노테이션을 보고 검증을 수행한다. 이렇게 글로벌 Validator가 적용되어 있기 때문에, '@Valid', '@Validated'만 적용하면 된다.
- 검증 오류가 발생하면, 'FieldError', 'ObjectError'를 생성해서 'BindingResult'에 담아준다.

#### 검증 순서
- 1. @ModelAttribute 각각의 필드에 타입 변환 시도
  - 1. 성공하면 다음으로
  - 2. 실패하면 typeMismatch로 FieldError 추가
- 2. Validator 적용
  
- 바인딩에 성공한 필드만 Bean Validation 적용
  - BeanValidator는 바인딩에 실패한 필드는 BeanValidation을 적용하지 않는다. (일단 모델 객체에 바인딩 받는 값이 정상으로 들어와야 검증도 의미가 있다.)
  
### Bean Validation - 에러 코드
- BeanValidation 메시지 찾는 순서
  - 1. 생성된 메시지 코드 순서대로 'messageSource' 에서 메시지 찾기
  - 2. 애노테이션의 'message'속성 사용 -> '@NotBlank(message="공백!{0}")'
  - 3. 라이브러리가 제공하는 기본 값 사용 -> 공백일 수 없습니다.
  
### Bean Validation - 오브젝트 오류
- Bean Validation에서 특정 필드(Field Error)가 아닌 해당 오브젝트 관련 오류(Object Error)는 '@ScriptAssert()'를 사용하면 된다.
- 그런데 실제 사용해보면 제약이 많고 복잡하다. 따라서 오브젝트 오류(글로벌 오류)의 경우 '@ScriptASsert'를 억지로 사용하는 것 보다는 오브젝트 오류 관련 부분만 직접 자바 코드로 작성하는 것을 권장한다.

### Bean Validation - 한계
- 데이터를 등록할 때와 수정할 때는 요구사항이 다를 수 있다.
- HTTP 요청은 언제든지 악의적으로 변경해서 요청할 수 있으므로 서버에서 항상 검증해야 한다.
- 등록과 수정은 같은 BeanValidation을 적용할 수 없다. 

### Bean Validadtion - groups
- 동일한 모델 객체를 등록할 때와 수정할 때 각각 다르게 검증하는 방법
  - BeanValidation의 groups 기능을 사용한다.
  - Item을 직접 사용하지 않고, ItemSaveForm, ItemUpdateForm 같은 폼 전송을 위한 별도의 모델 객체를 만들어서 사용한다.
  
- groups 기능을 사용해서 등록과 수정시에 각각 다르게 검증할 수 있다. 그런데 groups 기능을 사용하니 전반적으로 복잡도가 올라갔다.
- 실무에서는 주로 등록용 폼 객체와 수정용 폼 객체를 분리해서 사용한다.

### Form 전송 객체 분리 

#### 폼 데이터 전달에 Item 도매인 객체 사용
- HTML Form -> Item -> Controller -> Item -> Respository
  - 장점: Item 도메인 객체를 컨트롤러, 리포지토리까지 직접 전달해서 중간에 Item을 만드는 과정이 없어서 간단하다.
  - 단점: 간단한 경우에만 적용할 수 있다. 수정시 검증이 중복될 수 있고, groups를 사용해야 한다.
  
#### 폼 데이터 전달을 위한 별도의 객체 사용
- HTML Form -> ItemSaveForm -> Controller -> Item 생성 -> Repository
  - 장점: 전송하는 폼 데이터가 복잡해도 거기에 맞춘 별도의 폼 객체를 사용해서 데이터를 전달 받을 수 있다. 보통 등록과, 수정용으로 별도의 폼 객체를 만들기 때문에 검증이 중복되지 않는다.
  - 단점: 폼 데이터를 기반으로 컨트롤러에서 Item 객체를 생성하는 변환 과정이 추가된다.
  
- 수정의 경우 등록과 수정은 완전히 다른 데이터가 넘어온다. 검증 로직도 많이 달라진다. 그래서 'UpdateForm'이라는 별도의 객체로 전달받는 것이 좋다.
- 따라서 이렇게 폼 데이터 전달을 위한 별도의 객체를 사용하고, 등록, 수정용 폼 객체를 나누면 등록, 수정이 완전히 분리되기 때문에 groups를 적용할 일은 드물다.

### Bean Validation - HTTP 메시지 컨버터
- @Valid, @Validation는 HttpMessageConverter(@RequestBody)에도 적용할 수 있다.
- 참고
  - @ModelAttribute는 HTTP요청 파라미터(URL 쿼리 스트링, POST Form)를 다룰 때 사용한다.
  - @ReuqestBody는 HTTP Body의 데이터를 객체로 변환할 때 사용한다. 주로 API JSON 요청을 다룰 때 사용한다.
  
#### API의 경우 3가지 경우를 나누어 생각해야 한다.
- 성공 요청: 성공
- 실패 요청: JOSN을 객체로 생성하는 것 자체가 실패함
- 검증 오류 요청: JSON을 객체로 생성하는 것은 성공했고, 검증에서 실패함

  
#### @ModelAttribute vs @RequestBody
- HTTP 요청 파리미터를 처리하는 @ModelAttribute 는 각각의 필드 단위로 세밀하게 적용된다. 그래서
  특정 필드에 타입이 맞지 않는 오류가 발생해도 나머지 필드는 정상 처리할 수 있었다. 
- HttpMessageConverter 는 @ModelAttribute 와 다르게 각각의 필드 단위로 적용되는 것이 아니라,
  전체 객체 단위로 적용된다.
  따라서 메시지 컨버터의 작동이 성공해서 Item 객체를 만들어야 @Valid , @Validated 가 적용된다.
  - @ModelAttribute 는 필드 단위로 정교하게 바인딩이 적용된다. 특정 필드가 바인딩 되지 않아도 나머지
    필드는 정상 바인딩 되고, Validator를 사용한 검증도 적용할 수 있다.
  - @RequestBody 는 HttpMessageConverter 단계에서 JSON 데이터를 객체로 변경하지 못하면 이후
    단계 자체가 진행되지 않고 예외가 발생한다. 컨트롤러도 호출되지 않고, Validator도 적용할 수 없다.
    

## 로그인 처리1- 쿠키, 세션

### 로그인 처리하기 - 쿠키 사용
- 쿠키: 서버에서 로그인에 성공하면 HTTP 응답에 쿠키를 담아서 브라우저에 전달하자. 그러면 브라우저는 앞으로 해당 쿠키를 지속해서 보내준다.
- 쿠키에는 영속 쿠키와 세션 쿠키가 있다.
  - 영속 쿠키: 만료 날짜를 입력하면 해당 날자까지 유지
  - 세션 쿠키: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지
  
- 쿠키 생성 로직
  ```java
  Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
  response.addCookie(idCookie);
  ```
  - 로그인에 성공하면 쿠키를 생성하고 'HttpServletResponse'에 담는다. 쿠키 이름은 memberId이고, 같은 회원의 id를 담아둔다. 웹 브랄우저는 종료 전까지 회원의 id를 서버에 계속 보내줄 것이다.
  
#### 로그아웃 기능
- 로그 아웃 방법
  - 세션 쿠키이므로 웹 브라우저 종료시
  - 서버에서 해당 쿠키의 종료 날짜를 0으로 지정
  
### 쿠키와 보안 문제
- 쿠키를 사용해서 로그인id를 전달해서 로그인을 유지할 수 있다. 그런데 여기에는 심각한 보안 문제가 있다.

#### 보안 문제
- 쿠키의 값을 임의로 변경할 수 있다.
  - 클라이언트가 쿠키를 강제로 변경하면 다른 사용자가 된다.
  - 실제 웹브라우저 개발자모드 -> Application -> Cookie 변경으로 확인
  - Cookie: memberId=1 -> Cookie: memberId=2 (다른 사용자의 이름이 보임)
- 쿠키에 보관된 정보는 훔쳐갈 수 있다.
  - 정보가 웹 브라우저에도 보관되고, 네트워크 요청마다 계속 클라이언트에서 서버로 전달된다.
  - 쿠키의 정보가 나의 로컬 PC가 털릴 수도 있고, 네트워크 전송 구간에서 털릴 수도 있다.
- 해커가 쿠키를 한번 훔쳐가면 평생 사용할 수 있다.
  - 해커가 쿠키를 훔쳐가서 그 쿠키로 악의적인 요청을 계속 시도할 수 있다.
 
#### 대안
- 쿠키에 중요한 값을 노출하지 않고, 사용자 벼로 예측 불가능한 임의의 토큰(랜덤 값)을 노출하고, 서버에서 토큰과 사용자 id를 매핑해서 인식한다. 그리고 서버에서 토큰을 관리한다.
- 토큰은 해커가 임의로 값을 넣어도 찾을 수 없도록 예상 불가능 해야 한다.
- 해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 해당 토큰의 만료시간을 짧게(예:30분) 유지한다. 또는 해킹이 의심되는 경우 서버에서 해당 토큰을 강제로 제거하면 된다.

### 로그인 처리하기 - 세션 동작 방식
- 쿠키에 중요한 정보를 보관하는 방법은 여러가지 보안 이슈가 있었다. 이 문제를 해결하려면 결국 중요한 정보를 모두 서버에 저장해야 한다. 그리고 클라이언트와 서버는 추정 불가능한 임의의 식별자 값으로 연결해야 한다.
- 이렇게 서버에 중요한 정보를 보관하고 연결을 유지하는 방법을 세션이라 한다.


#### 세션 생성
- 세선 ID를 생성하는데, 추천 불가능해야 한다.
- UUID는 추정이 불가능하다.
  - Cookie: mySessionId=zz0101xx-bab9-4b92-9b32-dadb200f4b61
- 생성된 세션 ID와 세션에 보관할 값(memberA)를 서버의 세션 저장소에 보관한다.

#### 클라리언트와 서버느 결국 쿠키로 연결이 되어야 한다.
- 서버는 클라이언트에 mySesionId라는 이름으로 세션ID만 쿠키에 담아서 전달한다.
- 클라이언트는 쿠키 저장소에 mySessionId 쿠키를 보관한다.

- 여기서 중요한 포인트는 회원과 관련된 정보는 클라이언트에 전달하지 않는다는 것이다.
- 오직 추적 불가능한 세션 ID만 쿠키를 통해 클라이언트에 전달한다.

#### 클라이언트의 세션id 쿠키 전달
- 클라이언트는 요청시 항상 mySessionId 쿠키를 전달한다.
- 서버에서는 클라이언트가 전달한 mySessionId 쿠키 정보로 세션 저장소를 조회해서 로그인시 보관한 세션 정보를 사용한다.

#### 정리
- 세션을 사용해서 서버에서 중요한 정보를 관리하게 되었다. 덕분에 다음과 같은 보안 문제들을 해결할 수 있다.
  - 쿠키 값을 변조 가능 -> 예상 불가능한 복잡한 세션id를 사용한다.
  - 쿠키에 보관하는 정보는 클라이언트 해킹시 털릴 가능성이 있다. -> 세션id가 털려도 여기에는 중요한 정보가 없다.
  - 쿠키 탈취 후 사용 -> 해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 세션의 만료시간을 짧게 유지한다. 또는 해킹이 의심되는 경우 서버에서 해당 세션을 강제로 제거하면 된다.
  
### 로그인 처리하기 - 세션 직접 만들기, 사용
- 세션 생성
  - sessionId 생성 (임의의 추정 불가능한 랜덤 값)
  - 세션 저장소에 sessionId와 보관할 값 저장
  - sessionId로 응답 쿠키를 생성해서 클라이언트에 전달
- 세션 조회
  - 클라이언트가 요청한 sessionId 쿠키의 값으로, 세션 저장소에 보관한 값 조회
- 세션 만료
  - 클라이언트가 요청한 sessionId의 쿠키 값으로, 세션 저장소에 보관한 sessionId
  
### 로그인 처리하기 - 서블릿 HTTP 세션1
- 세션이라는 개념은 대부분의 웹 애프리케이션에 필요한 것이다. 서블릿은 세션을 위해 'HttpSession'이라는 기능을 제공하는데, 지금까지 나온 문제들을 해결해준다.

#### HttpSession 소개
- 서블릿이 제공하는 HttpSession도 결국 우리가 직접 만든 SessionManager와 같은 방식으로 동작한다.
- 서블릿을 통해 HttpSession을 생성하면 다음과 같은 쿠키를 생성한다. 쿠키 이름이 'JESSIONID'이고, 값은 추정 불가능한 랜덤값이다.
- Cookie: JESSONID=5B78E23B513F50164D6FDD8C97B0AD05

#### 세션의 생성과 조회
- 세션을 생성하려면 request.getSession(true)를 사용하면 된다.
- public HttpSession getSession(boolean create);

- request.getSession(true)
  - 세션이 있으면 기존 세션을 반환한다.
  - 세션이 없으면 새로운 세션을 생성해서 반환한다.
- request.getSession(false)
  - 세션이 있으면 기존 세션을 반환한다.
  - 세션이 없으면 세션을 생성하지 않는다. null을 반환한다.
- request.getSession(): 신규 세션을 생성하는 request.getSession(true)와 동일하다.

#### 세션에 로그인 회원 정보 보관
- session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);
- 세션에 데이터를 보관하는 방법은 request.setAttribute()와 비슷하다. 하나의 세션에 여러 값을 저장할 수 있다.

- request.getSession(false): request.getSession()를 사요앟면 기본 값이 create: true이므로, 로그인하지 않을 사용자도 의미없는 세션이 만들어진다. 따라서 세션을 찾아서 사용하는 시점에는 create: false 옵션을 사용해서 세션을 생성하지 않아야 한다.
- session.getAttribute(SessionConst.LOGIN_MEMBER): 로그인 시점에 세션에 보관한 회원 객체를 찾는다.


### 로그인 처리하기 - 서블릿 HTTP 세션2
- @SessionAttribute: 스프링은 세션을 더 편리하게 사용할 수 있도록 @SessionAttribute을 지원한다.

#### TrackingModes
- 로그인을 처음 시도하면 URL이 다음과 같이 jessionid를 포함하고 있는 것을 확인할 수 있다.
- 이것은 웹 브라우저가 쿠키를 지원하지 않을 때 쿠키 대신 URL을 통해서 세션을 유지하는 방법이다.
- URL 전달 방식을 끄고 항상 쿠키를 통해서만 세션을 유지하고 싶으면 옵션을 넣어주면 된다. 이렇게 하면 URL에 jessionid가 노출되지 않는다.



### 세션 정보와 타임아웃 설정
- sessionId: 세션 id, JESSIONID의 값
- maxInactiveInterval: 세션의 유효 시간, 예) 1800초
- creationTime: 세션 생성일시
- lastAccessedTime: 세션과 연결된 사용자가 최근에 서버에 접근한 시간, 클라이언트에서 서버로 sessoinId(JESSIONID)를 요청한 경우에 갱신된다.
- isNew: 새로 생성된 세션인지, 아니면 이미 과거에 만들어졌고, 클라이언트에서 서버로 sessionId(JESSIONID)를 요청해서 조회된 세션인지 여부

#### 세션 타임아웃 설정
- 세션은 사용자가 로그아웃을 직접 호출해서 session.invalidate()가 호출되는 경우에 삭제된다. 그런데 대부분의 사용자는 로그아웃을 선택하지 않고, 그냥 웹 브라우저를 종료한다.
문제는 HTTP가 비 연결성(ConnectionLess)이므로 서버 입장에서는 해당 사용자가 웹 브라우저를 종료한 것인지 아닌지를 인식할 수 없다. 따라서 서버에서 세션 데이터를 언제 삭제하는지 판단하기가 어렵다. 
  
- 이 경우 남아있는 세션을 무한정 보관하면 다음과 같은 문제가 발생할 수 있다.
  - 세션과 관련된 쿠키(JESSIONID)를 탈취 당했을 경우 오랜 시간이 지나도 해당 쿠키로 악의적인 요청을 할 수 있다.
  - 세션은 기본적으로 메모리에 생성된다. 메모리의 크기가 무한하지 않기 때문에 꼭 필요한 경우만 생성해서 사용해야 한다. 10만명의 사용자가 로그인하면 10만개의 세션이 생성되는 것이다. 
  
#### 세션의 종료 시점
- 생성 시점이 아니라 사용자가 서버에 최근에 요청한 시간을 기준으로 30분 정도를 유지해주는 것이다. 이렇게 하면 사용자가 서비스를 사용하고 있으면, 세션의 생존 시간이 30분으로 계속 늘어나게 된다. 
따라서 30분 마다 로그인해야 하는 번거로움이 사라진다. HttpSession은 이 방식을 사용한다.
  
#### 세션 타임아웃 발생
- 세션의 타임아웃 시간은 해당 세션과 관련된 'JESSIONID'를 전달하는 HTTP 요청이 있으면 현재 시간으로 다시 초기화 된다.
이렇게 초기화 되면 세션 타임아웃으로 설정한 시간동안 세션을 추가로 사용할 수 있다.
- LastAccessedTime 이후로 timeout 시간이 지나면, WAS가 내부에서 해당 세션을 제거한다.

#### 정리
- 서블릿의 HttpSession이 제공하는 타임아웃 기능 덕분에 세션을 안전하고 편리하게 사용할 수 있다. 실무에서 주의할 점은 세션에는 최소한의 데이터만 보관해야 한다는 점이다.
보관함 데이터 용량 * 사용자 수로 세션의 메모리 사용량이 급격하게 늘어나서 장애로 일어날 수 있다.
- 추가로 세션의 시간을 너무 길게 가져가면 메모리 사용이 계속 누적 될 수 있으므로 적당한 시간을 선택하는 것이 필요하다. 기본이 30분이라는 것을 기준으로 고민하면 된다.

## 로그인 처리2 - 필터, 인터셉터
### 서블릿 필터 - 소개
- 공통 관심 사항
  - 로그인 한 사용자만 상품 관리 페이지에 들어갈 수 있어야 한다. 앞에서 로그인을 하지 않은 사용자에게는 상품 관리 버튼이 보이지 않기 때문에 문제가 없어보이지만, 로그인 하지 않은 사용자도 다음 URL을 직접 호출하면 상품 관리 화면에 들어갈 수 있다. 
- 상품 관리 컨트롤러에서 로그인 여부를 체크하는 로직을 하나하나 작성하면 되겠지만, 상품관리의 모든 컨트롤러 로직에 공통으로 로그인 여부를 확인해야 한다. 더 큰 문제는 향후 로그인과 관련된 로직이 변경될 때 작성한 모든 로직을 다 수정해야 할 수 ㅣㅇㅆ다.
- 이렇게 애플리케이션 여러 로직에서 공통으로 관심이 있는 것을 공통 관심사(cross-cutting concern)라고 한다. 여기서는 등록, 수정, 삭제, 조회 등등 여러 로직에서 공동으로 인증에 대해서 관심을 가지고 있다.
- 이러한 공통 관심사는 스프링의 AOP로도 해결할 수 있지만, 웹과 관련된 공통 관심사는 서블릿 필터 또는 스프링 인터셉터를 사용하는 것이 좋다.
- 웹과 관련됙 공통 관심사를 처리할 때는 HTTP의 헤더나 URL의 정보들이 필요한데, 서블릿 필터나 스프링 인터셉터는 HttpServletRequest를 제공한다.

#### 서블릿 필터 소개
- 필터는 서블릿이 지원하는 수문장이다.
- 필터 흐름
  ```
  HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
  ```
  - 필터를 적용하면 필터가 호출 된 다음에 서블릿이 호출된다. 그래서 모든 고객의 요청 로그를 남기는 요구사항이 있다면 필터를 사용하면 된다. 참고로 필터는 특정 URL 패턴에 적용할 수 있다. '/*'이라고 하면 모든 요청에 필터가 적용된다.
  - 스프링을 사용하는 경우 여기서 말하는 서블릿은 스프링의 디스패처 서블릿으로 생각하면 된다.
- 필터 제한
  ```
  HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러 //로그인 사용자
  HTTP 요청 -> WAS -> 필터(적절하지 않은 요청이라 판단, 서블릿 호출 X) //비 로그인 사용자
  ```
  - 필터는 적절하지 않은 요청이라고 판단하면 거기에서 끝을 낼 수도 있다. 그래서 로그인 여부를 체크하기에 딱 좋다.
- 필터 체인
  ```
  HTTP 요청 -> WAS -> 필터 1 -> 필터2 -> 필터3 -> 서블릿 -> 컨트롤러
  ```
  - 필터는 체인으로 구성되는데, 중간에 필터를 자유롭게 추가할 수 있다. 예를 들어서 로그를 남기는 필터를 먼저 적용하고, 그 다음에 로그인 여부를 체크하는 필터를 만들 수 있다.
  
- 필터 인터페이스
  ```java
  import java.io.IOException;   public interface filter{ 
    public default void init(FilterConfog filterConfig) throws ServletException{}
    
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;    
   
    public default void destroy(){}
  }
  ```
  - 필터 인터페이스를 구현하고 등록하면 서블릿 컨테이너가 필터를 싱글톤 객체로 생성하고, 관리한다.
  - init(): 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출된다.
  - doFilter(): 고객의 요청이 올 때 마다 해당 메서드가 호출된다. 필터의 로직을 구현하면 된다.
  - destroy(): 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.
  
### 서블릿 필터 - 요청 로그
- public class LogFilter implements Filter {}
  - 필터를 사용하려면 필터 인터페이스를 구현해야 한다.
- doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
  - HTTP요청이 오면 doFilter가 호출된다.
  - ServletRequest request는 HTTP 요청이 아닌 경우까지 고려해서 만든 인터페이스이다. HTTP를 사용하면 HttpServletRequest httpRequest= (HttpServletRequest) request; 와 같이 다운 캐스팅 하면 된다.
- String uuid = UUID.randomUUID().toString();
  - HTTP 요청을 구분하기 위해 요청당 임의의 uuid를 생성해둔다.
- log.info("REQUEST [{}][{}]", uuid, requestURI);
  - uuid와 requestURI 를 출력한다.
- chain.doFilter(request, response);
  - 이 부분이 가장 중요하다. 다음 필터가 있으면 필터를 호출하고, 필터가 없으면 서블릿을 호출한다. 만약 이 로직을 호출하지 않으면 이 다음 단계로 진행되지 않는다.
  
#### webConfig - 필터 설정
- 필터를 등록하는 방법은 여러가지가 있지만, 스프링 부트를 사용한다면 FilterRegistrationBean을 사용해서 등록하면 된다.
- setFilter(new LogFilter()): 등록할 필터를 지정한다.
- setOrder(1): 필터는 체인으로 동작한다. 따라서 순서가 필요하다. 낮을 수록 먼저 동작한다.
- addUrlPatterns("/*"): 필터를 적용할 URL 패턴을 지정한다. 한번에 여러 패턴을 지정할 수 있다.

### 서블릿 필터 - 인증 체크
- 로그인 되지 않은 사용자는 상품 관리 뿐만 아니라 미래에 개발될 페이지에도 접근하지 못하도록 해야한다.

- whitelist= {"/", "/members/add", "/login", "/logout", "/css/*"};
  - 인증 필터를 적용해도 홈, 회원가입, 로그인 화면, css 같은 리소스에는 접근할 수 있어야 한다. 이렇게 화이트 리스트 경로는 인증과 무관하게 항상 허용한다. 화이트 리스트를 제외한 나머지 모든 경로에는 인증 체크 로직을 적용한다.
- isLoginCheckPath(RequestURI)
  - 화이트 리스트를 제외한 모든 경우에 인증 체크 로직을 적용한다.
- httpResponse.sendRedirect("/login?redirectURL="+ requestURI);
  - 로그인 이후에 원하는 페이지로 이동하기 위해, 요청한 경로인 requestURI를 /login에 쿼리 파라미터로 함께 전달한다. 
  
#### webCongif - loginCheckFiliter()
```java
@Bean
public FilterRegistrationBean loginCheckFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new
        FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LoginCheckFilter());
        filterRegistrationBean.setOrder(2);
        filterRegistrationBean.addUrlPatterns("/*");
        return filterRegistrationBean;
        }  
```
- setFilter(new LoginCheckFilter()): 로그인 필터를 등록한다.
- setOrder(2): 순서를 2번으로 설정 한다.
- addUrlPatterns("/*): 모든 요청에 로그인 필터를 적용한다.
