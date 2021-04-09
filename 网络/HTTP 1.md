# 协议格式
## 1. A Start-line 
> Request-Line | Status-Line

### example
> GET /hello.htm HTTP/1.1 (This is Request-Line sent by the client)

> HTTP/1.1 200 OK (This is Status-Line sent by the server)

## 2. Zero or more header fields followed by CRLF

## 3. An empty line 
i.e., a line with nothing preceding the CRLF.
indicating the end of the header fields.

## 4. Optionally a message-body

# 常见报头
## Accept
The Accept request-header field can be used to specify certain media types which are acceptable for the response.

A more elaborate example is

```
Accept: text/plain; q=0.5, text/html,
text/x-dvi; q=0.8, text/x-c
```
Verbally, this would be interpreted as “text/html and text/x-c are the preferred media types, but if they do not exist,
then send the text/x-dvi entity, and if that does not exist, send the text/plain entity.”

## Accept-Charset
The Accept-Charset request-header field can be used to indicate what character sets are acceptable for the response.

## Accept-Encoding
The Accept-Encoding request-header field is similar to Accept, but restricts the content-codings (section 3.5) that are acceptable in the response.

### example
Accept-Encoding: compress, gzip

## Accept-Language
The Accept-Language request-header field is similar to Accept, but restricts the set of natural languages that are preferred as a response to the request.

## Cache-Control
The Cache-Control general-header field is used to specify directives that MUST be obeyed by all caching mechanisms along the request/response chain. 

### public
Indicates that the response MAY be cached by any cache, even if it would normally be non-cacheable or cacheable only within a non-shared cache. 

### private
Indicates that all or part of the response message is intended for a single user and MUST NOT be cached by a shared cache. 

### no-cache
If the no-cache directive does not specify a field-name, then a cache MUST NOT use the response to satisfy a subsequent request without successful revalidation with the origin server.

## Content-Encoding
The Content-Encoding entity-header field is used as a modifier to the media-type. When present, its value indicates what additional content codings have been applied to the entity-body, and thus what decoding mechanisms must be applied in order to obtain the media-type referenced by the Content-Type header field. ContentEncoding is primarily used to allow a document to be compressed without losing the identity of its underlying
media type.

### example
Content-Encoding: gzip

## Content-Length
The Content-Length entity-header field indicates the size of the entity-body, in decimal number of OCTETs, sent to the recipient or, in the case of the HEAD method, the size of the entity-body that would have been sent had the request been a GET.

## Content-Type
The Content-Type entity-header field indicates the media type of the entity-body sent to the recipient or, in the case of the HEAD method, the media type that would have been sent had the request been a GET.

### example
Content-Type: text/html; charset=ISO-8859-4

## Date
The Date general-header field represents the date and time at which the message was originated.

## Expires
The Expires entity-header field gives the date/time after which the response is considered stale. A stale cache entry may not normally be returned by a cache (either a proxy cache or a user agent cache) unless it is first validated with the origin server (or with an intermediate cache that has a fresh copy of the entity).

## Host
The Host request-header field specifies the Internet host and port number of the resource being requested, as obtained from the original URI given by the user or referring resource.

## Last-Modified
The Last-Modified entity-header field indicates the date and time at which the origin server believes the variant
was last modified.

## Referer
The Referer[sic] request-header field allows the client to specify, for the server's benefit, the address (URI) of the resource from which the Request-URI was obtained.

### example
Referer: http://www.w3.org/hypertext/DataSources/Overview.html

## Server
The Server response-header field contains information about the software used by the origin server to handle the request. 

### example
Server: apache

# 常见状态码
## Successful1xx
This class of status code indicates a provisional response, consisting only of the Status-Line and optional headers, and is terminated by an empty line. There are no required headers for this class of status code. 

### 100 Continue
The client SHOULD continue with its request. This interim response is used to inform the client that the initial part of the request has been received and has not yet been rejected by the server. The client SHOULD continue by sending the remainder of the request or, if the request has already been completed, ignore this response. The server MUST send a final response after the request has been completed. 


## Successful 2xx
This class of status code indicates that the client’s request was **successfully** received, understood, and accepted.

### 200 OK
The request has succeeded. 

### 201 Created
The request has been fulfilled and resulted in a new resource being created. The request has been fulfilled and resulted in a new resource being created. 

### 202 Accepted
The request has been accepted for processing, but the processing has not been completed. The request might or might not eventually be acted upon, as it might be disallowed when processing actually takes place. There is no
facility for re-sending a status code from an asynchronous operation such as this.

### 204 No Content
The server has fulfilled the request but does not need to return an entity-body, and might want to return updated metainformation. 


## Redirection 3xx
### 300 Multiple Choices
The requested resource corresponds to any one of a set of representations, each with its own specific location, and agent-driven negotiation information (section 12) is being provided so that the user (or user agent) can select a preferred representation and redirect its request to that location.
### 301 Moved Permanently
The requested resource has been assigned a new permanent URI and any future references to this resource SHOULD use one of the returned URIs.

The new permanent URI SHOULD be given by the Location field in the response. Unless the request method was HEAD, the entity of the response SHOULD contain a short hypertext note with a hyperlink to the new URI(s).

### 302 Found
The requested resource resides temporarily under a different URI. Since the redirection might be altered on occasion, the client SHOULD continue to use the Request-URI for future requests.

The temporary URI SHOULD be given by the Location field in the response.

### 304 Not Modified
If the client has performed a conditional GET request and access is allowed, but the document has not been modified, the server SHOULD respond with this status code. 

### 307 Temporary Redirect
The requested resource resides temporarily under a different URI. Since the redirection MAY be altered on occasion, the client SHOULD continue to use the Request-URI for future requests.

The temporary URI SHOULD be given by the Location field in the response. 


## Client Error 4xx
### 400 Bad Request
The request could not be understood by the server due to malformed syntax. The client SHOULD NOT repeat the request without modifications.
### 401 Unauthorized
The request requires user authentication. 
### 403 Forbidden
The server understood the request, but is refusing to fulfill it. Authorization will not help and the request SHOULD NOT be repeated. If the request method was not HEAD and the server wishes to make public why the request has not been fulfilled, it SHOULD describe the reason for the refusal in the entity. If the server does not wish to make this information available to the client, the status code 404 (Not Found) can be used instead.
### 404 Not Found
The server has not found anything matching the Request-URI. No indication is given of whether the condition is temporary or permanent.

## Server Error 5xx
### 500 Internal Server Error
The server encountered an unexpected condition which prevented it from fulfilling the request.

### 502 Bad Gateway
The server, while acting as a gateway or proxy, received an invalid response from the upstream server it accessed in attempting to fulfill the request.

### 503 Service Unavailable
The server is currently unable to handle the request due to a temporary overloading or maintenance of the server.

### 504 Gateway Timeout
The server, while acting as a gateway or proxy, did not receive a timely response from the upstream server specified by the URI (e.g. HTTP, FTP, LDAP) or some other auxiliary server (e.g. DNS) it needed to access in attempting to complete the request.

# 工具使用
## curl
```
//http post请求设置Host与Content-Type两个报头
curl -d '{"key": "value"}' -H 'Host: www.vivo.com.cn' -H 'Content-Type:application/json'  http://110.23.2.323

//用curl设置cookies
curl 'http://man.linuxde.net' --cookie 'user=root;pass=123456'

//只打印头部信息
curl -I 'http://www.vivo.com.cn'

//打印详细信息
curl -v 'http://www.taobao.com'

//Http指定Host IP访问
curl -H 'Host:www.test.com' http://10.44.54.111/test.php
或
curl -x 10.44.54.111:80 http://www.test.com/test.php

//Https指定Host与IP访问
curl --resolve www.vivo.com.cn:443:183.61.27.136 https://www.vivo.com.cn

curl --resolve www.vivo.com.cn:183.61.27.136 https://www.vivo.com.cn
```


