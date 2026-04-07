# CSRF

## <mark style="color:$primary;">ABOUT</mark>

<mark style="color:red;">**Cross-Site Request Forgery**</mark> (also known as CSRF) is a web security vulnerability that allows an attacker to <mark style="color:purple;">**induce users to perform actions that they do not intend to perform.**</mark> It allows an attacker to partly circumvent the same origin policy, which is designed to prevent different websites from interfering with each other.

### <mark style="color:blue;">How CSRF Work?</mark>

{% hint style="warning" %}
For a CSRF attack to be possible, three key conditions must be in place:

* <mark style="color:yellow;">**Relevant action.**</mark> There is an action within the application that the attacker has a reason to induce. This might be a privileged action (such as modifying permissions for other users) or any action on user-specific data (such as changing the user's own password).
* <mark style="color:yellow;">**Cookie-based session handling.**</mark> Performing the action involves issuing one or more HTTP requests, and the application relies solely on session cookies to identify the user who has made the requests. There is no other mechanism in place for tracking sessions or validating user requests.
* <mark style="color:yellow;">**No unpredictable request parameters.**</mark> The requests that perform the action do not contain any parameters whose values the attacker cannot determine or guess. For example, when causing a user to change their password, the function is not vulnerable if an attacker needs to know the value of the existing password.
{% endhint %}

#### Example of CSRF Request

```http
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE

email=wiener@normal-user.com
```

{% hint style="success" %}
This meets the conditions required for CSRF:

* The action of changing the email address on a user's account is of interest to an attacker. Following this action, the attacker will typically be able to trigger a password reset and take full control of the user's account.
* The application uses a session cookie to identify which user issued the request. There are no other tokens or mechanisms in place to track user sessions.
* The attacker can easily determine the values of the request parameters that are needed to perform the action.
{% endhint %}

## <mark style="color:$primary;">APPROACH</mark>

<mark style="color:orange;">**1. Map state-changing endpoints**</mark> \
Only these matter. GET requests that read data are irrelevant. Target: password change, email change, account settings, admin actions, fund transfers, role modifications.

<mark style="color:orange;">**2. Assess session handling**</mark> \
Confirm the app relies solely on cookies for auth. If you see custom headers (`X-Requested-With`, auth bearer tokens in body), note them — they complicate exploitation but don't always prevent it.

<mark style="color:orange;">**3. Check for CSRF tokens**</mark> \
Present → locate them. Absent → you're likely done, move to craft. If present, test each defect class in order:

* Token absent entirely (just delete the parameter)
* Token not tied to session (use your token in another user's request)
* Token not validated server-side (send any garbage value)
* Token tied to non-session cookie (separate bypass path)

<mark style="color:orange;">**4. Check SameSite cookie attribute**</mark> \
`Strict` → delivery is severely limited, cross-site POST blocked. `Lax` → GET-based attacks survive, top-level navigation only. `None` → no restriction, full attack surface. No attribute set → browser default (increasingly `Lax`).

<mark style="color:orange;">**5. Check Referer / Origin validation**</mark> \
App may validate these instead of tokens. Test: strip header entirely, send null origin, send origin as subdomain of target.

<mark style="color:orange;">**6. Determine exploitable HTTP method**</mark> \
POST required → need full HTML form, hosted page. GET accepted → `<img>` tag, single URL, no external page needed.

<mark style="color:orange;">**7. Craft and deliver**</mark> \
Match method to mechanism. GET: embed in `<img src>` or direct URL. POST: auto-submitting HTML form, `<body onload="document.forms[0].submit()">`. Host on controlled infrastructure or inject into site with existing user traffic if available.

<mark style="color:orange;">**8. Confirm impact**</mark> \
Action executes silently under victim session. Verify the state change occurred. Chain forward if applicable — email change → password reset → account takeover.

### <mark style="color:blue;">DELIVERY</mark>

The delivery mechanisms for cross-site request forgery attacks are essentially the same as for reflected XSS. Typically, the attacker will place the malicious HTML onto a website that they control, and then induce victims to visit that website. This might be done by feeding the user a link to the website, via an email or social media message. Or if the attack is placed into a popular website (for example, in a user comment), they might just wait for users to visit the website.

Note that some simple CSRF exploits employ the GET method and can be fully self-contained with a single URL on the vulnerable website. In this situation, the attacker may not need to employ an external site, and can directly feed victims a malicious URL on the vulnerable domain. In the preceding example, if the request to change email address can be performed with the GET method, then a self-contained attack would look like this:

```html
<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">
```

## <mark style="color:$primary;">BURP COLLABORATOR</mark>

Manually creating the HTML needed for a CSRF exploit can be cumbersome, particularly where the desired request contains a large number of parameters, or there are other quirks in the request. The easiest way to construct a CSRF exploit is using the CSRF PoC generator that is built in to Burp Suite Professional:

1. Select a request anywhere in Burp Suite Professional that you want to test or exploit.
2. From the right-click context menu, select **Engagement tools / Generate CSRF PoC**.
3. Burp Suite will generate some HTML that will trigger the selected request (minus cookies, which will be added automatically by the victim's browser).
4. You can tweak various options in the CSRF PoC generator to fine-tune aspects of the attack. You might need to do this in some unusual situations to deal with quirky features of requests.
5. Copy the generated HTML into a web page, view it in a browser that is logged in to the vulnerable website, and test whether the intended request is issued successfully and the desired action occurs.

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$primary;">DEFENSE BYPASS</mark>

Basically the most common defenses are Same-Site or CSRF Tokens. There are separate sections for there, here I would write about some weird or unorthodox ways of protecting a website

### <mark style="color:red;">If Referer isn't present, skip validation</mark>

Pretty self-explanatory name

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0a6c009904a5919e84317c9b00170011.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="aboba&#64;gmail&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <meta name="referrer" content="no-referrer">
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```
