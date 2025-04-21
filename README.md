# Sending HTTP Headers with cURL

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/)

This guide will show you how to effectively use HTTP headers with cURL to improve your data collection and server communication capabilities:

- [Understanding HTTP Headers](#understanding-http-headers)
- [Getting Started with cURL Headers](#getting-started-with-curl-headers)
- [Viewing Default cURL Headers](#viewing-default-curl-headers)
- [Modifying Default Headers with -H](#modifying-default-headers-with--h)
- [Creating Custom Headers](#creating-custom-headers)
- [Working with Empty Headers](#working-with-empty-headers)
- [Deleting Headers](#deleting-headers)
- [Sending Multiple Headers at Once](#sending-multiple-headers-at-once)
- [Summary](#summary)

## Understanding HTTP Headers

Hypertext Transfer Protocol (HTTP) functions as a [stateless protocol](https://byjus.com/gate/difference-between-stateless-and-stateful-protocol/) following a client-server architecture where clients issue requests and await server responses. These requests contain important elements like HTTP method, server location, path, query parameters, and headers.

HTTP headers are essentially key-value pairs that transmit metadata and instructions between clients and servers. They play a crucial role in specifying parameters like content type, caching rules, and authentication methods, ensuring smooth and secure client-server interactions. For [web scraping](https://brightdata.com/blog/how-tos/what-is-web-scraping) operations, HTTP headers enable you to tailor requests by simulating different user agents, managing content negotiation, and handling authentication according to website requirements and protocols.

Common applications of HTTP headers in web scraping include altering the user-agent (UA), specifying response formats, performing conditional requests, and authenticating with application programming interfaces (APIs).

## Getting Started with cURL Headers

Before proceeding with this tutorial, ensure curl is installed on your system by executing this command in your terminal:

```sh
curl --version
```

If installed correctly, you'll receive version information like this:

```
curl 7.55.1 (Windows) libcurl/7.55.1 WinSSL
Release-Date: [unreleased]
Protocols: dict file ftp ftps http https imap imaps pop3 pop3s smtp smtps telnet tftp
Features: AsynchDNS IPv6 Largefile SSPI Kerberos SPNEGO NTLM SSL
```

If you encounter errors such as `curl is not recognized as an internal or external command, operable program or batch file` or `command not found`, you'll need to [install curl](https://curl.se/download.html).

You'll also need a service for examining headers, such as [httpbin.org](http://httpbin.org/), which provides a straightforward HTTP request and response service.

For those familiar with curl, you'll recognize that its basic syntax follows this pattern:

```sh
curl [options] [url]
```

This means that to retrieve content from `mywebpage.com`, you would run:

```sh
curl www.mywebpage.com
```

## Viewing Default cURL Headers

To examine the headers curl sends by default using httpbin.org, execute this command:

```sh
curl http://httpbin.org/headers
```

The response will display the headers sent:

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd2eb0-0617353714d52f3777c9c267"
  }
```

The `Accept`, `Host`, and `User-Agent` headers are included by default in curl requests.

The `Accept` header informs the server about media types the client can process. It communicates which content types the client will accept, facilitating content negotiation between client and server.

An `Accept` header indicating the client prefers JSON looks like:

```
Accept: application/json
```

The `User-Agent` field contains your client information, which in this case is the curl application with its version number (matching your installed version).

The `Host` header identifies the specific web domain (the host) and port number for the HTTP request. When no port is specified, default ports are assumed (port 80 for HTTP and port 443 for HTTPS).

`X-Amzn-Trace-Id` is not a default curl header but indicates your request was routed through an Amazon Web Services (AWS) service, like an AWS load balancer, and can be used for HTTP request tracing.

To confirm which headers curl sends by default, you can use verbose mode with either the `-v` or `--verbose` flag, which shows detailed request and response information, including headers.

Run this command to see default curl headers:

```sh
curl -v http://httpbin.org/headers
```

Your output will resemble:

```
- Trying 50.16.63.240...
* TCP_NODELAY set
* Connected to httpbin.org (50.16.63.240) port 80 (#0)
> GET /headers HTTP/1.1
> Host: httpbin.org
> User-Agent: curl/7.55.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Fri, 22 Mar 2024 07:18:00 GMT
< Content-Type: application/json
< Content-Length: 173
< Connection: keep-alive
< Server: gunicorn/19.9.0
< Access-Control-Allow-Origin: *
< Access-Control-Allow-Credentials: true
<
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd30a8-624365ad52781957578cd5b1"
  }
}
* Connection #0 to host httpbin.org left intact
```

Lines beginning with a greater-than sign (>) show what your client (curl) sent to the endpoint, confirming the following headers were transmitted:

- `GET` (HTTP method) to the endpoint/headers
- `Host` with value `httpbin.org`
- `User-Agent` with value `curl/7.55.1`
- `Accept` with value `*/*`

In the output, lines starting with a less-than sign (<), such as `< Content-Type: application/json`, reflect the response headers.

## Modifying Default Headers with -H

The `-H` or `--header` flag allows you to send custom headers to the server and is useful for testing.

For example, to change the `User-Agent` from `curl/7.55.1` to `Your-New-User-Agent`, use:

```sh
curl -H "User-Agent: Your-New-User-Agent" http://httpbin.org/headers
```

The response will show:

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "Your-New-User-Agent",
    "X-Amzn-Trace-Id": "Root=1-65fd5123-3ebe566a4681427c6996c72c"
  }
}
```

If you want to modify the `Accept` header from `*/*` (which accepts any content type) to `application/json` (which only accepts JSON content), run:

```sh
curl --header "Accept: application/json" http://httpbin.org/headers
```

The output will be:

```json
{
  "headers": {
    "Accept": "application/json",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd55c3-05c21f81770c1c5e6343b1fc"
  }
}
```

> **Note:**
>
> In this example, `--header` was used instead of `-H`. These flags are equivalent and perform the same function.

Since curl version 7.55.0, you can also use a file containing your headers. If your header file is named `header_file`, you can use:

```sh
curl -H @header_file
```

## Creating Custom Headers

Custom headers are developer-defined fields that provide additional information beyond standard HTTP headers.

To send a custom header with curl, use the `-H` flag. For instance, to send a custom header named `My-Custom-Header` with value `Value of custom header`, execute:

```sh
curl -H "My-Custom-Header: Value of custom header" http://httpbin.org/headers
```

The response will be:

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "My-Custom-Header": "Value of custom header",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd7d2a-3b683be160ff2965023b3a31"
  }
}
```

## Working with Empty Headers

Sometimes sending empty headers is necessary, such as when complying with specific API requirements that demand certain headers even without content. For example, the [HTTP Strict Transport Security (HSTS) header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) enforces secure HTTPS connections on websites. While this header typically includes directives about HSTS duration and behavior, sending it with an empty value ensures immediate HSTS enforcement.

Empty headers can also be used to clear previously set headers. To reset or clear a header that was set by default, sending an empty header can effectively remove its value.

To send an empty header with curl, specify the header name followed by a semicolon to indicate an empty value. This command shows how to send an empty custom header called `My-Custom-Header`:

```sh
curl -H "My-Custom-Header;" http://httpbin.org/headers
```

The output shows `My-Custom-Header` with an empty value:

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "My-Custom-Header": "",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd84e2-7a42d9d62a42741e448c426f"
  }
}
```

## Deleting Headers

To completely remove a header with curl, specify the header name followed by a colon with no subsequent value.

For example, to eliminate the default `User-Agent` header, use:

```sh
curl -H "User-Agent:" http://httpbin.org/headers
```

The response will not contain the `User-Agent` header, confirming it was removed:

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "X-Amzn-Trace-Id": "Root=1-65fd862d-13b181583501ae11046374a1"
  }
}
```

## Sending Multiple Headers at Once

So far, we've examined examples with single headers, but curl supports sending multiple headers simultaneously. Simply include multiple `-H` flags in your command.

For instance, to send two headers (`Custom-Header-1` and `Custom-Header-2`) with values `one` and `two` respectively, run:

```sh
curl -H "Custom-Header-1: one" -H "Custom-Header-2: two" http://httpbin.org/headers
```

The output will show:

```json
{
  "headers": {
    "Accept": "*/*",
    "Custom-Header-1": "one",
    "Custom-Header-2": "two",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd8781-143be3502c559bc5605fc6f1"
  }
}
```

## Summary

This article has covered the fundamentals of HTTP headers and demonstrated how to effectively manage them using curl.

For a comprehensive web scraping solution, consider Bright Data. They offer specialized tools and services including [proxy services](https://brightdata.com/proxy-types) that enhance anonymity and prevent IP blocking, as well as [Web Unlocker](https://brightdata.com/products/web-unlocker) to help access geographically restricted content without CAPTCHAs.

Begin your free trial today!