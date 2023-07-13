# item11: 가독성을 목표로 설계하라
* 개발자가 코드를 작성하는 데는 1분 걸리지만 읽는데는 10분 걸린다.
* 그만큼 가독성이 중요하다.

## 인식 부하 감소

다음의 코드중 어떤 코드가 더 좋을까?

```kotlin
//구현 A
if (person != null && person.isAdult) {
	view.showPerson(person)
} else {
	view.showError()
}

//구현 B
person?.takeIf { it. isAdult } 
	?.let(view::showPerson)
	?: view.showError()
```

B가 더 짧지만 A가 더 좋다.

* 가독성이란 코드를 읽고 얼마나 빠르게 이해할 수 있는지!
* 우리의 뇌가 얼마나 많은 관용구에 익숙해져 있는지에 따라 다르다.
* 코틀린 초보자에겐 A가 더 읽기 쉽다.
* B도 코틀린에선 일반적이지만 숙련자를 위한 코드는 지양하자.
* 코틀린의 대부분의 개발자는 첫번째 프로그래밍 언어가 아니다.
* 구현 A는 수정하기 쉽다.
  * if 블록에 작업을 추가해야 한다면 쉽게 추가 가능.
  * 구현 B 는 함수 참조가 불가해짐, 코드를 수정해야함.

```kotlin
//구현 A
if (person != null && person.isAdult) {
	view.showerson(person)
	view.hideProgressWithSuccess()
} else {
	view.showError()
	view.hideProgress()
}

//구현 B
person?.takelf{ it.isAdult }
	?.let {
		view.showPerson(it)
		view.hideProgressWithSuccess()
	} ?: run {
		view.showError()
		view.hideProgress()
	}
```

* 구현 A는 디버깅도 더 간단하다. (디버깅 도구 지원)
* intelij 리팩터링 도구 활용이 어렵다.
* 인지 부하를 줄이는 쪽으로 코딩하라.

## 극단적이 되지 않기
* let을 절대로 쓰면 안된다로 이해하면 안된다.
* 일반적인 곳엔 사용하자.
  * null 이 아닐때만 어떤 작업을 수행해야 하는 경우
* 코틀린 연산자들로 복잡해져도 가치가 있는곳엔 사용하자.

```kotlin
person?.let {
	print(it.name)
}
```

## 컨벤션

필자가 생각하는 최악의 코드

```kotlin
val abc = "A" { "B" } and "c"
print(abc) // ABC

// 위 코드가 기능하게 하려면, 다음과 같은 코드가 있어야 합니다.
operator fun String.invoke(f: ()->String): String = this + f()
infix fun String.and(s: String) = this + s
```

위 코드는 아래의 규칙을 위반한다.

* 연산자는 의미에 맞게 사용해야 한다.
  * invoke를 이러한 형태로 사용하면 안된다.
* 함수는 명시적이어야 한다.
  * and 라는 함수 이름이실제 구현은 문자열 결합이라 헷갈린다.
* 문자열을 결합하는 기능은 이미 내장되어 있다.
  * 이미 있는걸 다시 만들 필요는 없다.






