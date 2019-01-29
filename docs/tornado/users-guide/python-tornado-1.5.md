---
layout: default
title: Structure of a Tornado web application
parent: User's guide
grand_parent: Tornado
nav_order: 5
---

# Structure of a Tornado web application

Tornado 웹 어플리케이션은 일반적으로 하나 이상의 [`RequestHandler`][Link_RequestHandler] 서브클래스, 들어오는 요청(requests)을 핸들링하는(처리기로 라우팅하는) [`Application`][Link_Application] 오브젝트 및 `main()` 함수로 구성되어 서버를 시작합니다.

간단한 "hello world" 예제는 다음과 같습니다:

```python
import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello, world")

def make_app():
    return tornado.web.Application([
        (r"/", MainHandler),
    ])

if __name__ == "__main__":
    app = make_app()
    app.listen(8888)
    tornado.ioloop.IOLoop.current().start()
```

**The `Application` object**
---
[`Application`][Link_Application] 오브젝트는 요청을 처리기에 매핑하는 라우팅 테이블을 포함하며 전역 구성을 담당합니다.

라우팅 테이블은 [`URLSpec`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.URLSpec) 오브젝트들이 모여있는 list 거나 tuple 이며, 각각은 (적어도) 정규 표현식(regular expression)과 handler 클래스를 포함합니다. **Order matters**; 첫 번째로 매칭되는 규칙이 사용됩니다. 정규 표현식에 캡처 그룹이 포함되어 있으면이 그룹이 경로 인수이며 핸들러의 HTTP 메소드에 전달됩니다. dictionary가 [`URLSpec`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.URLSpec)의 세 번째 요소로 전달되는 경우 dictionary는 [`RequestHandler.initialize`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.initialize)에 전달되는 초기화 인수를 제공합니다. 마지막으로, [`URLSpec`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.URLSpec)은 [`RequestHandler.reverse_url`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.reverse_url)과 함께 사용될 수 있도록 이름을 가질 수 있습니다.

아래 예제에서 root URL인 `/`은 `MainHandler` 에 맵핑되고 `/story/` 형식 다음에 숫자가 `StoryHandler` 에 맵핑됩니다. 이 숫자는 `StoryHandler.get` 에 (문자열로) 전달됩니다.

```python
class MainHandler(RequestHandler):
    def get(self):
        self.write('<a href="%s">link to story 1</a>' %
                   self.reverse_url("story", "1"))

class StoryHandler(RequestHandler):
    def initialize(self, db):
        self.db = db

    def get(self, story_id):
        self.write("this is story %s" % story_id)

app = Application([
    url(r"/", MainHandler),
    url(r"/story/([0-9]+)", StoryHandler, dict(db=db), name="story")
    ])
```

[`Application`][Link_Application] 생성자는 응용 프로그램의 동작을 customize하고, optional 기능을 사용할 수있는 많은 키워드 인수를 사용합니다. 전체 목록은 [`Application.settings`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.Application.settings)를 참조하십시오.

**Subclassing `RequestHandler`**
---
Tornado 웹 응용 프로그램의 대부분의 작업은 [`RequestHandler`][Link_RequestHandler]의 하위 클래스에서 수행됩니다. handler subclass의 주요 진입점은 HTTP method 다음에 명명된 메소드입니다: get(), post(), etc. 각 핸들러는 서로 다른 HTTP 액션을 처리하기 위해 이러한 메소드 중 하나 이상을 정의 할 수 있습니다. 위에서 설명한 것처럼 이러한 메서드는 일치하는 라우팅 규칙의 캡처 그룹에 해당하는 인수를 사용하여 호출됩니다.

핸들러 내에서 [`RequestHandler.render`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.render) 또는 [`RequestHandler.write`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.write)와 같은 메소드를 호출하여 응답을 생성하십시오. `render()`는 이름으로 [`Template`](http://www.tornadoweb.org/en/stable/template.html#tornado.template.Template)을 로드하고 주어진 인자로 렌더링합니다. `write()`는 템플릿 기반이 아닌 출력에 사용됩니다. strings, bytes, 및 dictionaries를 허용합니다 (dict는 JSON으로 인코딩됩니다).

[`RequestHandler`][Link_RequestHandler]의 여러 메소드는 서브 클래스에서 overridde되고 응용 프로그램 전체에서 사용되도록 설계되었습니다. [`write_error`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.write_error) 및 [`get_current_user`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.get_current_user)와 같은 메서드를 재정의 한 `BaseHandler` 클래스를 정의한 다음 모든 특정 handler에 대해 [`RequestHandler`][Link_RequestHandler] 대신 `BaseHandler`를 서브 클래싱하는 것이 일반적입니다.

**Handling request input**
---
request handler는 `self.request`를 사용하여 현재 요청을 나타내는 객체에 액세스 할 수 있습니다. 전체 속성 목록은 [`HTTPServerRequest`](http://www.tornadoweb.org/en/stable/httputil.html#tornado.httputil.HTTPServerRequest)의 클래스 정의를 참조하십시오.

HTML form을 사용한 request 데이터는 사용자를 위해 파싱되어 [`get_query_argument`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.get_query_argument) 및 [`get_body_argument`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.get_body_argument)와 같은 메소드에서 사용할 수 있습니다.

```python
class MyFormHandler(tornado.web.RequestHandler):
    def get(self):
        self.write('<html><body><form action="/myform" method="POST">'
                   '<input type="text" name="message">'
                   '<input type="submit" value="Submit">'
                   '</form></body></html>')

    def post(self):
        self.set_header("Content-Type", "text/plain")
        self.write("You wrote " + self.get_body_argument("message"))
```

인수가 단일 값인지 또는 요소가 하나인 목록인지 여부에 대해 HTML 인코딩이 모호하기 때문에 [`RequestHandler`][Link_RequestHandler]는 응용 프로그램이 list를 기대하는지 여부를 나타낼 수있는 고유한 메서드를 제공합니다. list를 보려면 하나의 counterpart 대신 [`get_query_arguments`][Link_get_query_arguments] 및 [`get_body_arguments`][Link_get_body_arguments]를 사용하십시오.

Form을 통해 업로드 된 파일은 이름(`<input type="file">` 요소의 name)을 파일 목록을 매핑하는 `self.request.files`에서 사용할 수 있습니다. 각 파일은 `{ "filename": ..., "content_type": ..., "body": ... }` 형식의 dictionary입니다. 파일 객체는 파일이 form wrapper(예: `multipart/form-data` Content-Type)와 함께 업로드 된 경우에만 표시됩니다. 이 형식을 사용하지 않으면 업로드되지 않은 원시 데이터를 `self.request.body`에서 사용할 수 있습니다. 기본적으로 업로드 된 파일은 메모리에 완전히 버퍼링됩니다. 너무 커서 메모리를 편안하게 유지할 수없는 대용량 파일을 처리해야하는 경우 [`stream_request_body`][Link_stream_request_body] 클래스 데코레이터를 참조하십시오.

데모 디렉토리에서 [file_receiver.py][Link_file_receiver.py]는 두 가지 파일 업로드 수신 방법을 보여줍니다.

HTML 양식 인코딩의 단점(예 : 단수 대 복수 인수에 대한 모호함)으로 인해 Tornado는 form 인수를 다른 유형의 입력과 통합하려고 시도하지 않습니다. 특히 JSON request body는 구문 분석하지 않습니다. form-encoding 대신 JSON을 사용하려는 응용 프로그램은 [`prepare`][Link_prepare] 요청을 구문 분석하도록 재정의 할 수 있습니다.

```python
def prepare(self):
    if self.request.headers.get("Content-Type", "").startswith("application/json"):
        self.json_args = json.loads(self.request.body)
    else:
        self.json_args = None
```

**Overriding RequestHandler methods**
---

[`RequestHandler`][Link_RequestHandler]의 다른 메소드는 `get()`/`post()`/etc 외에도 필요한 경우 하위 클래스에 의해 재정의되도록 설계되었습니다. 모든 request에 대해 다음과 같은 일련의 호출이 수행됩니다.

1. 각 요청마다 새로운 [`RequestHandler`][Link_RequestHandler] 객체가 생성됩니다.
2. [`initialize()`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.initialize)는 [`Application`][Link_Application] 구성의 초기화 인수와 함께 호출됩니다. `initialize`는 일반적으로 멤버 변수에 전달 된 인수를 저장해야합니다. [`send_error`][Link_send_error]와 같은 출력이나 호출 메소드를 생성하지 않을 수도 있습니다.
3. [`prepare()`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.prepare) is called. This is most useful in a base class shared by all of your handler subclasses, as prepare is called no matter which HTTP method is used. prepare may produce output; if it calls finish (or redirect, etc), processing stops here.
4. [`prepare()`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.prepare)가 호출됩니다. 이것은 어떤 HTTP 메소드가 사용 되든 상관없이 `prepare`가 호출되기 때문에 모든 핸들러 서브 클래스가 공유하는 기본 클래스에서 가장 유용합니다. `prepare`는 output이 가능합니다. [`finish`][Link_finish] (또는 `redirect` 등)를 호출하면 처리가 여기서 멈춥니다.
5. One of the HTTP methods is called: get(), post(), put(), etc. If the URL regular expression contains capturing groups, they are passed as arguments to this method.
6. When the request is finished, on_finish() is called. For most handlers this is immediately after get() (etc) return; for handlers using the tornado.web.asynchronous decorator it is after the call to finish().

All methods designed to be overridden are noted as such in the [`RequestHandler`][Link_RequestHandler] documentation. Some of the most commonly overridden methods include:

* write_error - outputs HTML for use on error pages.
* on_connection_close - called when the client disconnects; applications may choose to detect this case and halt further processing. Note that there is no guarantee that a closed connection can be detected promptly.
* get_current_user - see User authentication
* get_user_locale - returns Locale object to use for the current user
* set_default_headers - may be used to set additional headers on the response (such as a custom Server header)

**Error Handling**
---

If a handler raises an exception, Tornado will call RequestHandler.write_error to generate an error page. tornado.web.HTTPError can be used to generate a specified status code; all other exceptions return a 500 status.

The default error page includes a stack trace in debug mode and a one-line description of the error (e.g. “500: Internal Server Error”) otherwise. To produce a custom error page, override RequestHandler.write_error (probably in a base class shared by all your handlers). This method may produce output normally via methods such as write and render. If the error was caused by an exception, an exc_info triple will be passed as a keyword argument (note that this exception is not guaranteed to be the current exception in sys.exc_info, so write_error must use e.g. traceback.format_exception instead of traceback.format_exc).

It is also possible to generate an error page from regular handler methods instead of write_error by calling set_status, writing a response, and returning. The special exception tornado.web.Finish may be raised to terminate the handler without calling write_error in situations where simply returning is not convenient.

For 404 errors, use the default_handler_class Application setting. This handler should override prepare instead of a more specific method like get() so it works with any HTTP method. It should produce its error page as described above: either by raising a HTTPError(404) and overriding write_error, or calling self.set_status(404) and producing the response directly in prepare().

**Redirection**
---

There are two main ways you can redirect requests in Tornado: RequestHandler.redirect and with the RedirectHandler.

You can use self.redirect() within a RequestHandler method to redirect users elsewhere. There is also an optional parameter permanent which you can use to indicate that the redirection is considered permanent. The default value of permanent is False, which generates a 302 Found HTTP response code and is appropriate for things like redirecting users after successful POST requests. If permanent is true, the 301 Moved Permanently HTTP response code is used, which is useful for e.g. redirecting to a canonical URL for a page in an SEO-friendly manner.

RedirectHandler lets you configure redirects directly in your Application routing table. For example, to configure a single static redirect:

```python
app = tornado.web.Application([
    url(r"/app", tornado.web.RedirectHandler,
        dict(url="http://itunes.apple.com/my-app-id")),
    ])
```

[`RedirectHandler`][Link_RedirectHandler] also supports regular expression substitutions. The following rule redirects all requests beginning with /pictures/ to the prefix /photos/ instead:

```python
app = tornado.web.Application([
    url(r"/photos/(.*)", MyPhotoHandler),
    url(r"/pictures/(.*)", tornado.web.RedirectHandler,
        dict(url=r"/photos/{0}")),
    ])
```

Unlike RequestHandler.redirect, RedirectHandler uses permanent redirects by default. This is because the routing table does not change at runtime and is presumed to be permanent, while redirects found in handlers are likely to be the result of other logic that may change. To send a temporary redirect with a RedirectHandler, add permanent=False to the RedirectHandler initialization arguments.

**Asynchronous handlers**
---

Certain handler methods (including prepare() and the HTTP verb methods get()/post()/etc) may be overridden as coroutines to make the handler asynchronous.

Tornado also supports a callback-based style of asynchronous handler with the tornado.web.asynchronous decorator, but this style is deprecated and will be removed in Tornado 6.0. New applications should use coroutines instead.

For example, here is a simple handler using a coroutine:

```python
class MainHandler(tornado.web.RequestHandler):
    async def get(self):
        http = tornado.httpclient.AsyncHTTPClient()
        response = await http.fetch("http://friendfeed-api.com/v2/feed/bret")
        json = tornado.escape.json_decode(response.body)
        self.write("Fetched " + str(len(json["entries"])) + " entries "
                   "from the FriendFeed API")
```

For a more advanced asynchronous example, take a look at the chat example application, which implements an AJAX chat room using long polling. Users of long polling may want to override on_connection_close() to clean up after the client closes the connection (but see that method’s docstring for caveats).

[Link_RequestHandler]: http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler
[Link_RedirectHandler]: http://www.tornadoweb.org/en/stable/web.html#tornado.web.RedirectHandler
[Link_Application]: http://www.tornadoweb.org/en/stable/web.html#tornado.web.Application
[Link_get_query_arguments]: http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.get_query_arguments
[Link_get_body_arguments]: http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.get_body_arguments
[Link_stream_request_body]: http://www.tornadoweb.org/en/stable/web.html#tornado.web.stream_request_body
[Link_file_receiver.py]: https://github.com/tornadoweb/tornado/tree/master/demos/file_upload/
[Link_prepare]: http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.prepare
[Link_send_error]: http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.send_error
[Link_finish]: http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.finish