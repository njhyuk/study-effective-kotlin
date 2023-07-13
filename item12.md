# item12: 연산자 오버로드를 할 때는 의미에 맞게 사용하라

연산자 오버로딩의 매력으로 인해 오용하기 쉽다.

팩토리얼을 구하는 함수를 생각해 보자.

```kotlin
fun Int.factorial(): Int = (1..this).product()

fun Iterable<Int>.product() : Int 
	= fold(1) { acc, i -> acc * i }
```

이는 Int 확장 함수라서 굉장히 편리하게 사용 할 수 있다.

```kotlin
print (10 * 6. factorial()) / / 7200
```

수학 기호에선 ! 으로 팩토리얼을 사용한다.

```
10 * 6!
```

위를 코틀린 연산자 오버로딩으로 만들어 낼 수 있다.

```kotlin
operator fun Int.not() = factorial()
print (10 * !6) // 7200
```

* 이렇게 쓰면 안된다. 이 함수의 이름이 not 이라는 것에 주목하자.
* 논리연산에 사용해야지 팩토리얼 연산에 사용하면 안된다.
* 모든 연산자는 연산자 대신 함수로도 호출 할 수 있다.
* not 함수 이름인데 팩토리얼 구현은 어울리지 않다.

```kotlin
print (10 * 6.not()) // 7200
```

예를들어 다음의 코드를 보자.

```kotlin
×+y == 2
```

이 코드는 언제나 다음과 같은 코드로 변환된다.

```kotlin
x.plus(y).equal(z)
```

* 참고로 만약 plus의 리턴 타입이 nullable이라면, 다음과 같이 변환된다.

```kotlin
(x.plus(y))?.equal(z) ?: (z == NULL)
```

* 연산자에 기대하는 역할을 벗어나게 된다.
* 따라서 팩토리얼을 계산하기 위해서 ! 연산자를 사용하면 안된다.
* 이는 관례에 어긋나기 때문이다.

## 분명하지 않은 경우

* 관례를 충족하는지 확실하지 않을때가 있다.
* 함수를 세 배 한다는건 무슨 의미일까?

예를들어 세번 반복하는 함수를 만들어 낼 수 있고 어떤 사람은 세번 호출한다는걸 쉽게 이해할 수 있다.

```kotlin
operator fun Int.times(operation: () -> Unit): () -> Unit =
	{ repeat (this) { operation () } }

3 * { print ("Hello") } // #9: HelloHelloHello
```

의미가  명확하지 않다면 infix 를 통한 확장함수를 사용하는 것이 좋다.

```kotlin
infix fun Int.timesRepeated(operation: ()->Unit) = {
repeat (this) < operation () }

val tripledHello = 3 timesRepeated { print ("'Hello"') }

tripledHello() // HelloHelloHello
```

사실은 이미 stdlib 에 구현되어 있다.

```kotlin
repeat (3) { print("Hello"') } // $8: HelloHelloHello
```

## 규칙을 무시해도 되는 경우

* 도메인 특화 언어 (DSL)은 무시해도 된다.
* 예를들어 고전적인 HTML DSL 은 다음과 같다.

```kotlin
body {
	div {
		+"Some text"
```
