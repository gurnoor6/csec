# Authentication
In this section, we'll look at some of the most common authentication mechanisms used by websites and discuss potential vulnerabilities in them.

## What is authentication?
Authentication is the process of verifying the identity of a given user or client.

There are three authentication factors into which different types of authentication can be categorized: 
* Something you know, such as a password.
* Something you have, that is, a physical object like a mobile phone or security token.
* Something you are or do, for example, your biometrics or patterns of behavior.

## What is the difference between authentication and authorization?
Authentication is the process of verifying that a user really is who they claim to be, whereas authorization involves verifying whether a user is allowed to do something. 

## How do authentication vulnerabilities arise?
* The authentication mechanisms are weak because they fail to adequately protect against brute-force attacks.
* Logic flaws or poor coding in the implementation allow the authentication mechanisms to be bypassed entirely by an attacker. This is sometimes referred to as "broken authentication". 

## What is the impact of vulnerable authentication?
The impact of authentication vulnerabilities can be very severe. Once an attacker has either bypassed authentication or has brute-forced their way into another user's account, they have access to all the data and functionality that the compromised account has. If they are able to compromise a high-privileged account, such as a system administrator, they could take full control over the entire application and potentially gain access to internal infrastructure. 

* [Password based vulnerabilities](authentication_vulnerabilities/password_based.md)
* [Multi factor authentication based vulnerabilities](authentication_vulnerabilities/multi_factor_based.md)
* [Vulnerabilities in other authentication mechanisms](authentication_vulnerabilities/others.md)




**P.S.-** The material has been taken from [here](https://portswigger.net/web-security/authentication). So if you wish to learn more about any of the above sections or want to gain hands on experience with such attacks, do check it out!! 


