# what is path traversal 

- directory traversal (path traversal): a vulnerability that allows an attacker to read files on the server that is running the application.

## How to find path traversal

## Black box perspective

- map the application.
- test identified instances with common directory traversal payload.
    ex:- ../../../../etc/passwrd
    if there is filter ones: ....//....//....//etc/passwrd
    from absolute path: /etc/passwrd
    url encoded(n times) to pass the detecting :`%2e%2e%2f`
    Modify the `filename` parameter, giving it the value:`/var/www/images/../../../etc/passwd`
    if there is detector for the file type: `filename=../../../etc/passwd%00.png`
## How to prevent a path traversal attack

The most effective way to prevent path traversal vulnerabilities is to avoid passing user-supplied input to filesystem APIs altogether. Many application functions that do this can be rewritten to deliver the same behavior in a safer way.

If you can't avoid passing user-supplied input to filesystem APIs, we recommend using two layers of defense to prevent attacks:

- Validate the user input before processing it. Ideally, compare the user input with a whitelist of permitted values. If that isn't possible, verify that the input contains only permitted content, such as alphanumeric characters only.
- After validating the supplied input, append the input to the base directory and use a platform filesystem API to canonicalize the path. Verify that the canonicalized path starts with the expected base directory.

Below is an example of some simple Java code to validate the canonical path of a file based on user input:

`File file = new File(BASE_DIRECTORY, userInput); if (file.getCanonicalPath().startsWith(BASE_DIRECTORY)) { // process file }`