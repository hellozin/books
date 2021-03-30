- [Part 1. 코틀린 소개](#part-1-코틀린-소개)
- [Part 2. 코틀린 기초](#part-2-코틀린-기초)
- [Part 3. 함수 정의와 호출](#part-3-함수-정의와-호출)
- [Part 4. 클래스, 객체, 인터페이스](#part-4-클래스-객체-인터페이스)

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
