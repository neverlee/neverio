title: Nginx request body filter
categories: 小笔记
date: 2016-03-09 18:53:09
tags: [nginx,filter,模块]
description: Nginx的request body filter介绍
---

Nginx中如果需要处理请求体，之后交由其他模块处理，常用的方式有三种：

1. 用一个content除非的handle模块接收处理请求体，处理完成后内部跳转给其他location。比如nginx-upload-module就是采用这种方法。
2. 注册一个rewrite阶段的handle模块接收处理请求体。之后按原nginx模块流程。比如form-input-nginx-module和ngx_json_post_module就是采用这模式
3. 注册一个nginx request body filter，使用类似response body filter的方式进行处理。

当然这三种方法的适用场景并不相同。这里主要介绍方法3，适用于流式处理请求body数据。

首先，可以用类似注册response filter的方式在模块的postconfigure方法中注册request body filter
```c
static ngx_http_request_body_filter_pt  ngx_http_next_request_body_filter;

static ngx_int_t ngx_http_example_post_conf(ngx_conf_t *cf)
{
    // register input filter
    ngx_http_next_request_body_filter = ngx_http_top_request_body_filter;
    ngx_http_top_request_body_filter = ngx_http_example_request_body_filter;

    return NGX_OK;
}

```
之后像编写response body filter一样完成request body filter方法就可以了。当然，这里需要注意的是，如果经过这个filter处理之后改变了原本请求的body的长度，记得一定要在最后修正头部的content length，以免让之后的模块或upstream拿到错误的content length。
```c

static ngx_int_t
ngx_http_example_request_body_filter(ngx_http_request_t *r, ngx_chain_t *in)
{
    // ...

    if (last_buf_in_this_filter) {
        r->headers_in.content_length_n = cry->total_len;
        r->headers_in.content_length->value.data = ngx_palloc(r->pool, NGX_OFF_T_LEN);
        if (r->headers_in.content_length->value.data == NULL) {
            return NGX_HTTP_INTERNAL_SERVER_ERROR;
        }
        r->headers_in.content_length->value.len =
            ngx_sprintf(r->headers_in.content_length->value.data, "%O", r->headers_in.content_length_n)
            - r->headers_in.content_length->value.data;
    }

    return ngx_http_next_request_body_filter(r, in);
}

```

而正常情况（在没有第三模块注册request body filter时），该request body filter的执行顺序是在ngx_http_request_body_length_filter或ngx_http_request_body_chunked_filter之后执行，在这里处理数据不用担心chunked编码的问题，而对应的ngx_http_next_request_body_filter（也就是注册模块前的ngx_http_top_request_body_filter）正好是ngx_http_request_body_save_filter。当然，要注意的，request body filter只有在真正接收请求body时才会被执行，比如在handle里请求接收数据，或者向后端upstream转发请求body时。


> 作者原创，转载请注明出处
