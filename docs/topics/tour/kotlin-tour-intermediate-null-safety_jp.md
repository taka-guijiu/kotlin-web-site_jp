[//]: # (title: Null safety)

<no-index/>


初心者向けツアーでは、コード内で `null` 値を扱う方法を学びました。
この章では、ヌルセーフ機能の一般的な使用例と、それらを最大限に活用する方法について解説します。

## Smart casts and safe casts
## スマートキャストとセーフキャスト

Kotlinでは、明示的な宣言がなくても型を推論できる場合があります。
Kotlinに対して、変数やオブジェクトを特定の型に属するものとして扱うよう指示することを、**型変換**と呼びます。
型が自動的にキャストされる場合（型推論が行われる場合など）、それは**スマートキャスト**と呼ばれます。

### is and !is operators
### is および !is 演算子

型変換の仕組みについて詳しく見る前に、オブジェクトが特定の型であるかどうかを確認する方法を見てみましょう。
For this, you can use the
`when` または `if` の条件式と組み合わせて使用する `is` および `!is` 演算子：

* `is` は、オブジェクトが指定された型であるかどうかを調べ、ブール値を返します。
* `!is` は、オブジェクトがその型を**持っていない**かどうかを調べ、ブール値を返します。

For example:

```kotlin
fun printObjectType(obj: Any) {
    when (obj) {
        is Int -> println("It's an Integer with value $obj")
        !is Double -> println("It's NOT a Double")
        else -> println("Unknown type")
    }
}

fun main() {
    val myInt = 42
    val myDouble = 3.14
    val myList = listOf(1, 2, 3)
  
    // The type is Int
    printObjectType(myInt)
    // It's an Integer with value 42

    // The type is List, so it's NOT a Double.
    printObjectType(myList)
    // It's NOT a Double

    // The type is Double, so the else branch is triggered.
    printObjectType(myDouble)
    // Unknown type
}
```
{kotlin-runnable="true" id="kotlin-tour-null-safety-casts"}

> `is` および `!is` 演算子と組み合わせて `when` 条件式を使用する方法の例については、[Open およびその他の特殊クラス](kotlin-tour-intermediate-open-special-classes.md#sealed-classes) の章ですでに紹介されています。
> 
{style="tip"}

### as and as? operators
### `as` および `as` 演算子

オブジェクトを他の型に明示的に_キャスト_するには、`as`演算子を使用します。
これには、Null許容型からNull非許容型への型変換も含まれます。
キャストができない場合、プログラムは**実行時に**クラッシュします。
だからこそ、これは****安全でない**型変換演算子**と呼ばれるのです。**

```kotlin
fun main() {
//sampleStart
    val a: String? = null
    val b = a as String

    // Triggers an error at runtime
    print(b)
//sampleEnd
}
```
{kotlin-runnable="true" validate="false" id="kotlin-tour-null-safety-as-operator"}

オブジェクトを明示的にヌル不可型にキャストし、失敗時にエラーをスローするのではなく `null` を返すには、`as?` 演算子を使用します。
`as?` 演算子は、失敗してもエラーを発生させないため、**安全な**演算子と呼ばれています。

```kotlin
fun main() {
//sampleStart
    val a: String? = null
    val b = a as? String

    // Returns null value
    print(b)
    // null
//sampleEnd
}
```
{kotlin-runnable="true" id="kotlin-tour-null-safety-safe-operator"}

`as?` 演算子をエルビス演算子 `?:` と組み合わせることで、数行のコードを1行にまとめることができます。
たとえば、次の `calculateTotalStringLength()` 関数は、さまざまな種類の文字列が混在するリストに含まれるすべての文字列の合計長さを計算します：

```kotlin
fun calculateTotalStringLength(items: List<Any>): Int {
    var totalLength = 0

    for (item in items) {
        totalLength += if (item is String) {
            item.length
        } else {
            0  // Add 0 for non-String items
        }
    }

    return totalLength
}
```

例：

* `totalLength` 変数をカウンターとして使用します。
* `for` ループを使用して、リスト内のすべての項目を順に処理します。
* `if` ステートメントと `is` 演算子を使用して、現在の項目が文字列であるかどうかを確認します：
  * そうである場合、その文字列の長さがカウンタに加算されます。
  * そうでない場合、カウンタはインクリメントされません。
* `totalLength` 変数の最終的な値を返します。

このコードは次のように簡略化できます：

```kotlin
fun calculateTotalStringLength(items: List<Any>): Int {
    return items.sumOf { (it as? String)?.length ?: 0 }
}
```

この例では、[`.sumOf()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/sum-of.html) 拡張関数を使用しており、次のようなラムダ式を指定しています：

* リスト内の各項目について、`as?` を使用して `String` への安全なキャストを実行します。
* 呼び出しが `null` 値を返さない場合に、安全な呼び出し `?.` を使用して `length` プロパティにアクセスします。
* エルビス演算子 `?:` を使用し、セーフコールが `null` 値を返した場合に `0` を返します。

## ヌル値とコレクション

Kotlinでは、コレクションを扱う際、`null`値の処理や不要な要素の除外が必要になることがよくあります。
Kotlinには、リスト、セット、マップ、その他のコレクションを扱う際に、クリーンで効率的かつnullセーフなコードを書くために利用できる便利な関数が用意されています。

リストから `null` 値をフィルタリングするには、[`filterNotNull()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/filter-not-null.html) 関数を使用します：

```kotlin
fun main() {
//sampleStart
    val emails: List<String?> = listOf("alice@example.com", null, "bob@example.com", null, "carol@example.com")

    val validEmails = emails.filterNotNull()

    println(validEmails)
    // [alice@example.com, bob@example.com, carol@example.com]
//sampleEnd
}
```
{kotlin-runnable="true" id="kotlin-tour-null-safety-filternotnull"}

リストを作成する際に、`null` 値のフィルタリングを直接行いたい場合は、[`listOfNotNull()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/list-of-not-null.html) 関数を使用してください：

```kotlin
fun main() {
//sampleStart
    val serverConfig = mapOf(
        "appConfig.json" to "App Configuration",
        "dbConfig.json" to "Database Configuration"
    )

    val requestedFile = "appConfig.json"
    val configFiles = listOfNotNull(serverConfig[requestedFile])

    println(configFiles)
    // [App Configuration]
//sampleEnd
}
```
{kotlin-runnable="true" id="kotlin-tour-null-safety-listofnotnull"}

これらの例の両方において、すべての項目が `null` 値の場合、空のリストが返されます。

Kotlin には、コレクション内の値を検索するために使用できる関数も用意されています。
値が見つからない場合、エラーを発生させるのではなく、`null`値を返します：

* [`maxOrNull()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/max-or-null.html) は、最大値を求めます。最大値が存在しない場合は、`null` を返します。
* [`minOrNull()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/min-or-null.html) は、最小値を検索します。最小値が存在しない場合は、`null` を返します。

例えば：

```kotlin
fun main() {
//sampleStart
    // 1週間にわたって記録された気温
    val temperatures = listOf(15, 18, 21, 21, 19, 17, 16)
  
    // その週の最高気温を算出する
    val maxTemperature = temperatures.maxOrNull()
    println("Highest temperature recorded: ${maxTemperature ?: "No data"}")
    // Highest temperature recorded: 21

    // その週の最低気温を算出する
    val minTemperature = temperatures.minOrNull()
    println("Lowest temperature recorded: ${minTemperature ?: "No data"}")
    // Lowest temperature recorded: 15
//sampleEnd
}
```
{kotlin-runnable="true" id="kotlin-tour-null-safety-collections"}

この例では、Elvis演算子 `?:` を使用して、関数が `null` 値を返した場合に、出力文を返すようにしています。

> `maxOrNull()` および `minOrNull()` 関数は、`null` 値を含んで**いない**コレクションで使用するように設計されています。
> そうしないと、関数が目的の値を見つけられなかったのか、それとも `null` 値を見つけたのかが判別できません。
>
{style="note"}

条件に一致する単一の項目を見つけるには、ラムダ式とともに [`singleOrNull()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/single-or-null.html) 関数を使用できます。
該当するものが存在しない場合、または一致する項目が複数ある場合、この関数は `null` 値を返します：

```kotlin
fun main() {
//sampleStart
    // 1週間にわたって記録された気温
    val temperatures = listOf(15, 18, 21, 21, 19, 17, 16)

    // 30度の日がちょうど1日あったかどうかを確認する
    val singleHotDay = temperatures.singleOrNull{ it == 30 }
    println("Single hot day with 30 degrees: ${singleHotDay ?: "None"}")
    // Single hot day with 30 degrees: None
//sampleEnd
}
```
{kotlin-runnable="true" id="kotlin-tour-null-safety-singleornull"}

> The `singleOrNull()` function is designed to be used with collections that **don't** contain `null` values.
>
{style="note"}

Some functions use a lambda expression to transform a collection and return `null` values if they can't
fulfill their purpose.

To transform a collection with a lambda expression and return the first value that isn't `null`, use the 
[`firstNotNullOfOrNull()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/first-not-null-of-or-null.html) function. If no such value exists, the function returns a `null` value:

```kotlin
fun main() {
//sampleStart
    data class User(val name: String?, val age: Int?)

    val users = listOf(
        User(null, 25),
        User("Alice", null),
        User("Bob", 30)
    )

    val firstNonNullName = users.firstNotNullOfOrNull { it.name }
    println(firstNonNullName)
    // Alice
//sampleEnd
}
```
{kotlin-runnable="true" id="kotlin-tour-null-safety-firstnotnullofornull"}

To use a lambda expression to process each collection item sequentially and create an accumulated value (or return a 
`null` value if the collection is empty) use the [`reduceOrNull()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/reduce-or-null.html) function:

```kotlin
fun main() {
//sampleStart
    // Prices of items in a shopping cart
    val itemPrices = listOf(20, 35, 15, 40, 10)

    // Calculate the total price using the reduceOrNull() function
    val totalPrice = itemPrices.reduceOrNull { runningTotal, price -> runningTotal + price }
    println("Total price of items in the cart: ${totalPrice ?: "No items"}")
    // Total price of items in the cart: 120

    val emptyCart = listOf<Int>()
    val emptyTotalPrice = emptyCart.reduceOrNull { runningTotal, price -> runningTotal + price }
    println("Total price of items in the empty cart: ${emptyTotalPrice ?: "No items"}")
    // Total price of items in the empty cart: No items
//sampleEnd
}
```
{kotlin-runnable="true" id="kotlin-tour-null-safety-reduceornull"}

This example also uses the Elvis operator `?:` to return a printed statement if the function returns a `null` value.

> The `reduceOrNull()` function is designed to be used with collections that **don't** contain `null` values.
>
{style="note"}

Explore Kotlin's [standard library](https://kotlinlang.org/api/core/kotlin-stdlib/) to find more functions that you can 
use to make your code safer.

## Early returns and the Elvis operator

In the beginner tour, you learned how to use [early returns](kotlin-tour-functions.md#early-returns-in-functions) to stop
your function from being processed further than a certain point. You can use the Elvis operator `?:` with an early return
to check preconditions in a function. This approach is a great way to keep your code concise because you don't need to use
nested checks. The reduced complexity of your code also makes it easier to maintain. For example:

```kotlin
data class User(
    val id: Int,
    val name: String,
    // List of friend user IDs
    val friends: List<Int>
)

// Function to get the number of friends for a user
fun getNumberOfFriends(users: Map<Int, User>, userId: Int): Int {
    // Retrieves the user or return -1 if not found
    val user = users[userId] ?: return -1
    // Returns the number of friends
    return user.friends.size
}

fun main() {
    // Creates some sample users
    val user1 = User(1, "Alice", listOf(2, 3))
    val user2 = User(2, "Bob", listOf(1))
    val user3 = User(3, "Charlie", listOf(1))

    // Creates a map of users
    val users = mapOf(1 to user1, 2 to user2, 3 to user3)

    println(getNumberOfFriends(users, 1))
    // 2
    println(getNumberOfFriends(users, 2))
    // 1
    println(getNumberOfFriends(users, 4))
    // -1
}
```
{kotlin-runnable="true" id="kotlin-tour-null-safety-early-return"}

In this example:

* There is a `User` data class that has properties for the user's `id`, `name` and list of friends.
* The `getNumberOfFriends()` function:
  * Accepts a map of `User` instances and a user ID as an integer.
  * Accesses the value of the map of `User` instances with the provided user ID.
  * Uses an Elvis operator to return the function early with the value of `-1` if the map value is a `null` value.
  * Assigns the value found from the map to the `user` variable.
  * Returns the number of friends in the user's friends list by using the `size` property.
* The `main()` function:
  * Creates three `User` instances. 
  * Creates a map of these `User` instances and assigns them to the `users` variable. 
  * Calls the `getNumberOfFriends()` function on the `users` variable with values `1` and `2` that returns two friends for `"Alice"` and one friend for `"Bob"`.
  * Calls the `getNumberOfFriends()` function on the `users` variable with value `4`, which triggers an early return with a value of `-1`.

You may notice that the code could be more concise without an early return. However, this approach needs multiple safe 
calls because the `users[userId]` might return a `null` value, making the code slightly harder to read:

```kotlin
fun getNumberOfFriends(users: Map<Int, User>, userId: Int): Int {
    // Retrieve the user or return -1 if not found
    return users[userId]?.friends?.size ?: -1
}
```
{validate="false"}

Although this example checks only one condition with the Elvis operator, you can add multiple checks to cover any critical
error paths. Early returns with the Elvis operator prevent your program from doing unnecessary work and make your code 
safer by stopping as soon as a `null` value or invalid case is detected.

For more information about how you can use `return` in your code, see [Returns and jumps](returns.md).

## Practice {completion-point="true"}

### Exercise 1 {initial-collapse-state="collapsed" collapsible="true" id="null-safety-exercise-1"}

You are developing a notification system for an app where users can enable or disable different types of notifications.
Complete the `getNotificationPreferences()` function so that:

1. The `validUser` variable uses the `as?` operator to check if `user` is an instance of the `User` class. If it isn't, return an empty list.
2. The `userName` variable uses the Elvis `?:` operator to ensure that the user's name defaults to `"Guest"` if it is `null`.
3. The final return statement uses the `.takeIf()` function to include email and SMS notification preferences only if they are enabled.
4. The `main()` function runs successfully and prints the expected output.

> The [`takeIf()` function](scope-functions.md#takeif-and-takeunless) returns the original value if the given condition is true,
> otherwise it returns `null`. For example:
>
> ```kotlin
> fun main() {
>     // The user is logged in
>     val userIsLoggedIn = true
>     // The user has an active session
>     val hasSession = true
> 
>     // Gives access to the dashboard if the user is logged in
>     // and has an active session
>     val canAccessDashboard = userIsLoggedIn.takeIf { hasSession }
> 
>     println(canAccessDashboard ?: "Access denied")
>     // true
> }
> ```
>
{style = "tip"}

|--|--|

```kotlin
data class User(val name: String?)

fun getNotificationPreferences(user: Any, emailEnabled: Boolean, smsEnabled: Boolean): List<String> {
    val validUser = // Write your code here
    val userName = // Write your code here

    return listOfNotNull( /* Write your code here */)
}

fun main() {
    val user1 = User("Alice")
    val user2 = User(null)
    val invalidUser = "NotAUser"

    println(getNotificationPreferences(user1, emailEnabled = true, smsEnabled = false))
    // [Email Notifications enabled for Alice]
    println(getNotificationPreferences(user2, emailEnabled = false, smsEnabled = true))
    // [SMS Notifications enabled for Guest]
    println(getNotificationPreferences(invalidUser, emailEnabled = true, smsEnabled = true))
    // []
}
```
{validate="false" kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-null-safety-exercise-1"}

|--|--|

```kotlin
data class User(val name: String?)

fun getNotificationPreferences(user: Any, emailEnabled: Boolean, smsEnabled: Boolean): List<String> {
    val validUser = user as? User ?: return emptyList()
    val userName = validUser.name ?: "Guest"

    return listOfNotNull(
        "Email Notifications enabled for $userName".takeIf { emailEnabled },
        "SMS Notifications enabled for $userName".takeIf { smsEnabled }
    )
}

fun main() {
    val user1 = User("Alice")
    val user2 = User(null)
    val invalidUser = "NotAUser"

    println(getNotificationPreferences(user1, emailEnabled = true, smsEnabled = false))
    // [Email Notifications enabled for Alice]
    println(getNotificationPreferences(user2, emailEnabled = false, smsEnabled = true))
    // [SMS Notifications enabled for Guest]
    println(getNotificationPreferences(invalidUser, emailEnabled = true, smsEnabled = true))
    // []
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution" id="kotlin-tour-null-safety-solution-1"}

### Exercise 2 {initial-collapse-state="collapsed" collapsible="true" id="null-safety-exercise-2"}

You are working on a subscription-based streaming service where users can have multiple subscriptions, but **only one 
can be active at a time**. Complete the `getActiveSubscription()` function so that it uses the `singleOrNull()` function
with a predicate to return a `null` value if there is more than one active subscription:

|--|--|

```kotlin
data class Subscription(val name: String, val isActive: Boolean)

fun getActiveSubscription(subscriptions: List<Subscription>): Subscription? // Write your code here

fun main() {
    val userWithPremiumPlan = listOf(
        Subscription("Basic Plan", false),
        Subscription("Premium Plan", true)
    )

    val userWithConflictingPlans = listOf(
        Subscription("Basic Plan", true),
        Subscription("Premium Plan", true)
    )

    println(getActiveSubscription(userWithPremiumPlan))
    // Subscription(name=Premium Plan, isActive=true)

    println(getActiveSubscription(userWithConflictingPlans))
    // null
}
```
{validate="false" kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-null-safety-exercise-2"}

|--|--|

```kotlin
data class Subscription(val name: String, val isActive: Boolean)

fun getActiveSubscription(subscriptions: List<Subscription>): Subscription? {
    return subscriptions.singleOrNull { subscription -> subscription.isActive }
}

fun main() {
    val userWithPremiumPlan = listOf(
        Subscription("Basic Plan", false),
        Subscription("Premium Plan", true)
    )

    val userWithConflictingPlans = listOf(
        Subscription("Basic Plan", true),
        Subscription("Premium Plan", true)
    )

    println(getActiveSubscription(userWithPremiumPlan))
    // Subscription(name=Premium Plan, isActive=true)

    println(getActiveSubscription(userWithConflictingPlans))
    // null
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution 1" id="kotlin-tour-null-safety-solution-2-1"}

|--|--|

```kotlin
data class Subscription(val name: String, val isActive: Boolean)

fun getActiveSubscription(subscriptions: List<Subscription>): Subscription? =
    subscriptions.singleOrNull { it.isActive }

fun main() {
    val userWithPremiumPlan = listOf(
        Subscription("Basic Plan", false),
        Subscription("Premium Plan", true)
    )

    val userWithConflictingPlans = listOf(
        Subscription("Basic Plan", true),
        Subscription("Premium Plan", true)
    )

    println(getActiveSubscription(userWithPremiumPlan))
    // Subscription(name=Premium Plan, isActive=true)

    println(getActiveSubscription(userWithConflictingPlans))
    // null
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution 2" id="kotlin-tour-null-safety-solution-2-2"}

### Exercise 3 {initial-collapse-state="collapsed" collapsible="true" id="null-safety-exercise-3"}

You are working on a social media platform where users have usernames and account statuses. You want to see the list of 
currently active usernames. Complete the `getActiveUsernames()` function so that the [`mapNotNull()` function](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/map-not-null.html)
has a predicate that returns the username if it is active or a `null` value if it isn't:

|--|--|

```kotlin
data class User(val username: String, val isActive: Boolean)

fun getActiveUsernames(users: List<User>): List<String> {
    return users.mapNotNull { /* Write your code here */ }
}

fun main() {
    val allUsers = listOf(
        User("alice123", true),
        User("bob_the_builder", false),
        User("charlie99", true)
    )

    println(getActiveUsernames(allUsers))
    // [alice123, charlie99]
}
```
{validate="false" kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-null-safety-exercise-3"}

|--|--|

> Just like in Exercise 1, you can use the [`takeIf()` function](scope-functions.md#takeif-and-takeunless) when you check
> if the user is active.
>
{ style = "tip" }

|--|--|

```kotlin
data class User(val username: String, val isActive: Boolean)

fun getActiveUsernames(users: List<User>): List<String> {
    return users.mapNotNull { user ->
        if (user.isActive) user.username else null
    }
}

fun main() {
    val allUsers = listOf(
        User("alice123", true),
        User("bob_the_builder", false),
        User("charlie99", true)
    )

    println(getActiveUsernames(allUsers))
    // [alice123, charlie99]
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution 1" id="kotlin-tour-null-safety-solution-3-1"}

|--|--|

```kotlin
data class User(val username: String, val isActive: Boolean)

fun getActiveUsernames(users: List<User>): List<String> =
    users.mapNotNull { user -> user.username.takeIf { user.isActive } }

fun main() {
    val allUsers = listOf(
        User("alice123", true),
        User("bob_the_builder", false),
        User("charlie99", true)
    )

    println(getActiveUsernames(allUsers))
    // [alice123, charlie99]
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution 2" id="kotlin-tour-null-safety-solution-3-2"}

### Exercise 4 {initial-collapse-state="collapsed" collapsible="true" id="null-safety-exercise-4"}

You are working on an inventory management system for an e-commerce platform. Before processing a sale, you need to check
if the requested quantity of a product is valid based on the available stock.

Complete the `validateStock()` function so that it uses early returns and the Elvis operator (where applicable) to check if:

* The `requested` variable is `null`.
* The `available` variable is `null`.
* The `requested` variable is a negative value.
* The amount in the `requested` variable is higher than in the `available` variable.

In all of the above cases, the function must return early with the value of `-1`.

|--|--|

```kotlin
fun validateStock(requested: Int?, available: Int?): Int {
    // Write your code here
}

fun main() {
    println(validateStock(5,10))
    // 5
    println(validateStock(null,10))
    // -1
    println(validateStock(-2,10))
    // -1
}
```
{validate="false" kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-null-safety-exercise-4"}

|--|--|

```kotlin
fun validateStock(requested: Int?, available: Int?): Int {
    val validRequested = requested ?: return -1
    val validAvailable = available ?: return -1

    if (validRequested < 0) return -1
    if (validRequested > validAvailable) return -1

    return validRequested
}

fun main() {
    println(validateStock(5,10))
    // 5
    println(validateStock(null,10))
    // -1
    println(validateStock(-2,10))
    // -1
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution" id="kotlin-tour-null-safety-solution-4"}

<seealso></seealso>

<list columns="2" id="tour-nav">
  <li>
    <a as="button" href="kotlin-tour-intermediate-properties.md" mode="outline" icon="arrow-left" icon-position="left">Previous step</a>
  </li>
  <li>
    <a as="button" href="kotlin-tour-intermediate-libraries-and-apis.md" mode="classic" icon="arrow-right" icon-position="right">Next step</a>
  </li>
</list>
