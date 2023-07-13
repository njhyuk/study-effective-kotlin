# item 9: use를 사용하여 리소스를 닫아라
더이상 필요하지 않을때 close 메서드로 명시적으로 닫아야 하는 리소스들.

* InputStream, OutputStream
* java.sql.Connection
* java.io.Reader(FileReader, BufferedReader, CSSParser)
* java.new.Socket과 java.util.Scanner

## AutoCloseable
* 이러한 리소스들은 AutoCloseable 을 상속 받은 Closeable 인터페이스를 구현 하고 있음.
* 이러한 리소스들은 레퍼런스가 없어질 때, 가비지 컬렉터가 처리함.
* 굉장히 느리고 쉽게 처리되지 않음.
* 명시적으로 close 메서드를 호출해 주는게 좋음.

## 전통적인 처리 방식
* try-finally 블록을 사용한 처리
  * 복잡함
  * try 블록과 finally 블록 내부의 오류가 서로 공유 되지 않음

```kotlin
fun countCharactersInfile(path: String): Int {
	val reader = BufferedReader (FileReader (path))
	try {
		return reader.lineSequence().sumBy { it. length }
	} finally {
	 reader.close()
	}
}
```

## 제안 : use 방식
> Closeable / AutoCloseable 을 구현한 객체를 처리하는 방법 

```kotlin
fun countCharactersInFile (path: String): Int {
	val reader = BufferedReader(FileReader(path))
	reader.use {
		return reader.lineSequence().sumBy { it.length }
	}
}
```

## 추가 : 한줄씩 처리 해야 하는 경우
> 메모리에 파일의 내용을 한 줄 씩만 유지, 대용량 파일 처리시 사용

```kotlin
fun countCharactersInFile (path: String): Int {
	File(path).useLines { lines ->
		return lines.sumBy { it.length }
	}
}
```





