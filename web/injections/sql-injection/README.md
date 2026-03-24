# SQL Injection

## <mark style="color:$primary;">ABOUT</mark>

<mark style="color:red;">**SQL injection (SQLi)**</mark> is a <mark style="color:purple;">**critical web security vulnerability**</mark> that allows an attacker to interfere with the queries that an application makes to its database. This manipulation can enable an attacker to view data they are not normally able to retrieve, such as data belonging to other users or any other information the application can access. In many operational scenarios, an attacker can <mark style="color:purple;">**modify or delete this data**</mark>, causing persistent changes to the application's content or behavior, which fundamentally compromises the integrity of the system.

{% hint style="warning" %}
&#x20;**Start small, then steadily build toward greater impact**
{% endhint %}

## <mark style="color:$primary;">WHERE TO LOOK</mark>

Most SQL injection vulnerabilities occur within the **WHERE** clause of a **SELECT** query. Most experienced testers are familiar with this type of SQL injection. However, SQL injection vulnerabilities can occur at any location within the query, and within different query types. Some other common locations where SQL injection arises are:

* In **UPDATE** statements, within the updated values or the **WHERE** clause.
* In **INSERT** statements, within the inserted values.
* In **SELECT** statements, within the table or column name.
* In **SELECT** statements, within the **ORDER BY** clause.&#x20;

{% hint style="info" %}
So the main idea is to learn application and search for where is most likely SQL database requests in back-end, and test payloads there
{% endhint %}

## <mark style="color:$primary;">APPROACH</mark>

How to detect SQL injection vulnerabilities

You can detect SQL injection manually using a systematic set of tests against every entry point in the application. To do this, you would typically submit:

* The single quote character `'` and <mark style="color:orange;">**look for errors or other anomalies**</mark>.
* Some SQL-specific syntax that evaluates to the base (original) value of the entry point, and to a different value, and <mark style="color:orange;">**look for systematic differences in the application responses.**</mark>
* <mark style="color:orange;">**Boolean conditions**</mark> such as `OR 1=1` and `OR 1=2`, and look for differences in the application's responses.
* Payloads designed to trigger <mark style="color:orange;">**time delays**</mark> when executed within a SQL query, and look for differences in the time taken to respond.
* OAST payloads designed to <mark style="color:orange;">**trigger an out-of-band network interaction**</mark> when executed within a SQL query, and monitor any resulting interactions.&#x20;

{% hint style="info" %}
Alternatively, you can find the majority of SQL injection vulnerabilities quickly and reliably using Burp Scanner.
{% endhint %}

## <mark style="color:$primary;">DIFFERENT CONTEXT</mark>

You can perform SQL injection attacks using any controllable input that is processed as a SQL query by the application. For example, some websites take input in JSON or XML format and use this to query the database.

These different formats may provide different ways for you to obfuscate attacks that are otherwise blocked due to WAFs and other defense mechanisms. Weak implementations often look for common SQL injection keywords within the request, so you may be able to bypass these filters by encoding or escaping characters in the prohibited keywords. For example, the following XML-based SQL injection uses an XML escape sequence to encode the S character in SELECT:

```xml
<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
```

This will be decoded server-side before being passed to the SQL interpreter.

<details>

<summary>XML HTML Entity Obfuscation</summary>

#### FROM

```sql
UNION SELECT username || '~' || password FROM users
```

#### TO

```xml
&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;&#x20;&#x75;&#x73;&#x65;&#x72;&#x6e;&#x61;&#x6d;&#x65;&#x20;&#x7c;&#x7c;&#x20;&#x27;&#x7e;&#x27;&#x20;&#x7c;&#x7c;&#x20;&#x70;&#x61;&#x73;&#x73;&#x77;&#x6f;&#x72;&#x64;&#x20;&#x46;&#x52;&#x4f;&#x4d;&#x20;&#x75;&#x73;&#x65;&#x72;&#x73;
```

<figure><img src="../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

</details>

## <mark style="color:$primary;">EXAMPLES</mark>

The following payloads demonstrate basic retrieval techniques used in standard testing scenarios. The first payload utilizes a comment indicator (`--`) to remove the trailing SQL constraints, effectively displaying unreleased products. The second payload injects a boolean tautology (`OR 1=1`) to force the database to return every record in the table, bypassing specific category filters.

{% hint style="info" %}
IN SOME EXAMPLES I WOULD USE SCREENSHOTS FROM OBSIDIAN, BECAUSE IT CORRECTLY INTERPRETS SQL REQUESTS
{% endhint %}

On the back-end SQL query probably looks like this:

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

In first payload we use -- comment indicator to cut off the next part of request.

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

We can see here SQL syntax interpretation: part after **Gifts** is seen as a comment.

The other way we can add out payload is by using logical operators, like `OR`

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

In this scenario we use OR 1=1, and because 1=1 is always true. Database would show us everything&#x20;

{% hint style="danger" %}
Yes we are using OR 1=1 as example. but that payload is considered aggressive, because it can be seen by other non-seen part of request which can delete some tables as example. And together you an have devastating consequences, so don't be a gonk and don't use that anywhere except training.
{% endhint %}

## <mark style="color:$primary;">PREVENTION</mark>

You can prevent most instances of SQL injection using parameterized queries instead of string concatenation within the query. These parameterized queries are also know as "prepared statements".

The following code is vulnerable to SQL injection because the user input is concatenated directly into the query:

```java
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery(query);
```

You can rewrite this code in a way that prevents the user input from interfering with the query structure:

```java
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```

## <mark style="color:$primary;">ADDITIONAL NOTES</mark>

It is important to note that SQL injection attacks can be performed using any controllable input processed as a SQL query by the application, including input formats like JSON or XML. Different formats may provide unique methods to obfuscate attacks that might otherwise be blocked by WAFs. For instance, an XML-based injection can use escape sequences to encode specific characters, such as encoding the "S" in `SELECT(999 &#x53;ELECT...)`, which is decoded server-side before execution19.

## <mark style="color:$primary;">RESOURCES</mark>

{% embed url="https://portswigger.net/web-security/sql-injection/cheat-sheet" %}

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection" %}
