# Vulnerabilities in other authentication mechanisms
In addition to the basic login functionality, most websites provide supplementary functionality to allow users to manage their account. For example, users can typically change their password or reset their password when they forget it. These mechanisms can also introduce vulnerabilities that can be exploited by an attacker. 

## Keeping users logged in
A common feature is the option to stay logged in even after closing a browser session. This is usually a simple checkbox labeled something like "Remember me" or "Keep me logged in".

This functionality is often implemented by generating a "remember me" token of some kind, which is then stored in a persistent cookie. As possessing this cookie effectively allows you to bypass the entire login process, it is best practice for this cookie to be impractical to guess. However, some websites generate this cookie based on a predictable concatenation of static values, such as the username and a timestamp. Some even use the password as part of the cookie. This approach is particularly dangerous if an attacker is able to create their own account because they can study their own cookie and potentially deduce how it is generated. Once they work out the formula, they can try to brute-force other users' cookies to gain access to their accounts. 

Even if the attacker is not able to create their own account, they may still be able to exploit this vulnerability. Using the usual techniques, such as XSS, an attacker could steal another user's "remember me" cookie and deduce how the cookie is constructed from that. If the website was built using an open-source framework, the key details of the cookie construction may even be publicly documented.

## Resetting user passwords
In practice, it is a given that some users will forget their password, so it is common to have a way for them to reset it. As now the users cannot be authenticated using their passwords, this feature needs to be implemented very securely.

### Sending passwords by email
Generally, when you reset a password, you receive an email with a new temporary password which is valid for a short period of time. This approach is susceptible to man in the middle attacks, so is vulnerable. Also, amny users sync their emails across multiple devices, transmitting sensitive data over insecure channels.

### Resetting passwords using a URL
A more robust method of resetting passwords is to send a unique URL to users that takes them to a password reset page. Less secure implementations of this method use a URL with an easily guessable parameter to identify which account is being reset, for example: ` http://vulnerable-website.com/reset-password?user=victim-user`.
In this example, an attacker could change the user parameter to refer to any username they have identified. They would then be taken straight to a page where they can potentially set a new password for this arbitrary user. 

### Password reset poisoning
This method relies on manipulating the HTTP request headers so that the link for password reset that the victim user receives, is what the attacker controls. So when the victim clicks on that link, the attacker gets access to the original URL that can be used for the password reset of the victim. To get more idea about how it actually works, solve the lab for it [here](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-poisoning) .

## Changing user passwords
Typically what happens is that when you click on the link for resetting the password, you are redirected to a page that asks for a new password twice. Once you enter them and submit the form, the password is reset. However, if there is a hidden `username` field, that is used to display the form, the attacker might manipulate it to reset the password of any user, without even having them to click the link!!