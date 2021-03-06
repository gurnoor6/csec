# XML External Entity (XXE) Injection
## What is XML External Entity Injection?
XXE is a web security vulnerability that allows an attacker to interfere with an application's processing of XML data. It often allows an attacker to view files on the application server filesystem, and to interact with any backend or external systems that the application itself can access.

## How do XXE Vulnerabilities arise?
XXE vulnerabilities arise because the XML specification contains various potentially dangerous features, and standard parsers support these features even if they are not normally used by the application.

## Types of XXE Attacks
* **Exploiting XXE to retrieve files -** In this type, an external entity is defined containing the contents of a file, and returned in the application's response. To perform an XXE injection attack that retrieves an arbitrary file from the server's filesystem, you need to modify the submitted XML in two ways:
    * Introduce a DOCTYPE element that defines an external entity containing the path to the file.
    * Edit a data value in the XML that is returned in the application's response, to make use of the defined external entity.

    Let us see an example of this attack. Suppose a shopping application checks for the stock level of a product by submitting the following XML to the server:
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <stockCheck><productId>381</productId></stockCheck>
    ```
    The application performs no particular defenses against XXE attacks, so you can exploit the XXE vulnerability to retrieve the /etc/passwd file by submitting the following XXE payload:
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
    <stockCheck><productId>&xxe;</productId></stockCheck>
    ```
    This XXE payload defines an external entity &xxe; whose value is the contents of the /etc/passwd file and uses the entity within the productId value. This causes the application's response to include the contents of the file:
    ```
    Invalid product ID: root:x:0:0:root:/root:/bin/bash
    daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
    bin:x:2:2:bin:/bin:/usr/sbin/nologin
    ```

    You can try this out [here](https://portswigger.net/web-security/xxe/lab-exploiting-xxe-to-retrieve-files)

* **Exploiting XXE to perform SSRF attacks -** Another impact of XXE attacks is that they can be used to perform SSRF (server-side request forgery). This can be a serious vulnerability in which the server-side application can be induced to make HTTP requests to any URL that the server can access. You can read about SSRF attacks [here](https://portswigger.net/web-security/ssrf) before continuing.

    To exploit an XXE vulnerability to perform an SSRF attack, you need to define an external XML entity using the URL that you want to target, and use the defined entity within a data value. If you can use the defined entity within a data value that is returned in the application's response, then you will be able to view the response from the URL within the application's response, and so gain two-way interaction with the backend system. If not, then you will only be able to perform blind SSRF attacks (which can still have critical consequences).

    For example, the following XXE will cause the server to make a back-end HTTP request to an internal system within the organization's infrastructure.
    `<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>`

    You can try this out [here](https://portswigger.net/web-security/xxe/lab-exploiting-xxe-to-perform-ssrf)

* **Blind XXE Vulnerabilities -** When the application does not return the values of any defined external entities in its responses, and so direct retrieval of server-side files is not possible, such an instance of XXE Vulnerability can still be exploited and detected, and is called blind. More advanced techniques are required for this, so we won't go into much detail. You can sometimes use out-of-band techniques to find vulnerabilities and exploit them to exfiltrate data. And you can sometimes trigger XML parsing errors that lead to disclosure of sensitive data within error messages.

    If you wish to read up more about this in detail, visit this [link](https://portswigger.net/web-security/xxe/blind)

* **Finding hidden attack surface for XXE injection -** In some rare cases, the attack surface of XXE vulnerabilities is less visible. If you look in the right places, you will find XXE attack surface in requests that do not contain any XML.
    * **XInclude Attacks -** Some applications receive client-submitted data, embed it on the server-side into an XML document, and then parse the document. An example of this occurs when client-submitted data is placed into a backend SOAP request, which is then processed by the backend SOAP service.

        In this situation, you cannot carry out a classic XXE attack, because you don't control the entire XML document and so cannot define or modify a DOCTYPE element. However, you might be able to use XInclude instead. XInclude is a part of the XML specification that allows an XML document to be built from sub-documents. You can place an XInclude attack within any data value in an XML document, so the attack can be performed in situations where you only control a single item of data that is placed into a server-side XML document.

        To perform an XInclude attack, you need to reference the XInclude namespace and provide the path to the file that you wish to include. For example:
        ```
        <foo xmlns:xi="http://www.w3.org/2001/XInclude">
        <xi:include parse="text" href="file:///etc/passwd"/></foo>
        ```

        You can try this out [here](https://portswigger.net/web-security/xxe/lab-xinclude-attack)

    * **XXE attacks via file upload -** Some applications allow users to upload files which are then processed server-side. Some common file formats use XML or contain XML subcomponents. Examples of XML-based formats are office document formats like DOCX and image formats like SVG.

        For example, an application might allow users to upload images, and process or validate these on the server after they are uploaded. Even if the application expects to receive a format like PNG or JPEG, the image processing library that is being used might support SVG images. Since the SVG format uses XML, an attacker can submit a malicious SVG image and so reach hidden attack surface for XXE vulnerabilities.

        You can try out exploiting XXE via image file upload [here](https://portswigger.net/web-security/xxe/lab-xxe-via-file-upload)

    * **XXE attacks via modified content type -** Most POST requests use a default content type that is generated by HTML forms, such as application/x-www-form-urlencoded. Some web sites expect to receive requests in this format but will tolerate other content types, including XML.

        For example, if a normal request contains the following:
        ```
        POST /action HTTP/1.0
        Content-Type: application/x-www-form-urlencoded
        Content-Length: 7

        foo=bar
        ```
        Then you might be able submit the following request, with the same result:
        ```
        POST /action HTTP/1.0
        Content-Type: text/xml
        Content-Length: 52

        <?xml version="1.0" encoding="UTF-8"?><foo>bar</foo>
        ```
        If the application tolerates requests containing XML in the message body, and parses the body content as XML, then you can reach the hidden XXE attack surface simply by reformatting requests to use the XML format.

## Finding and testing XXE Vulnerabilities
You have two options - manual or automatic. Burp Suite Professional can quickly and reliably find the vast majority of XXE Vulnerabilities for you.

Manually testing for XXE vulnerabilities generally involves:
* Testing for file retrieval by defining an external entity based on a well-known operating system file and using that entity in data that is returned in the application's response.
* Testing for blind XXE vulnerabilities by defining an external entity based on a URL to a system that you control, and monitoring for interactions with that system. Burp Collaborator client is perfect for this purpose.
* Testing for vulnerable inclusion of user-supplied non-XML data within a server-side XML document by using an XInclude attack to try to retrieve a well-known operating system file.

## Preventing XXE Vulnerabilities
Virtually all XXE vulnerabilities arise because the application's XML parsing library supports potentially dangerous XML features that the application does not need or intend to use. The easiest and most effective way to prevent XXE attacks is to disable those features.

Generally, it is sufficient to disable resolution of external entities and disable support for XInclude. This can usually be done via configuration options or by programmatically overriding default behavior. Consult the documentation for your XML parsing library or API for details about how to disable unnecessary capabilities.
