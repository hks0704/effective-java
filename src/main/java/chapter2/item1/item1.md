# 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라
- 인스턴스를 얻는 방법으로는 public 생성자가 전통적이다.
- 그런데 정적 팩터리 메서드(static factory method)라는 방법이 하나 더 있고 모든 프로그래머가 알고 있어야 된다.
- 정적 팩터리 메서드는 디자인 패턴 중 팩터리 메서드(factory method)와는 다른 개념이다.
```java
// 클래스의 인스턴스를 반환하는 정적 팩토리 메서드 예제
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```
## 정적 팩터리 메서드의 장점
1. 이름을 가질 수 있다.
2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

## 정적 팩터리 메서드의 단점
1. 이름을 가질 수 있다.
2. 프로그래머가 찾기 어렵다.
- 생성자처럼 API 설명에 명확하게 드러나지 않기 때문에 API문서를 잘 작성하는 게 필요하다.
- 흔히 사용하는 명명 방식
  - from
  - of
  - valueOf
  - instance 또는 getInstance
  - create 또는 newInstance
  - getType
  - newType
  - type

## 핵심 정리