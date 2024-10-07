---
title: 파이썬의 *args와 **kwargs 매개변수
author: kjb4494
date: 2021-02-12 15:00:00 +0900
categories: [Python]
tags: [python, args, kwargs]
---

### 예제

1. 코드

   ```python
   def args_test(*args, **kwargs):
       print(args)
       print(kwargs)

       result = 0
       for arg in args:
           if isinstance(arg, int):
               result += arg
       print(result)

   if __name__ == '__main__':
       args_test(1, 2, 1, 2, 7, '1', '64', 3.14, 9, 3, 6, 8, key1='a', key2='b', etc='c')
   ```

1. 결과
   ```text
   (1, 2, 1, 2, 7, '1', '64', 3.14, 9, 3, 6, 8)
   {'key1': 'a', 'key2': 'b', 'etc': 'c'}
   39
   ```

### 정리

> - \*args

    키워드가 없는 매개변수들의 집합이다.
    for문을 이용해 각 매개변수에 순차적으로 접근할 수 있다.

> - \*\*kwargs

    키워드를 가진 매개변수들의 딕셔너리다.
    key값을 이용해 특정 매개변수에 접근할 수 있다.

기초 중의 기초지만 간단하게 정리해봄 ;)
