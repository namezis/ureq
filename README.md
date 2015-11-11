# ureq
Micro C library for handling HTTP requests on low resource systems. Please note that ureq is still in heavy development and new features are continuously added. Despite this, it behaves very well in stability tests and current user-end interface won't be probably changed.

-------------------------------------------------------------------------------

## How does it work?

ureq is a middleware that sits between any kind of tcp server and the core of your application. Use it to **dispatch urls to functions**, like this: `ureq_serve("/", my_function, UREQ_GET);`. It supports **basic http functionality** and can be used anywhere **system resources are low** (it was built for embedded systems, e.g. ESP8266) or where sophisticated http features aren't needed. And yes, **it understands RESTful**.

## Basic usage

As said before, `ureq` is used for dispatching. **Let's add a couple of urls then**:
```
ureq_serve("/", s_home, UREQ_GET);
ureq_serve("/post", s_post, UREQ_POST);
...
ureq_serve("/all", s_all, UREQ_ALL);

```
How does it work? **When there's a request for an url** (`/`, `/all`, `/post`), **with corresponding method** (`UREQ_GET`, `UREQ_POST`, `UREQ_PUT`, `UREQ_DELETE`,  `UREQ_ALL`), **ureq calls the corresponding function** (`s_home()`, `s_post()`, ...,  `s_all()`). What's `UREQ_ALL` method? It calls a function connected to the url, no matter which type of method was used.

But wait, how should such function look like? The only requirement is that **it has to return a string** that will be sent to a client. For example:

```
char *some_function() {
    return "<h1>Some text!</h1>";
}
```

You want to **parse GET parameters** or do something with **POST data**? No problem, just define your function with a pointer to `HttpRequest`:

```
char *fancy_function(HttpRequest *r) {
    // Do something here, see detailed usage and examples for more info
    return "<h1>Some text!</h1>";
}
```

Keep in mind that adding struct argument to the function's definition is **not obligatory**.

**We connected some urls to functions**, what's next?

We have to **create a structure** that holds all the data and initialize it. And we have to do it per every request. Simply put such line in your program:
```
HttpRequest r = ureq_init(request);
```
`request` in that case is everything you got from the client.

Now let the magic happen. Put such loop in your code:
```
while(ureq_run(&r)) send(req.response.data, req.len);
```
and do all the sending job inside it (`send` is our sending function in that case, remember to replace it with one used by your server backend, `req.response.data` is the response generated by ureq, and `req.len` is its length.

If you wish, do something else with `HttpRequest` now. When you're ready, call
```
ureq_close(&req);
```
To perform a **cleanup**. See `examples/example.c` for basic server-less request parser. See `examples/example-server.c` for basic unix http server.

**That's all.**

-------------------------------------------------------------------------------

Let's summarize:

```
#include "ureq.h"

char *s_home() {
    return "home";
}

int main() {
    ureq_serve("/", s_home, UREQ_GET);

    char request[] = "GET / HTTP/1.1\n"
                     "Host: 127.0.0.1:80\n";

    HttpRequest r = ureq_init(request);
    while(ureq_run(&r)) printf("%s\n", r.response.data);

    ureq_close(&r);
    ureq_finish();

    return 0;
}
```

-------------------------------------------------------------------------------

## Detailed usage
This part of `README` needs to be improved, please treat it as an early draft.

To take **precise control of server's response**, modify **HttpRequest** struct's fields **inside page connected to framework via `ureq_serve`**. Reading `ureq.h` file may provide many useful informations.

Let's take a look at `Response` struct, which is initialized in every request struct:

```
typedef struct ureq_response_t {
    int  code;
    char *mime;
    char *header;
    char *data;
    char *file;
} UreqResponse;
```

Except `*data`, these parameters can be set from your page functions. Some examples:

#### Set response code
```
r->response.code = 302;
```

#### Set custom response header
```
r->response.header = "Location: /";
```

#### Set response mime-type:
```
r->reponse.mime = "text/plain";
```

## Load test
With the help of Reddit and Hacker News users, `ureq` was benchmarked on the `ESP8266`.

![charts](https://solusipse.net/varia/esp8266/esp8266stats.png)

For more data, go to: http://ureq.solusipse.net.

## Roadmap
- minimal templating system
- file sending on unix systems
- todos from comments in files

## License
See `LICENSE`.
