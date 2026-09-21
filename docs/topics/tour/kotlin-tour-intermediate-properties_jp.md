[//]: # (title: Properties)

<no-index/>


初心者向けツアーでは、プロパティを使ってクラスインスタンスの特性を宣言する方法や、それらにアクセスする方法を学びました。
この章では、Kotlinにおけるプロパティの仕組みについてさらに詳しく掘り下げ、コード内でプロパティを活用するその他の方法についても解説します。

## Backing fields
## バッキングフィールド


Kotlinでは、プロパティにはデフォルトの`get()`および`set()`関数（プロパティアクセサと呼ばれる）が用意されており、これらが値の取得や変更を処理します。
これらのデフォルト関数はコード上では明示的には表示されませんが、コンパイラが裏でプロパティへのアクセスを管理するために自動的に生成しています。
これらのアクセサは、**バッキングフィールド**を使用して、プロパティの実際の値を格納します。

以下のいずれかが当てはまる場合、バッキングフィールドが存在します：

* プロパティに対して、デフォルトの `get()` または `set()` 関数を使用します。
* コード内で `field` キーワードを使用して、プロパティの値にアクセスしようとします。

> `get()` および `set()` 関数は、ゲッターやセッターとも呼ばれます。
>
{style="tip"}

たとえば、次のコードには `category` プロパティがありますが、このプロパティにはカスタム `get()` 関数や `set()` 関数が定義されていないため、デフォルトの実装が使用されます：

```kotlin
class Contact(val id: Int, var email: String) {
    var category: String = ""
}
```

内部的には、これは次の擬似コードに相当します：

```kotlin
class Contact(val id: Int, var email: String) {
    var category: String = ""
        get() = field
        set(value) {
            field = value
        }
}
```
{validate="false"}

この例では：

* `get()` 関数は、`field`からプロパティの値`""`を取得します。
* `set()` 関数は `value` をパラメータとして受け取り、それを`field`に割り当てます。ここで、`value` は `""` です。

Access to the backing field is useful when you want to add extra logic in your `get()` or `set()` functions 
without causing an infinite loop. For example, you have a `Person` class with a `name` property:


```kotlin
class Person {
    var name: String = ""
}
```

`name` プロパティの最初の文字を大文字にする必要があるため、[`.replaceFirstChar()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/replace-first-char.html) および [`.uppercase()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/uppercase-char.html) 拡張関数を使用するカスタム `set()` 関数を作成します。
しかし、`set()` 関数内でそのプロパティを直接参照すると、無限ループが発生し、実行時に `StackOverflowError` が発生します：

```kotlin
class Person {
    var name: String = ""
        set(value) {
            // This causes a runtime error
            name = value.replaceFirstChar { firstChar -> firstChar.uppercase() }
        }
}

fun main() {
    val person = Person()
    person.name = "kodee"
    println(person.name)
    // Exception in thread "main" java.lang.StackOverflowError
}
```
{validate="false" kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-properties-stackoverflow"}

この問題を解決するには、`set()` 関数内で `field` キーワードを使ってバッキングフィールドを参照するようにします。

```kotlin
class Person {
    var name: String = ""
        set(value) {
            field = value.replaceFirstChar { firstChar -> firstChar.uppercase() }
        }
}

fun main() {
    val person = Person()
    person.name = "kodee"
    println(person.name)
    // Kodee
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-properties-backingfield"}

バッキングフィールドは、ロギングを追加したり、プロパティの値が変更された際に通知を送信したり、新旧のプロパティ値を比較する追加のロジックを使用したりする場合にも役立ちます。

詳細については、[バッキングフィールド](properties.md#backing-fields)を参照してください。

## Extension properties
## 拡張プロパティ

拡張関数と同様に、拡張プロパティも存在します。
拡張プロパティを使用すると、ソースコードを変更することなく、既存のクラスに新しいプロパティを追加することができます。
ただし、Kotlinの拡張プロパティには、**バックフィールド**は存在しません。
つまり、`get()` および `set()` 関数を自分で記述する必要があります。
さらに、バッキングフィールドがないため、状態を保持することができません。

拡張プロパティを宣言するには、拡張したいクラスの名前の後に `.` を付け、その後にプロパティ名を記述します。
通常のクラスプロパティと同様に、プロパティの型を宣言する必要があります。
例えば：

```kotlin
val String.lastChar: Char
```
{validate="false"}

拡張プロパティは、継承を使用せずにプロパティに計算された値を格納したい場合に最も役立ちます。
拡張プロパティは、引数が1つだけ（レシーバー）の関数のように動作すると考えてください。

たとえば、`Person` というデータクラスがあり、そこに `firstName` と `lastName` という 2 つのプロパティがあるとします。

```kotlin
data class Person(val firstName: String, val lastName: String)
```

`Person` データクラスを変更したり、そこから継承したりすることなく、その人物のフルネームにアクセスできるようにしたい。
これを行うには、カスタム `get()` 関数を持つ拡張プロパティを作成します：

```kotlin
data class Person(val firstName: String, val lastName: String)

// Extension property to get the full name
val Person.fullName: String
    get() = "$firstName $lastName"

fun main() {
    val person = Person(firstName = "John", lastName = "Doe")

    // Use the extension property
    println(person.fullName)
    // John Doe
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-properties-extension"}

> 拡張プロパティは、クラスの既存のプロパティを上書きすることはできません。
> 
{style="note"}

拡張関数と同様に、Kotlinの標準ライブラリでは拡張プロパティも広く利用されています。例えば、`CharSequence` の [`lastIndex` プロパティ](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/last-index.html) を参照してください。

## Delegated properties
## 委譲プロパティ

[クラスとインターフェース](kotlin-tour-intermediate-classes-interfaces.md#delegation)の章で、デリゲーションについてすでに学習しました。
また、プロパティの委譲を使用して、そのプロパティのアクセサを別のオブジェクトに委譲することもできます。
これは、データベースのテーブルやブラウザのセッション、マップなどに値を保存するなど、単純なバッキングフィールドでは対応できないような、より複雑なプロパティの保存要件がある場合に役立ちます。
委譲プロパティを使用すると、プロパティの取得や設定に関するロジックが、委譲先のオブジェクト内にのみ含まれるため、定型コードも削減されます。

構文はクラスでの委譲の使用と似ていますが、動作するレベルが異なります。
プロパティを宣言し、その後に `by` キーワードと、委譲先のオブジェクトを指定します。
例えば：

```kotlin
val displayName: String by Delegate
```

ここでは、委譲されたプロパティ `displayName` は、そのプロパティアクセサに関して `Delegate` オブジェクトを参照しています。

委譲先のすべてのオブジェクトには、**必ず** `getValue()` 演算子関数が用意されている必要があります。Kotlin はこの関数を使用して、委譲されたプロパティの値を取得します。
プロパティが可変である場合、Kotlinがその値を設定できるように、`setValue()` 演算子関数も用意する必要があります。

デフォルトでは、`getValue()` および `setValue()` 関数は次のような構成になっています：

```kotlin
operator fun getValue(thisRef: Any?, property: KProperty<*>): String {}

operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {}
```
{validate="false"}

In these functions:

* `operator` キーワードは、これらの関数を演算子関数として指定し、`get()` および `set()` 関数をオーバーロードできるようにします。
* `thisRef` パラメータは、委譲されたプロパティを**含む**オブジェクトを指します。
デフォルトでは、型は `Any?` に設定されていますが、より具体的な型を宣言する必要がある場合があります。
* `property` パラメータは、その値が参照または変更されるプロパティを指します。
このパラメータを使用すると、プロパティの名前や型などの情報にアクセスできます。デフォルトでは、型は `KProperty<*>` に設定されていますが、`Any?` を使用することもできます。
コード内でこれを変更することについては、心配する必要はありません。

`getValue()` 関数の戻り値の型はデフォルトで `String` ですが、必要に応じて変更することができます。

`setValue()` 関数には、プロパティに割り当てられる新しい値を格納するために使用される `value` という追加のパラメータがあります。

では、実際にはどのような感じになるのでしょうか？
たとえば、ユーザーの表示名のような計算プロパティを用意したいとします。その計算処理は負荷が高く、アプリケーションのパフォーマンスが重要なため、このプロパティは一度だけ計算されるようにしたいとします。
委譲プロパティを使用すると、表示名をキャッシュできるため、計算は一度だけ行われ、パフォーマンスに影響を与えることなくいつでもアクセスできるようになります。

まず、デリゲート先のオブジェクトを作成する必要があります。この場合、そのオブジェクトは `CachedStringDelegate` クラスのインスタンスになります：

```kotlin
class CachedStringDelegate {
    var cachedValue: String? = null
}
```

`cachedValue` プロパティには、キャッシュされた値が格納されています。
`CachedStringDelegate` クラス内で、委譲されたプロパティの `get()` 関数に期待する動作を、`getValue()` 演算子の関数本体に追加します。

```kotlin
class CachedStringDelegate {
    var cachedValue: String? = null

    operator fun getValue(thisRef: Any?, property: Any?): String {
        if (cachedValue == null) {
            cachedValue = "Default Value"
            println("Computed and cached: $cachedValue")
        } else {
            println("Accessed from cache: $cachedValue")
        }
        return cachedValue ?: "Unknown"
    }
}
```

`getValue()` 関数は、`cachedValue` プロパティが `null` かどうかを確認します。`null` の場合、この関数は `「Default value」` を代入し、ログ記録のために文字列を出力します。
`cachedValue` プロパティがすでに計算されている場合、そのプロパティは `null` ではありません。
この場合、ログ出力のために別の文字列が出力されます。最後に、この関数はエルビス演算子を使用して、キャッシュされた値を返すか、値が `null` の場合は `「Unknown」` を返します。

これで、キャッシュしたいプロパティ（`val displayName`）を `CachedStringDelegate` クラスのインスタンスに委譲できるようになりました：

```kotlin
class CachedStringDelegate {
    var cachedValue: String? = null

    operator fun getValue(thisRef: User, property: Any?): String {
        if (cachedValue == null) {
            cachedValue = "${thisRef.firstName} ${thisRef.lastName}"
            println("Computed and cached: $cachedValue")
        } else {
            println("Accessed from cache: $cachedValue")
        }
        return cachedValue ?: "Unknown"
    }
}

class User(val firstName: String, val lastName: String) {
    val displayName: String by CachedStringDelegate()
}

fun main() {
    val user = User("John", "Doe")

    // 最初のアクセス時に値を計算してキャッシュする
    println(user.displayName)
    // Computed and cached: John Doe
    // John Doe

    // それ以降のアクセスでは、キャッシュから値が読み込まれます
    println(user.displayName)
    // Accessed from cache: John Doe
    // John Doe
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-properties-delegated"}

この例では：

* ヘッダーに `firstName` と `lastName` の 2 つのプロパティを持ち、クラス本体に `displayName` という 1 つのプロパティを持つ `User` クラスを作成します。
* `displayName` プロパティを `CachedStringDelegate` クラスのインスタンスに委譲します。
* `User` クラスのインスタンスを `user` という名前で作成します。
* `user` インスタンスの `displayName` プロパティにアクセスした結果を出力します。

なお、`getValue()` 関数では、`thisRef` パラメータの型が `Any?` 型からオブジェクト型 `User` へと絞り込まれている点に注意してください。
これは、コンパイラが `User` クラスの `firstName` および `lastName` プロパティにアクセスできるようにするためです。

### Standard delegates

Kotlinの標準ライブラリには、便利なデリゲートがいくつか用意されているため、毎回一から作成する必要はありません。
これらのデリゲートのいずれかを使用する場合、標準ライブラリが自動的に `getValue()` および `setValue()` 関数を提供するため、これらを定義する必要はありません。

#### Lazy properties

プロパティが最初にアクセスされたときのみ初期化するには、遅延プロパティを使用します。
標準ライブラリでは、デリゲーションのために `Lazy` インターフェースが提供されています。

`Lazy` インターフェースのインスタンスを作成するには、`lazy()` 関数を使用し、`get()` 関数が初めて呼び出されたときに実行されるラムダ式を引数として渡します。
その後、`get()` 関数を再度呼び出しても、最初の呼び出し時に返されたのと同じ結果が返されます。
レイジープロパティでは、ラムダ式を渡すために [トレーリングラムダ](kotlin-tour-functions.md#trailing-lambdas) の構文を使用します。

For example:

```kotlin
class Database {
    fun connect() {
        println("Connecting to the database...")
    }

    fun query(sql: String): List<String> {
        return listOf("Data1", "Data2", "Data3")
    }
}

val databaseConnection: Database by lazy {
    val db = Database()
    db.connect()
    db
}

fun fetchData() {
    val data = databaseConnection.query("SELECT * FROM data")
    println("Data: $data")
}

fun main() {
    // データベース接続への初回アクセス
    fetchData()
    // Connecting to the database...
    // Data: [Data1, Data2, Data3]

    // 以降のアクセスでは、既存の接続が使用されます
    fetchData()
    // Data: [Data1, Data2, Data3]
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-properties-lazy"}

この例では：

* `connect()` および `query()` というメンバ関数を持つ `Database` クラスがあります。
* `connect()` 関数はコンソールに文字列を出力し、`query()` 関数は SQL クエリを受け取ってリストを返します。
* `databaseConnection` プロパティは、遅延読み込みプロパティです。
* `lazy()` 関数に渡されるラムダ式：
  * `Database` クラスのインスタンスを作成します。
  * このインスタンス（`db`）の `connect()` メンバ関数を呼び出します。
  * インスタンスを返します。
* `fetchData()` という関数があり、その機能は次のとおりです：
  * `databaseConnection` プロパティの `query()` 関数を呼び出して、SQL クエリを作成します。
  * SQLクエリを`data`変数に代入します。
  * `data` 変数の値をコンソールに出力します。
* `main()` 関数は `fetchData()` 関数を呼び出します。初めて呼び出された際、lazy プロパティが初期化されます。
2回目の呼び出しでは、1回目の呼び出しと同じ結果が返されます。

遅延プロパティは、初期化にリソースを大量に消費する場合だけでなく、コード内でそのプロパティが使用されない可能性がある場合にも役立ちます。
さらに、レイジープロパティはデフォルトでスレッドセーフであるため、並行処理環境での作業において特に有益です。

詳細については、[遅延プロパティ](delegated-properties.md#lazy-properties)を参照してください。

#### Observable properties
#### 観測可能なプロパティ（オブザーバルプロパティ）

プロパティの値が変更されたかどうかを監視するには、オブザーバブルプロパティを使用します。
オブザーバブルプロパティは、プロパティ値の変化を検出し、その情報を基に何らかの反応を引き起こしたい場合に役立ちます。
標準ライブラリには、委譲を行うための `Delegates` オブジェクトが用意されています。

オブザーバブルプロパティを作成するには、まず `kotlin.properties.Delegates.observable` をインポートする必要があります。
次に、`observable()` 関数を使用し、プロパティが変更されるたびに実行されるラムダ式を引数として渡します。
遅延プロパティと同様に、オブザーバブルプロパティも、ラムダ式を渡すために [トレーリングラムダ](kotlin-tour-functions.md#trailing-lambdas) の構文を使用します。

例えば：

```kotlin
import kotlin.properties.Delegates.observable

class Thermostat {
    var temperature: Double by observable(20.0) { _, old, new ->
        if (new > 25) {
            println("Warning: Temperature is too high! ($old°C -> $new°C)")
        } else {
            println("Temperature updated: $old°C -> $new°C")
        }
    }
}

fun main() {
    val thermostat = Thermostat()
    thermostat.temperature = 22.5
    // Temperature updated: 20.0°C -> 22.5°C

    thermostat.temperature = 27.0
    // Warning: Temperature is too high! (22.5°C -> 27.0°C)
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-properties-observable"}

この例では：

* `Thermostat` クラスには、オブザーバブルプロパティ `temperature` が含まれています。
* `observable()` 関数は `20.0` を引数として受け取り、それを使ってプロパティを初期化します。
* `observable()` 関数に渡されるラムダ式：
  * 3つのパラメータがあります：
    * `_`：プロパティそのものを指します。
    * `old`：プロパティの以前の値。
    * `new`：プロパティの新しい値です。
  * `new` パラメータが `25` より大きいかどうかを確認し、その結果に応じてコンソールに文字列を出力します。
* `main()` 関数：
  * `Thermostat` クラスのインスタンスを `thermostat` という名前で作成します。
  * インスタンスの `temperature` プロパティの値を `22.5` に更新します。これにより、温度の更新情報を表示する print 文が実行されます。
  * インスタンスの `temperature` プロパティの値を `27.0` に更新すると、警告を含む出力文が実行されます。

観測可能なプロパティは、ロギングやデバッグの目的だけでなく、さまざまな場面で役立ちます。
また、UIの更新や、データの有効性の確認といった追加のチェックを行う場合などにも利用できます。

詳細については、[オブザーバブルプロパティ](delegated-properties.md#observable-properties)を参照してください。

## 演習 {completion-point="true"}

### 課題 1 {initial-collapse-state="collapsed" collapsible="true" id="properties-exercise-1"}

あなたは書店で在庫管理システムを運用しています。
在庫はリストとして保存されており、各項目は特定の書籍の数量を表しています。
たとえば、`listOf(3, 0, 7, 12)` は、その書店には第1巻が3冊、第2巻が0冊、第3巻が7冊、第4巻が12冊あることを意味します。

`findOutOfStockBooks()` という関数を記述し、在庫切れとなっているすべての書籍のインデックスのリストを返すようにしてください。

<deflist collapsible="true">
    <def title="Hint 1">
        標準ライブラリの <a href="https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/indices.html"><code>indices</code></a> 拡張プロパティを使用してください。
    </def>
</deflist>

<deflist collapsible="true">
    <def title="Hint 2">
        手動で可変リストを作成して返す代わりに、<a href="https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/build-list.html"><code>buildList()</code></a> 関数を使用して、リストを作成・管理することができます。 <code>buildList()</code> 関数は、前の章で学んだ、レシーバーを持つラムダ式を使用しています。
    </def>
</deflist>

|--|--|

```kotlin
fun findOutOfStockBooks(inventory: List<Int>): List<Int> {
    // Write your code here
}

fun main() {
    val inventory = listOf(3, 0, 7, 0, 5)
    println(findOutOfStockBooks(inventory))
    // [1, 3]
}
```
{validate="false" kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-properties-exercise-1"}

|---|---|
```kotlin
fun findOutOfStockBooks(inventory: List<Int>): List<Int> {
    val outOfStockIndices = mutableListOf<Int>()
    for (index in inventory.indices) {
        if (inventory[index] == 0) {
            outOfStockIndices.add(index)
        }
    }
    return outOfStockIndices
}

fun main() {
    val inventory = listOf(3, 0, 7, 0, 5)
    println(findOutOfStockBooks(inventory))
    // [1, 3]
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution 1" id="kotlin-tour-properties-solution-1-1"}

|---|---|
```kotlin
fun findOutOfStockBooks(inventory: List<Int>): List<Int> = buildList {
    for (index in inventory.indices) {
        if (inventory[index] == 0) {
            add(index)
        }
    }
}

fun main() {
    val inventory = listOf(3, 0, 7, 0, 5)
    println(findOutOfStockBooks(inventory))
    // [1, 3]
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution 2" id="kotlin-tour-properties-solution-1-2"}

### 課題 2 {initial-collapse-state="collapsed" collapsible="true" id="properties-exercise-2"}

キロメートルとマイルの両方で距離を表示する必要がある旅行アプリがあります。
キロメートル単位の距離をマイルに変換するための、`Double` 型の `asMiles` という拡張プロパティを作成します。

> キロメートルをマイルに変換する計算式は、`マイル = キロメートル × 0.621371` です。
>
{style="note"}

<deflist collapsible="true">
    <def title="Hint">
        拡張プロパティには、カスタム <code>get()</code> 関数が必要であることを忘れないでください。
    </def>
</deflist>

|---|---|

```kotlin
val // Write your code here

fun main() {
    val distanceKm = 5.0
    println("$distanceKm km is ${distanceKm.asMiles} miles")
    // 5.0 km is 3.106855 miles

    val marathonDistance = 42.195
    println("$marathonDistance km is ${marathonDistance.asMiles} miles")
    // 42.195 km is 26.218757 miles
}
```
{validate="false" kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-properties-exercise-2"}

|---|---|
```kotlin
val Double.asMiles: Double
    get() = this * 0.621371

fun main() {
    val distanceKm = 5.0
    println("$distanceKm km is ${distanceKm.asMiles} miles")
    // 5.0 km is 3.106855 miles

    val marathonDistance = 42.195
    println("$marathonDistance km is ${marathonDistance.asMiles} miles")
    // 42.195 km is 26.218757 miles
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution" id="kotlin-tour-properties-solution-2"}

### 課題 3 {initial-collapse-state="collapsed" collapsible="true" id="properties-exercise-3"}

クラウドシステムの状態を把握できるシステムヘルスチェッカーがあります。
ただし、ヘルスチェックを実行するために実行できる2つの機能は、処理負荷が高いものです。
lazyプロパティを使用してチェックを初期化し、処理負荷の高い関数が必要なときにのみ実行されるようにします：

|---|---|

```kotlin
fun checkAppServer(): Boolean {
    println("Performing application server health check...")
    return true
}

fun checkDatabase(): Boolean {
    println("Performing database health check...")
    return false
}

fun main() {
    // Write your code here

    when {
        isAppServerHealthy -> println("Application server is online and healthy")
        isDatabaseHealthy -> println("Database is healthy")
        else -> println("System is offline")
    }
    // Performing application server health check...
    // Application server is online and healthy
}
```
{validate="false" kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-properties-exercise-3"}

|---|---|
```kotlin
fun checkAppServer(): Boolean {
    println("Performing application server health check...")
    return true
}

fun checkDatabase(): Boolean {
    println("Performing database health check...")
    return false
}

fun main() {
    val isAppServerHealthy by lazy { checkAppServer() }
    val isDatabaseHealthy by lazy { checkDatabase() }

    when {
        isAppServerHealthy -> println("Application server is online and healthy")
        isDatabaseHealthy -> println("Database is healthy")
        else -> println("System is offline")
    }
   // Performing application server health check...
   // Application server is online and healthy
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution" id="kotlin-tour-properties-solution-3"}

### 課題 4 {initial-collapse-state="collapsed" collapsible="true" id="properties-exercise-4"}

あなたは、シンプルな予算管理アプリを作っています。
このアプリは、ユーザーの残りの予算の変動を監視し、予算が一定の閾値を下回った際にはユーザーに通知する必要があります。
`Budget` クラスがあり、初期予算額を含む `totalBudget` プロパティが初期化されています。
クラス内で、`remainingBudget` という名前のオブザーバブルプロパティを作成し、以下を出力するようにしてください：

* 予算額が当初予算の20%を下回った場合に警告を表示する。
* 予算が前回の金額から増額された際に表示される励ましのメッセージ。

|---|---|

```kotlin
import kotlin.properties.Delegates.observable

class Budget(val totalBudget: Int) {
    var remainingBudget: Int // Write your code here
}

fun main() {
    val myBudget = Budget(totalBudget = 1000)
    myBudget.remainingBudget = 800
    myBudget.remainingBudget = 150
    // Warning: Your remaining budget (150) is below 20% of your total budget.
    myBudget.remainingBudget = 50
    // Warning: Your remaining budget (50) is below 20% of your total budget.
    myBudget.remainingBudget = 300
    // Good news: Your remaining budget increased to 300.
}
```
{validate="false" kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-properties-exercise-4"}

|---|---|
```kotlin
import kotlin.properties.Delegates.observable

class Budget(val totalBudget: Int) {
    var remainingBudget: Int by observable(totalBudget) { _, oldValue, newValue ->
        if (newValue < totalBudget * 0.2) {
            println("Warning: Your remaining budget ($newValue) is below 20% of your total budget.")
        } else if (newValue > oldValue) {
            println("Good news: Your remaining budget increased to $newValue.")
        }
    }
}

fun main() {
    val myBudget = Budget(totalBudget = 1000)
    myBudget.remainingBudget = 800
    myBudget.remainingBudget = 150
    // Warning: Your remaining budget (150) is below 20% of your total budget.
    myBudget.remainingBudget = 50
    // Warning: Your remaining budget (50) is below 20% of your total budget.
    myBudget.remainingBudget = 300
    // Good news: Your remaining budget increased to 300.
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution" id="kotlin-tour-properties-solution-4"}

<seealso></seealso>

<list columns="2" id="tour-nav">
  <li>
    <a as="button" href="kotlin-tour-intermediate-open-special-classes_jp.md" mode="outline" icon="arrow-left" icon-position="left">Previous step</a>
  </li>
  <li>
    <a as="button" href="kotlin-tour-intermediate-null-safety_jp.md" mode="classic" icon="arrow-right" icon-position="right">Next step</a>
  </li>
</list>
