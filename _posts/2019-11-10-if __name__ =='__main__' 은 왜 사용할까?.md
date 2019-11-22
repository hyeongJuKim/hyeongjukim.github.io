---
layout: post
title:  "if __name__ == '__main__' 은 왜 사용할까?"
author: "HyeongJu"
comments: true
---

__name__ 은 모듈의 이름을 저장하는 파이썬 내장 변수이다.  
파이썬 파일은 두 가지 경우로 실행이 될 수 있다.  

1.직접 실행하는 경우
```
__name__ 변수에 문자열 '__main__' 이 할당된다.  
함수가 아닌 0 deps의 코드를 실행.  
```
2.import하는 경우
```
__name__ 변수에 해당 모듈의 이름이 할당.  
함수가 아닌 0 deps의 코드를 실행.  
```

__name__ 을 통해 파이썬 파일이 어떻게 실행되었는지를 알 수가 있다.  
그리고 모듈을 import하면서 원치 않는 코드가 실행 될 수가 있다.  
이것을 방지하기 위해 if __name__ == '__main__' 조건문을 통해 분기처리를 하는 것이다.  

ex) 예제 코드  
1.print_hello.py를 직접 실행  

print_hello.py
``` python
def print_hello:
    print('hello')

print('print_hello.py __name__:', __name__)
print('test hello')
```
실행결과  
``` python
print_hello.py __name__: __main__
test hello
```
__name__ 이라는 변수에는 main 이 담겨져 있고, 0 deps 코드가 실행이 된다.

2.다른 파일에서 import  
main.py
``` python
import print_hello

print('main.py __name__:', __name__)
```
실행결과  
```
print_hello.py __name__: hello
test hello
main.py __main__
```
__name__ 이라는 변수에는 print_hello 가 담겨지고, print_hello.py 를 import 하면서 0 deps 코드를 실행되었다.  

정리  
아래 처럼 조건을 통하여 원하는 분기처리를 하면된다.  
``` python
if __name__ == '__main__':
    print('직접 실행됨')
else:
    print('import되어 실행됨')
```


