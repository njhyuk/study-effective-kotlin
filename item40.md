# item 40 equals 규약을 지켜라
## 동등성
코틀린에는 두가지 동등성이 있다.
* 구조적 동등성
  * a == b 는
  * a가 nullable 이 아니라면
    * a.equals(b) 으로 변환됨
  * a가 nullable 이라면
    * a?.equals(b) ?: (b === null) 으로 변환됨
* 레퍼런스적 동등성
  * === 연산자로 확인함
  * 두 피연산자가 같은 객체를 가리지면 true 를 리턴함
  * 다만 다른 타입 두 객체를 비교하는건 허용 안함

## equals가 필요한 이유
* 두 객체가 기본 생성자의 프로퍼티가 같다면, 같은 객체로 보고 싶을때
* data class 를 쓰면 자동으로 위와 같이 동작한다!
* ex)
  * name1 = FullName("노준혁")
  * name2 = FullName("노준혁")
  * name1 == name2
* data class 는 일부 프로퍼티만 비교해야 될 때도 사용 가능하다
  * ex) 캐시 프로퍼티는 비교하고 싶지 않을때
    * 그냥 class 였다면 equals 재정의 필요 하지만..
    * data class(name: String) { private var cachedName: String }
    * 단, 생성자에 없는 프로퍼티는, copy 로 복사되지 않음
    * 코틀린에서는 데이터 클래스 쓰면 equals 필요 없다

## equals 규약
* 반사적 동작
  * x가 null이 아니면?
    * x.equals(x) 는 true 를 리턴해야 한다
* 대칭적 동작
  * x와 y가 널이 아닌 값이라면?
    * x.equals(y)는 y.equals(z) 와 같은 결과여야 한다
* 연속적 동작
  * x, y, z 가 널이 아닌 값이면?
    * 그리고 x.equals(y) 와 y.equals(z) 가 true 라면?
      * x.equals(z) 도 true 여야 한다.
* 일관적 동작
  * x와 y가 널이 아닌 값이라면?
    * x.equals(y) 는 여러번 실행해도 항상 같은 결과여야 한다
* 널과 관련된 동작
  * x가 null이 아니면?
    * x.equals(null) 은 항상 false 여야 한다

규약엔 없지만 equals, toString, hashCode 매우 빠를거라 예측되므로, 빠르게 동작되어야 한다.

## 안티패턴
### 반사적 동작
`x.equals(x)` 는 true 가 나와야 하는데, 다음의 예시는 반사되지 않음
```kotlin
class Time(val millsArgs: Long, val isNow: Boolean) {
	val mills: Long get() =
		if (isNow) System.currentTimeMills()
		else millsArgs
}

val now = Time(isNow = true)

now == now // 때로는 true, 때로는 false
```
### 대칭적 동작
x == y 와 y == x 가 같아야 하는데, 다음의 예시는 대칭되지 않음
```kotlin
// Complex 라고 정의한 클래스를 Dobule 과 비교할 수 있게 미리 구현해둠

Complex(1.0).equals(1.0) // true
1.0.eqauls(Complex(1.0)) // false
```
### 연속적 동작
x == y 와 y == z 와 x==z 가 같아야 하는데, 다음의 예시는 연속되지 않음
```
* Datetime 과 Date 를 비교할 수 있게 구현
  * Datetime 과 Datetime 을 비교할 때 더 많은 프로퍼티를 확인
    * 날짜가 같지만 시간이 다른 두 Datetime 을 비교하면 false
    * 날짜가 같은 Date 를 비교하면 true
```

특별한 이유가 없으면 equals 구현 비추, 만들어야 한다면 일관 규약을 지켜야 한다.


#이펙티브-코틀린
