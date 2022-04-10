
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