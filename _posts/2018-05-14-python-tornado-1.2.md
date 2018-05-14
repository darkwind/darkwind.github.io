---
title: "Tornado 1.2. User's guide - Asynchronous and non-Blocking I/O"
date: 2018-05-14 12:02:00
layout: post
categories: [python]
tags: [tornado]
---

Asynchronous and non-Blocking I/O
---
실시간 웹 기능을 사용하려면 대부분의 유휴 연결이 오래 있어야합니다. 기존의 동기식 웹 서버에서 이것은 각 사용자에게 하나의 스레드를 할당하는 것을 의미합니다. 이는 매우 비쌉니다.(자원 낭비 / 비효율적).

동시 연결 비용을 최소화하기 위해 Tornado는 single-threaded event loop를 사용합니다. 즉, 한 번에 하나의 작업만 활성화 될 수 있으므로 모든 응용 프로그램 코드는 asynchronous 및 non-blocking을 목표로해야합니다.

asynchronous 및 non-blocking이라는 용어는 서로 밀접하게 관련되어 있으며 종종 같은 의미로 사용되지만 실제로는 동일하지 않습니다.


Blocking
---
function은 리턴되기 전에 어떤 일이 일어날 때까지 기다리는 동안 **차단(blocks)** 됩니다. function은 network I/O, disk I/O, mutexes 등 여러 이유로 차단(blocks) 될 수 있습니다. 실제로 모든 함수는 실행 중이고 CPU를 사용하면서 차단됩니다 (극단적인 예제로 CPU 블로킹이 다른 종류의 블로킹만큼 심각하게 받아 들여지는 이유는 설계 상 수백 밀리 초의 CPU 시간을 사용하는 [bcrypt](http://bcrypt.sourceforge.net/)와 같은 암호 해시 기능을 사용하는 것이 일반적인 네트워크 또는 디스크 액세스보다 훨씬 더 중요하기 때문입니다.)

function은 일부 측면에서는 차단 될 수 있고 다른 부분에서는 차단되지 않을 수 있습니다. Tornado의 기준에서는 모든 종류의 차단이 최소화되어야하지만 일반적으로는 network I/O의 차단에 대해서만 이야기합니다.


Asynchronous
--
비동기 함수는 완료되기 전에 반환되며 일반적으로 응용 프로그램에서 향후 작업을 트리거하기 전에 백그라운드에서 일부 작업이 수행됩니다 (리턴하기 전에 수행 할 모든 작업을 수행하는 일반 동기(**synchronous**) 함수와 다르게). 비동기 인터페이스에는 다양한 스타일이 있습니다.

  * Callback argument
  * Return a placeholder ([`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future), `Promise`, `Deferred`)
  * Deliver to a queue
  * Callback registry (e.g. POSIX signals)

어떤 유형의 인터페이스가 사용되는지에 상관없이 정의에 의한 비동기 함수는 호출자와 다르게 상호 작용합니다. 호출자에 투명하게 비동기식 동기 함수를 만드는 무료 방법은 없습니다. ([gevent](http://www.gevent.org/)와 같은 시스템은 경량 스레드를 사용하여 비동기 시스템에 필적하는 성능을 제공하지만 실제로 비동기로 만들지는 않습니다.)

Examples
---

다음은 동기 함수 샘플입니다.
```python
from tornado.httpclient import HTTPClient

def synchronous_fetch(url):
    http_client = HTTPClient()
    response = http_client.fetch(url)
    return response.body
```

아래는 콜백 인수와 비동기 방식으로 다시 작성된 동일한 함수가 있습니다.
```python
from tornado.httpclient import AsyncHTTPClient

def asynchronous_fetch(url, callback):
    http_client = AsyncHTTPClient()
    def handle_response(response):
        callback(response.body)
    http_client.fetch(url, callback=handle_response)
```

다음은 콜백 대신 [`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future)를 사용합니다.
```python
from tornado.concurrent import Future
from tornado.httpclient import AsyncHTTPClient

def async_fetch_future(url):
    http_client = AsyncHTTPClient()
    my_future = Future()
    fetch_future = http_client.fetch(url)
    fetch_future.add_done_callback(
        lambda f: my_future.set_result(f.result()))
    return my_future
```

[`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future)의 원시 버전은 좀 더 복잡하지만, 그럼에도 `Futures`는 두 가지 큰 장점을 가지고 있기 때문에 Tornado에서는 권장하고 있습니다. `Future.result` 메서드는 간단하게 예외를 발생시킬 수 있기 때문에 오류 처리를 일관성있게 만들 수 있으며(콜백 지향 인터페이스에서 흔히 발생하는 특수 오류 처리(ad-hoc error handling)와는 대조적으로), `Futures`는 coroutine과 함께 사용할 수 있습니다. Coroutines에 대해서는 이 가이드의 다음 섹션에서 자세히 설명합니다. 다음은 원래의 동기 버전과 매우 유사한 coroutine 버전의 샘플 함수입니다:

```python
from tornado import gen

@gen.coroutine
def fetch_coroutine(url):
    http_client = AsyncHTTPClient()
    response = yield http_client.fetch(url)
    raise gen.Return(response.body)
```

`raise gen.Return(response.body)`는 생성자가 값을 반환 할 수 없는 Python 2의 아티팩트입니다. 이 문제를 해결하기 위해 Tornado의 coroutines은 [`Return`](http://www.tornadoweb.org/en/stable/gen.html#tornado.gen.Return) 이라는 특별한 exception을 발생시킵니다. coroutine은 이 예외를 잡아서 반환 값처럼 취급합니다. Python 3.3 이후부터는, `return response.body`가 동일한 결과를 얻습니다.
