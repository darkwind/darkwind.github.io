---
title: "User's guide - Structure of a Tornado web application"
date: 2018-05-14 12:05:00
layout: post
categories: [python]
tags: [tornado, user-guide]
---


Tornado 웹 어플리케이션은 일반적으로 하나 이상의 [`RequestHandler`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler) 서브클래스, 들어오는 요청(requests)을 핸들링하는(처리기로 라우팅하는) [`Application`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.Application) 오브젝트 및 `main()` 함수로 구성되어 서버를 시작합니다.

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

The `Application` object
---
[`Application`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.Application) 오브젝트는 요청을 처리기에 매핑하는 라우팅 테이블을 포함하며 전역 구성을 담당합니다.

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

[`Application`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.Application) 생성자는 응용 프로그램의 동작을 customize하고, optional 기능을 사용할 수있는 많은 키워드 인수를 사용합니다. 전체 목록은 [`Application.settings`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.Application.settings)를 참조하십시오.

Subclassing `RequestHandler`
---
Tornado 웹 응용 프로그램의 대부분의 작업은 [`RequestHandler`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler)의 하위 클래스에서 수행됩니다. handler subclass의 주요 진입점은 HTTP method 다음에 명명된 메소드입니다: get(), post(), etc. 각 핸들러는 서로 다른 HTTP 액션을 처리하기 위해 이러한 메소드 중 하나 이상을 정의 할 수 있습니다. 위에서 설명한 것처럼 이러한 메서드는 일치하는 라우팅 규칙의 캡처 그룹에 해당하는 인수를 사용하여 호출됩니다.

핸들러 내에서 [`RequestHandler.render`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.render) 또는 [`RequestHandler.write`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.write)와 같은 메소드를 호출하여 응답을 생성하십시오. `render()`는 이름으로 [`Template`](http://www.tornadoweb.org/en/stable/template.html#tornado.template.Template)을 로드하고 주어진 인자로 렌더링합니다. `write()`는 템플릿 기반이 아닌 출력에 사용됩니다. strings, bytes, 및 dictionaries를 허용합니다 (dict는 JSON으로 인코딩됩니다).

Many methods in RequestHandler are designed to be overridden in subclasses and be used throughout the application. It is common to define a BaseHandler class that overrides methods such as write_error and get_current_user and then subclass your own BaseHandler instead of RequestHandler for all your specific handlers.
