# item 50 컬렉션 처리 단계 수를 제한하라

- 모든 컬렉션 처리 메소드는 비용이 많이 든다.
    - 내부적으로 계산을 위해 추가적 컬렉션을 만든다.
    - 시퀀스도 시퀀스 전체를 랩하는 객체가 만들어진다.
    - 컬렉션 처리 단계 수를 제한하는 것이 좋다.

```kotlin
class Student (val name: String?)

// Bad
fun List<Student>.getNames(): List<String> = 
		this.map { it.name }
		.filter{ it != null }
		.map { it!! }

// Good
fun List<Student>.getNames(): List<String> = 
		this.map{ it. name }.filterNotNull()

// Best
fun List<Student>.getNames(): List<String> = 
		this.mapNotNull { it.name }
```

![Untitled](item%2050%20%E1%84%8F%E1%85%A5%E1%86%AF%E1%84%85%E1%85%A6%E1%86%A8%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%20%E1%84%83%E1%85%A1%E1%86%AB%E1%84%80%E1%85%A8%20%E1%84%89%E1%85%AE%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%86%AB%E1%84%92%E1%85%A1%E1%84%85%E1%85%A1%20144d4a18b191463aafe418fe4f56f039/Untitled.png)

대부분의 컬렉션 처리 단계는 전체 컬렉션에 대한 반복과, 중간 컬렉션 생성 단계 비용이 발생하니 주의가 필요하다.