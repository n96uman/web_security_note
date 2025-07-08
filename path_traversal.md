# what is path traversal 

- directory traversal (path traversal): a vulnerability that allows an attacker to read files on the server that is running the application.

## How to find path traversal

## Black box perspective

- map the application.
- test identified instances with common directory traversal payload.
    ex:- ../../../../etc/passwrd
## white box perspective

- identify the instances where user-supplied input is being passed to file APIs or as parameters to the operating system.