# File Upload

## <mark style="color:$primary;">ABOUT</mark>

<mark style="color:red;">**File Upload**</mark> vulnerabilities are when a web server allows users to upload files to its filesystem without sufficiently validating things like their name, type, contents, or size. Failing to properly enforce restrictions on these could mean that even a basic image upload function can be used to upload arbitrary and potentially dangerous files instead. This could even include server-side script files that enable remote code execution.

In some cases, the act of uploading the file is in itself enough to cause damage. Other attacks may involve a follow-up HTTP request for the file, typically to trigger its execution by the server.

The impact of file upload vulnerabilities generally depends on two key factors:

* Which aspect of the file the website fails to validate properly, whether that be its size, type, contents, and so on.
* What restrictions are imposed on the file once it has been successfully uploaded.

## <mark style="color:$primary;">1. Bypassing Extension Filters</mark>

### <mark style="color:blue;">Method: Case Sensitivity</mark>

Even the most exhaustive blacklists can potentially be bypassed using classic obfuscation techniques. Let's say the validation code is case sensitive and fails to recognize that `exploit.pHp` is in fact a .php file. If the code that subsequently maps the file extension to a MIME type is not case sensitive, this discrepancy allows you to sneak malicious PHP files past validation that may eventually be executed by the server.

### <mark style="color:blue;">Method: Multiple Extensions</mark>

You can also achieve similar results using the following techniques:

1. Provide multiple extensions. Depending on the algorithm used to parse the filename, the following file may be interpreted as either a PHP file or JPG image: `exploit.php.jpg`.
2. The code only tests whether the file name contains an image extension; a straightforward method of passing the regex test is through Double Extensions. For example, if the .jpg extension was allowed, we can add it in our uploaded file name and still end our filename with .php (e.g. `shell.jpg.php`), in which case we should be able to pass the whitelist test, while still uploading a PHP script that can execute PHP code. Let's intercept a normal upload request, and modify the file name to (`shell.jpg.php`), and modify its content to that of a web shell.

### <mark style="color:blue;">Method: Recursive Stripping Bypass</mark>

Other defenses involve stripping or replacing dangerous extensions to prevent the file from being executed. If this transformation isn't applied recursively, you can position the prohibited string in such a way that removing it still leaves behind a valid file extension.

1. For example, consider what happens if you strip .php from the following filename: `exploit.p.phphp`.

### <mark style="color:blue;">Method: Trailing Characters</mark>

Add trailing characters.

1. Some components will strip or ignore trailing whitespaces, dots, and suchlike: `exploit.php.`.

### <mark style="color:blue;">Method: URL Encoding</mark>

Try using the URL encoding (or double URL encoding) for dots, forward slashes, and backward slashes.

1. If the value isn't decoded when validating the file extension, but is later decoded server-side, this can also allow you to upload malicious files that would otherwise be blocked: `exploit%2Ephp`.

### <mark style="color:blue;">Method: Null Byte Injection</mark>

Add semicolons or URL-encoded null byte characters before the file extension. If validation is written in a high-level language like PHP or Java, but the server processes the file using lower-level functions in C/C++, for example, this can cause discrepancies in what is treated as the end of the filename.

1. Example: `exploit.asp%00.jpg`
2. Example: `(shell.php%00.jpg)` works with PHP servers with version 5.X or earlier, as it causes the PHP web server to end the file name after the (`%00`), and store it as (`shell.php`), while still passing the whitelist.

A null byte is a control character with a value of zero. In C-based languages (which form the foundation of operating systems and web servers), a "string" is just a sequence of characters in memory. The system needs a special marker to know where the string ends. That marker is the null byte (`\0`). When a C-based function reads a string, it reads one character at a time until it hits this null byte. It then stops, considering everything after it to be part of a different, unrelated piece of data. In your example, `exploit.php%00.jpg` (which becomes `exploit.php\0.jpg`):

1. The PHP Filter (High-Level) reads the _entire_ string and sees that it ends with .jpg.
2. The File System (Low-Level, C-based) reads `exploit.php`, hits the `\0`, and stops. It saves the file with the name `exploit.php`.

### <mark style="color:blue;">Method: Semicolon Injection</mark>

Add semicolons or URL-encoded null byte characters before the file extension.

1. Example: `exploit.asp;.jpg`

### <mark style="color:blue;">Method: Windows Colon Injection (ADS)</mark>

The same may be used with web applications hosted on a Windows server by injecting a colon (`:`) before the allowed file extension.

1. Example: `(e.g. shell.aspx:.jpg)`, which should also write the file as (`shell.aspx`).

### <mark style="color:blue;">Method: General Character Injection</mark>

Finally, let's discuss another method of bypassing a whitelist validation test through Character Injection. We can inject several characters before or after the final extension to cause the web application to misinterpret the filename and execute the uploaded file as a PHP script. The following are some of the characters we may try injecting:

1. `%20`
2. `%0a`
3. `%00`
4. `%0d0a`
5. `/`
6. `.\`
7. `.`
8. `…`
9. `:`

Each character has a specific use case that may trick the web application to misinterpret the file extension. Similarly, each of the other characters has a use case that may allow us to upload a PHP script while bypassing the type validation test.

### <mark style="color:blue;">Method: Multibyte Unicode Characters</mark>

Try using multibyte unicode characters, which may be converted to null bytes and dots after unicode conversion or normalization.

1. Sequences like `xC0 x2E`, `xC4 xAE` or `xC0 xAE` may be translated to `x2E` if the filename parsed as a UTF-8 string, but then converted to ASCII characters before being used in a path.

## <mark style="color:$primary;">2. Bypassing Content & Config Filters</mark>

### <mark style="color:blue;">Method: Content-Type Spoofing</mark>

1. FILE UPLOAD BYPASS TACTIC 1: Content-Type replacement.

### <mark style="color:blue;">Method: Magic Byte Spoofing</mark>

The second and more common type of file content validation is testing the uploaded file's MIME-Type. Multipurpose Internet Mail Extensions (MIME) is an internet standard that determines the type of a file through its general format and bytes structure. This is usually done by inspecting the first few bytes of the file's content, which contain the File Signature or Magic Bytes.&#x20;

{% hint style="info" %}
Also you can add magic bytes by using hexeditor files for checking and file tool to check if it worked
{% endhint %}

1. For example, if a file starts with (`GIF87a` or `GIF89a`), this indicates that it is a GIF image, while a file starting with plaintext is usually considered a Text file.
2. If we change the first bytes of any file to the GIF magic bytes, its MIME type would be changed to a GIF image, regardless of its remaining content or extension.
3. Tip: Many other image types have non-printable bytes for their file signatures, while a GIF image starts with ASCII printable bytes (as shown above), so it is the easiest to imitate.
4. Furthermore, as the string `GIF8` is common between both GIF signatures, it is usually enough to imitate a GIF image.
5. Similarly, certain file types may always contain a specific sequence of bytes in their header or footer. These can be used like a fingerprint or signature to determine whether the contents match the expected type.
6. For example, JPEG files always begin with the bytes `FF D8 FF`.

{% embed url="https://medium.com/@wakedxy/bypassing-file-upload-restriction-using-magic-bytes-ae59fb5bb383" %}

### <mark style="color:blue;">Method: Polyglot Files</mark>

This (magic byte validation) is a much more robust way of validating the file type, but even this isn't foolproof.

1. Using special tools, such as ExifTool, it can be trivial to create a polyglot JPEG file containing malicious code within its metadata.

### <mark style="color:blue;">Method: Overriding Apache Configuration (.htaccess)</mark>

As we discussed in the previous section, servers typically won't execute files unless they have been configured to do so.

1. For example, before an Apache server will execute PHP files requested by a client, developers might have to add the following directives to their /etc/apache2/apache2.conf file:

```apacheconf
LoadModule php_module /usr/lib/apache2/modules/libphp.so
AddType application/x-httpd-php .php
```

1. Many servers also allow developers to create special configuration files within individual directories in order to override or add to one or more of the global settings.
2. Apache servers, for example, will load a directory-specific configuration from a file called `.htaccess` if one is present.

### <mark style="color:blue;">Method: Overriding IIS Configuration (web.config)</mark>

Similarly, developers can make directory-specific configuration on IIS servers using a `web.config` file.

1.  This might include directives such as the following, which in this case allows JSON files to be served to users:<br>

    ```xml
    <staticContent>
    <mimeMap fileExtension=".json" mimeType="application/json" />
    </staticContent>
    ```
2. Web servers use these kinds of configuration files when present, but you're not normally allowed to access them using HTTP requests. However, you may occasionally find servers that fail to stop you from uploading your own malicious configuration file.
3. In this case, even if the file extension you need is blacklisted, you may be able to trick the server into mapping an arbitrary, custom file extension to an executable MIME type.

## <mark style="color:$primary;">3. Exploiting Server-Side Logic</mark>

### <mark style="color:blue;">Server File Handling Basics</mark>

If this file type is non-executable, such as an image or a static HTML page, the server may just send the file's contents to the client in an HTTP response. If the file type is executable, such as a PHP file, and the server is configured to execute files of this type, it will assign variables based on the headers and parameters in the HTTP request before running the script. The resulting output may then be sent to the client in an HTTP response. If the file type is executable, but the server is not configured to execute files of this type, it will generally respond with an error. However, in some cases, the contents of the file may still be served to the client as plain text. Such misconfigurations can occasionally be exploited to leak source code and other sensitive information. You can see an example of this in our information disclosure learning materials.

While it's clearly better to prevent dangerous file types being uploaded in the first place, the second line of defense is to stop the server from executing any scripts that do slip through the net. As a precaution, servers generally only run scripts whose MIME type they have been explicitly configured to execute. Otherwise, they may just return some kind of error message or, in some cases, serve the contents of the file as plain text instead.

### <mark style="color:blue;">Method: Path Traversal</mark>

A directory to which user-supplied files are uploaded will likely have much stricter controls than other locations on the filesystem that are assumed to be out of reach for end users. If you can find a way to upload a script to a different directory that's not supposed to contain user-supplied files, the server may execute your script after all.

1. Web servers often use the `filename` field in `multipart/form-data` requests to determine the name and location where the file should be saved.
2. It is not always necessary to aggressively use Path Traversal many directories back, sometimes one is enough.

### <mark style="color:blue;">Method: Race Conditions (URL-based Uploads)</mark>

Similar race conditions can occur in functions that allow you to upload a file by providing a URL. In this case, the server has to fetch the file over the internet and create a local copy before it can perform any validation. As the file is loaded using HTTP, developers are unable to use their framework's built-in mechanisms for securely validating files. Instead, they may manually create their own processes for temporarily storing and validating the file, which may not be quite as secure. Continue.

1. For example, if the file is loaded into a temporary directory with a randomized name, in theory, it should be impossible for an attacker to exploit any race conditions. If they don't know the name of the directory, they will be unable to request the file in order to trigger its execution.
2. On the other hand, if the randomized directory name is generated using pseudo-random functions like PHP's `uniqid()`, it can potentially be brute-forced.
3. To make attacks like this easier, you can try to extend the amount of time taken to process the file, thereby lengthening the window for brute-forcing the directory name. One way of doing this is by uploading a larger file.
4. If it is processed in chunks, you can potentially take advantage of this by creating a malicious file with the payload at the start, followed by a large number of arbitrary padding bytes.

## <mark style="color:$primary;">4. Alternative Payloads & General Notes</mark>

### <mark style="color:blue;">Attack: Client-Side (Stored XSS)</mark>

Although you might not be able to execute scripts on the server, you may still be able to upload scripts for client-side attacks.

1. For example, if you can upload HTML files or SVG images, you can potentially use `<script>` tags to create stored XSS payloads.
2. If the uploaded file then appears on a page that is visited by other users, their browser will execute the script when it tries to render the page. Note that due to same-origin policy restrictions, these kinds of attacks will only work if the uploaded file is served from the same origin to which you upload it.

### <mark style="color:blue;">Attack: File Format (XXE)</mark>

If the uploaded file seems to be both stored and served securely, the last resort is to try exploiting vulnerabilities specific to the parsing or processing of different file formats.

1. For example, you know that the server parses XML-based files, such as Microsoft Office `.doc` or `.xls` files, this may be a potential vector for XXE injection attacks.

### <mark style="color:blue;">General Notes & Payloads</mark>

1. When you have successfully uploaded your payload, just put your commands after the variable `?cmd=` (ex: `?cmd=ls -la`).
2. `<?php system($_GET["cmd"]);?>`
3. 400 Bad Request this is a classic response to some filters, it may be necessary to bypass them or obfuscate the feed.

