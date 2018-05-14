---
title: "Tornado 1.3. User's guide - Coroutines"
date: 2018-05-14 12:03:00
layout: post
categories: [python]
tags: [tornado]
---

Coroutines
===

**Coroutines** 는 Tornado에서 비동기 코드를 작성하는데 권장되는 방법입니다. Coroutines은 Python `yield` 키워드를 사용하여 콜백 체인 대신 실행을 일시 중지했다가 다시 시작합니다. ([gevent](http://www.gevent.org/)와 같은 프레임워크에서 볼 수 있는 협력형 경량 스레드를 coroutines이라고도 하지만, Tornado에서는 모든 coroutines이 명시적 컨텍스트 스위치를 사용하며 비동기 함수라고 합니다).

Coroutines은 동기식 코드만큼이나 간단하지만 스레드를 사용하지 않아도 됩니다. 또한 컨텍스트 스위치가 발생할 수 있는 곳의 수를 줄임으로서 동시성을 추론하기가 더 쉽습니다([make concurrency easier](https://glyph.twistedmatrix.com/2014/02/unyielding.html)).

Example

```python
from tornado import gen

@gen.coroutine
def fetch_coroutine(url):
    http_client = AsyncHTTPClient()
    response = yield http_client.fetch(url)
    # In Python versions prior to 3.3, returning a value from
    # a generator is not allowed and you must use
    #   raise gen.Return(response.body)
    # instead.
    return response.body
```

Python 3.5: `async` and `await`
---

Python 3.5에서는 `async`와 `await` 키워드가 도입되었습니다(이러한 키워드를 사용하는 함수는 "native coroutines"라고 부르기도 합니다). Tornado 4.3 부터는 대부분의 `yield`기반 coroutines 대신 사용할 수 있습니다(제한 사항은 다음 단락을 참조 바랍니다). `@gen.coroutine decorator`를 사용하여 함수를 정의하는 대신 `async def foo()`를 사용하고 `yield` 대신 `await`를 사용하면 됩니다. 이 문서의 나머지 부분에서는 이전 버전의 Python과의 호환성을 위해 여전히 `yield` 스타일을 사용하고 있지만 `async`와 `await`는 사용 가능한 경우 더 빠르게 실행됩니다:

```python
async def fetch_coroutine(url):
    http_client = AsyncHTTPClient()
    response = await http_client.fetch(url)
    return response.body
```

`await` 키워드는 `yield` 키워드보다 다재다능하지 않습니다. 예를 들어 `yield`기반 coroutine에서는 `Futures` 리스트를 생성할 수 있지만 native coroutine에서는 tornado.gen.multi에 리스트를 래핑(wrap)해야합니다. 또한 [`concurrent.futures`](https://docs.python.org/3.6/library/concurrent.futures.html#module-concurrent.futures)와의 통합을 제거합니다. [`tornado.gen.convert_yielded`](http://www.tornadoweb.org/en/stable/gen.html#tornado.gen.convert_yielded)를 사용하여 `yield`로 작업 할 수있는 모든 것을 await와 함께 사용할 수있는 양식으로 변환 할 수 있습니다:

```python
async def f():
    executor = concurrent.futures.ThreadPoolExecutor()
    await tornado.gen.convert_yielded(executor.submit(g))
```

> native coroutines은 특정 프레임워크에서는 보이지 않으며 (예: tornado.gen.coroutine 또는 asyncio.coroutine과 같은 데코레이터를 사용하지 않음) 모든 coroutine은 서로 호환되지 않습니다. coroutine runner가 호출되어 첫 번째 coroutine에 의해 선택되고, 그 다음에 곧바로 호출되는 await를 모든 coroutine이 공유합니다. Tornado coroutine runner는 다용도로 설계되어 모든 프레임 워크에서 기다릴 수있는 객체를 허용합니다. 다른 coroutine runner는 제한 될 수 있습니다 (예 : asyncio coroutine 러너는 다른 프레임 워크의 coroutine을 허용하지 않습니다). 이러한 이유로 복수 프레임 워크를 결합한 응용 프로그램에는 Tornado coroutine runner를 사용하는 것이 좋습니다. 이미 asyncio 러너를 사용중인 coroutine에서 Tornado runner를 사용하여 coroutine을 호출하려면 tornado.platform.asyncio.to_asyncio_future 어댑터를 사용하십시오.

How it works
---

`yield`를 포함하는 함수는 **generator**(생성자) 입니다. 모든 generator는 비동기식입니다. 호출 될 때 완료까지 실행되는 대신 generator 객체를 반환합니다. `@gen.coroutine` decorator는 `yield` 표현식을 통해 generator와 통신하고 [`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future)를 반환하여 coroutine의 호출자와 통신합니다.

다음은 coroutine decorator의 내부 루프의 간소화 버전입니다:

```python
# Simplified inner loop of tornado.gen.Runner
def run(self):
    # send(x) makes the current yield return x.
    # It returns when the next yield is reached
    future = self.gen.send(self.next)
    def callback(f):
        self.next = f.result()
        self.run()
    future.add_done_callback(callback)
```

decorator는 generator에서 [`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future)를 수신하고, 해당 [`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future)가 완료 될 때까지(블로킹없이) 대기 한 다음 [`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future)를 "unwrap"한 다음 결과를 `yield` 표현식(expression)의 결과로 generator로 다시 보냅니다. 대부분의 비동기 코드는 비동기 함수에서 반환 된 [`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future)를 `yield` 표현식에 즉시 전달하는 경우를 제외하고는 [`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future) 클래스를 직접 건드리지 않습니다.


How to call a coroutine
---

Coroutine은 정상적인 방법으로 예외를 발생시키지 않습니다. 제기 된 모든 예외는 그것이 산출 될 때까지 [`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future)에 갇힐 것입니다. 이것은 올바른 방법으로 Coroutine을 호출하는 것이 중요하다는 것을 의미합니다. 그렇지 않으면 오류가 눈에 띄지 않을 수 있습니다:

```python
@gen.coroutine
def divide(x, y):
    return x / y

def bad_call():
    # This should raise a ZeroDivisionError, but it won't because
    # the coroutine is called incorrectly.
    divide(1, 0)
```

거의 모든 경우에 Coroutine을 호출하는 함수는 모두 Coroutine 자체여야하며 호출시 `yield` 키워드를 사용해야합니다. super 클래스에 정의 된 메서드를 재정의하는 경우 문서에서 coroutine이 허용되는지 여부를 확인하십시오 (문서에서 메서드가 "coroutine 일 수 있음"또는 "[`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future)를 반환 할 수 있음"이라고 표시되어 있어야 합니다).

```python
@gen.coroutine
def good_call():
    # yield will unwrap the Future returned by divide() and raise
    # the exception.
    yield divide(1, 0)
```

때로는 결과를 기다리지 않고 coroutine을 "발사하고 잊어 버리기(fire and forget)"를 원할 수 있습니다. 이 경우에는 [`IOLoop`](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop)에서 호출을 담당하게하는 [`IOLoop.spawn_callback`](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop.spawn_callback)을 사용하는 것이 좋습니다. 실패 할 경우 [`IOLoop`](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop)은 스택 추적(stack trace)을 기록합니다.

```python
# The IOLoop will catch the exception and print a stack trace in
# the logs. Note that this doesn't look like a normal call, since
# we pass the function object to be called by the IOLoop.
IOLoop.current().spawn_callback(divide, 1, 0)
```

이 방법으로 [`IOLoop.spawn_callback`](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop.spawn_callback)을 사용하는 것은 `@gen.coroutine`을 사용하는 함수에서는 **권장** 되지만 `async def`를 사용하는 함수에서는 **필수** 입니다.(그렇지 않으면 coroutine runner가 시작되지 않습니다.)

마지막으로, 프로그램의 최상위 레벨에서 *IOLoop* 가 아직 실행되고 있지 않으면 [`IOLoop`](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop)를 시작하고, coroutine을 실행 한 다음 [`IOLoop.run_sync`](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop.run_sync) 메서드로 [`IOLoop`](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop)를 중지 할 수 있습니다. 이것은 배치 지향 프로그램(batch-oriented program)의 main 함수를 시작하는 데 자주 사용됩니다.

```python
# run_sync() doesn't take arguments, so we must wrap the
# call in a lambda.
IOLoop.current().run_sync(lambda: divide(1, 0))
```

Coroutine patterns
---

Calling blocking functions(블로킹 함수 호출하기)
---

The simplest way to call a blocking function from a coroutine is to use IOLoop.run_in_executor, which returns Futures that are compatible with coroutines:

coroutine에서 블로킹 함수를 호출하는 가장 간단한 방법은 coroutine과 호환되는 `Futures`를 반환하는 [`IOLoop.run_in_executor`](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.IOLoop.run_in_executor)를 사용하는 것입니다.

```python
@gen.coroutine
def call_blocking():
    yield IOLoop.current().run_in_executor(blocking_func, args)
```

Parallelism(병렬처리)
---

[`coroutine`](http://www.tornadoweb.org/en/stable/gen.html#tornado.gen.coroutine) 데코레이터는 값이 `Futures`인 lists와 dicts를 인식하고 그 모든 `Futures`를 병렬(parallel)로 기다립니다.

```python
@gen.coroutine
def parallel_fetch(url1, url2):
    resp1, resp2 = yield [http_client.fetch(url1),
                          http_client.fetch(url2)]

@gen.coroutine
def parallel_fetch_many(urls):
    responses = yield [http_client.fetch(url) for url in urls]
    # responses is a list of HTTPResponses in the same order

@gen.coroutine
def parallel_fetch_dict(urls):
    responses = yield {url: http_client.fetch(url)
                        for url in urls}
    # responses is a dict {url: HTTPResponse}
```

Lists 와 dicts 는 `await`와 함께 사용하기 위해 [`tornado.gen.multi`](http://www.tornadoweb.org/en/stable/gen.html#tornado.gen.multi) 로 포장(wrapped)되어야 합니다:

```Python
async def parallel_fetch(url1, url2):
    resp1, resp2 = await gen.multi([http_client.fetch(url1),
                                    http_client.fetch(url2)])
```

Interleaving
---

[`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future)를 즉시 적용(yielding)하는 대신 저장하는 것이 유용하기 때문에 기다리기 전에 다른 작업을 시작할 수 있습니다.

```Python
@gen.coroutine
def get(self):
    fetch_future = self.fetch_next_chunk()
    while True:
        chunk = yield fetch_future
        if chunk is None: break
        self.write(chunk)
        fetch_future = self.fetch_next_chunk()
        yield self.flush()
```

이 패턴은 `@gen.coroutine`에서 가장 유용합니다. 만약 `fetch_next_chunk()`가 `async def`를 사용하면 `fetch_future = tornado.gen.convert_yielded(self.fetch_next_chunk())`로 호출하여 백그라운드 처리를 시작해야만 합니다.

Looping
---

native coroutines에서는 `async for` 를 사용할 수 있습니다. 이전 버전의 Python에서는 `for` 또는 `while` 루프의 반복마다 `yield` 결과를 얻을 수 있는 방법이 없으므로 Looping은 coroutine으로는 까다롭습니다. 대신 [Motor](https://motor.readthedocs.io/en/stable/)의 예제와 같이 루프 조건을 결과에 액세스하는 것과 분리해야합니다:

```Python
import motor
db = motor.MotorClient().test

@gen.coroutine
def loop_example(collection):
    cursor = db.collection.find()
    while (yield cursor.fetch_next):
        doc = cursor.next_object()
```

Running in the background
---

[`PeriodicCallback`](http://www.tornadoweb.org/en/stable/ioloop.html#tornado.ioloop.PeriodicCallback) 은 coroutines에서는 일반적으로 사용되지 않습니다. 대신, coroutine은 `while True:` 루프를 포함하고 [`tornado.gen.sleep`](http://www.tornadoweb.org/en/stable/gen.html#tornado.gen.sleep)를 사용할 수 있습니다:

```Python
@gen.coroutine
def minute_loop():
    while True:
        yield do_something()
        yield gen.sleep(60)

# Coroutines that loop forever are generally started with
# spawn_callback().
IOLoop.current().spawn_callback(minute_loop)
```

때로는 복잡한 루프가 더 바람직 할 수 있습니다. 예제에서 이전 루프는 `60+N`초 마다 실행됩니다. 여기서 `N`은 `do_something()`의 실행 시간입니다. 정확히 60초 마다 실행하려면 위의 Interleaving패턴을 사용하삽시오:

```Python
@gen.coroutine
def minute_loop2():
    while True:
        nxt = gen.sleep(60)   # Start the clock.
        yield do_something()  # Run while the clock is ticking.
        yield nxt             # Wait for the timer to run out.
```
