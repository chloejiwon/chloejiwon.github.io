---
title: "python의 process와 thread"
excerpt: "GIL이 뭐야!"

categories:
  - python
tags:
  - python
last_modified_at: 2020-09-20T01:11:00-05:00
---

> python 의 병렬 처리, process와 thread에 대해 알아보자. 그리고 python의 특별한 GIL도.. 🙄

# Process 와 Thread

아주 기초적인 process와 thread의 차이점은 다음과 같다. 

- process : process란 메모리에 적재되어(load) 실행되고 있는 프로그램이다.
- thread : Thread는 process내에서 자원, memory map을 공유하며 동작하는 흐름의 단위이다. 자원을 공유하기 때문에 race condition과 같은 상황(=예를 들어 A thread가 한 전역 변수에 write하려고 하고 B thread도 동일한 전역 변수를 write하려고 할 때. 전역 변수를 예시로 들었으나 여러 가지 모든 자원이 될 수 있다.) 이 생기기 때문에, thread는 lock & unlock, acquire & release하는 동기화가 필수적이다. *동기화 잘못하면 abort나면서 그 프로세스가 죽거나, 어쩔 땐 죽어도 release되지 않아 다음 동작이 실행 안되는 등.... 하여튼 이건 해보면 잘 알듯 😨어쨌든 잘 써야 한다.*

process는 메모리를 새로 할당 받기 때문에 (=각자가 고유한 메모리 맵을 가진다) thread와는 달리 메모리를 많이 사용하며, thread는 light하지만 python 구조 때문에 CPU bound task에서는 multi thread을 이용하는 효과를 못 볼 수 도 있다. 

기본적으로 일 할때 pthread를 많이 사용하곤 하는데 (RTOS 어플리케이션이라 1 프로세스의 멀티 쓰레딩 환경이다. 😐) python은 process / thread를 어떻게 사용하는 지 궁금. 

# python의 threading

python의 threading을 말하려면 꼭 짚고 넘어가야 하는 개념이 있다. 바로 **GIL**이 그것이다. GIL은 Global Interpreter Lock의 줄임말이다. 

## GIL(Global Interpreter Lock)

그럼 GIL이 대체 뭐냐! Python interpreter가 한 thread만, 하나의 bytecode만 실행할 수 있도록 하는 것이다. Global하게 적용되는 lock인 것이다. 😶(당황) GIL은 하나의 thread에게 모든 자원의 점유를 허락하여, 다른 thread는 자원을 acquire하기 전에는 아예 실행조차 될 수 없다. 여기서 당연히 드는 의문. ❓아니 그러면 basically 멀티 쓰레딩이 아니잖아!! 

그 의문점을 가지고(워워..), 왜 python이 GIL을 쓰는지 이유나 들어보자. 일단 알아야 할 것은 python의 모든 것은 객체이다. (여기서 C만 쓰는 나로서는 약간 이해 안되기 시작. 애초에 언어 구조가 다르게 생겨서.. ) 각 Python object는 GC를 위해 reference count를 가지고 있다. reference count는 병렬 프로그래밍에서 흔히 접할 수 있는 (아까 내가 얘기한) 공유 자원 접근 시의 race condition문제를 겪을 수 있다. 그러면 python은 각 개체가 reference count를 접근 할 때마다 개별 mutex를 활용해서 thread-safe하도록 만들기에는 성능 상 손해, deadlock의 위험이 있는 것. 그래서 Interpreter를 그냥 한 thread돌 때 잠궈 버렸다. 🔒GIL로 하나의 thread가 interpreter의 모든 것을 가져간다. GIL때문에 다른 언어와 같이 multi threading 프로그래밍은 제한되었으나 single threading환경에서의 성능은 좋다고 한다. (성능 비교는 안 해봐서 모르지만)

## 그럼 왜 python에서 thread를 써?

이전까지만 읽으면 python 어차피 single thread 환경이라는 건데 왜 여러 thread를 사용하는 지 의문이 든다.  (나 또한..) 그런데 좀만 더 생각해보면 python에서도 multi thread로 구현하면 좋을 케이스가 있다. 프로그램이 하는 일을 생각해 봤을 때 CPU bound task와 I/O bound task로 나눌 수 있다. CPU bound task는 CPU에게 일을 많이 시키는 일이고, I/O bound task는 I/O와 관련된 일인데 입출력, 소리내기 등이 있을 수 있겠다. CPU bound task를 python에서 multi thread로 구현하면 느려진다. 왜냐하면 thread간 lock acquire, release하는 과정에서 context switching 등 오버헤드가 발생하기 때문이다. 차라리 single thread로 만드는 편이 성능이 좋다. 하지만 I/O bound task는 대부분의 시간을 I/O에 할애하기 때문에 그동안은 CPU가 논다! Thread가 interpreter를 점유하다가 I/O작업이 발생하면 interpreter 점유를 release하기 때문에 I/O bound task에서는 multi thread를 쓰는 의미가 있다. 

# python의 process

그럼 이번에는 python의 multi processing에 대해 알아보도록 한다. python은 interpreter 언어로서 순차적으로 동작하기 때문에, 병렬 프로그래밍을 위해선 별도의 모듈인 multi processing모듈을 사용해야 한다. multi processing 모듈을 사용하면 process를 생성하기 때문에 효과적으로 GIL을 피할 수 있다. (python개발자들이 GIL때문에 많이 힘들어 한 듯😂)

예를 들면 multi processing 모듈을 사용하여 이런 식으로 구현 하면 된다. 

```python
from multiprocessing import Process, Queue

def work(id, end, result):
	total = 0	
	for i in range(end):
				total += i
	result.put(total)

if __name__ == '__main__':
	result = Queue()
	p1 = Process(target=work, args=(1, 1000000, result))
	# 생략
	p1.start()
	p1.join()
```

# 마무리

아직 까진 python 에서 프로세싱 이나 threading을 사용할 일이 없었는데 머신 러닝이나 딥 러닝 오픈소스에서 데이터 load나 feature abstract 부분에서 사용하는 것을 볼 수 있었다. 앞으로는 많이 사용하게 될테니 연습해둬야지! 👶🏽

# reference

[[Python] 파이썬 멀티 쓰레드(thread)와 멀티 프로세스(process)](https://monkey3199.github.io/develop/python/2018/12/04/python-pararrel.html)