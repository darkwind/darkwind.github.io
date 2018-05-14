---
title: "Tornado 1.1. User's guide - Introduction"
date: 2018-05-14 12:01:00
layout: post
categories: [python]
tags: [tornado]
---

[Tornado](http://www.tornadoweb.org/)는 원래 [FriendFeed](https://en.wikipedia.org/wiki/FriendFeed)에서 개발된 Python 웹프레임워크 및 비동기 네트워킹 라이브러리입니다.  Tornado는 non-blocking network I/O를 사용하여 수만개의 연결을 확장 할 수 있으므로 [long polling](http://en.wikipedia.org/wiki/Push_technology#Long_polling), [WebSockets](http://en.wikipedia.org/wiki/WebSocket) 및 각 사용자에게 long-lived connection이 필요한 기타 응용 프로그램에 이상적입니다.

Tornado는 크게 네 가지 주요 구성 요소로 나눌 수 있습니다.

* 웹프레임워크(웹 응용 프로그램을 작성하기 위해 서브 클래싱 된 [`RequestHandler`](http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler) 및 다양한 지원 클래스).
* HTTP의 클라이언트 및 서버 사이드 구현 ([`HTTPServer`](http://www.tornadoweb.org/en/stable/httpserver.html#tornado.httpserver.HTTPServer) 및 [`AsyncHTTPClient`](http://www.tornadoweb.org/en/stable/httpclient.html#tornado.httpclient.AsyncHTTPClient)).
* [`IOLoop`](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop) 및 [`IOStream`](http://www.tornadoweb.org/en/stable/iostream.html#tornado.iostream.IOStream) 클래스를 포함하는 비동기 네트워킹 라이브러리. HTTP 구성 요소의 기본 요소로 사용되며 다른 프로토콜을 구현하는 데에도 사용할 수 있습니다.
* coroutine 라이브러리 ([`tornado.gen`](http://www.tornadoweb.org/en/stable/gen.html#module-tornado.gen))는 비동기 코드가 연결 콜백보다 간단한 방법으로 작성되도록합니다.

Tornado **웹프레임워크**와 **HTTP서버** 조합은 [WSGI](http://www.python.org/dev/peps/pep-3333/)에 대한 full-stack 대안을 제공합니다. Tornado 웹프레임워크를 WSGI 컨테이너([`WSGIAdapter`](http://www.tornadoweb.org/en/stable/wsgi.html#tornado.wsgi.WSGIAdapter))에서 사용하거나, Tornado HTTP서버를 다른 WSGI 프레임 워크([`WSGIContainer`](http://www.tornadoweb.org/en/stable/wsgi.html#tornado.wsgi.WSGIContainer))의 컨테이너로 사용할 수는 있지만, 이러한 각 조합에는 한계가 있으며 토네이도를 최대한 활용하려면 Tornado의 웹프레임워크와 HTTP서버를 함께 사용해야합니다.
