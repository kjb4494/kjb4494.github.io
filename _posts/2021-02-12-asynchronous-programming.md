---
title: 파이썬 비동기 프로그래밍
author: kjb4494
date: 2021-02-12 16:00:00 +0900
categories: [Python]
tags: [python, async]
---

### 동기 vs 비동기 코드 비교

- 동기 프로그래밍

  1. 코드

     ```python
     import requests
     import time


     def get_google_response():
         start = time.time()
         res = requests.get('https://www.google.com')
         return res.status_code, time.time() - start


     def normal_code():
         start = time.time()
         for _ in range(20):
             status_code, taken_time = get_google_response()
             print(status_code, ': ', taken_time)
         print('total: {}'.format(time.time() - start))


     if __name__ == '__main__':
         normal_code()
     ```

  1. 결과
     ```text
     200 :  0.3122532367706299
     200 :  0.34461331367492676
     200 :  0.36156463623046875
     200 :  0.3558688163757324
     200 :  0.35318613052368164
     200 :  0.37262439727783203
     200 :  0.3737161159515381
     200 :  0.3346893787384033
     200 :  0.39562082290649414
     200 :  0.35920214653015137
     200 :  0.35454726219177246
     200 :  0.35430455207824707
     200 :  0.3461928367614746
     200 :  0.3401949405670166
     200 :  0.3195078372955322
     200 :  0.3500397205352783
     200 :  0.3499133586883545
     200 :  0.32646942138671875
     200 :  0.34032320976257324
     200 :  0.3709850311279297
     total: 7.015817165374756
     ```

- 비동기 프로그래밍

  1. 코드

     ```python
     import requests
     import time
     import asyncio


     async def get_google_response(loop):
         start = time.time()
         res = await loop.run_in_executor(None, requests.get, 'https://www.google.com')
         return res.status_code, time.time() - start


     def async_code():
         start = time.time()
         loop = asyncio.get_event_loop()
         tasks = [get_google_response(loop) for _ in range(20)]
         completed_tasks, _ = loop.run_until_complete(asyncio.wait(tasks))
         for done in completed_tasks:
             status_code, taken_time = done.result()
             print(status_code, ': ', taken_time)
         print('total: {}'.format(time.time() - start))
         loop.close()


     if __name__ == '__main__':
         async_code()
     ```

  1. 결과
     ```text
     200 :  0.29973459243774414
     200 :  0.2707829475402832
     200 :  0.2817347049713135
     200 :  0.32273435592651367
     200 :  0.31773948669433594
     200 :  0.3037395477294922
     200 :  0.5270824432373047
     200 :  0.32169365882873535
     200 :  0.3247337341308594
     200 :  0.489962100982666
     200 :  0.2887296676635742
     200 :  0.5300889015197754
     200 :  0.31470251083374023
     200 :  0.2817227840423584
     200 :  0.5450804233551025
     200 :  0.31772708892822266
     200 :  0.2817366123199463
     200 :  0.2977299690246582
     200 :  0.29676270484924316
     200 :  0.3117361068725586
     total: 0.5750386714935303
     ```

---

### 처리 결과를 기다리기 싫다.

위 코드는 구글 사이트에 요청을 보내 응답을 받는 것을 20번 반복해 각 요청-응답 시간과 총 소요시간을 출력하는 코드다.

동기식으로 처리했을 때는 하나의 요청-응답이 끝나야 다음 요청-응답을 진행하기 때문에 총 소요시간이 모든 작업 시간을 합한 값과 비슷하게 나왔다.

하지만 비동기식으로 처리했을 때는 각 요청 작업이 응답을 기다리지 않고 바로 다음 요청 작업을 수행하기 때문에 총 소요시간이 확연하게 줄어든 것을 볼 수 있다.

### 코루틴

```python
async def get_google_response(loop):
    start = time.time()
    res = await loop.run_in_executor(None, requests.get, 'https://www.google.com')
    return res.status_code, time.time() - start
```

파이썬에서 비동기식 프로그래밍을 할 때는 함수 앞에 async를 붙인다. 그래야 이 함수가 코루틴이 된다.

원래 파이썬은 지원하는 특정 함수만 코루틴으로써 사용할 수 있다. 코루틴이 아닌 일반 함수를 코루틴처럼 실행할 수 있게 만드는 것이 loop.run_in_executor() 이다.

```python
loop = asyncio.get_event_loop()
tasks = [get_google_response(loop) for _ in range(20)]
completed_tasks, _ = loop.run_until_complete(asyncio.wait(tasks))
loop.close
```

비동기식 작업에는 기다림이 없고 막히면 다음 작업을 실행한다. 해당 코드는 그런 루프를 만드는 과정이다.

### 키워드를 요구하는 매개변수를 전달해야할 때는?

```python
import requests
import time
import asyncio


async def get_google_response(loop, timeout, stream):
    start = time.time()

    def requests_wrapper():
        return requests.get('https://www.google.com', timeout=timeout, stream=stream)

    res = await loop.run_in_executor(None, requests_wrapper)
    return res.status_code, time.time() - start


def async_code():
    start = time.time()
    loop = asyncio.get_event_loop()
    tasks = [get_google_response(loop, timeout=0.5, stream=True) for _ in range(20)]
    completed_tasks, _ = loop.run_until_complete(asyncio.wait(tasks))
    for done in completed_tasks:
        status_code, taken_time = done.result()
        print(status_code, ': ', taken_time)
    print('total: {}'.format(time.time() - start))
    loop.close()


if __name__ == '__main__':
    async_code()
```

코루틴에 wrapper 함수를 내장해 키워드 매개변수를 전달해주면 된다.
