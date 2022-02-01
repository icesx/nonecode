Goaccess
=====

分析access log

## install



```
sudo apt install goaccess
```



## 使用

### 自定义日志
日志定义格式
```
"%h %l %u [%{yyyy-MM-dd HH:mm:ss}t] \"%r\" %s %b \"%{Referer}i\" \"%{User-agent}i\" %D %F"
```
日志样例为

```
172.16.15.1 - - [2020-04-21 10:22:57] "POST /vehicle/getCount HTTP/1.1" 200 90 "-" "Mozilla/5.0 (Linux; Android 9; MI 8 Lite Build/PKQ1.181007.001; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/77.0.3865.92 Mobile Safari/537.36 Html5Plus/1.0" 7 6
172.16.15.1 - - [2020-04-21 10:24:01] "POST /authentication/check?account=17070014588&token=b0ffefae-6bd0-4085-b8e9-b6f5e53d84b2 HTTP/1.1" 200 138 "-" "Mozilla/5.0 (Linux; Android 9; Redmi Note 7 Pro Build/PKQ1.181203.001; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/80.0.3987.99 Mobile Safari/537.36 Html5Plus/1.0" 21 20
```



对应的goaccess命令为

```
goaccess -f access_log.log --log-format='%h %l %^ [%d %t] "%m %U %H" %s %b "%R" "%u" %D %^' --date-format='%Y-%m-%d' --time-format='%H:%M:%S' --agent-list --with-output-resolver
```

%d 对应 --data-format='%Y-%m-%d'

%t 对应 --time-format='%H:%M:%S'

### 在线查看

```
goaccess -f access_log.log --log-format='%h %l %^ [%d %t] "%m %U %H" %s %b "%R" "%u" %D %^' --date-format='%Y-%m-%d' --time-format='%H:%M:%S' --agent-list --with-output-resolver --output=report.html
```

下载report.html到本地就可以查看报告

### nginx 日志

```
goaccess /logs/nginx/access.log  --log-format=COMBINED --output=report.html
```



##  Goaccess 格式规范

`%x` A date and time field matching the time-format and date-format variables. This is used when a timestamp is given instead of the date and time being in two separate variables.

`%t`time field matching the time-format variable.

`%d`date field matching the **date-format** variable.

`%v`The server name according to the canonical name setting (Server Blocks or Virtual Host).

`%e`This is the userid of the person requesting the document as determined by HTTP authentication.

`%h`host (the client IP address, either IPv4 or IPv6)

`%r`The request line from the client. This requires specific delimiters around the request (single quotes, double quotes, etc) to be parsable. Otherwise, use a combination of special format specifiers such as `%m`, `%U`, `%q` and `%H` to parse individual fields.

- **Note:** Use either `%r` to get the full request **OR** `%m`, `%U`, `%q` and `%H` to form your request, do not use both.

`%m`The request method.

`%U`The URL path requested.

- **Note:** If the query string is in `%U`, there is no need to use `%q`. However, if the URL path, does not include any query string, you may use `%q` and the query string will be appended to the request.

`%q`The query string.

`%H`The request protocol.

`%s`The status code that the server sends back to the client.

`%b`The size of the object returned to the client.

`%R`The "Referer" HTTP request header.

`%u`The user-agent HTTP request header.

`%D`The time taken to serve the request, in microseconds.

`%T`The time taken to serve the request, in seconds with milliseconds resolution.

`%L` The time taken to serve the request, in milliseconds as a decimal number.

`%^`Ignore this field.

`%~`Move forward through the log string until a non-space (!isspace) char is found.

`~h`The host (the client IP address, either IPv4 or IPv6) in a X-Forwarded-For (XFF) field.

其中 date-format 格式为 strftime函数的格式，参考地址

`http://man7.org/linux/man-pages/man3/strftime.3.html`

## apache 日志规范

| Format String   | Description                                                  |
| :-------------- | :----------------------------------------------------------- |
| `%%`            | The percent sign.                                            |
| `%a`            | Client IP address of the request (see the `mod_remoteip` module). |
| `%{c}a`         | Underlying peer IP address of the connection (see the `mod_remoteip` module). |
| `%A`            | Local IP-address.                                            |
| `%B`            | Size of response in bytes, excluding HTTP headers.           |
| `%b`            | Size of response in bytes, excluding HTTP headers. In CLF format, *i.e.* a '`-`' rather than a 0 when no bytes are sent. |
| `%{VARNAME}C`   | The contents of cookie VARNAME in the request sent to the server. Only version 0 cookies are fully supported. |
| `%D`            | The time taken to serve the request, in microseconds.        |
| `%{VARNAME}e`   | The contents of the environment variable VARNAME.            |
| `%f`            | Filename.                                                    |
| `%h`            | Remote hostname. Will log the IP address if `HostnameLookups` is set to `Off`, which is the default. If it logs the hostname for only a few hosts, you probably have access control directives mentioning them by name. See [the Require host documentation](http://httpd.apache.org/docs/current/mod/mod_authz_host.html#reqhost). |
| `%{c}h`         | Like `%h`, but always reports on the hostname of the underlying TCP connection and not any modifications to the remote hostname by modules like `mod_remoteip`. |
| `%H`            | The request protocol.                                        |
| `%{VARNAME}i`   | The contents of `VARNAME:` header line(s) in the request sent to the server. Changes made by other modules (e.g. `mod_headers`) affect this. If you're interested in what the request header was prior to when most modules would have modified it, use `mod_setenvif` to copy the header into an internal environment variable and log that value with the `%{VARNAME}e` described above. |
| `%k`            | Number of keepalive requests handled on this connection. Interesting if `KeepAlive` is being used, so that, for example, a '1' means the first keepalive request after the initial one, '2' the second, etc...; otherwise this is always 0 (indicating the initial request). |
| `%l`            | Remote logname (from identd, if supplied). This will return a dash unless `mod_ident` is present and `IdentityCheck` is set `On`. |
| `%L`            | The request log ID from the error log (or '-' if nothing has been logged to the error log for this request). Look for the matching error log line to see what request caused what error. |
| `%m`            | The request method.                                          |
| `%{VARNAME}n`   | The contents of note VARNAME from another module.            |
| `%{VARNAME}o`   | The contents of `VARNAME:` header line(s) in the reply.      |
| `%p`            | The canonical port of the server serving the request.        |
| `%{format}p`    | The canonical port of the server serving the request, or the server's actual port, or the client's actual port. Valid formats are `canonical`, `local`, or `remote`. |
| `%P`            | The process ID of the child that serviced the request.       |
| `%{format}P`    | The process ID or thread ID of the child that serviced the request. Valid formats are `pid`, `tid`, and `hextid`. `hextid` requires APR 1.2.0 or higher. |
| `%q`            | The query string (prepended with a `?` if a query string exists, otherwise an empty string). |
| `%r`            | First line of request.                                       |
| `%R`            | The handler generating the response (if any).                |
| `%s`            | Status. For requests that have been internally redirected, this is the status of the *original* request. Use `%>s` for the final status. |
| `%t`            | Time the request was received, in the format `[18/Sep/2011:19:18:28 -0400]`. The last number indicates the timezone offset from GMT |
| `%{format}t`    | The time, in the form given by format, which should be in an extended `strftime(3)` format (potentially localized). If the format starts with `begin:` (default) the time is taken at the beginning of the request processing. If it starts with `end:` it is the time when the log entry gets written, close to the end of the request processing. In addition to the formats supported by `strftime(3)`, the following format tokens are supported:`sec`number of seconds since the Epoch`msec`number of milliseconds since the Epoch`usec`number of microseconds since the Epoch`msec_frac`millisecond fraction`usec_frac`microsecond fractionThese tokens can not be combined with each other or `strftime(3)` formatting in the same format string. You can use multiple `%{format}t` tokens instead. |
| `%T`            | The time taken to serve the request, in seconds.             |
| `%{UNIT}T`      | The time taken to serve the request, in a time unit given by `UNIT`. Valid units are `ms` for milliseconds, `us` for microseconds, and `s` for seconds. Using `s` gives the same result as `%T` without any format; using `us` gives the same result as `%D`. Combining `%T` with a unit is available in 2.4.13 and later. |
| `%u`            | Remote user if the request was authenticated. May be bogus if return status (`%s`) is 401 (unauthorized). |
| `%U`            | The URL path requested, not including any query string.      |
| `%v`            | The canonical `ServerName` of the server serving the request. |
| `%V`            | The server name according to the `UseCanonicalName` setting. |
| `%X`            | Connection status when response is completed:`X` =Connection aborted before the response completed.`+` =Connection may be kept alive after the response is sent.`-` =Connection will be closed after the response is sent. |
| `%I`            | Bytes received, including request and headers. Cannot be zero. You need to enable `mod_logio` to use this. |
| `%O`            | Bytes sent, including headers. May be zero in rare cases such as when a request is aborted before a response is sent. You need to enable `mod_logio` to use this. |
| `%S`            | Bytes transferred (received and sent), including request and headers, cannot be zero. This is the combination of %I and %O. You need to enable `mod_logio` to use this. |
| `%{VARNAME}^ti` | The contents of `VARNAME:` trailer line(s) in the request sent to the server. |
| `%{VARNAME}^to` | The contents of `VARNAME:` trailer line(s) in the response sent from the server. |

