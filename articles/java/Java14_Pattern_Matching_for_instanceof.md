> 제목: Java14 new features - (1) Pattern Matching for instanceof (Preview)  
> 출처: https://openjdk.java.net/jeps/305  
  
### 요약
- Pattern Matching을 이용하여 instanceof 연산자 개선
- Pattern Matching은 객체에서 요소를 조건에 따라 추출할 때 공통적으로 사용되는 코드를 간결하고 안전하게 작성 가능
- instanceof 연산자를 사용할 때 개발자가 자연스럽게 함께 작성하는 코드를 언어 레벨에서 지원
- 하지만 Preview 스펙이라 언제 없어질지 모름, 마이너 버전 업데이트나 다음 Java15 업데이트에서 사라질 수도 있음

### 내용
- Before
  - 아래 코드같이 instanceof 연산자를 사용할 때, 개발자가 자연스럽게 함께 작성하는 코드가 존재
  ```java
  public static void test(Object obj) {
    if (obj instanceof String) {
      String s = (String) obj; // 이거
    }
  }
  ```
  - 위 코드에서 총 3가지가 실행됨: validate, type casting, declaration
  - 이런 코드를 작성하는 것은 피곤하고 지루하며, 이런 반복적인 코드는 오류를 발생할 수 있음
- After
  - Pattern Matching을 이용하여 instanceof 연산자 개선 시작
  - Pattern
    - 객체의 원하는 모양을 간결하게 표현 가능
    - target에 적용될 타입과 target이 그 타입에 해당될 경우 할당될 변수의 조합으로 이루어짐
  - Matching
    - target이 Pattern에서 정의한 모양에 일치하는지 검사함
  - instanceof 연산자가 단순 타입 검사에서 타입 검사 **패턴**으로 기능이 확장됨
  - ex)
    ```java
    @SuppressWarnings("preview")
    public static void test(Object obj) {
      if (obj instanceof Integer i) { // Integer i 코드가 type test pattern
        i.doubleValue();
      } else {
        i.doubleValue(); // ERROR!, 여기서 i 사용 못함
        obj.getClass().getName();
      }

      if (!(obj instanceof String s)) {
        s.equalsIgnoreCase("test"); // ERROR!, 여기서 s 사용 못함
        obj.getClass();
      } else {
        s.equalsIgnoreCase("aaa");
      }

      Double d = 0;
      if (!(obj instanceof Double d)) {
        d.toString(); // test 메서드에 선언된 d
      } else {
        d.toString(); // if문 안에 선언된 d
      }

      if (obj instanceof String s && s.length() > 5) { // && 연산자 뒤에 사용 가능
        s.equalsIgnoreCase("testtest");
      }

      if (obj instanceof String s || s.length() > 5) { // || 연산자 뒤에 사용 불가능
        s.equalsIgnoreCase("testtest");
      }
    }
    ```
