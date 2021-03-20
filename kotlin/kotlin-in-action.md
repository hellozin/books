# Part 1. 코틀린 소개

- 코틀린은 타입 추론을 지원하는 정적 타입 지정 언어다.
- 객체지향과 함수형 프로그래밍 스타일을 모두 지원한다.

[책 예제를 포함한 여러 코틀린 예제](https://try.kotlinlang.org)

# Part 2. 코틀린 기초

**기본적인 함수의 구조**

```kotlin
fun main(args: Array<String>) {
    println("Hello, World!")
}
```

```kotlin
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}

fun max(a: Int, b: Int) = if (a > b) a else b
```

- 함수는 클래스 내가 아닌 최상위 수준에 존재할 수 있다.
- println 과 같이 자바 표준 라이브러리 함수에 대한 wrapper 를 제공한다.
- 세미콜론을 붙이지 않아도 된다.
- 반환 타입은 뒤에 명시한다.
- if 와 같이 루푸를 제외한 대부분의 제어 구조는 statement 가 아닌 expression 이다.
- 함수 본문이 block 인 경우는 반드시 반환 타입을 지정하고 return문으로 반환 값을 지정한다.

**변수**

- val: value 의 약자로 immutable 한 변수이다.
- var: variable 의 약자로 mutable 한 변수이다.
- 변수 지정과 동시에 초기화 하지 않는다면 타입 추론을 할 수 없기 때문에 타입을 명시해야 한다.

**문자열 템플릿**

```kotlin
var name = "Bob"
var age = 20
println("Hello, $name!")
println("I'm ${age} years old.")
```

**클래스와 프로퍼티**

```kotlin
class Person(
    val name: String,
    var age: Int
)

fun greeting() {
    val person = Person("Bob", 20)
    // person.name = "Aba" (X, immutable)
    person.age = 21
    println("${person.name} is ${person.age} years old.")
}
```

- Person 클래스에 var, val로 선언한 필드를 프로퍼티라고 한다.
- var 프로퍼티는 setter 와 getter, val 프로퍼티는 getter 가 제공된다.
- 별도의 setter, getter 메소드 없이 프로퍼티 명으로 접근 가능하다.

```kotlin
class Person(var name: String, var age: Int) {
    val originName = name
    val isAdult: Boolean
        get() {
            return age > 20
        }
}
```

- custom setter, getter 는 위와 같이 지정할 수 있다.

**디렉터리와 패키지**

```kotlin
package me.hellozin.model
import me.hellozin.model.Person
```

- 패키지를 지정하면 해당 파일 내 모든 선언(클래스, 함수, 프로퍼티)이 해당 패키지에 속하게 된다.
- 같은 패키지에 속해 있다면 파일이 달라도 동일하게 사용할 수 있다.
- 클래스와 함수 모두 동일하게 import 할 수 있다.
- 자바와 달리 디렉터리 구조와 패키지 구조를 같게 할 필요는 없다.
  - 하지만 같게 유지하는 것을 권장한다.
  - 다만 작은 크기의 여러 클래스를 하나의 파일에서 관리하는 것은 괜찮다.

**enum 과 when**

```kotlin
enum class Color(val r: Int, val g: Int, var b: Int) {
    RED(255,0,0), ORANGE(255,165,0), YELLOW(255,255,0);

    fun rgb() = (r & 256 + g) * 256 + b
}
```

- enum class 내에 함수를 정의하는 경우 반드시 상수 목록 뒤에 세미콜론을 붙여야 한다.

```kotlin
fun like(color: Color) {
    when (color) {
        RED, ORANGE -> "like"
        YELLOW -> "dislike"
    }
}```

```kotlin
fun describe(input: String) {
    when (input) {
        "name" -> "bob"
        "class" -> "A"
        else -> "none"
    }
}
```

- enum 타입의 경우 모든 상수 목록을 처리하면 else 문을 사용하지 않아도 된다.

**스마트 캐스트**

```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int =
    when (e) {
        is Num ->
            e.value
        is Sum ->
            eval(e.right) + eval(e.left)
        else ->
            throw IllegalArgumentException("Unknown expression")
    }

fun main(args: Array<String>) {
    println(eval(Sum(Num(1), Num(2))))
}
```

- `is` 로 변수 타입을 검사 할 수 있다.
- `is` 로 타입을 검사한 후에는 해당 타입으로 **스마트 캐스트** 된다.
- 스마트 캐스트는 아래 조건에 한해 적용됩니다
  - Reference: [Type checks and casts](https://kotlinlang.org/docs/typecasts.html)
  - val local properties: local delegated properties를 제외한 모든 경우
  - val properties
    - 프로퍼티 visibility 가 private 혹은 internal 인 경우
    - 캐스트 확인이 프로퍼티의 정의와 같은 모듈(컴파일 시 포함되는 그룹)에서 이루어질 경우
  - var local variables: local delegated property 가 아니고 캐스트 확인과 프로퍼티 사용 사이 변경사항이 없는 경우
    - 람다 내부의 수정사항은 확인할 수 없다.
  - var properties: 언제든 변경될 수 있기 때문에 불가능하다.

**루프**

```kotlin
while (condition) { ... }
do { ... } while (condition)
```

```kotlin
for (i in 1..10) {
    println(i)
}

for (i in 10 downTo 1 step 2) {
    println(i)
}

val list = arrayListOf(1,2,3)
for ((index, val) in list.withIndex()) {
    println("$index: $val")
}

val map = TreeMap<Int, Int>()
for ((key, val) in map) {
    println("$key: $val")
}
```

**in**

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
```

**예외처리**

```kotlin
fun readNumber(reader: BufferedReader): Int? {
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    }
    catch (e: NumberFormatException) {
        return null
    }
    finally {
        reader.close()
    }
}
```

```kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        null
    }

    println(number)
}

fun main(args: Array<String>) {
    val reader = BufferedReader(StringReader("not a number"))
    readNumber(reader)
}
```

- 자바와 달리 함수 선언 뒤에 throws 키워드가 붙지 않는다.
- try 도 expression 으로 사용할 수 있다.
  - if 와 달리 반드시 본문을 {} 로 감싸야 한다.
- 코틀린은 모든 exception 을 unchecked 로 처리한다.