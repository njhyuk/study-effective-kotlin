# item 49

하나 이상의 처리 단계를 가진 경우에는 시퀀스를 사용하라

```kotlin
interface Iterable<out T> {
	operator fun iterator() : Iterator<T>
}

interface Sequence<out T> {
	operator fun iterator() : Iterator<T>
}
```

- 이터러블과 시퀀스는 동일해 보이지만 완전히 다른 목적으로 설계됨
- 시퀀스는 무엇보다 지연(lazy) 처리됨
    - 데코레이터 패턴으로 꾸며진 새로운 시퀀스가 리턴됨
    - 최종적 계산은 toList, count 등의 최종 연산이 이루어질 때 수행됨
- 이터러블은 처리 함수를 사용할 때 마다 연산이 이루어짐

```kotlin
val seq = sequence0f (1,2,3)
val filtered = seq.filter { print("f$it "); it % 2 == 1 }
println(filtered) // FilteringSequence@.. (아직 연산 X)

val asList = filtered.toList()
// f1 f2 f3 (toList 하니까 연산되서 출력)
printin(asList) // [1, 3]
```

## 순서의 중요성

- 시퀀스는 요소 하나하나에 지정한 연산을 한꺼번에 적용함
    - (element-by-element order 또는 lazy order라고 부름)
- 이러터블은 요소 전체를 차근차근 연산
    - (eager order 또는 step-by-step order 라고 부름)

```kotlin
// 이터러블 처리
// 1,2,3 순회 필터, 순회 맵, 순회 forEach
listof(1,2,3)
.filter { print("F$it, "); it % 2 == 1 }
.map { print("M$it, "); it * 2 }
.forEach { print("E$it, ")
// 출력 : F1, F2, F3, M1, M3, E2, E6

// 시퀀스 처리
// 1 먼저 (필터,맵,each)
// 그다음 2 필터
// 그다음 3 필터,맵,each
sequence0f(1,2,3)
.filter { print("F$it, "); it % 2 == 1  }
.map { print("M$it, "); it * 2 }
.forEach { print("E$it, ") }
// 출력 : F1, MI, EZ, F2, F3, M3, E6
```

![Untitled](item%2049%203eb867152482407da19e741710cece2f/Untitled.png)

만약 전통적 반복문으로 구현 했다면?

```kotlin
for (e in listOf (1,2,3)) {
	print("F$e, ")
	if (e % 2 = 1) {
		print ("M$e, ")
		val mapped = e * 2
		print ("E$mapped, ")
	}
}
// 출력: F1, M1, EZ, F2, F3, M3, E6 -> 시퀀스 처리 결과와 같다!
```

즉 시퀀스 처리에 사용되는 방법이 기본적 반복문 조건문을 사용하는 코드와 같으므로 자연스럽다.

## 최소 연산

앞의 요소 10개만 필요한 경우?

- 이터러블
    - 중간 연산 개념이 없어, 원하는 처리를 컬렉션 전체에 적용
- 시퀀스
    - 중간 연산 개념이 있어서 앞의 요소 10개에만 원하는 처리

![Untitled](item%2049%203eb867152482407da19e741710cece2f/Untitled%201.png)

## 무한 시퀀스

- 시퀀스는 최종 연산이 일어나기 전까지, 어떠한 연산 X
- 무한 시퀀스를 만들고, 필요한 부분까지만 값을 추출하는 것도 가능

### generateSequence 로 만들기

첫번째 요소와 그 다음 요소를 계산하는 방법을 지정

```kotlin
generateSequence(1) { it + 1 }
	.map { it * 2 }
	.take(10)
	.forEach { print("$it, ") }
// 출력: 2, 4, 6, 8, 10, 12, 14, 16, 18, 20,
```

### sequence 로 만들기

시퀀스 빌더는 중단 함수(suspending function) 내부에서 yield로 값을 하나씩 만들어냄

```kotlin
val fibonacci = sequence {
	yield (1)

	var current = 1
	var prev = 1

	while (true) {
		yield(current)

		val temp = prev
		prev = current
		current += temp
	}
}

print(fibonacci.take(10).toList())
// [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
```

무한 시퀀스는 값 활용 개수를 지정하지 않으면 무한하게 반복함

```kotlin
print(fibonacci.tolist()) //종료되지 않습니다.
```

무한 반복에 빠지는 경우가 은근 많다.

조건문 쓰려고 any를 썼는데 true를 리턴하지 못하면, 무한 반복에 빠짐.

take와 first 정도만 사용하는 것이 좋음.

## 시퀀스는 각각의 단계에서 컬렉션을 만들지 X

```kotlin
numbers.filter { it % 10 = 0 3  } // 여기에서 컬렉션 하나
.map { it * 2 } // 여기에서 컬렉션 하나
sum()
//전체적으로 2개의 컬렉션이 만들어집니다.

numbers.asSequence()
.filter { it % 10 == 0 }
.map { it * 2 }
sum()
// 컬렉션이 만들어지지 않습니다.
```

극단적으로 1.53GB 의 CSV 파일을 filter, map, each 한다면?

중간 컬렉션 3번 생성, 연산.. OOM 발생함.

시퀀스를 사용하면 효과적임.

필자의 경험으로 하나 이상의 처리단계가 필요한 컬렉션은 성능 향상 20~40%

## 시퀀스가 빠르지 않은 경우

- stdlib의 sorted
    - sorted는 시퀀스를 List 로 변환 한 뒤에, 자바 stdlib 의 sort 를 사용해 처리함
    - 컬랙션 처리보다 느려짐
- 무한시퀀스에서 sorted를 적용하면 무한 반복 이슈

```kotlin
generateSequence (0) { it + 1 ).take(10).sorted().toList()
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

generateSequence(0) { it + 1 ).sorted().take(10).toList()
//종료되지 않습니다.
// 따라서 어떤 값도 리턴하지 않습니다.
```

sorted는 sequence보다 Collection이 더 빠른 희귀한 예

## 자바 스트림의 경우

코틀린의 시퀀스와 비슷한 형태로 동작함

```kotlin
productsList.asSequence()
	.filter { it. bought }
	.map { it.price }
	.average()

productslist.stream()
	.filter { it.bought }
	.mapToDouble { it.price }
	.average()
	.orElse(0.0)
```

자바 8의 스트림도 lazy하게 작동하며, 마지막 처리 단계에서 연산함

### 차이점

- 시퀀스가 더 많은 처리 함수를 가지고 사용하기 쉬움
    - 스트림 : collect(Collectors.toList())
    - 시퀀스 : toList()
- 자바 스트림은 병렬 함수를 사용해서 병렬 모드 실행 가능
- 시퀀스는 JVM, Javascript, Native 환경에서 사용 가능, 스트림은 JVM 환경에서만 사용 가능
- 필자의 경험
    - 병렬 모드로 이득을 얻을 수 있다면 자바 스트림
    - 일반적인 경우는 코틀린 시퀀스 사용 추천
    
    ## 시퀀스 정리
    
    - 자연스러운 처리 순서를 유지합니다.
    - 최소한만 연산합니다.
    - 무한 시퀀스 형태로 사용할 수 있습니다.
    - 각각의 단계에서 컬렉션을 만들어 내지 않습니다.