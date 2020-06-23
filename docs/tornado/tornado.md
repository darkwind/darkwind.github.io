---
layout: default
title: Tornado
nav_order: 2
has_children: true
permalink: /docs/tornado
---

# Tornado

Tornado는 원래 FriendFeed에서 개발 된 Python 웹 프레임 워크 및 비동기 네트워킹 라이브러리입니다.
Tornado는 Non-Bloking 네트워크 I/O를 사용하여 수만 개의 연결을 확장 할 수 있으므로 Long polling, WebSocket 및 각 사용자에게 장기간 연결해야하는 기타 응용 프로그램에 이상적입니다.

Hello, world
---
다음은 Tornado를 사용한 간단한 "Hello, world” 예제입니다.
```python
import tornado.ioloopimport tornado.web

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
이 예제는 토네이도의 비동기 기능을 사용하지 않습니다. (비동기는 [채팅 샘플](https://github.com/tornadoweb/tornado/tree/stable/demos/chat) 을 참고하세요.)

Threads and WSGI
---
토네이도는 대부분의 Python 웹프레임워크와 다릅니다. [WSGI](https://wsgi.readthedocs.io/en/latest/)를 기반으로하지 않으며 일반적으로 프로세스당 하나의 스레드만 사용하여 실행됩니다. 비동기 프로그래밍에 대한 Tornado의 접근법에 대한 자세한 내용은 [User 's Guide](http://www.tornadoweb.org/en/stable/guide.html) 를 참조하십시오.
WSGI에 대한 일부 지원은 [`tornado.wsgi`](http://www.tornadoweb.org/en/stable/wsgi.html#module-tornado.wsgi) 모듈 에서 사용할 수 있지만 개발의 초점이 아니며 대부분의 응용 프로그램은 WSGI 대신 직접적 으로 Tornado의 자체 인터페이스(예: [`tornado.web`](http://www.tornadoweb.org/en/stable/web.html#module-tornado.web))를 사용하도록 작성되어야합니다.
일반적으로 Tornado 코드는 스레드로부터 안전하지 않습니다. 토네이도에서 다른 스레드로부터 안전하게 호출 할 수있는 유일한 방법은 [IOLoop.add_callback](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop.add_callback) 입니다. 또한 [IOLoop.run_in_executor](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop.run_in_executor)를 사용해서 다른 스레드에서 비동기 적으로 블로킹 함수를 실행할 수도 있지만 `run_in_executor`에 전달 된 함수는 토네이도 객체를 참조하지 않아야합니다. `run_in_executor`는 블러킹 코드와 상호 작용할 때 권장되는 방법입니다.

Installation
---
```bash
$ pip install tornado
```
Tornado는 [PyPI](http://pypi.python.org/pypi/tornado)에 나열되어 있으며 `pip`를 사용하여 설치할 수 있습니다. 소스 배포판으로 설치시 존재하지 않는 데모 애플리케이션이 포함되어 있으므로 소스 tarball 사본을 다운로드하거나 [git 저장소](https://github.com/tornadoweb/tornado)를 복제 할 수도 있습니다.

Prerequisites: Tornado는 Python 2.7 및 3.4+에서 실행됩니다. Python 2.7.9에서는 `ssl` 모듈의 업데이트가 필요합니다. `pip` 또는 `setup.py install`에 의해 자동으로 설치될 패키지 외에도 다음과 같은 유용한 옵션 패키지가 있습니다.
* [pycurl](http://pycurl.sourceforge.net/)은 `tornado.curl_httpclient` 옵션에 사용됩니다. Libcurl 버전 7.22 이상이 필요합니다.
* [Twisted](http://www.twistedmatrix.com/)는 [`tornado.platform.twisted`](http://www.tornadoweb.org/en/stable/twisted.html#module-tornado.platform.twisted) 클래스와 함께 사용할 수 있습니다.
* [pycares](https://pypi.python.org/pypi/pycares)는 threads가 적절하지 않을 때 사용할 수있는 대체 Non-blocking DNS 확인 프로그램입니다.
* [monotonic](https://pypi.python.org/pypi/monotonic) 또는 [Monotime](https://pypi.python.org/pypi/Monotime)은 monotonic clock에 대한 지원을 추가하여 클록 조정이 빈번한 환경에서 안정성을 향상시킵니다. **Python 3에서는 더 이상 필요하지 않습니다.**

**Platforms:** Tornado는 유닉스와 같은 플랫폼에서 실행되어야합니다. 최고의 성능과 확장성을 위해서는 Linux(with `epoll`)나 BSD(with `kqueue`)가 프로덕션 배포에 권장됩니다(MacOS는 BSD에서 파생되어 kqueue를 지원하지만, 네트워킹 성능은 일반적으로 좋지 않으므로 개발 용도로만 권장됩니다). 이 구성은 공식적으로 지원되지 않으며 개발 용도로만 권장됩니다. Tornado는 Windows에서도 실행됩니다. 하지만 Tornado IOLoop 인터페이스를 재 작업하지 않으면 기본 Tornado Windows IOLoop 구현을 추가하거나 AsyncIO 또는 Twisted와 같은 프레임 워크에서 Windows의 IOCP 지원을 활용할 수 없습니다.(윈도우에서 쓰지 말란 소리)

Documentation(원문 링크)
---
* [User’s guide](http://www.tornadoweb.org/en/stable/guide.html)
  * [Introduction](http://www.tornadoweb.org/en/stable/guide/intro.html)
  * [Asynchronous and non-Blocking I/O](http://www.tornadoweb.org/en/stable/guide/async.html)
  * [Coroutines](http://www.tornadoweb.org/en/stable/guide/coroutines.html)
  * [`Queue` example - a concurrent web spider](http://www.tornadoweb.org/en/stable/guide/queues.html)
  * [Structure of a Tornado web application](http://www.tornadoweb.org/en/stable/guide/structure.html)
  * [Templates and UI](http://www.tornadoweb.org/en/stable/guide/templates.html)
  * [Authentication and security](http://www.tornadoweb.org/en/stable/guide/security.html)
  * [Running and deploying](http://www.tornadoweb.org/en/stable/guide/running.html)
* [Web framework](http://www.tornadoweb.org/en/stable/webframework.html)
  * [`tornado.web` — `RequestHandler` and `Application` classes](http://www.tornadoweb.org/en/stable/web.html)
  * [`tornado.template` — Flexible output generation](http://www.tornadoweb.org/en/stable/template.html)
  * [`tornado.routing` — Basic routing implementation](http://www.tornadoweb.org/en/stable/routing.html)
  * [`tornado.escape` — Escaping and string manipulation](http://www.tornadoweb.org/en/stable/escape.html)
  * [`tornado.locale` — Internationalization support](http://www.tornadoweb.org/en/stable/locale.html)
  * [`tornado.websocket` — Bidirectional communication to the browser](http://www.tornadoweb.org/en/stable/websocket.html)
* [HTTP servers and clients](http://www.tornadoweb.org/en/stable/http.html)
  * [`tornado.httpserver` — Non-blocking HTTP server](http://www.tornadoweb.org/en/stable/httpserver.html)
  * [`tornado.httpclient` — Asynchronous HTTP client](http://www.tornadoweb.org/en/stable/httpclient.html)
  * [`tornado.httputil` — Manipulate HTTP headers and URLs](http://www.tornadoweb.org/en/stable/httputil.html)
  * [`tornado.http1connection` – HTTP/1.x client/server implementation](http://www.tornadoweb.org/en/stable/http1connection.html)
* [Asynchronous networking](http://www.tornadoweb.org/en/stable/networking.html)
  * [`tornado.ioloop` — Main event loop](http://www.tornadoweb.org/en/stable/ioloop.html)
  * [`tornado.iostream` — Convenient wrappers for non-blocking sockets](http://www.tornadoweb.org/en/stable/iostream.html)
  * [`tornado.netutil` — Miscellaneous network utilities](http://www.tornadoweb.org/en/stable/netutil.html)
  * [`tornado.tcpclient` — IOStream connection factory](http://www.tornadoweb.org/en/stable/tcpclient.html)
  * [`tornado.tcpserver` — Basic `IOStream`-based TCP server](http://www.tornadoweb.org/en/stable/tcpserver.html)
* [Coroutines and concurrency](http://www.tornadoweb.org/en/stable/coroutine.html)
  * [`tornado.gen` — Generator-based coroutines](http://www.tornadoweb.org/en/stable/gen.html)
  * [`tornado.locks` – Synchronization primitives](http://www.tornadoweb.org/en/stable/locks.html)
  * [`tornado.queues` – Queues for coroutines](http://www.tornadoweb.org/en/stable/queues.html)
  * [`tornado.process` — Utilities for multiple processes](http://www.tornadoweb.org/en/stable/process.html)
* [Integration with other services](http://www.tornadoweb.org/en/stable/integration.html)
  * [`tornado.auth` — Third-party login with OpenID and OAuth](http://www.tornadoweb.org/en/stable/auth.html)
  * [`tornado.wsgi` — Interoperability with other Python frameworks and servers](http://www.tornadoweb.org/en/stable/wsgi.html)
  * [`tornado.platform.caresresolver` — Asynchronous DNS Resolver using C-Ares](http://www.tornadoweb.org/en/stable/caresresolver.html)
  * [`tornado.platform.twisted` — Bridges between Twisted and Tornado](http://www.tornadoweb.org/en/stable/twisted.html)
  * [`tornado.platform.asyncio` — Bridge between `asyncio` and Tornado](http://www.tornadoweb.org/en/stable/asyncio.html)
* [Utilities](http://www.tornadoweb.org/en/stable/utilities.html)
  * [`tornado.autoreload` — Automatically detect code changes in development](http://www.tornadoweb.org/en/stable/autoreload.html)
  * [`tornado.concurrent` — Work with `Future` objects](http://www.tornadoweb.org/en/stable/concurrent.html)
  * [`tornado.log` — Logging support](http://www.tornadoweb.org/en/stable/log.html)
  * [`tornado.options` — Command-line parsing](http://www.tornadoweb.org/en/stable/options.html)
  * [`tornado.stack_context` — Exception handling across asynchronous callbacks](http://www.tornadoweb.org/en/stable/stack_context.html)
  * [`tornado.testing` — Unit testing support for asynchronous code](http://www.tornadoweb.org/en/stable/testing.html)
  * [`tornado.util` — General-purpose utilities](http://www.tornadoweb.org/en/stable/util.html)
* [Frequently Asked Questions](http://www.tornadoweb.org/en/stable/faq.html)

Discussion and support
---
[Tornado 개발자 메일링 리스트](http://groups.google.com/group/python-tornado)에서 Tornado에 대해 토론하고, [GitHub issue tracker](https://github.com/tornadoweb/tornado/issues)에서 버그를 보고 할 수 있습니다. 추가 자원에 대한 링크는 [Tornado wiki](https://github.com/tornadoweb/tornado/wiki/Links)에서 찾을 수 있습니다. 새로운 릴리즈는 [공지사항 메일링 리스트](http://groups.google.com/group/python-tornado-announce)에서 발표됩니다.

Tornado는 [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html)에서 사용할 수 있습니다.

모든 문서는 [Creative Commons 3.0](http://creativecommons.org/licenses/by/3.0/)에 따라 사용이 허가되었습니다.



{: .fs-6 .fw-300 }