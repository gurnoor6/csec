# Cross-site Request Forgery (CSRF)
## What is CSRF?
Cross-site Request Forgery is a vulnerability that allows and attacker to induce users to perform actions that they do not intend to perform. For a CSRF attack, three key conditions must be fulfilled:
1. A relevant action. There is an action with the application that the attacker has a reason to induce.
2. Cookie-based session handling. The action involves issuing HTTP requests, and the application relies on session cookies solely to identify the user.
3. No unpredictable request parameters. The requests that perform the action do not contain any parameters whose values the attacker cannot determine or guess.

## How does CSRF work?
For example, suppose an application has the functionality to allow users to change their email address. The corresponding HTTP request might look like:

```
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE

email=wiener@normal-user.com
```

This satisfies the key conditions for CSRF:
1. The action of changing the email address on a user's account is relevant.
2. The application uses a session cookie to identify which user issued the request. No other mechanism exists to check the same.
3. The attacker can easily determine the values of the request parameters that are needed to perform the action.

The attacker can construct a webpage containing the following HTML:

```
<html>
  <body>
    <form action="https://vulnerable-website.com/email/change" method="POST">
      <input type="hidden" name="email" value="pwned@evil-user.net" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

If a victim user visits the attacker's web page, the following will happen, firstly the attacker's page will trigger an HTTP request to the vulnerable web site.
Now if the user is logged in to the vulnerable site, their browser will automatically include the session cookie in the request.
The vulnerable web site will process the request in the normal way, treat it as having been made by the victim user, and change their email address.

## How to construct a CSRF Attack?
You have two options, either do the manual cumbersome process like the above example, possibly for multiple parameters, or use the inbuilt CSRF PoC Generator of Burp Suite Professional.
You can try this out [here](https://portswigger.net/web-security/csrf/lab-no-defenses).

## How to deliver a CSRF Exploit?
Typically, the attacker will place the malicious HTML onto a web site that they control, and then induce victims to visit that web site. This might be done by feeding the user a link to the web site, via an email or social media message. Or if the attack is placed into a popular web site (for example, in a user comment), they might just wait for users to visit the web site.

## What is the impact of a CSRF Attack?
This attack has the potential to change the email address or password of a user's account, or to make a funds transfer. Depending on the nature of the action, the attacker might be able to gain full control over the user's account. If the compromised user has a privileged role within the application, then the attacker might be able to take full control of all the application's data and functionality.

## Preventing CSRF Attacks
A robust way of preventing these if by including CSRF Tokens within relevant requests. You can read about these in more detail [here](https://portswigger.net/web-security/csrf/tokens). In general, the token should be:
* Unpredictable with high entropy, as for session tokens in general.
* Tied to the user's session.
* Strictly validated in every case before the relevant action is executed.

