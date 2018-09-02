# Chalesh [Challenge]
[![Travis branch](https://img.shields.io/travis/1995parham/chalesh/master.svg?style=flat-square)](https://travis-ci.org/1995parham/chalesh)

## Introduction
[ArvanCloud](https://www.arvancloud.com/en/) Recruitment Challenge (Sep 2018).

- Goal: Add a unique id to `error_log` for per request context
- The unique id must be an inner defined Nginx variable
- The cost of this operation must be low

## Solution
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
