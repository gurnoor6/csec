# Vulnerabilities in Password Based Login
In this scenario, the mere fact that the user knows the secret password is taken as sufficient proof of the user's identity. Consequently, the security of the website would be compromised if an attacker is able to either obtain or guess the login credentials of another user. 

## Brute-force attacks
A brute-force attack is when an attacker uses a system of trial and error in an attempt to guess valid user credentials. These attacks are typically automated using wordlists of usernames and passwords, and plenty of tools are available for this purpose (Burp Suite is one such tool). 

### Brute-forcing usernames
Often times the attacker knows the username of the victim. But there are cases where he might not know that. So he needs to find the username by brute forcing.  Usernames are especially easy to guess if they conform to a recognizable pattern, such as an email address. For example, it is very common to see business logins in the format `firstname.lastname@somecompany.com` . However, even if there is no obvious pattern, sometimes even high-privileged accounts are created using predictable usernames, such as admin or administrator. 

### Brute-forcing passwords
Passwords can similarly be brute-forced, with the difficulty varying based on the strength of the password. Many websites adopt some form of password policy, which forces users to create high-entropy passwords that are, theoretically at least, harder to crack using brute-force alone. This typically involves enforcing passwords with: 

* A minimum number of characters
* A mixture of lower and uppercase letters
* At least one special character

However, while high-entropy passwords are difficult for computers alone to crack, we can use a basic knowledge of human behavior to exploit the vulnerabilities that users unwittingly introduce to this system. Rather than creating a strong password with a random combination of characters, users often take a password that they can remember and try to crowbar it into fitting the password policy. For example, if `mypassword` is not allowed, users may try something like `Mypassword1!` or `Myp4$$w0rd` instead. 

### Username enumeration
Username enumeration is when an attacker is able to observe changes in the website's behavior in order to
identify whether a given username is valid. 

While attempting to brute-force a login page to find out the username, you should pay particular attention to any differences in:

* **Status codes**: During a brute-force attack, the returned HTTP status code is likely to be the same for the vast majority of guesses because most of them will be wrong. If a guess returns a different status code, this is a strong indication that the username was correct. It is best practice for websites to always return the same status code regardless of the outcome, but this practice is not always followed.

* **Error messages**: Sometimes the returned error message is different depending on whether both the username AND password are incorrect or only the password was incorrect. It is best practice for websites to use identical, generic messages in both cases, but small typing errors sometimes creep in.

* **Response times**: If most of the requests were handled with a similar response time, any that deviate from this suggest that something different was happening behind the scenes. This is another indication that the guessed username might be correct. For example, a website might only check whether the password is correct if the username is valid. This extra step might cause a slight increase in the response time. This may be subtle, but an attacker can make this delay more obvious by entering an excessively long password that the website takes noticeably longer to handle.

### Flawed brute-force protection
Many websites adopt measures to prevent brute force attacks. The two most common ways of preventing brute-force attacks are: 

* Locking the account that the remote user is trying to access if they make too many failed login attempts.
* Blocking the remote user's IP address if they make too many login attempts in quick succession.

Both approaches offer varying degrees of protection, but neither is invulnerable, especially if implemented using flawed logic.

For example, you might sometimes find that your IP is blocked if you fail to log in too many times. In some implementations, the counter for the number of failed attempts resets if the IP owner logs in successfully. This means an attacker would simply have to log in to their own account every few attempts to prevent this limit from ever being reached.
 
##### Account locking
One way in which websites try to prevent brute-forcing is to lock the account if certain suspicious criteria are met, usually a set number of failed login attempts. Just as with normal login errors, responses from the server indicating that an account is locked can also help an attacker to enumerate usernames. 

The following method can be used to work around this kind of protection:

* Establish a list of candidate usernames that are likely to be valid. This could be through username enumeration or simply based on a list of common usernames. 
* Decide on a very small shortlist of passwords that you think at least one user is likely to have. Crucially, the number of passwords you select must not exceed the number of login attempts allowed. For example, if you have worked out that limit is 3 attempts, you need to pick a maximum of 3 password guesses. 
* Using a tool such as Burp Intruder, try each of the selected passwords with each of the candidate usernames. This way, you can attempt to brute-force every account without triggering the account lock. You only need a single user to use one of the three passwords in order to compromise an account.

##### User rate limiting
Another way websites try to prevent brute-force attacks is through user rate limiting. In this case, making too many login requests within a short period of time causes your IP address to be blocked. Typically, the IP can only be unblocked in one of the following ways:

* Automatically after a certain period of time has elapsed
* Manually by an administrator
* Manually by the user after successfully completing a CAPTCHA

User rate limiting is sometimes preferred to account locking due to being less prone to username enumeration and denial of service attacks. However, it is still not completely secure. There are several ways an attacker can manipulate their apparent IP in order to bypass the block. 