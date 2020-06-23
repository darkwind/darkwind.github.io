---
layout: default
title: Queue example - a concurrent web spider
parent: User's guide
grand_parent: Tornado
nav_order: 4
---

# [`Queue`](https://docs.python.org/3.6/library/queue.html#module-queue) example - a concurrent web spider

Tornado의 [`tornado.queues`](http://www.tornadoweb.org/en/stable/queues.html#module-tornado.queues) 모듈은 module은 coroutines에 대한 asynchronous producer / consumer pattern 을 구현합니다.
이는 Python 표준 라이브러리의 [`queue`](https://docs.python.org/3.6/library/queue.html#module-queue) 모듈에 의해 threads 용으로 구현 된 패턴과 유사합니다.

대기열에 항목이 있을 때까지 [`Queue.get`](http://www.tornadoweb.org/en/stable/queues.html#tornado.queues.Queue.get) 을 생성하는 coroutune이 일시 중지됩니다. 큐의 최대 크기가 설정되어 있으면 다른 항목을 넣을 공간이 생길 때까지 [`Queue.put`](http://www.tornadoweb.org/en/stable/queues.html#tornado.queues.Queue.put)을 생성하는 coroutine이 일시 중지됩니다.

[`Queue`](http://www.tornadoweb.org/en/stable/queues.html#tornado.queues.Queue)는 0에서 시작하는 미완료 작업 수를 유지(maintains)합니다. [`put`](http://www.tornadoweb.org/en/stable/queues.html#tornado.queues.Queue.put)은 카운트를 증가시키고, [`task_done`](http://www.tornadoweb.org/en/stable/queues.html#tornado.queues.Queue.task_done)은 감소시킵니다.

아래 web-spider 예제에서 queue는 base_url만 포함하고 시작합니다. 작업자가 페이지를 가져 오면 링크를 구문 분석하고 새 작업을 queue에 넣은 다음 [`task_done`](http://www.tornadoweb.org/en/stable/queues.html#tornado.queues.Queue.task_done)을 호출하여 카운터를 한 번 감소시킵니다. 결국 작업자는 이전에 URL이 가지고 있는 모든 페이지를 가져오고 대기열에는 작업이 남아 있지 않습니다. 따라서 작업자가 [`task_done`](http://www.tornadoweb.org/en/stable/queues.html#tornado.queues.Queue.task_done)을 호출하면 카운터가 0으로 감소합니다.  [`join`](http://www.tornadoweb.org/en/stable/queues.html#tornado.queues.Queue.join) 을 기다리고 있는 메인 coroutine은 일시중지되지 않고 완료됩니다.

```python
#!/usr/bin/env python

import time
from datetime import timedelta

try:
    from HTMLParser import HTMLParser
    from urlparse import urljoin, urldefrag
except ImportError:
    from html.parser import HTMLParser
    from urllib.parse import urljoin, urldefrag

from tornado import httpclient, gen, ioloop, queues

base_url = 'http://www.tornadoweb.org/en/stable/'
concurrency = 10


@gen.coroutine
def get_links_from_url(url):
    """Download the page at `url` and parse it for links.

    Returned links have had the fragment after `#` removed, and have been made
    absolute so, e.g. the URL 'gen.html#tornado.gen.coroutine' becomes
    'http://www.tornadoweb.org/en/stable/gen.html'.
    """
    try:
        response = yield httpclient.AsyncHTTPClient().fetch(url)
        print('fetched %s' % url)

        html = response.body if isinstance(response.body, str) \
            else response.body.decode(errors='ignore')
        urls = [urljoin(url, remove_fragment(new_url))
                for new_url in get_links(html)]
    except Exception as e:
        print('Exception: %s %s' % (e, url))
        raise gen.Return([])

    raise gen.Return(urls)


def remove_fragment(url):
    pure_url, frag = urldefrag(url)
    return pure_url


def get_links(html):
    class URLSeeker(HTMLParser):
        def __init__(self):
            HTMLParser.__init__(self)
            self.urls = []

        def handle_starttag(self, tag, attrs):
            href = dict(attrs).get('href')
            if href and tag == 'a':
                self.urls.append(href)

    url_seeker = URLSeeker()
    url_seeker.feed(html)
    return url_seeker.urls


@gen.coroutine
def main():
    q = queues.Queue()
    start = time.time()
    fetching, fetched = set(), set()

    @gen.coroutine
    def fetch_url():
        current_url = yield q.get()
        try:
            if current_url in fetching:
                return

            print('fetching %s' % current_url)
            fetching.add(current_url)
            urls = yield get_links_from_url(current_url)
            fetched.add(current_url)

            for new_url in urls:
                # Only follow links beneath the base URL
                if new_url.startswith(base_url):
                    yield q.put(new_url)

        finally:
            q.task_done()

    @gen.coroutine
    def worker():
        while True:
            yield fetch_url()

    q.put(base_url)

    # Start workers, then wait for the work queue to be empty.
    for _ in range(concurrency):
        worker()
    yield q.join(timeout=timedelta(seconds=300))
    assert fetching == fetched
    print('Done in %d seconds, fetched %s URLs.' % (
        time.time() - start, len(fetched)))


if __name__ == '__main__':
    io_loop = ioloop.IOLoop.current()
    io_loop.run_sync(main)
```
