# item 39 태그 클래스보다 클래스 계층을 사용하라
* 상수 모드를 태그(tag) 라고 부르며, 태그를 포함한 클래스를 "태그 클래스" 라고 부름
* 태그 클래스의 문제
  * 서로 다른 책임을 한 클래스에 태그로 구분
  * 아래 예시, 태그에 따라 값 비교 클래스

```kotlin
class ValueMatcher<T> private constructor(
	private val value: T? = null, 	private val matcher: Matcher
) {

fun match (value: T?) = when (matcher) {
	Matcher.EQUAL -> value == this.value
	Matcher.NOT_EQUAL -> value != this.value
	Matcher.LIST_EMPTY -> value is List<*> && value.isEmpty()
	Matcher.LIST_NOT_EMPTY -> value is List<*> && value.isNotEmpty()
}

enum class Matcher {
	EQUAL, NOT_EQUAL, LIST_EMPTY, LIST_NOT_EMPTY
}

companion object {
	fun <T> equal(value: T) =
		ValueMatcher<T>(value = value, matcher = Matcher.EQAUL)

	fun <T> notEqual(value: T) =
		ValueMatcher<T>(value = value, matcher = Matcher.NOT_EQAUL)

	fun <T> emptyList() =
		ValueMatcher<T>(matcher = Matcher.LIST_EMPTY)

	fun <T> notEmptyList() =
		ValueMatcher<T>(matcher = Matcher.LIST_NOT_EMPTY)
}
```

* 위 클래스의 단점, 한 클래스에 여러 모드 처리를 위한 상용구 추가
* 여러 목적 지원을 위해, 프로퍼티가 일관적이지 않게 사용됨
  * emptyList() notEmptyList() 는 value 를 사용하지 않음
* 팩토리 메서드를 사용해야함
  * 그렇지 않으면 객체가 태그가 필요한 프로퍼티로 생성이 된건지 확인이 어려움

## sealed 한정자 

* 코를린은 태그 클래스보다 sealed 클래스 사용
* 외부파일에서 서브클래스를 만드는 행위 자체를 제한함
* when 을 사용할때 else 브랜치 만들 필요 없음

```kotlin
sealed class ValueMatcher<T> {

abstract fun match(value: T): Boolean

class Equal<T>(val value: T) : ValueMatcher<T>() {
	override fun match(value: T): Boolean =
		value == this.value
}

class NotEqual<T> (val value: T) : ValueMatcher<T>() {
	override fun match(value: T): Boolean =
		value != this.value
}
```

## sealed 한정자를 상태 패턴으로도 쓰기

```kotlin
sealed class WorkoutState

class PrepareState(val exercise: Exercise) : WorkoutState()

class ExerciseState(val exercise: Exercise) : WorkoutState()

object DoneState : WorkoutState()

class WorkoutPresenter() {
	private var state: WorkoutState = states.first()
}
```

>  enum 으로만 쓰는것보다 어떤 이점이 있을까?

#이펙티브-코틀린
