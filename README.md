# Chalesh [Challenge]

## Introduction
[ArvanCloud](https://www.arvancloud.com/en/) Recruitment Exam (Sep 2018).

- Goal: Add a unique id to error_log for per request context
- The unique id must be an inner defined Nginx variable
- The cost of this operation must be low

## Solution
First define a variable in nginx variables `src/http/ngx_http_variables.c` as follow:

```c
{ ngx_string("arvan_unique_id"), NULL, ngx_http_variable_header,
      offsetof(ngx_http_request_t, headers_in.arvn_unique_id), 0, 0 },
```
