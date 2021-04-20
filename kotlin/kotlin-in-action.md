- [Part 1. 코틀린 소개](#part-1-코틀린-소개)
- [Part 2. 코틀린 기초](#part-2-코틀린-기초)
- [Part 3. 함수 정의와 호출](#part-3-함수-정의와-호출)
- [Part 4. 클래스, 객체, 인터페이스](#part-4-클래스-객체-인터페이스)
- [Part 5. 람다 프로그래밍](#part-5-람다-프로그래밍)
- [Part 6. 코틀린 타입 시스템](#part-6-코틀린-타입-시스템)
- [Part 7. 연산자 오버로딩과 기타 관례](#part-7-연산자-오버로딩과-기타-관례)

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

# Part 3. 함수 정의와 호출

**컬렉션**

코틀린에서는 자바 컬렉션과 코틀린 자체 컬렉션을 모두 지원한다.

**자바 컬렉션**

```kotlin
val list = arrayListOf(1,2,3)
val set = hashSetOf(1,2,3)
val map = hashMapOf(1 to "one", 2 to "two", 3 to "three")

println(list.javaClass) // javaClass == getClass()
>> class java.util.ArrayList
```

- `to` 도 일반 함수이며 뒤에서 자세히 배운다.
- 자바 컬렉션을 사용하기 때문에 기존 자바 코드와 상호작용하기 쉽다.

**코틀린 컬렉션**

자바 컬렉션에 비해 더 많은 기능을 제공한다.

```kotlin
val strings = listOf("first", "second", "fourteenth")
println(string.last())

val numbers = setOf(1, 14, 2)
println(numbers.max())
```

**이름 붙인 인자**

```kotlin
val result1 = joinToString(collection, ", ", "[", "]")
val result2 = joinToString(collection, separator = ", ", prefix = "[", postfix = "]")
```

**디폴트 파라미터**

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "[",
    postfix: String = "]"
)

val result1 = joinToString(collection)
```

**확장 함수**

원하는 클래스의 멤버 함수처럼 동작하는 함수를 클래스 밖에서 정의할 수 있다.

```
package strings
fun String.lastChar(): Char = this.get(this.length -1)

println("Kotlin".lastChar())
```
- `String.lastChar()`에서 클래스 이름인 `String`을 수신 객체 타입(receiver type) 이라고 한다.
- `this.get..`에서 함수가 호출되는 대상인 `this`를 수신 객체(receiver object) 라고 한다.
- 일반 메소드와 마찬가지로 확장함수 본문에서도 `this`를 생략 할 수 잇다.

```kotlin
fun String.lastChar(): Cahr = get(length -1)
```

- 다만 확장함수에서 receiver type 의 private, protect 멤버를 사용 할 수는 없다.

```kotlin
import strings.lastChar

import strings.*

import strings.lastChar as last
```

- 확장 함수를 사용하기 위해서는 사용하는 코드에서 import를 해야 한다.
- 함수 이름이 충돌하는 경우 `as` 를 통해 이름을 바꿔 해결할 수 있다.
- 확장 함수는 정적 메소드와 같이 동작하기 때문에 하위 클래스에서 오버라이드 할 수 없다.
- 확장 함수와 클래스의 멤버 함수가 이름, 시그니처 모두 동일하다면 멤버 함수가 호출된다.

**확장 프로퍼티**

```kotlin
val String.lastChar: Char
    get() = get(length - 1)
var StringBuilder.lastChar: Char
    get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
    }
```

- 확장 프로퍼티는 기존 클래스에 필드를 추가할 방법이 없기 때문에 상태를 가질 수는 없다.
- 기본 getter 구현을 제공하지 않기 때문에 최소한 getter 는 구현해야 한다.
- 초기화 코드도 쓸 수 없다?

**컬렉션 처리**

앞서 살펴봤던 코틀린 컬렉션의 추가 기능은 자바 컬렉션에 확장 함수를 더해 제공되고 있다.

**가변 인자**

```kotlin
fun myfun(vararg values: String) { ... }
```

배열을 넘기고 싶을 때 자바는 그냥 넘길 수 있지만 코틀린은 스프레드 연산자를 통해 배열을 명시적으로 풀어야 한다.

```kotlin
fun main(args: Array<String>) {
    val list = listOf("args: ", *args)
    println(list)
}
```

**중위 호출**

앞서 map을 초기화 할때 `to` 함수를 사용했는데 이렇게 사용하는 방식을 중위 호출 이라고 한다.

```kotlin
val map = hashMapOf(1 to "one", 2 to "two", 3 to "three")
```

- 중위 호출은 수신 객체와 유일한 메소드 인자 사이에 메소드 이름을 넣어 사용한다.
    - `1.to("one")`
    - `1 to "one"`
- 함수를 중위 호출 가능하게 만들기 위해서는 infix 라는 prefix 를 붙여준다.
    - `infix fun Any.to(other: Any) = Pair(this, other)`

**구조 분해 선언**

```kotlin
val (number, name) = 1 to "one"
```

- 위와 같이 선언하는 것을 구조 분해 선언이라고 한다.
- 7장에서 자세히 배운다.

**문자열과 정규식**

- 코틀린 문자열은 자바 문자열과 같다
- 다만 조금 더 명확하고 편리한 사용을 위한 확장 함수를 제공한다.

```kotlin
fun main(args: Array<String>) {
    println("12.345-6.A".split("\\.|-".toRegex()))
}
```

- 자바와 달리 일반 문자열과 정규식을 처리하는 함수가 분리되어 있어 쉽게 알 수 있다.

```kotlin
fun main(args: Array<String>) {
    println("12.345-6.A".split(".", "-"))
}
```

- 여러개의 구분자를 지원하는 함수도 제공한다.

```kotlin
fun parsePath(path: String) {
    val directory = path.substringBeforeLast("/")
    val fullName = path.substringAfterLast("/")

    val fileName = fullName.substringBeforeLast(".")
    val extension = fullName.substringAfterLast(".")

    println("Dir: $directory, name: $fileName, ext: $extension")
}

fun main(args: Array<String>) {
    parsePath("/Users/yole/kotlin-book/chapter.adoc")
}
```

- 다양한 케이스의 편리한 기능을 제공한다.

```kotlin
val kotlinLogo = """| //
                   .|//
                   .|/ \"""

fun main(args: Array<String>) {
    println(kotlinLogo.trimMargin("."))
}
```

- `"""` 로 감싼 문자열은 이스케이프를 할 필요가 없다.

**로컬 함수와 확장**

```kotlin
class User(val id: Int, val name: String, val address: String)

fun User.validateBeforeSave() {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
               "Can't save user $id: empty $fieldName")
        }
    }

    validate(name, "Name")
    validate(address, "Address")
}

fun saveUser(user: User) {
    // User.validateBeforeSave를 여기에 정의 할 수도 있지만
    // 중첩된 함수의 깊이가 너무 깊어지는 것을 방지하기 위해 권장하지는 않는다.
    user.validateBeforeSave()

    // Save user to the database
}

fun main(args: Array<String>) {
    saveUser(User(1, "", ""))
}

```

- User 클래스의 확장함수로 선언함으로써 내부 프로퍼티에 직접 접근이 가능하다.
- User 클래스와 validate 로직을 분리할 수 있다.

# Part 4. 클래스, 객체, 인터페이스

**인터페이스**

- 추상 메소드, 구현된 메소드를 정의할 수 있다.
- 상태(필드)는 들어갈 수 없다.
- 자바의 extends와 implements를 모두 콜론(`:`)으로 처리한다.

```kotlin
interface Clickable {
    fun click()
}
class Button : Clickable {
    override fun click() = println("I was clicked")
}
```

- 구현할 메소드에는 override 변경자가 꼭 있어야 한다.

같은 이름과 시그니쳐를 가진 메소드를 가진 두개의 인터페이스를 동시에 구현하면

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")
}

interface Focusable {
    fun setFocus(b: Boolean) = println("I ${if (b) "got" else "lost"} focus.")
    fun showOff() = println("I'm focusable!")
}

class Button : Clickable, Focusable {
    override fun click() = println("I was clicked")

    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

- Button 클래스에서 `showOff()` 메소드를 override하지 않으면 컴파일 에러가 난다.

**상속**

- 코틀린 클래스는 기본적으로 상속이 불가능하다.
- 상속하고 싶은 경우 상속을 원하는 클래스, 프로퍼티, 메소드에 open 변경자를 붙여준다.

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")
}

open class RichButton : Clickable {
    fun disable() {} // 하위 클래스에서 오버라이드 할 수 없다.
    open fun animate() {} // 하위 클래스에서 오버라이드 할 수 있다.
    override fun click() {} // override 한 메소드는 기본적으로 열려있다.(open)
    // final override fun click() {} // override 한 메소드를 열고 싶지 않은 경우 final 키워드를 사용한다.
}
```

- 기본 상속 가능 상태를 final로 함으로써 다양한 경우에 스마트 캐스트를 적용할 수 있게 되었다.

**추상 클래스**

- 추상 클래스는 추상 멤버가 존재하기 때문에 하위 클래스에서 이를 구현하는 것이 일반적이다.
- 이 때문에 추상 멤버 앞에는 open 변경자를 명시할 필요가 없다.

```kotlin
abstract class Animated {
    abstract fun animate()
    open fun stopAnimating() {} // 추상 멤버가 아닌 경우는
    fun animateTwice() {} // 여전히 open 여부를 선택할 수 있다.
}
```

**상속 제어** **변경자**

- `final`: default, 오버라이드 할 수 없음
- `open`: 오버라이드 할 수 있음
- `abstract`: 반드시 오버라이드 해야 함
- `override`: 상위 멤버를 오버라이드 함

**가시성 변경자 (visivility modifier)**

- `public`: default, 모든 곳에서 볼 수 있다.
- `internal`: 같은 모듈 안에서만 볼 수 있다.
    - 모듈: 한번에 컴파일되는 코틀린 파일 그룹
- `protected`: 클래스 멤버에 지정된 경우 하위 클래스에서만 볼 수 있고 최상위 선언에는 적용할 수 없다.
- `private`: 클레스 멤버에 지정된 경우 캍은 클래스에서만 볼 수 있고 최상위 선언에 지정된 경우 같은 파일 내에서만 볼 수 있다.
- 코틀린에는 package-private이 없다.

```kotlin
internal open class TalkativeButton: Focusable {
	private fun yell() = println("Hey!")
	protected fun whisper() = println("Let's talk!")
}

fun TalkativeButton.giveSpeech() { // internal 클래스를 public 확장함수로 참조할 수 없다.
	yell() // 확장 함수는 private에 접근할 수 없다.
	whisper() // 확장 함수는 protected에 접근할 수 없다.
}
```

- 자바에는 internal에 맞는 가시성이 없기 때문에 public으로 사용된다. 이때문에 접근 범위가 달라지는 문제가 생길 수 있다.
- 이를 통해 생길 문제를 예방하기 위해 코틀린 컴파일러가 internal 멤버의 이름을 이상하게 바꾼다.

**중첩 클래스**

```kotlin
import java.io.Serializable

interface State: Serializable

interface View {
    fun getCurrentState(): State
    fun restoreState(state: State) {}
}

class Button : View {
    override fun getCurrentState(): State = ButtonState()

    override fun restoreState(state: State) { /*...*/ }

    class ButtonState : State { /*...*/ }
}

/* java */
public class Button implements View {
	@Override
	public State get CurrentState() {
		return new ButtonState();
	}

	@Override
	public void restoreState(State state) {..}

	public class ButtonState implements State {..}
}
```

- 아래 자바 코드로 직렬화를 하면 `NotSerializableException`이 발생한다.
    - 자바에서 중첩 클래스는 기본적으로 내부 클래스가 되고 외부 클래스를 묵시적으로 참조한다.
    - 이 문제를 해결하려면 `ButtonState`를 static class로 지정해야 한다.
- 코틀린의 중첩 클래스는 기본적으로 내부 클래스가 아니다.
- 코틀린에서 내부 클래스를 사용하려면 `inner` 변경자를 붙여야 한다.

```kotlin
class Outer {
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
```

- 내부 클래스에서 외부 클래스를 참조할 때는 `this@외부클래스` 와 같이 사용한다.

**제한된 하위 클래스**

```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.left)
        else ->
            throw IllegalArgumentException("Unknown expression")
    }
```

- `Expr` 인터페이스가 상속 가능한 상태이기 때문에 분기 시 `else` 문이 꼭 필요하다.

```kotlin
sealed class Expr {
    class Num(val value: Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
}

fun eval(e: Expr): Int =
    when (e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)
    }
```

- `sealed(봉인된)` 키워드를 사용하면  하위 클래스를 모두 중첩 클래스로 정의해야 한다.
    - 이 제한은 코틀린 1.1에서 완화되었다. 1.1 부터는 파일 내 아무데서나 하위 클래스를 만들 수 있다.
- 하위 클래스가 제한되어 별도의 `else` 분기가 필요없어졌다.
- `sealed` 클래스는 자동으로 `open` 이 된다.

**생성자**

```kotlin
class User(val name: String)
```

- 클래스 이름 뒤 소괄호에 들어가는 코드를 주 생성자(primary constructor)라고 한다.

```kotlin
class User constructor(_nickname: String) {
    val nickname: String
    init {
        nickname = _nickname
    }
}
```

- 명시적으로 선언하면 위와 같은 형태가 된다.
- constructor 키워드는 주, 부 생성자 정의를 시작할 때 사용된다.
- init 키워드는 초기화 블록을 시작한다.
- 필요한 경우 여러 초기화 블록을 선언할 수 있다.
- 언더바(`_`) 대신 자바처럼 `this`를 사용해도 된다.(`this.name = name`)

```kotlin
class User(val nickname: String, val isSubscribed: Boolean = true)
```

- 디폴트 값을 지정할 수 있다.
- 모든 파라미터에 디폴트 값을 지정하면 컴파일러가 자동으로 기본 생성자를 만들어주는데 이는 기본 생성자를 필요로 하는 자바 라이브러리와의 통합을 쉽게 해준다.

```kotlin
open class User(val name: String)
class MyUser(val nickname: String): User(nickname)
```

- 기반 클래스의 생성자를 호출해야 할 경우 위와 같이 값을 지정할 수 있다.

```kotlin
open class Button
class RadioButton: Button()
```

- 생성자를 정의하지 않으면 컴파일러가 인자 없이 아무 동작도 하지 않는 생성자를 만들어주고 이를 상속 할 경우 하위 클래스는 반드시 기반 클래스의 생성자를 호출해야 한다.
- 뒤에 소괄호가 붙는지 여부를 보고 인터페이스인지 클래스인지 확인할 수도 있다.

```kotlin
class Secretive private constructor () {}
```

- 클래스를 외부에서 인스턴스화 하지 못하게 하려면 모든 생성자를 private으로 지정하면 된다.

**부 생성자**

```kotlin
open class View {
    constructor(ctx: Context) {}
    constructor(ctx: Context, attr: Attribute) {}
}

class Button: View {
    constructor(ctx: Context): this(ctx, MY_STYLE) {}
    constructor(ctx: Context, attr: Attribute): super(ctx, attr) {}
}
```

- constructor 키워드로 지정할 수 있다.
- `this` 키워드로 클래스 내 다른 생성자를, `super` 키워드로 기반 클래스의 생성자를 호출할 수 있다.
- 클래스에 주 생성자가 없다면 반드시 상위 클래스를 초기화하거나 다른 생성자에게 위임해야 한다.

**추상 프로퍼티**

```kotlin
interface User {
    val nickname: String
}

class PrivateUser(override val nickname: String) : User

class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@')
}

fun getFacebookName(accountId: Int) = "fb:$accountId"

class FacebookUser(val accountId: Int) : User {
    override val nickname = getFacebookName(accountId)
}
```

- 코틀린에서는 인터페이스에 추상 프로퍼티 선언을 넣을 수 있다.
- 인터페이스는 상태를 가질 수 없기 때문에 구현할 클래스에서 해당 프로퍼티의 값을 처리해야 한다.

```kotlin
interface User {
    val email: String
    val nickname: String
        get() = email.substringBefore('@')
}
```

- `email`은 하위 클래스에서 반드시 오버라이드 해야 한다.
- `nickname`은 오버라이드 하지 않고 사용할 수 있다.

**Getter와 Setter에서 backing field 접근**

```kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            println("""
                Address was changed for $name:
                "$field" -> "$value".""".trimIndent())
            field = value
        }
}
```

- 커스텀 접근자 내에서 backing field에 접근해 원하는 로직을 추가할 수 있다.
- 접근은 `field` 라는 특별한 식별자를 통해 할 수 있다.
- `field` 식별자를 사용하지 않으면 컴파일러가 backing field를 자동으로 생성하지 않는다.
    - ? 프로퍼티가 val 인 경우에는 게터에 field가 없으면 되지만, var 인 경우에는 게터나 세터 모두에 field가 없어야 한다.

**접근자의 가시성 변경**

- 접근자의 가시성은 기본적으로 프로퍼티의 가시성과 같다.
- get, set 앞에 가시성 변경자를 추가해 변경할 수 있다.

```kotlin
class LengthCounter {
    var counter: Int = 0
        private set

    fun addWord(word: String) {
        counter += word.length
    }
}
```

**프로퍼티에 대해 나중에 다룰 내용**

p. 171

**데이터 클래스**

주로 정의하는 메소드들

```kotlin
class Client(val name: String, val postalCode: Int) {

    override fun equals(other: Any?): Boolean {
        if (other == null || other !is Client) return false
        return name == other.name && postalCode == other.postalCode
    }

    override fun hashCode(): Int = name.hashCode() * 31 + postalCode

    override fun toString() = "Client(name=$name, postalCode=$postalCode)"
}
```

- 코틀린은 `==` 연산자 사용 시 equals를 호출한다. 참조 비교는 `===` 를 사용한다.

어떤 클래스가 데이터를 저장하는 역할만 수행한다면 toString, equals, hashCode를 반드시 오버라이드 해야한다. 코틀린은 이런 메소드를 자동으로 생성하고 보이지않게 내부적으로 처리하는 `data` 클래스를 제공한다.

```kotlin
data class Client(val name: String, val postalCode: Int)
```

생성되는 메소드

- equals() / hashCode()
    - 주 생성자의 프로퍼티를 기반으로 생성된다.
- toString (`"Client(name=Bob, postalCode=973293)"`)
- copy()
    - 아래에서 설명한다
- componentN()
    - 7장에서 설명한다

**copy 메소드**

- 데이터 클래스는 모든 프로퍼티를 val로 선언해 불변 객체로 유지하기를 권장하지만 원한다면 var을 사용할 수 있다.
- copy는 불변 객체를 더 편하게 사용할 수 있도록 객체를 복사하며 일부 프로퍼티를 변경할 수 있는 메소드이다.

```kotlin
data class Client(val name: String, val postalCode: Int)

fun main(args: Array<String>) {
    val bob = Client("Bob", 973293)
    println(bob.copy(postalCode = 382555))
}
```

**클래스 위임**

- 코틀린은 상위 클래스의 변경이 하위 클래스에 영향을 미치는 단점을 보완하기 위해 기본적으로 final로 취급하고 open을 통해 확장할 수 있도록 했다.
- 상속을 허용하지 않는 클래스에 새로운 동작을 추가해야 할 경우 주로 사용하는 데코레이터 패턴을 `by` 키워드를 통해 제공한다.

```kotlin
interface Base {
    fun getName(): String
    fun getAge(): Int
}

class BaseImpl : Base {
    override fun getName() = "hello"
    override fun getAge() = 10
}

class MyBaseImpl(val baseImpl : BaseImpl = BaseImpl()) : Base by baseImpl {
    override fun getName() = baseImpl.getName() + ", bye"
}
```

**object 키워드**

object 키워드를 사용하는 여러 상황

- **객체 선언**(object declaration)은 싱글턴을 정의하는 방법 중 하나이다.
- **동반 객체**(companion object)는 인스턴스 메소드는 아니지만 어떤 클래스와 관련있는 메소드와 팩토리 메소드를 담을 때 쓰인다. 동반 객체 메소드에 접근할 때는 동반 객체가 포함된 클래스의 이름을 사용할 수 있다.
- 객체 식은 자바의 **익병 내부 클래스**(anonymous inner class) 대신 쓰인다.

**객체 선언**

- 코틀린은 객체 선언, `object` 키워드를 통해 싱글톤을 쉽게 제공한다.
- 객체 선언에는 프로퍼티, 메소드, 초기화 블록이 들어갈 수 있지만 주, 부 생성자는 사용할 수 없다.
- 인터페이스나 클래스를 상속할 수 있다.

```kotlin
object CaseInsensitiveFileComparator : Comparator<File> {
    override fun compare(file1: File, file2: File): Int {
        return file1.path.compareTo(file2.path,
                ignoreCase = true)
    }
}

fun main(args: Array<String>) {
    println(CaseInsensitiveFileComparator.compare(
        File("/User"), File("/user")))
    val files = listOf(File("/Z"), File("/a"))
    println(files.sortedWith(CaseInsensitiveFileComparator))
}
```

- 클래스 내부에서 객체 선언을 사용할 수도 있다.

```kotlin
data class Person(val name: String) {
    object NameComparator : Comparator<Person> {
        override fun compare(p1: Person, p2: Person): Int =
            p1.name.compareTo(p2.name)
    }
}
```

- 자바에서 object 클래스를 사용하려면 `클래스이름.INSTANCE.*` 와 같은 포멧으로 호출할 수 있다.
    - `CaseInsensitiveFileComparator.INSTANCE.compare`

**동반 객체**

- 코틀린에는 static 멤버가 없어 최상위 함수나 객체 선언으로 대신할 수 있다.
- 객체 내의 private 멤버에 접근이 필요한 경우 중첩된 객체 선언을 멤버로 정의해 사용할 수 있다.
- 이때, `companion` 키워드를 사용하면 상위 클래스의 이름으로 객체의 프로퍼티, 메소드에 접근할 수 있다.

```kotlin
class A {
    companion object {
        fun bar() {
            println("Companion object called")
        }
    }
}

fun main(args: Array<String>) {
    A.bar()
}
```

- 동반 객체는 private 생성자를 호출하기 좋은 장소다.
    - 동반 객체는 정의된 클래스의 모든 private 멤버에 접근할 수 있다.
    - 팩토리 메소드를 구현하기 적합하다.

**여러 생성자를 사용하는 경우**

```kotlin
fun getFacebookName(accountId: Int) = "fb:$accountId"

class User {
    val nickname: String

    constructor(email: String) {
        nickname = email.substringBefore('@')
    }

    constructor(facebookAccountId: Int) {
        nickname = getFacebookName(facebookAccountId)
    }
}

fun main(args: Array<String>) {
    val subscribingUser = User.newSubscribingUser("bob@gmail.com")
    val facebookUser = User.newFacebookUser(4)
    println(subscribingUser.nickname)
}
```

**동반 객체를 사용하는 경우**

```kotlin
fun getFacebookName(accountId: Int) = "fb:$accountId"

class User private constructor(val nickname: String) {
    companion object {
        fun newSubscribingUser(email: String) =
            User(email.substringBefore('@'))

        fun newFacebookUser(accountId: Int) =
            User(getFacebookName(accountId))
    }
}

fun main(args: Array<String>) {
    val subscribingUser = User.newSubscribingUser("bob@gmail.com")
    val facebookUser = User.newFacebookUser(4)
    println(subscribingUser.nickname)
}
```

- 생성자를 private으로 선언해 팩토리 메소드를 통해서만 인스턴스를 생성할 수 있다.
- 필요한 경우 각각의 팩토리 메소드에서 다른 객체를 반환할수도 있다.
- 동반 객체는 하위 클래스에서 오버라이드 할 수 없어 클래스를 확장할 경우 여러 생성자를 활용하는것이 더 좋다.

**동반 객체를 일반 객체처럼 사용**

- 동반 객체도 클래스 안에 정의된 일반 객체이기 때문에 이름을 붙이거나, 상속, 확장함수 사용 등이 가능하다.

```kotlin
interface Person {
    fun getRole(): String
}

class Student() {
    // 이름 붙이기, 인터페이스 구현
    companion object Role : Person {
        override fun getRole() = "student"
    }
}

fun printRole(person: Person) {
    println(person.getRole())
}

class Professor() {
    companion object {}
}

// 확장함수
fun Professor.Companion.getRole(): String = "professor"

fun main(args: Array<String>) {
    printRole(Student)
    printRole(Student.Role)
    println(Professor.getRole())
}
```

- 자바에서 접근할 때는 `클래스이름.Companion.*` 으로 호출할 수 있다.

**객체 식**

- 자바의 익명 내부 클래스와 유사한 익명 객체(anonymous object)를 정의할때도 object 키워드를 활용할 수 있다.
- 객체 식을 변수에 담아 사용할수도 있다.
- 객체 식 내부에서 로컬 변수를 사용할 수 있고 자바와 달리 final이 아닌 변수도 사용 가능하다.
- 익명 객체는 싱글턴이 아니다.

```kotlin
interface Person {
    fun getRole(): String
}

fun printRole(person: Person) {
    println(person.getRole())
}

fun main(args: Array<String>) {
    printRole(
        object : Person {
            override fun getRole() = "anonymous"
        }
    )

    // or

    var defaultRole = "anonymous"
    val roleHandler = object : Person {
        override fun getRole(): String = defaultRole
    }
    printRole(roleHandler)
}
```

# Part 5. 람다 프로그래밍

일련의 동작을 변수에 저장하거나 다른 함수에 넘겨야 할 때 람다를 사용한다.

```kotlin
data class Person(val name: String, val age: Int)

fun main(args: Array<String>) {
    val people = listOf(
        Person("a", 1),
        Person("b", 2),
        Person("c", 3)
    )
    println(people.maxByOrNull { it.age })      // 함수를 인자로 전달한다.
    println(people.maxByOrNull(Person::age))    // 멤버 참조는 뒤에서 설명한다.
}
```


**문법**

- 대부분의 경우 함수에 인자로 람다를 지정하고 변수를 선언해 저장할 수도 있다.
- 코틀린 람다 식은 중괄호로 둘러싸여 있다.

```kotlin
{ x: Int, y: Int -> x + y }
```


- 필요하다면 람다 식을 직접 호출할 수도 있다.
  - 거의 사용되지 않고 직접 호출하는 것이 더 명확하다.
- `run`은 인자로 받은 람다를 실행해주는 라이브러리 함수이다.
- 코틀린 람다 호출에는 아무 오버헤드가 없다. 8장에서 배운다.

```kotlin
{ println(10) } ()
run { println(10) }
```


- 처음 봤던 람다 식을 호출하는 방법은 다음과 같다.

```kotlin
// 가장 명시적인 호출
people.maxByOrNull({ p: Person -> p.age })

// 함수의 마지막 인자가 람다 식이면 괄호 밖으로 뺄 수 있다.
people.maxByOrNull() { p: Person -> p.age }

// 함수의 유일한 인자가 람다 식이고 괄호 밖으로 뺐다면 빈 괄호를 생략할 수 있다.
people.maxByOrNull { p: Person -> p.age }

// 타입 추론을 사용해 명시적인 타입 지정을 생략할 수 있다.
people.maxByOrNull { p -> p.age }

// 인자가 단 하나뿐이고 타입을 추론할 수 있는 경우 it 라는 이름으로 파라미터를 사용할 수 있다.
people.maxByOrNull { it.age }
```
- 단, `it`를 사용할 때 람다 안에 람다가 중첩되거나 의미를 명확하게 해야하는 경우 파라미터 이름을 지정하는 것이 좋다.


- 람다 식을 변수에 저장할 때는 타입 추론을 할 수 없어 타입을 명시해야 한다.

```kotlin
val getAge = { p: Person -> p.age }
```


- 본문이 여러 줄로 이뤄진 경우 마지막 식이 람다의 결과 값이 된다.

```kotlin
val sum = { x: Int, y: Int ->
    println("sum of $x, $y")
    x + y
}
```


**포획된 변수 (captured variable)**

- 자바의 익명 내부 클래스와 같이 코틀린 람다 식도 정의된 메소드의 로컬 변수를 람다 내부에서 사용할 수 있다.
- 자바와 달리 final 변수가 아니어도 접근 및 수정이 가능하다.

```kotlin
fun printProblemCounts(responses: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0

    responses.forEach {
        if (it.startsWith("4")) {
            clientErrors++
        } else if (it.startsWith("5")) {
            serverErrors++
        }
    }
    println("$clientErrors client errors, $serverErrors server errors")
}
```

- 위와 같이 람다 내부에서 사용하는 외부 변수를 포획한 변수(captured variable)라고 한다.
  - 람다의 모든 참조가 포함된 닫힌(closed) 객체 그래프를 클로저라고 한다.
  - 이 클로저는 람다 코드와 함께 저장된다.
- 일반적으로 로컬 변수는 정의된 함수와 생명주기를 함께 하지만 포획된 변수는 달라질 수 있다.


- 자바에서는 final 변수만 람다에서 사용할 수 있다.
- 하지만 해당 변수만 저장하는 배열이나 클래스를 선언하고 final로 선언하면 내부 값은 변경 가능하지만 람다에서도 사용할 수 있게 된다.
- 코틀린 코드도 내부적으로 위와 같은 방법으로 적용된다.

```kotlin
class Ref<T> (val value: T)
val counter = Ref(0)
val inc = { counter.value++ }
```


**멤버 참조**

- 이미 선언되어 있는 함수를 람다로 넘기고 싶을 때 멤버 참조를 통해 함수를 값으로 바꿀 수 있다.

```kotlin
val getAge = Person::age
// 위 코드는 아래와 같은 역할을 한다.
val getAge = { person: Person -> person.age }
```


- 멤버 참조 뒤에는 괄호를 붙이지 않는다.
- 멤버 참조는 해당 멤버를 호출하는 람다와 같은 타입이다.
  - 함수형 언어에서는 이런 경우를 에타 변환이라고 한다.

```kotlin
people.maxByOrNull(Person::age)
people.maxByOrNull { p -> p.age }
people.maxByOrNull { it.age }
```


- 최상위 함수나 프로퍼티도 참조할 수 있다.

```kotlin
fun salute() = println("Salute!")
run(::salute)
```


- 람다를 통해 인자가 여럿인 함수를 호출해야 하는 경우 참조만 제공해 더 편하게 사용할 수 있다.

```kotlin
val action = { person: Person, message: String ->
    sendEmail(person, message)
}

val action = ::sendEmail

val person = Person("Bob", 20)
val message = "hi"
run { action(person, message) }
run { simpleAction(person, message) }
```


- 생성자와 확장함수도 동일하게 참조해 사용할 수 있다.

```kotlin
// 생성자 참조
val createPerson = ::Person
createPerson("Alice", 20)

// 확장함수 참조
fun Person.isAdult() = age >= 20
val predicate = Person::isAdult
```


- 바운드 멤버 참조 (코틀린 1.1)
  - 코틀린 1.0에서는 참조한 멤버함수를 호출할 때 해당 클래스의 인스턴스를 제공해야 했다.
  - 코틀린 1.1부터는 멤버 참조를 생성할 때 인스턴스를 함께 저장하고 그에대한 멤버를 호출한다.
  - 따라서 호출 시 수신 대상 객체를 별도로 지정하지 않아도 된다. (아래 `val person` 에 해당)

```kotlin
val person = Person("Dmitry", 20)

// 코틀린 1.0
val getAge = Person::age
println(getAge(person)) // 수식 객체를 넘겨줘야 한다.

// or
val getAge = { person.age } // captured variable
println(getAge())

// 코틀린 1.1
val getAge = person::age
println(getAge())
```


**컬렉션 함수형 API**

컬렉션을 다루는 코틀린 표준 라이브러리 몇개를 살펴보자.

```kotlin
// filter
// predicate를 만족하는 원소로만 이루어진 새 컬렉션을 반환
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.filter { it.age > 30 })

// map
// 주어진 람다를 모든 원소에 적용한 결과를 모아 새 컬렉션으로 반환
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.map { it.name })

// 주의: 필요하지 않은 계산을 반복하지 말자.
// 람다는 신경써서 작업하지 않으면 굉장히 비효율적인 식이 될 수 있다.
people.filter { it.age == people.maxBy(Person::age)!!.age } // maxBy를 매번 계산한다.

val maxAge = people.maxBy(Person::age)!!.age
people.filter { it.age == maxAge }

// mapKeys, mapValues
val numbers = mapOf(0 to "zero", 1 to "one")
println(numbers.mapValues { it.value.toUpperCase() })
```

- `all`: 모든 원소가 predicate를 만족하는지 여부
- `any`: 하나의 원소라도 predicate를 만족하는지 여부
- `count`: 조건을 만족하는 원소의 수
- `find`, `firstOrNull`: 조건을 만족하는 첫번째 원소

- `all`과 `any`는 상호 배타적이기 때문에 !(not)을 쓰지 않는 것이 가독성에 좋다.
- `count`와 `size`를 적재적소에 사용하자.
  - `count` 는 조건을 만족하는 원소에 대한 중간 컬렉션을 생성하지 않아 효율적이다.

[예제](https://try.kotlinlang.org/#/Kotlin%20in%20Action/Chapter%205/5.2/2_1_AllAnyCountAndFindApplyingAPredicateToACollection.kt)


- `groupBy`는 어떤 특성에 따라 여러 그룹으로 나누고 싶을 때 사용한다.
  - 어떤 특성이 키, 여러 그룹이 값이 맵을 반환한다.

```kotlin
val people = listOf(
    Person("Alice", 31),
    Person("Bob", 29),
    Person("Carol", 31)
)
println(people.groupBy { it.age })
>>> {31=[Person(name=Alice, age=31), Person(name=Carol, age=31)], 29=[Person(name=Bob, age=29)]}
```


- `flatten`: 주어진 컬렉션을 하나의 리스트로 모아서 반환
- `flatMap`: 주어진 람다를 모든 원소에 map 하고 결과를 flatten 한 리스트를 반환

```kotlin
val books = listOf(
    Book("Thursday Next", listOf("Jasper Fforde")),
    Book("Mort", listOf("Terry Pratchett")),
    Book("Good Omens", listOf("Terry Pratchett","Neil Gaiman"))
)
println(books.flatMap { it.authors }.toSet())

val deepList = listOf(listOf(1),listOf(2),listOf(3))
println(deepList.flatten())
```


**시퀸스와 지연 계산**

앞에서 배운 map이나 filter 같은 함수는 결과 컬렉션을 즉시(eagerly) 생성하기 때문에 많은 원소에 대해 함수를 체이닝 할 경우 중간 결과 컬렉션이 많은 리소스를 사용하게 된다.
시퀀스를 사용하면 중간 임시 컬렉션을 생성하지 않고 위와 같은 처리를 할 수 있다.

- 한번에 하나씩 열거될 수 있는 원소를 표현하는 Sequence 인터페이스를 기반으로 한다.
- Sequence 안에는 iterator라는 단 하나의 메소드가 존재한다.
- 중간 연산은 항상 지연 계산되고 최종 연산 시 연산이 실행된다.

```kotlin
listOf(1, 2, 3, 4).asSequence()
    .map { print("map($it) "); it * it }
    .filter { print("filter($it) "); it % 2 == 0 }
    .toList()   // 최종 연산이 없으면 위의 map과 filter도 실행되지 않는다.
```


- 시퀸스의 결과는 시퀸스이다.
  - 인덱스를 통한 랜덤 엑세스 등 다른 API 메소드가 필요하면 `toList()`와 같이 컬렉션으로 변환한다.
- 원소가 많을 때는 중간 임시 컬렉션에 대한 오버헤드가 커져 시퀸스를 사용하는 것이 유리하다.


- 연산 수행 순서
  - 모든 원소에 대해 각각의 연산을 적용하는 컬렉션과 달리 시퀸스는 각각의 원소에 대해 모든 연산을 적용하는 순서로 이루어진다.
  - 따라서 연산을 적용하다 결과가 나오게되면 이후의 원소는 처리하지 않는다.

```kotlin
listOf(1, 2, 3, 4).asSequence()
    .map { it * it }.find { it > 3 }
```

![image](https://user-images.githubusercontent.com/10545416/113511515-1cc3de00-959b-11eb-83c1-b94076bb41a6.png)


- 연산 순서
  - map이나 filter와 같은 연산의 순서에 따라 결과는 같지만 변환의 전체 횟수는 달라진다.

```kotlin
val people = listOf(
    Person("A", 10),
    Person("B", 15),
    Person("C", 20),
    Person("D", 25)
)
println(people.asSequence().map(Person::age).filter { it >= 20 }.toList())
println(people.asSequence().filter { it.age >= 20 }.map(Person::age).toList())
```

![image](https://user-images.githubusercontent.com/10545416/113511733-5ba66380-959c-11eb-90c5-35532900e93e.png)


- 자바의 스트림과 코틀린의 시퀸스는 같은 개념이다.
- 코틀린은 자바 6에 맞춰져 있어 시퀸스를 별도로 제공하고 있다.
- 자바의 스트림은 CPU 병렬 실행 등의 기능을 제공하기 때문에 자바 버전에 따라 적절한 방식을 선택한다.


- `generateSequence` 함수로 시퀸스를 생성할 수 있다.

```kotlin
val naturalNumbers = generateSequence(0) { it + 1 }
val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
println(numbersTo100.sum())
```

```kotlin
fun File.isInsideHiddenDirectory() =
        generateSequence(this) { it.parentFile }.any { it.isHidden }

fun main(args: Array<String>) {
    val file = File("/Users/svtk/.HiddenDir/a.txt")
    println(file.isInsideHiddenDirectory())
}
```


**자바 함수형 인터페이스 활용**

- 코틀린 람다는 자바 API에도 아무 문제없이 사용할 수 있다.
- 추상 메소드가 단 하나 있는 인터페이스를 함수형 인터페이스 또는 SAM(single abstract method) 인터페이스라고 한다.
- 함수형 인터페이스를 인자로 받는 자바 메소드에 코틀린 람다를 넘길 수 있다.
  - 코틀린에서 함수를 인자로 받을때는 함수형 인터페이스가 아닌 코틀린의 함수 타입을 인자 타입으로 사용해야 한다.

```kotlin
void postponeComputation(int delay, Runnable computation);

postponeComputation(1000) { println("delayed more than 1s.") }
```


- 코틀린 컴파일러가 자동으로 Runnable을 구현한 익명 클래스와 인스턴스를 만들어준다.
- 메소드를 호출할 때마다 인스턴스가 생성되는 익명 클래스와 달리 코틀린의 람다는 변수를 포획하지 않는다는 가정 하에 기존의 익명 객체를 재사용한다.
  - 이를 명시적으로 표현하면 아래와 같다.

```kotlin
val runnable = Runnable { println("delayed more than 1s.") } // SAM 생성자 (아래 설명한다.)
fun handleComputation () {
    postponeComputation(1000, runnable)
}
```

- 만약 람다가 주변 변수를 포획하면 위와 같이 매번 호출 시 같은 인스턴스를 사용할 수 없어 새로운 인스턴스를 생성한다.


- 지금까지 자바 메소드에서 코틀린을 호출하는 경우를 살펴봤는데 컬렉션을 확장한 메소드에 람다를 넘기는 경우는 다르다.
- 대부분의 코틀린 확장함수는 inline 키워드가 붙어있고 해당 함수에 람다를 넘기면 익명 클래스도 생성되지 않는다.
- 이 내용은 8장에서 배운다.


**SAM 생성자**

SAM 생성자는 컴파일러가 람다를 함수형 인터페이스의 인스턴스로 변환할 수 있게 생성해주는 함수이다. 함수형 인터페이스의 인스턴스를 반환하는 등 컴파일러가 자동으로 변환하지 못하는 경우 명시적으로 SAM 생성자를 사용할 수 있다.

```kotlin
fun createAllDoneRunnable: Runnable {
    return Runnable { println("All Done.") }
}
createAllDoneRunnable().run()
>>> All Done.
```

- SAM 생성자의 이름은 함수형 인터페이스의 이름과 같다.
- 인스턴스를 변수에 저장해야 하는 경우에도 SAM 생성자를 활용할 수 있다.

```kotlin
val listener = OnClickListener { view -> 
    val text = when (view.id) {
        R.id.button1 -> "First"
        R.id.button2 -> "Second"
        else -> "Unknown"
    }
    toast(text)
}
```

- 익명 객체와 달리 람다에는 인스턴스 자신을 가리키는 `this`가 없다.
- 람다 안에서의 this는 람다를 둘러싼 클래스의 인스턴스를 가리킨다.


**with & apply**

- 수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있게 해주는 것을 수신 객체 지정 람다라고 한다.

[예제](https://try.kotlinlang.org/#/Kotlin%20in%20Action/Chapter%205/5.5/1_1_Alphabet.kt)


- with문은 특별한 구문처럼 보이지만 파라미터가 2개인 함수이다.
  - `with(stringBuilder, {...})`
  - 첫번째 인자로 받은 객체가 두번째 인자로 받은 람다의 수신 객체가 된다.
  - 수신 객체의 프로퍼티와 함수에 this를 생략할 수 있지만 사용하고 싶은 외부 메소드와 이름이 충돌하면 아래와 같이 사용할 수 있다.
    - `this@외부클래스이름.외부메소드()`


- apply는 with와 유사하지만 항상 수신 객체를 반환한다.
- apply는 확장 함수로 정의되어 있다.
- 자바의 Builder 패턴과 같이 인스턴스를 만들며 즉시 프로퍼티를 초기화 하는 경우 유용하다

```kotlin
val person = Person().apply {
    name = "Bob"
    age = 20
}
```

# Part 6. 코틀린 타입 시스템

**nullable 타입**

- 코틀린의 타입은 기본적으로 null이 될 수 없다.
- 예를 들면 다음과 같은 코드는 컴파일 에러가 발생한다

```kotlin
fun strLen(string: String) = string.length
strLen(null)
```

- null을 입력받고 싶은 경우 타입 뒤에 `?`를 붙여줘야 한다.

```kotlin
strLen(string: String?) = string.length
```

- nullable 변수로 메소드를 호출할 수 없다.
- nullable 값을 non-nullable 변수에 대입할 수 없다.
- nullable 값을 non-nullable 함수 파라미터에 전달할 수 없다.

```kotlin
val a: String?
a.length // x
val b = a // x
```

- null 검사 후에는 컴파일러가 non-nullable 타입처럼 취급한다.
- 하지만 코틀린은 nullable을 더 편리하게 처리할 수 있는 도구들을 제공한다.

```kotlin
fun strLenSafe(s: String?): Int = if (s != null) s.length else 0
```

- nullable 타입은 `?.`를 통해 안전하고 편리하게 처리할 수 있다.
- `?.` 앞의 타입이 null이 아닐 경우에만 뒤의 operation이 실행된다.
- 이렇게 반환된 값은 nullable(`Type?`)이다.
- `foo?.bar()`
  - `foo != null` -> `foo.bar()`
  - `foo == null` -> `null`

```kotlin
if (s != null) s.length else null
s?.length
```

- 연쇄해서 사용하면 편한 경우도 있다.

```kotlin
class Address(val streetAddress: String, val zipCode: Int,
              val city: String, val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun Person.countryName(): String {
   val country = this.company?.address?.country
   return if (country != null) country else "Unknown" // 이부분은 아래에서 개선해보자.
}
```

**앨비스 연산자**

- null 대신 default 값을 사용하고 싶을 때 유용하다.
- `foo ?: bar`
  - `foo != null` -> `foo`
  - `foo == null` -> `bar`

```kotlin
...

fun Person.countryName(): String {
   val address = this.company?.address ?: throw Exception("No address")
   with (address) {
       println(streetAddress)
       println("$zipCode $city $country")
   }
}
```

**안전한 캐스트 as?**

- ClassCastException을 피하려면 `is`로 타입이 변환 가능한지 확인 후 `as`를 사용해야 한다.
- `as?` 를 통해 더 쉽게 처리할 수 있다.
- 엘비스 연산자와 함께 사용하면 더욱 편하게 처리할 수 있다.
- `foo as? Type`
  - `foo is Type` -> `foo as Type`
  - `foo !is Type` -> `null`

```kotlin
class Person(val firstName: String, val lastName: String) {
   override fun equals(o: Any?): Boolean {
      val otherPerson = o as? Person ?: return false

      return otherPerson.firstName == firstName &&
             otherPerson.lastName == lastName
   }

   override fun hashCode(): Int =
      firstName.hashCode() * 37 + lastName.hashCode()
}
```

**Non-Null Assertion**

- `!!`를 붙여서 nullable 타입을 명시적으로 non-nullable 바꿀 수 있다.
- `foo!!`
  - `foo != null` -> foo
  - `foo == null` -> NPE
- NPE가 발생할 경우 null이 존재하는 라인이 아닌 해당 assertion을 선언한 라인을 가리킨다.

```kotlin
fun ignoreNulls(s: String?) {
    val sNotNull: String = s!!  // NPE는 여기서 발생한다.
    println(sNotNull.length)    // 여기가 아니라
}
```

- non-null이 보장되는 경우 assertion을 통해 코드를 간소화 할 수 있다.

```kotlin
data class Person(var name: String?, var age: Int = 10)

class Processor(var person: Person) {
    fun isEnabled(): Boolean = person.name != null

    fun processName() { // isEnabled()이 true일 경우에만 실행된다고 가정
        val value = person.name!!
        println(value.length)   // assertion이 없으면 null 체크 없이 호출할 수 없다.
    }
}
```

```kotlin
// NPE가 발생할 경우 line 정보는 제공하지만 expression 정보는 제공하지 않기 때문에
// 한줄에 여러 assertion을 쓰는 것은 자제해야 한다.
person.company!!.address!!.country
```

**let 함수**

- nullable expression을 더욱 쉽게 처리할 수 있다.
- `foo?.let { ...it }`
  - `foo != null` -> 람다 실행
  - `foo == null` -> 아무일도 일어나지 않는다.

```kotlin
fun sendEmail(email: String) = println(email)

var email: String? = "kotlin@mail.com"
email = null
// sendEmail(email)
email?.let(::sendEmail) // email이 null이면 아무일도 수행하지 않는다.
```

**프로퍼티 lazy init**

- non-null인 프로퍼티를 생성자가 아닌 메소드에서 초기화 할 수 없다.
- 이를 가능하게 하려면 lateinit 키워드를 붙여줘야 한다.
- val 프로퍼티는 생성자 안에서 반드시 초기화해야 하는 final 필드이기 때문에 var 프로퍼티에만 lateinit을 적용할 수 있다.

```kotlin
class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    private var myService: MyService? = null

    @Before fun setUp() {
        myService = MyService()
    }

    @Test fun testAction() {
        Assert.assertEquals("foo",
            myService!!.performAction()) // nullable이기 때문에 assertion을 사용한다.
    }
}
```

```kotlin
class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    private lateinit var myService: MyService // lateinit 덕분에 non-null 타입을 나중에 초기화 할 수 있다.

    @Before fun setUp() {
        myService = MyService()
    }

    @Test fun testAction() {
        Assert.assertEquals("foo",
            myService.performAction()) // non-null이기 때문에 null 체크도 필요없다.
    }
}
```

**nullable 타입의 확장**

- nullable 타입에 대한 확장함수를 통해 null 처리를 더욱 쉽게 할 수 있다.
- 이는 확장함수에서만 가능하고 일반 멤버 호출은 객체 인스턴스를 통해 디스패치 되기 때문에 null 체크를 하지 않는다.
- 일반적으로는 non-nullable한 함수를 작성하고 필요한 경우 nullable에 대한 확장을 추가하는 것이 좋다.

```kotlin
fun String?.isNullOrBlank(): Boolean = 
    this == null || this.isBlank() // 두번째 this는 스마트 캐스트가 적용된다.

fun verifyUserInput(input: String?) {
    if (input.isNullOrBlank()) { // null에 대해 호출해도 예외가 발생하지 않는다.
        println("Please fill in the required fields")
    }
}

fun main(args: Array<String>) {
    verifyUserInput(" ")
    verifyUserInput(null)
}
```

**타입 파라미터의 nullable**

- 함수나 클래스의 모든 타입 파라미터는 기본적으로 nullable하다.

```kotlin
// T의 타입은 Any?로 추론된다.
fun <T> printHash(t: T) {
    println(t?.hashCode()) // t는 nullable이다.
}
```

- non-nullable로 지정하고 싶은 경우 타입 상한(upper bound)를 지정해야 한다.
- 이에대한 자세한 내용은 9장 제네릭스에서 배운다.

```kotlin
fun <T:Any> printHash(t: T) {
    println(t?.hashCode()) // t는 non-nullable이다.
}
```

**nullable과 자바**

- Nullable에 대한 애노테이션이 있는 경우 아래와 같이 처리된다.
  - `@Nullable String` -> `String?`
  - `@NotNull String` -> `String`
- Nullable 애노테이션이 없는 경우는 플랫폼 타입으로 처리된다.
  - 플랫폼 타입은 nullable, non-nullable 모두 처리가 가능하다.
  - 모든 책임은 개발자에게 있다.
  - public 함수의 경우 코틀린 컴파일러가 non-nullable 파라미터와 수신 객체에 대한 null 검사를 추가하기 때문에 자바 클래스를 사용할 때는 null 체크 후 사용하는 것이 좋다.
  - 플랫폼 타입에 대한 표현은 `Type!`으로 처리된다.
  - 자바의 인터페이스나 클래스 메소드를 오버라이드 할 때 nullable을 선택할 수 있다.

**코틀린 원시 타입**

- 코틀린은 원시 타입과 래퍼 타입을 구분하지 않는다.
- 따라서 원시 타입 값에 대해 메소드를 호출할 수 있다.

```kotlin
val progress: Int = 120
val percent = progress.coerceIn(0, 100)
```

- 원시 타입을 nullable로 선언할 수도 있다.
  - `Int?`, `Boolean?` 등
  - 자바에서는 래퍼 타입으로 처리된다.
- null 체크를 한 뒤에 일반적인 값처럼 사용이 가능하다.

```kotlin
data class Person(val name: String,
                  val age: Int? = null) {

    fun isOlderThan(other: Person): Boolean? {
        if (age == null || other.age == null)
            return null
        return age > other.age
    }
}
```

**숫자 변환**

- 코틀린은 숫자를 자동 변환하지 않는다.
- 대신 Boolean을 제외한 원시 타입에 대해 변환 함수를 제공한다.
  - `toByte()`, `toShort()`, `toChar()` 등
- 원시 타입에 대한 리터럴은 다음과 같다.
  - Long: 100L
  - Double: 0.1, 1.0e10, 1.5e-20
  - Float: 3.14f
  - Hex: 0xCAFEBENE
  - Bin: 0b00010001
  - 코틀린 1.1 부터 숫자에 밑줄(`_`)을 추가할 수 있다. (1_000)

- 타입이 정해진 변수나 파라미터로 전달할때는 자동으로 변환이 된다.

```kotlin
fun foo(l: Long) = println(l)
val b: Byte = 1
val l = b + 1L
foo(100)
```

- ?? 코틀린은 overflow 검사에 대해 추가 비용이 들지 않는다.

**Any**

- 코틀린의 최상위 타입으로 자바의 Object와 유사하다.
  - 자바의 Object는 코틀린에서 `Any!`(플랫폼 타입)으로 처리된다.

**Unit**

- 의미없는 반환을 의미하며 자바의 void와 유사하다.
- 반환 타입을 선언하지 않으면 Unit 타입을 반환한다.
- 자바의 void와 달리 Unit을 타입 인자로 쓸 수 있어 반환이 필요없는 제네릭 파라미터를 처리할 때 편하다.

```kotlin
interface Processor<T> {
    fun process(): T
}

class NoResultProcessor: Processor<Unit> {
    override fun process() {
        ... without return expression // return 문을 명시하지 않으면 컴파일러가 return Unit을 넣어준다.
    }
}
```

**Nothing**

- 반환 값이라는 개념이 전혀 의미가 없을 때 사용된다.
- 함수의 반환 타입이나 반환 타입으로 쓰일 타입 파라미터로만 쓸 수 있다.

```kotlin
fun fail(message: String): Nothing {
    throw IllegalStateException(message)
}

val address = company.address ?: fail("No address")
println(address.city) // 컴파일러가 fail의 반환타입을 보고 address가 non-nullable임을 알게 된다.
```

**nullable과 컬렉션**

- 컬렉션의 타입 인자도 `?`를 통해 nullable을 결정한다.

```kotlin
fun readNumbers(reader: BufferedReader): List<Int?> {
    val result = ArrayList<Int?>()
    for (line in reader.lineSequence()) {
        try {
            val number = line.toInt()
            result.add(number)
        }
        catch(e: NumberFormatException) {
            result.add(null)
        }
    }
    return result
}
```

- `?`의 위치에 따라 nullable의 범위가 바뀐다.
  - `List<Int?>`, `List<Int>?`

![image](https://user-images.githubusercontent.com/10545416/114746861-27e0f000-9d8b-11eb-9bd0-3665045d0b4f.png)

**읽기 전용 컬렉션과 변경 가능한 컬렉션**

- 코틀린 컬렉션은 읽기와 쓰기에 대한 인터페이스를 분리했다.
  - `kotlin.collections.Collection`에는 읽기 연산만 존재한다.
  - 쓰기 연산은 `kotlin.collections.MutableCollection`을 사용한다.

![image](https://user-images.githubusercontent.com/10545416/114747615-ff0d2a80-9d8b-11eb-9bca-4ab013892851.png)

- 가능하면 항상 읽기 전용 컬렉션을 사용해라.
  - 읽기 전용 컬렉션은 변경 가능 컬렉션일 수 있다는 점을 명심해라. (반드시 thread-safe 하지 않다.)

![image](https://user-images.githubusercontent.com/10545416/114747727-1d732600-9d8c-11eb-9e90-30f7a20117ee.png)

- 자바의 컬렉션에 대해 읽기 전용, 변경 가능이라는 표현을 제공하고 자바의 컬렉션이 변경 가능 인터페이스를 상속하는 형태로 제공한다.

![image](https://user-images.githubusercontent.com/10545416/114748214-a5593000-9d8c-11eb-999c-c20b69fd0cd8.png)

- 컬렉션 생성 함수에 따른 분류

![image](https://user-images.githubusercontent.com/10545416/114748261-b1dd8880-9d8c-11eb-9df2-626a361f2978.png)

- setOf()와 mapOf()는 자바 표준 라이브러리의 인스턴스를 반환하기 때문에 내부적으로는 변경 가능한 클래스이지만 미래에는 불변 인스턴스로 변경될 수 있다.

- 자바에서는 읽기 전용 컬렉션을 지원하지 않기 때문에 코틀린에서 읽기 전용 컬렉션을 넘기면 변경이 가능하다.
- 이에 대한 처리는 개발자가 신경써야 한다.
- 위 문제는 널이 아닌 원소로 이루어진 컬렉션에도 적용된다.

**컬렉션과 플랫폼 타입**

- 자바에서 선언한 컬렉션 변수는 코틀린에서 플랫폼 타입으로 본다.
  - 해당 컬렉션에 대한 읽기 전용, 변경 가능 여부를 선택할 수 있다.
- 일반적인 경우 문제가 될 일이 별로 없다.
- 컬렉션 타입을 시그니처로 가지는 자바 메소드를 오버라이드 하는 경우 신경쓸 부분이 생긴다.
  - 컬렉션의 nullable 여부
  - 컬렉션의 원소의 nullable 여부
  - 컬렉션의 mutable 여부
  
p294-p296 참고

**객체의 배열과 원시 타입의 배열**

```kotlin
fun main(args: Array<String>) {
    for (i in args.indices) { // 인덱스 값을 알기위해 array.indices 확장 함수 사용
         println("Argument $i is: ${args[i]}") // array[index]로 배열 원소 접근
    }
}
```

- 배열을 만드는 다양한 방법
  - arrayOf(1,2,3)
  - arrayOfNulls<Int>(3)
  - Array<Int>(10) { i -> i + 1 }
    - 람다는 배열의 인덱스를 받아서 해당 위치에 들어갈 원소를 반환한다.

```kotlin
val letters = Array<String>(26) { i -> ('a' + i).toString() }
```

- 컬렉션을 배열로 만들때는 `toTypedArray`를 사용하면 편하다.

```kotlin
val strings = listOf("a", "b", "c")
println("%s/%s/%s".format(*strings.toTypedArray())) // *: 스프레드 연산자
```

- 배열 타입의 타입 인자는 항상 객체 타입이 된다.
  - `Array<Int>` -> `Integer[]`
- 원시 타입의 배열을 표현하려면 각각의 타입에 맞는 배열 클래스를 사용한다.
  - `IntArray` -> `int[]`
  - `ByteArray` -> `byte[]`
  - `CharArray` -> `char[]`
  - 등

```kotlin
IntArray(5) // [0,0,0,0,0]
intArrayOf(0,1,2) // [0,1,2]
IntArray(3) { i -> (i*2) } // [0,2,4]
```

- 컬렉션에서 사용한 확장 함수, map, filter 등을 Array에도 사용할 수 있다.

```kotlin
fun main(args: Array<String>) {
    args.forEachIndexed { index, element ->
        println("Argument $index is: $element")
    }
}
```

# Part 7. 연산자 오버로딩과 기타 관례

- 기존 언어에서 제공하는 문법과 미리 정의한 함수를 연결해 주는 기법을 코틀린에서는 관례(convention) 이라고 한다.
  - 클래스에 `plus`라는 메소드를 정의하면 인스턴스에 대해 `+` 연산을 사용할 수 있다.
- 위와같은 기능을 타입에 의존하는 자바와 달리 코틀린에서 컨벤션을 사용하는 이유는 기존의 자바 클래스를 코틀린 언어에 적용하기 위해서이다.
  - 기존 자바 클래스가 새로운 인터페이스를 구현하도록 할 수 없기 때문에 확장함수를 구현하면서 컨벤션에 따라 이름을 붙이면 쉽게 처리가 가능하다.

**산술 연산자**

- `BigInteger`에서 `add` 대신 `+`를 사용하거나 컬렉션에 원소를 추가할 때 `+=`를 사용할 수 있다면 편할것이다.
- `plus`라는 특별한 이름의 함수를 정의하고 `+` 연산자로 이 함수를 호출한다.
- 연산자를 오버로딩할때는 함수앞에 `operation` 키워드가 있어야 한다.

```kotlin
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}

fun main(args: Array<String>) {
    val p1 = Point(10, 20)
    val p2 = Point(30, 40)
    println(p1 + p2)
}
```

- 멤버 함수 대신 확장 함수로 정의할 수도 있다.
- 확장 함수로 정의하는 경우가 더 일반적인 패턴이다.

```kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}
```

- 오버로딩을 하더라도 연산 우선순위는 기존과 같다.
- 오버로딩이 가능한 이항 산술 연산자는 다음과 같다.
  - a * b : times
  - a / b : div
  - a % b : mod(1.1부터 rem)
    - rem : remainder
  - a + b : plus
  - a - b : minus

- 피연산자가 같은 타입일 필요는 없다.
- 다만 교환 법칙은 성립하지 않는다.
  - `a op b` != `b op a`, `b op a`에 대한 함수를 추가해야 가능하다.

```kotlin
operator fun Point.times(scale: Double): Point {
    return Point((x * scale).toInt(), (y * scale).toInt())
}

fun main(args: Array<String>) {
    val p = Point(10, 20)
    println(p * 1.5)
}
```

- 일반 함수와 동일하게 오버로딩이 가능하다.
- 비트 연산은 코틀린에서 제공하지 않아 오버로딩도 불가능하다.
  - 대신 아래의 일반 함수를 제공하기 때문에 필요한 경우 이를 활용하면 된다.
    - and: 비트 곱
    - or: 비트 합
    - xor: 비트 배타 합
    - 등등

**복합 대입 연산자**

- `+=`와 같이 여러 산술연산을 합친 연산자에 대해 설명한다.

```kotlin
operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}

fun main(args: Array<String>) {
    var point = Point(1, 2)
    point += Point(3, 4) // point는 새로운 인스턴스가 할당된다.
    println(point)
}
```

- 변수의 참조를 변경하지 않고 값을 수정하고 싶은 경우는 Unit 타입을 반환하는 `plusAssign`과 같은 함수를 정의하면 된다.

```kotlin
operation fun <T> MutableCollection<T>.plusAssign(element: T) {
    this.add(element)
}

val numbers = ArrayList<Int>()
numbers += 42
```

- 문법상 `plus`와 `plusAssign`을 모두 정의하고 둘 다 `+=`에 사용 가능한 경우 컴파일 에러가 발생한다.
- 클래스의 변경 가능성을 판단해 적절한 함수만 정의하는 것이 좋다.
- 코틀린 표준 라이브러리는 컬렉션에 대해 아래와 같은 접근법을 제공한다.
  - `+`, `-`: 새로운 컬렉션 반환
  - `+=`, `-=`:
    - mutable 컬렉션: 현재 컬렉션 객체의 상태를 수정
    - read only 컬렉션: 변경을 적용한 복사본 반환

**단항 연산자**

- 단항 연산자도 이항 연산자와 크게 다르지 않다.
- 단항 연산자 오버로딩 함수는 인자를 받지 않는다.
- 오버로딩이 가능한 단항 산술 연산자는 다음과 같다.
  - +a : unaryPlus
  - -a : unaryMinus
  - !a : not
  - ++a, a++ : inc
  - --a, a-- : dec
  - unary: "유너리" 라고 읽네요

```kotlin
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}

fun main(args: Array<String>) {
    val p = Point(10, 20)
    println(-p)
}
```

```kotlin
operator fun BigDecimal.inc() = this + BigDecimal.ONE

fun main(args: Array<String>) {
    var bd = BigDecimal.ZERO
    println(bd++) // 0
    println(++bd) // 2
}
```

**비교 연산자 오버로딩: equals**

- == : equals
  - 내부에서 자동으로 null 검사를 하기 때문에 nullable 타입에도 사용할 수 있다.
  - `a == b` -> `a?.equals(b) ?: (b == null)`
  - `equals`는 `Any`에 정의된 메소드이므로 override가 필요하다.
  - `Any`의 `equals`에 `operator` 키워드가 이미 명시되어 있기 때문에 오버라이드하는 메소드에는 추가하지 않아도 된다.
  - `Any`의 `equals`가 확장 함수보다 우선순위가 높아 `equals`는 확장 함수로 정의할 수 없다.
- === : 두 객체가 같은 인스턴스인지 확인한다.
  - `equals` 구현 시 자기 자신과의 비교를 최적화 할때 사용되는 경우가 많다.
  - 이 연산자는 오버로딩 할 수 없다.

```kotlin
class Point(val x: Int, val y: Int) {
    override fun equals(obj: Any?): Boolean {
        if (obj === this) return true
        if (obj !is Point) return false
        return obj.x == x && obj.y == y
    }
}
```

**순서 연산자: compareTo**

- Comparable 인터페이스를 구현하면 순서 연산자를 오버라이딩 할 수 있다.
- `a >= b` -> `a.compareTo(b) >= 0`

```kotlin
class Person(val firstName: String, val lastName: String) : Comparable<Person> {
    override fun compareTo(other: Person): Int {
        return compareValuesBy(this, other, 
            Person::lastName, Person::firstName)
    }
}
```

**인덱스 연산자**

- 배열 원소 뿐만 아니라 맵에 접근할 때도 대괄호를 사용할 수 있는 이유는 인덱스 연산자 덕분이다.
  - `val value = map[key]`, `mutableMap[key] = newValue`
- `get`, `set` 메소드로 오버라이드 할 수 있다.

```kotlin
operator fun Point.get(index: Int): Int {
    return when(index) {
        0 -> x
        1 -> y
        else ->
            throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

fun main(args: Array<String>) {
    val p = Point(10, 20)
    println(p[1])
}
```

**in**

- `a in b` -> `b.contains(a)`

```kotlin
data class Point(val x: Int, val y: Int)

data class Rectangle(val upperLeft: Point, val lowerRight: Point)

operator fun Rectangle.contains(p: Point): Boolean {
    return p.x in upperLeft.x until lowerRight.x &&
           p.y in upperLeft.y until lowerRight.y
}

fun main(args: Array<String>) {
    val rect = Rectangle(Point(10, 20), Point(50, 50))
    println(Point(20, 30) in rect)
    println(Point(5, 5) in rect)
}
```

**rangeTo**

- `start..end` -> `start.rangeTo(end)`
- `Comparable` 인터페이스를 구현하면 아래와 같은 `rangeTo` 함수를 사용할 수 있다.
  - `operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>`

**iterator**

- `for 루프` 내의 `in`은 앞에서 다룬 `in`과는 의미가 다르다.
- `for (x in list) {...}` -> `list.iterator()` -> `hasNext(), next()`

```kotlin
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
    // LocalDate에 대한 Iterator를 구현한다
    object : Iterator<LocalDate> {
        var current = start

        // compareTo 관례를 활용한다
        override fun hasNext() =
            current <= endInclusive

        override fun next() = current.apply {
            current = plusDays(1)
        }
    }

fun main(args: Array<String>) {
    val newYear = LocalDate.ofYearDay(2017, 1)
    val daysOff = newYear.minusDays(1)..newYear
    for (dayOff in daysOff) { println(dayOff) }
}
```

**구조 분해 선언과 component()**

- 구조 분해 선언을 위해 `componentN()`이라는 함수를 호출한다.
- `val (a, b) = p` -> `val a = p.component1() \ val b = p.component2()`
- data 클래스의 주 생성자 프로퍼티는 컴파일러가 자동으로 `componentN()` 함수를 만들어준다.
- 여러 값을 한번에 반환하고 싶을 경우 모든 값을 저장하는 데이터 클래스를 만들고 이를 반환하면 된다.

```kotlin
data class NameComponents(val name: String, val extension: String)

fun splitFilename(fullName: String): NameComponents {
    val (name, extension) = fullName.split('.', limit = 2)
    return NameComponents(name, extension)
}

fun main(args: Array<String>) {
    val (name, ext) = splitFilename("example.kt")
    println(name)
    println(ext)
}
```

- 코틀린 표준 라이브러리는 맨 앞의 다섯 원소에 대한 componentN을 제공한다.  
- 위 예제의 `NameComponents`처럼 별도 클래스를 생성하지 않고 `Pair`, `Triple`을 사용할 수도 있다.

**구조 분해 선언과 루프**

- 위에서 배운 관례를 활용해 아래 예제를 더 잘 이해할 수 있다.

```kotlin
fun printEntries(map: Map<String, String>) {
    // in -> map에 대한 iteration
    // (key, value) -> Map.Entry에 대한 구조 분해 선언
    for ((key, value) in map) {
        println("$key -> $value")
    }
}
```

**위임 프로퍼티**

- 필드에 값을 저장할 때 처리하는 로직을 직접 구현할 수 있다.
  - 값을 필드가 아닌 DB, 세션, 맵 등에 저장할 수 있다.
- 이러한 로직(위임 객체)은 직접 구현할 수도, 코틀린이 제공하는 것을 사용할 수도 있다.

```kotlin
class Foo {
    var p: Type by Delegate() // by 라는 키워드로 위임을 명시한다
}

class Delegate {
    operator fun getValue(...) {...}
    operator fun setValue(..., value: Type) {...}
}

// TO-BE
class Foo {
    private val delegate = Delegate()
    var p: Type
    set(value: Type) = delegate.setValue(..., value)
    get() = delegate.getValue(...)
}
```

**by lazy()를 통한 초기화 지연**

- 먼저 배운 내용을 바탕으로 초기화 지연을 구현해보자.

```kotlin
class Email { /*...*/ }
fun loadEmails(person: Person): List<Email> {
    println("Load emails for ${person.name}")
    return listOf(/*...*/)
}

class Person(val name: String) {
    private var _emails: List<Email>? = null

    val emails: List<Email>
       get() {
           if (_emails == null) {
               _emails = loadEmails(this)
           }
           return _emails!!
       }
}

fun main(args: Array<String>) {
    val p = Person("Alice")
    p.emails // print o
    p.emails // print x
}
```

- 지연 초기화가 필요한 필드가 많아지면 코드 수정이 귀찮아진다.
- thread-safe 하지 않다.
- 위임 프로퍼티로 이런 문제들을 해결할 수 있다.
  - 데이터 저장에 쓰이는 backing 프로퍼티와 오직 한번만 초기화되는 getter 로직을 캡슐화한다.
  - 인자로 넘긴 람다를 값을 초기화 할 때 사용한다.
  - 기본적으로 thread-safe하다

```kotlin
class Person(val name: String) {
    val emails by lazy { loadEmails(this) }
}
```

**위임 프로퍼티 구현**

[예제 참고](https://try.kotlinlang.org/#/Kotlin%20in%20Action/Chapter%207/7.5/3_1_ImplementingDelegatedProperties.kt)

**프로퍼티 값을 맵에 저장**

- `p.name` -> `_attributes.getValue(p, prop)` -> `_attributes[prop.name]`

```kotlin
class Person {
    private val _attributes = hashMapOf<String, String>()

    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }

    // as-is
    val name: String get() = _attributes["name"]!!
    // to-be
    val name: String by _attributes
}

fun main(args: Array<String>) {
    val p = Person()
    val data = mapOf("name" to "Dmitry", "company" to "JetBrains")
    for ((attrName, value) in data)
       p.setAttribute(attrName, value)
    println(p.name)
}
```

**프레임워크와 위임 프로퍼티**

```kotlin
// 테이블을 표현하는 Users는 object 키워드로 싱글턴 객체가 되었다
object Users: IdTable() {
    val name = varchar("name", length=50).index()
    val age = integer("age)
}

// 
class User(id: EntityId): Entity(id) {
    var name: String by Users.name
    var age: Int by Users.age
}
```

- `user.age += 1` -> `user.ageDelegate.setValue(user.ageDelegate.getValue() + 1)`
  - 대략적인 흐름