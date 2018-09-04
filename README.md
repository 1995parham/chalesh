# Chalesh [Challenge]
[![Travis branch](https://img.shields.io/travis/1995parham/chalesh/master.svg?style=flat-square)](https://travis-ci.org/1995parham/chalesh)

## Introduction
[ArvanCloud](https://www.arvancloud.com/en/) Recruitment Challenge (Sep 2018).

- Goal: Add a unique id to `error_log` for per request context
- The unique id must be an inner defined Nginx variable
- The cost of this operation must be low

## Solution (2 Sep)
First define a variable in nginx variables `src/http/ngx_http_variables.c` as follow:

```c
{ ngx_string("arvan_unique_id"), NULL, ngx_http_variable_header,
      offsetof(ngx_http_request_t, headers_in.arvn_unique_id), 0, 0 },
```
Then add variable for it to `ngx_http_headers_in_t` and finally add
its http header line parser to `ngx_http_headers_in`.

```c
ngx_table_elt_t                  *arvn_unique_id;
```

```c
{ ngx_string("Arvan-Unique-ID"),
                 offsetof(ngx_http_headers_in_t, arvn_unique_id),
                 ngx_http_process_header_line },
```

With this solution, clients can provide `Arvan-Unique-ID` in their request
and admins can distinguish their request from the others.

If you want to see this manually created variable in `error_log` you must
change error log format from code and you cannot do this by configuration.
Following code snippet adds `Arvan-Unique-ID` to `error_log` in `ngx_http_log_error_handler`:

```c
    if (r->headers_in.arvn_unique_id) {
        p = ngx_snprintf(buf, len, ", arvn_unique_id: \"%V\"", &r->headers_in.arvn_unique_id->value);
        len -= p - buf;
        buf = p;
    }
```

Samples from `logs/error.log` and `logs/access.log`  present below:

```
2018/09/02 20:28:03 [error] 95138#0: *1 "/usr/local/nginx/html/index.html" is not found (2: No such file or directory), client: 192.168.73.2, server: localhost, request: "GET / HTTP/1.1", arvn_unique_id: "10", host: "192.168.73.4:8080"
```

```
-: 192.168.73.2 - - [02/Sep/2018:20:16:12 +0430] "GET / HTTP/1.1" 404 169 "-" "PostmanRuntime/7.2.0" "-"
10: 192.168.73.2 - - [02/Sep/2018:20:16:25 +0430] "GET / HTTP/1.1" 404 169 "-" "PostmanRuntime/7.2.0" "-"
```

## Solution (4 Sep)
In this new solution we want to print `$request_id` when users do not use `Arvan-Unique-ID` in their
request. In order to doing this we must get `request_id` from request variables and print it in
`ngx_http_log_error_handler` but when `ngx_http_log_error_handler` is called there is no `request_id`
in the variables beacuse we need to index them before use. So I decided to print request object memory
address as unique id :smile:.
