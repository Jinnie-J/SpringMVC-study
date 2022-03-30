
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