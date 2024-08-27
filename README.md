# 2024SummerTIL
# 부스트캠프 챌린지 수료
<img width="684" alt="image" src="https://github.com/user-attachments/assets/653cbeca-2cd7-4cf9-8093-01380ef1dcdb">

# 프로세스의 메모리 구조

기본적으로 프로그램을 실행하는 과정은 다음과 같은 과정으로 진행된다.

1. 메모리에서 실행될 수 있는 바이트코드나 기계어를 메모리 위에 로드한다.(이때 해당 데이터를 Code 영역에 저장)
2. 프로그램이 실행되면서 사용될 전역 함수나 해당 언어에서 지원하는 원시 타입이 아닌 사용할 선언 타입 같은 것들을 미리 저장해 놓는다.(이때 해당 데이터를 Data 영역에 저장)
3. 프로그램의 시작점인 함수가 실행되면서 실행된 함수는 하나의 스택 프레임으로 정의되면서 콜스택에 쌓이게 된다.
4. 스택 프레임에는 해당 함수가 실행되면서 사용될 원시 타입의 데이터, 힙 영역을 통해서 동적할당 받은 메모리 공간에 대한 메모리 주소 참조값 등을 포함한다. 그리고 이러한 것들의 용량이 곧 스택 프레임의 사이즈가 된다.
5. 함수의 실행이 완료되면 콜스택에서 해당 함수에 대한 스택 프레임이 사라진다. 그러면 콜스택은 다음 스택 프레임(바로 밑에 있는 프레임)을 참조하게 된다.
6. 만약 콜스택에 스택 프레임이 더 이상 존재하지 않게되면, 프로세스는 종료하게 된다.

# 왜 프로세스는 콜스택을 스택 자료 구조를 통해서 구현하였을까?

프로세스가 스택 자료 구조를 통해서 콜스택을 통해 함수의 흐름을 제어하는 이유는 기본적으로 함수의 동작과 관련이 있다. 함수는 함수 내부의 함수를 호출할 수 있고, 이러한 내부 함수가 완전히 실행이 종료될 때까지 바깥에 있는 함수는 더 이상 실행을 이어나가면 안된다.(여기서 비동기 함수는 고려하지 않는다. 하지만 따지고 보면, 비동기 함수 또한 **해당 규칙을 잘 지키고 있는 것**이라고 볼 수 있다. ⇒ 비동기 함수는 실행되고 콜스택에서 사라지기 때문.(이 부분에 대해서 나중에 잘 설명할 수 있는 기회가 있다면 좋겠다. 반드시 비동기 함수로 선언됐다 하더라도 콜스택에서 빠져나가는 건 아니기 때문이다.))

이러한 규칙에 의거해서 해당 동작을 가장 잘 지킬 수 있는 것은 스택 자료 구조이다. 스택 자료 구조는 LIFO(Last In First Out ⇒ 나중에 들어간 것이 가장 먼저 나온다.) 구조를 채택하기 때문에, 이러한 함수 흐름을 매우 효율적으로 구현할 수 있기 떄문이다. 

# GC(Garbage Collector)

실행 중인 함수가 종료되고 스택 프레임은 콜스택에서 사라지게 된다. 하지만, 스택 프레임에서 할당받은 Heap 공간을 스택 프레임이 종료될 때 자동으로 해제하진 않는다.(물론 직접 개발자가 메모리 할당을 해제하는 언어도 존재한다. C나… 아무튼 그런 언어도 있다.) 만약 메모리를 해제하지 않으면, 프로세스에서 사용되는 메모리의 양이 기하급수적으로 커질 위험이 존재한다. 그리고 이러한 문제는 해당 프로세스 뿐만 아니라, 다른 프로세스에도 영향을 미칠 수 있다.

따라서 보통 언어들은 가비지 컬렉터라는 자동 메모리 할당 해제 기계를 도입한다. 만약 스택 프레임과 연결되어 있지 않는 메모리 공간이 존재할 경우, 해당 메모리 공간을 해제시킨다. 이것이 기본적으로 GC의 역할이며, GC는 특정한 순간에 실행이 된다. ⇒ 반드시 스택 프레임이 사라질 때마다 GC가 실행되는 것이 아니다!

# CallStack과 Execution Context Stack

<img width="821" alt="1" src="https://github.com/user-attachments/assets/40f27965-e046-44a3-a7ec-492a8a99a4d5">


같은 거니까 혼용해서 사용하지 말자.(정확히 말하면 같진 않지만 똑같은 역할을 수행한다.) ECMAScript에서는 Execution context stack이라고 하는데, 결과적으로 CallStack이다. 그리고 running execution context는 현재 참조되고 있는 스택 프레임을 말한다.

<img width="831" alt="2" src="https://github.com/user-attachments/assets/3d44a6fa-a32b-4b29-99fe-3c807aaf989c">


# V8 Engine의 프로세스 메모리 구조

![3](https://github.com/user-attachments/assets/0f55c691-cbd7-4c34-9739-ca6bdbafb6ba)


V8 Engine은 우리가 운영체제 수업에서 흔히 들었던 Code와 Data영역이 존재하지 않는다.

Data 영역은 Call Stack의 Global frame이라는 이름으로 편입되었고, Code 영역은 힙 메모리 영역에 편입되어 있다.

- Stack(Call Stack): 운영체제에서 배웠던 Stack 영역에 대한 역할과 동일하다. 단지, Data 영역을 Stack에서 관리함으로써 Global frame이라는 프레임이 Call Stack에 추가된다.
- Heap: 운영체제에서 배운 Heap 영역에 대한 역할과 동일하지만, 기본적으로 Code 영역에 대한 부분이 Heap 영역에 포함된다.
    - Young generation:
    - Old generation:
    - Large object space: young generation 또는 old generation에서도 저장되지 못할 정도로 큰 객체를 보관하는 장소이다. 해당 부분은 GC의 영향을 받지 않는다.
    - Code space: 운영체제에서 배웠던 Code 영역이 Heap 영역으로 편입된 것이다. Javascript가 JIT 컴파일러 방식으로 코드를 실시간으로 바이트 코드로 바꾸기 때문에, 실시간으로 컴파일을 해야 하는 특성상, Heap 영역에 저장하는 것이다. 또한 Code가 반드시 Code space에 저장되는 것이 아닌, Large object space에 저장될 수 있는데 이 경우는 매우 큰 바이트 코드를 저장해 Code space 영역에 대한 제한을 초과하는 경우 발생한다.
    - Cell space, property cell space, map space: 아직도 정확한 용도를 못 찾았습니다.

# V8 Engine의 GC

# 다양한 GC 처리 방식

## Parallel

<img width="796" alt="4" src="https://github.com/user-attachments/assets/b9df6da3-7397-4cee-ba83-16eff5b8a9d9">


## Incremental

<img width="771" alt="5" src="https://github.com/user-attachments/assets/0d35f15a-8009-463b-810c-676650dd9bf9">


## Concurrent

<img width="768" alt="6" src="https://github.com/user-attachments/assets/88a095e5-c7a4-4dd5-92d1-39c28840324d">


# V8 Engine의 GC 처리 방식

## Minor GC

<img width="762" alt="7" src="https://github.com/user-attachments/assets/058f99b2-39c3-4bfc-8f11-2944b1d74397">


## Major GC

<img width="770" alt="8" src="https://github.com/user-attachments/assets/74ec05c9-ad21-4644-b71b-736339f2bcd0">


# V8 Engine의 전반적인 GC 처리 방식(Idle-time GC)

<img width="783" alt="9" src="https://github.com/user-attachments/assets/e2af068f-63e0-4fe7-bc00-a7e4918ccc7b">


[Idle-Time Garbage-Collection Scheduling - ACM Queue](https://queue.acm.org/detail.cfm?id=2977741)

# 자바스크립트에서의 메모리 누수 방지법

- 전역 변수 잘 쓰지 않기(알파이자 오메가)
- console 함수 사용하지 않기(실제 프로덕션 환경에서는 사용 금지, 개발 환경에서는 OK!)
    - 왜 그런지는 다음과 같은 링크를 참조해 봤는데 아직 더 공부해야 할 것 같습니다!(확인되는 대로 갱신 예정) 하지만 제 추측으로는 브라우저(또는 Console 객체일 수도 있습니다.)에서 출력 객체에 대한 메모리 값을 참조하기 때문에 일어나는 문제가 아닐까 하고 생각해봤습니다.
    
    [console: log() static method - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/console/log_static)
    
    [console - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/console)
    
    [Console Standard](https://console.spec.whatwg.org/#printer)
    
- 사용한 타이머 잘 해제하기

# 출처

[Call stack](https://en.wikipedia.org/wiki/Call_stack)

[🚀 Demystifying memory management in modern programming languages](https://deepu.tech/memory-management-in-programming/)

[🚀 Visualizing memory management in V8 Engine (JavaScript, NodeJS, Deno, WebAssembly)](https://deepu.tech/memory-management-in-v8/)

[Avoiding Memory Leaks in Node.js: Best Practices for Performance | AppSignal Blog](https://blog.appsignal.com/2020/05/06/avoiding-memory-leaks-in-nodejs-best-practices-for-performance.html)

[Trash talk: the Orinoco garbage collector · V8](https://v8.dev/blog/trash-talk)

[당신이 모르는 자바스크립트의 메모리 누수의 비밀](https://ui.toast.com/weekly-pick/ko_20210611)

[Idle-Time Garbage-Collection Scheduling - ACM Queue](https://queue.acm.org/detail.cfm?id=2977741)

# 브라우저의 이벤트 루프

## 정의

이벤트 루프는 ‘자바스크립트는 싱글 스레드를 기반으로 동작한다.’라고 불리게 하는 언어의 핵심이며, 프로그램의 렌더링 주기와 콜스택과 밀접한 관련이 있다.

Html Spec에서는 이벤트 루프에 대한 뜻을 명확히 하지 않지만 어디서 사용하는지 나타낸다.

> to coordinate events, user interaction, scripts, rendering, networking, and so forth, user agents must use event loops.

이벤트, 사용자의 상호작용, 스크립트, 렌더링, 네트워킹 등을 조정하려면 유저 에이전트는 이벤트를 루프를 반드시 사용해야 한다.
> 

여기서 에이전트란 ECMAScript에 정의되어 있는데 그 정의는 다음과 같다. 

> An agent comprises a set of ECMAScript execution contexts, an excution context stack, a running execution context

에이전트는 ECMAScript의 실행 컨텍스트, 실행 컨텍스트 스택, 동작 중인 실행 컨텍스트를 하나로 통합한 것이다.
> 

다시 말하면 에이전트는 ECMAScript 를 실행하는 실행 컨텍스트를 말하며 그 외의 실행에 필요한 스택 등을 통합하여 부르는 말이다.

자바스크립트가 ECMAScript의 사양을 따르고 있는 언어라는 것을 알고 있다면, 에이전트는 자바스크립트에서 얼마나 중요한 역할을 하는 지 알 수 있다.

다시 말하면 이벤트 루프는 에이전트가 자바스크립트에서 자신의 이벤트, 사용자의 상호작용, 스크립트, 렌더링, 네트워크를 적절히 조정할 때, 사용되는 매커니즘이라고 볼 수 있다.

## 이벤트 루프의 종류

이벤트 루프는 2가지 종류가 있는데, window event loop, worker event loop가 있다.

### window event loop

Window 객체(브라우저의 전역 객체는 Window 객체이다.)가 있는 브라우저에서 사용되는 이벤트 루프를 말한다. 다른 말로 본다면, 브라우저에서 일어나는 대부분의 일(키 입력, 네트워크 요청 등)은 해당 이벤트 루프에서 처리된다고 볼 수 있다. 

Node.js에서의 전역 객체를 보고 싶다면, 다음 자료를 보는 것을 추천한다.

[전역 객체](https://www.notion.so/43c0cc596aa844558f93389ff9bf6703?pvs=21)

### worker event loop

web worker나 service worker에서 사용되는 이벤트 루프를 말하며, 해당 이벤트 루프를 통해서 각 워커들은 비동기 작업을 처리할 때 사용한다. 다시 말하면 워커 스레드는 각 스레드마다 자신만의 이벤트 루프를 하나씩 가지고 있다.

자 그럼 보통 자바스크립트에서 사용되는 Node.js 런타임 환경에 대한 이벤트 루프는 어떠한 종류의 이벤트 루프일까?

아쉽게도 이 둘 어디에도 해당되지 않는다.  Node.js는 Html Spec에 적혀 있는 이벤트 루프의 두 가지 종류를 정확히 따르지 않고 자신만의 이벤트 루프를 구축해서 사용한다.

하지만 결과적으로 Html Spec에 정의되어 있는 요소를 크게 벗어나지 않게 구현되어 있다.(추후에 따로 깊게 공부해 보는 게 좋을 거 같다.)

[Node.js — The Node.js Event Loop](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick)

## 이벤트 루프의 구성 요소

Html Spec을 살펴보면 이벤트 루프는 하나 이상의 태스크 큐를 가지고 있다. 태스크 큐는 태스크들을 모아둔 집합을 말한다. 여기서 중요한 것은 큐가 아니라 set으로 구현되어 있다는 것이다. set으로 구현되어 있는 이유는 이벤트 루프의 프로세싱 모델이 첫 번째 동작가능한 태스크를 선택된 큐로부터 정보를 받아오는 것이지, 빼는 방식이 아니기 때문이다.

그리고 이벤트 루프는 그렇게 정보를 받아온 것을 저장하고 있다가, 콜스택이 빈 것을 확인하면 해당 정보를 콜스택에 넘겨주는 역할을 한다. 그리고 가지고 있던 정보를 비우는데, 이 때 다시 한 번 여러 개의 태스크 큐 중 하나를 선택해서 태스크를 가져온다.

또한 이벤트 루프는 마이크로태스크 큐도 가지고 있는데, 마이크로태스크 큐는 그냥 태스크 큐와 다르며, 여러 개를 가질 수 있는 태스크 큐와 달리 마이크로태스크 큐 하나만 가질 수 있다.(마이크로태스크 큐와 태스크 큐에서 보관되는 태스크는 나중에 설명한다.)

이것 외에도 마이크로태스크 큐와 관련된 마이크로태스크 체크포인트라는 boolean 타입의 프로퍼티 또한 가지고 있다. true일 경우 마이크로태스크 큐에서 작업을 가져와 실행하고 있다는 것을 나타내며, 한 개의 마이크로태스크 작업이 끝나기 전까지 마이크로태스크 큐에서 계속해서 태스크를 가져오는 것을 방지한다. 이러한 변수가 필요한 이유는 마이크로태스크 큐에 저장된 태스크들을 처리하는 방식에서 확인할 수 있는데, 이벤트 루프는 한 주기(브라우저에서는 16ms의 주기를 가지고 동작한다.)에 마이크로태스크 큐와 태스크 큐를 관찰하며, 콜스택이 비어있는지 확인 후 콜스택이 비어있다면 마이크로태스크 큐 → 태스크 큐의 우선 순위로 태스크를 꺼내와 작업을 하기 때문이다. 이때, 한 주기에서 마이크로태스크 큐는 큐가 비어질 때까지 태스크를 가져오고 처리하는 방식이며, 태스크 큐는 하나의 태스크만 꺼내와서 처리한다.

즉, 이벤트 루프는 한 주기에 마이크로태스크 큐 → 큐가 비워질 때까지 이벤트 루프가 태스크를 가져와서 처리, 태스크 큐 → 선택된 태스크 큐에서 태스크 하나만 꺼내서 처리를 한다.

이벤트 루프는 한 주기(브라우저에서는 16ms의 주기를 가지고 동작한다.)에 마이크로태스크 큐와 태스크 큐를 관찰하며, 콜스택이 비어있는지 확인 후, 콜스택이 비어있다면 태스크를 큐에서 꺼내와 콜스택에 올린다.

이것 외에도 이벤트 루프 종류별로 추가적으로 가지는 요소가 있는데, window event loop는 DOMHighResTimeStamp를 가지며, 해당 변수는 마지막 렌더 기회의 시간을 의미하며, 동시에 Idle 상태의 시작 시간을 나타내기도 한다.

이벤트 루프에 대한 설명은 여기서 마친다. 이정도만 이해해도 Node.js 런타임 환경에서 비동기 함수가 실제로 이벤트 루프를 통해 어떻게 동작하는지 알 수 있다.

> **주의할 점**
Node.js는 Html Spec을 기반으로 하지만 정확히 Html Spec에서 명시된 내용처럼 구현되어 있지 않다는 점이다. 이 점을 꼭 유의해야 한다! 다행히 이번에는 Html Spec을 기반으로 어느 정도 설명해도 비동기 함수의 처리 방식을 이해하는데 전혀 무리가 없다.
> 

# 비동기 함수의 처리 방식(async, promise, web api)

여기서는 비동기 함수의 정의나 콜백 지옥을 어떻게 벗어나는지에 대해 전혀 설명하지 않는다. Promise 객체와 async 함수가 등장하게 된 변천사 및 사용법은 해당 링크에서 보는 것을 추천한다.

[자바스크립트 비동기 처리와 콜백 함수](https://joshua1988.github.io/web-development/javascript/javascript-asynchronous-operation/)

[자바스크립트 Promise 쉽게 이해하기](https://joshua1988.github.io/web-development/javascript/promise-for-beginners/)

[자바스크립트 async와 await](https://joshua1988.github.io/web-development/javascript/js-async-await/)

알아야 할 핵심은 다음과 같다.

1. **async function은 결국 new Promise 객체이다.(근데 자동 resolve 기능이 포함된) async function을 호출할 경우 new Promise(() ⇒ {async function 내부 코드})가 실행된다.**
2. **비동기 함수가 호출되면 await 키워드 부분을 만날 때까지는 콜스택에서 머무르며 동기적인 동작을 처리한다.**
3. **Web Api 비동기 함수 또는 await 키워드를 만났을 경우 해당 함수가 실행될 수 있는 위치로 이동하고 (web api, libuv 등) 콜스택에서 없어진다.**
4. **Web Api 비동기 함수 또는 await 키워드 뒤에 있는 함수가 실행되고 만약 거기에 Promise.then 객체가 있다면 마이크로태스크 큐로 전달된다.(다시 말해 promise에 관련된 콜백함수 then, catch, finally 등은 마이크로태스크 큐로 전달된다.) 만약, setTimeout같은 web api에 해당되는 함수라면, 해당 함수 내부에 있는 콜백함수는 태스크 큐로 전달된다.**
    1. 여기서 재밌는 사실은 Promise 사양이 정의된 Promiseaplus에서는 해당 부분이 마이크로태스크 큐로 구현되거나 태스크 큐로 구현되어야 한다고 표현했다는 것이다. 하지만 마이크로태스크 큐에 넣어진 것을 보니, 결국 promise 객체의 콜백 함수는 마이크로태스크 큐에 넣어지는 것으로 결정한 것 같다.
5. **만약 처음 실행됐던 비동기 함수의 뒷 부분이 남았다면, 해당 부분도 마이크로태스크 큐에 추가된다.(하지만 이것은 4번의 후속 함수(then, catch, finally)가 마이크로태스크 큐에 넣어진 다음 행해진다.)**
6. **이벤트루프는 마이크로태스크 큐와 태스크 큐가 비어있는지 확인하는데, 이때 마이크로태스크가 우선순위로 처리된다. 차이점은 마이크로태스크 큐는 이벤트 루프의 한 주기(16ms)에서 비어질 때까지 처리가되고 태스크 큐는 한 주기에 한 번만 처리된다는 것이다. 그리고 마이크로태스크 큐와 태스크 큐는 모두 한 주기에 처리된다. 여기서 처리된다는 말은 콜스택에 해당 콜백 함수를 올린다는 뜻이다.**
7. **올려진 콜백 함수는 다시 콜스택에서 동기적으로 실행된다.**

### 실험 코드

```jsx
async function asyncFunction() {
  console.log('Async Function Start');

  await new Promise((resolve) => {
    console.log("test")
    resolve();
    }
  ).then(() => {
    console.log("microfirst")
  });

  console.log('Async Function End');
}

console.log('Script Start');

asyncFunction().then(() => {
  console.log('Async Function Complete');
});

console.log('Script End');
```

<img width="437" alt="24" src="https://github.com/user-attachments/assets/e3e5017b-f14e-454c-8a06-8651a8261407">


```jsx
async function asyncFunction() {
  console.log('Async Function Start');

  new Promise((resolve) => {
    console.log("test")
    resolve();
    }
  ).then(() => {
    console.log("microfirst")
  });

  console.log('Async Function End');
}

console.log('Script Start');

asyncFunction().then(() => {
  console.log('Async Function Complete');
});

console.log('Script End');
```

<img width="483" alt="25" src="https://github.com/user-attachments/assets/35efeae4-7da4-4971-9b1c-2f5478c19107">


# 출처

[HTML Standard](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)

[ECMAScript® 2025 Language Specification](https://tc39.es/ecma262/#sec-agents)

[async function - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function#description)

[Promises/A+](https://promisesaplus.com/)

[자바스크립트와 이벤트 루프 : NHN Cloud Meetup](https://meetup.nhncloud.com/posts/89)

[JavaScript의 queueMicrotask()와 함께 마이크로태스크 사용하기 - Web API | MDN](https://developer.mozilla.org/ko/docs/Web/API/HTML_DOM_API/Microtask_guide#마이크로태스크)

[웹 API](https://ko.wikipedia.org/wiki/웹_API)

# 버전 관리 시스템

버전 관리 시스템은 파일 변화를 시간에 따라 기록했다가 나중에 특정 시점의 버전을 다시 꺼내올 수 있는 시스템이다. 버전 관리 시스템은 개발자들에 한정되어서 사용할 수 있는 것이 아니라 디자이너, 작가들도 모두 사용할 수 있다.

예를 들어 디자이너들은 버전 관리 시스템을 사용해서 이미지를 변경 사항마다 저장을 하고 어떤 사람이 해당  이미지를 수정했는지 추적할 수도 있다. 개발자들도 마찬가지로 소스 코드를 관리할 때 이러한 방식으로 사용할 수 있다.

# Git의 구성

처음 git init 명령어를 사용하면 .git 폴더가 생성되고 해당 폴더에 들어가면 다음과 같은 폴더 및 파일들을 확인할 수 있다.

<img width="507" alt="10" src="https://github.com/user-attachments/assets/2f5c1b65-71cd-4eb6-8477-afc442e33887">


- HEAD: git 저장소가 현재 어떤 브랜치를 가리키고 있는지 표시하는 파일이다.
- config: git 저장소의 설정 정보를 보관하는 파일이다.
- description: 프로젝트에 대한 설명을 포함하는 파일이다.
- hooks: git에서 어떠한 이벤트(커밋이나 머지 등)가 생겼을 때 자동으로 실행되는 스크립트를 보관하는 폴더이다. 어떠한 이벤트가 있는지는 아래 링크를 참고
    
    [Git - Git Hooks](https://git-scm.com/book/ko/v2/Git맞춤-Git-Hooks)
    
- info: 추가적인 정보와 설정이 저장되는 폴더이다. 내부에는 기본적으로 exclude 파일이 있는데 해당 파일에는 git 시스템이 무시하는 패턴을 지정할 수 있다. .gitignore와 비슷하다고 생각할 수 있겠지만 exclude에 정의된 설정은 로컬에서만 적용된다는 차이점이 있다.
- obejcts: git에서 버전 관리를 통해서 추적된 파일들의 데이터를 보관하는 폴더이다. 추적된 파일들의 데이터는 zlib 방식으로 압축되어 보관되고 압축된 파일들의 이름은 압축되기 전 데이터들을 기반으로 만든 sha-1 해시코드의 앞 2자리를 제외한 나머지 부분으로 결정된다. 앞 2자리는 해당 데이터 파일을 보관할 상위 폴더를 만들 때 쓰인다.
    - 예를 들어, test라는 파일을 만들었고 해당 test 파일 데이터를 기반으로 한 sha-1 해시 코드의 값이 e69de29bb2d1d6434b8b29ae775ad8c2e48c539 라면 objects 폴더는 e6 폴더가 없다면 생성하고 해당 폴더 내부에 9de29bb2d1d6434b8b29ae775ad8c2e48c539라는 이름을 가진 파일을 만든다. 그리고 해당 파일 내부의 압축된 test 파일 데이터를 보관한다.
- refs: git에서 현재 추적중인 브랜치들에 대한 정보를 가지고 있다. HEAD는 해당 refs 폴더를 통해서 현재 보고있는 브랜치를 찾아서 표시한다.
- index: 현재 git에서 추적하고 있는 파일들의 정보를 가지고 있는 파일이다. 해당 파일은 바이너리 파일이기 때문에 cat 명령어로 직접 열 수는 없고 git ls-files —stage 명령어를 통해서 확인할 수 있다.
명령어를 사용해서 해당 파일을 보면 objects 폴더에 저장되어 있는 압축된 test 파일 데이터의 이름을 확인할 수 있다.
    - git ls-files —stage 명령어를 통해서 test 파일의 전체 해시코드와 원래 파일 이름을 확인할 수 있다. 전체 해시코드를 활용하여 objects 폴더에서 앞의 2글자로 된 폴더로 이동하면 나머지 부분 글자로 이루어진 파일을 하나 발견할 수 있다. 그것이 압축된 test 데이터의 파일이다.
        
<img width="486" alt="11" src="https://github.com/user-attachments/assets/72a374f4-cfc6-4d9c-9edc-b8a39c50c904">

        
<img width="640" alt="12" src="https://github.com/user-attachments/assets/b4a25403-a490-4efc-a8b1-7109f7f93841">

        

# Git의 다양한 상태

버전 관리 시스템인 git은 크게 3가지 상태를 통해서 버전을 관리한다.

![13](https://github.com/user-attachments/assets/94c0a8c4-3843-4b0f-bd3a-7c7940d1692e)


- Working Directory: 사용자가 실제로 작업하고 있는 로컬 공간. 사용자는 Working Directory에서 작업을 하고 있다가, 작업된 내용을 저장하고 싶을 때 Staging Area에 작업된 내용들을 올린다.
- Staging Area: .git directory에 옮겨지기 전 작업된 정보들을 보관하고 있는 임시 공간. 여기서는 아직 작업된 내용이 저장되지 않으며, 최종적으로 여기서 .git directory로 이동해야 작업된 내용을 하나의 버전으로서 저장할 수 있다.
- .git directory(Repository): Git 저장소가 위치한 곳으로, 사용자가 작업한 내용들을 버전 별로 보관하는 실제 공간을 말한다. ‘.git’ 디렉터리에 커밋된 모든 변경 사항들이 버전 히스토리를 통해서 관리되며 이곳을 통해 이전 버전으로 돌아가거나 다양한 버전을 확인할 수 있다. .git 디렉터리는 프로젝트의 모든 버전 관리 정보를 포함하고 있고 git의 핵심이 되는 부분이다.

![14](https://github.com/user-attachments/assets/68e77cb4-8b5a-451a-a4f1-4f714e63dd65)


git은 파일에 대해서 다음과 같은 4가지 상태를 이용해서 파일을 관리한다.

- Untracked: .git/index 에서 목록에 없는 파일들은 Untracked 상태로 규정된다. 버전 관리 시스템을 통해서 해당 파일을 관리하고 싶다면, Untracked 상태를 git add 명령어를 통해서 Staged 상태 올리고 이후 git commit을 통해 등록하여야 한다.
- Unmodified: .git/index 에서 목록에 해당 파일이 존재하고 해당 파일이 버전 관리 시스템에서 관리하고 있는 파일의 이름과 내용이 동일할 경우 Unmodified 상태로 규정된다.
- Modified: Unmodified 상태에서 파일을 수정한 경우 Modified 상태로 규정된다. 해당 상태는 추후 Staged 상태로 올리고 이후 git commit을 통해서 Unmodified 상태로 규정될 수 있다.
- Staged: Staged 상태는 해당 파일이 현재 스테이징 환경, 다시 말해 커밋되기 전에 변경된 상태를 임시 저장하는 공간에 있다는 상태를 말한다. 여기서 추가적으로 git commit을 통해 변경된 상태를 완전히 버전 관리 시스템에 등록해야 변경된 상태를 하나의 버전으로 저장할 수 있다. git commit 이후 Staged 상태에 있던 파일들은 Unmodified 상태로 규정된다.

# Git Object

아까 전 objects 폴더에는 압축된 파일들의 데이터가 sha-1 해시 코드의 이름으로 보관된다고 하였다. 이러한 방식으로 보관되는 Git Object는 3가지 종류로 나뉜다.

## blob object

working directory에서 작업한 파일(untracked or modified 상태의 파일)을 staged 상태로 올릴 때 생성되는 object 파일을 말한다. blob object가 생성되는 시점은 git add 명령어를 통해서 untracked 또는 modified 상태의 파일을 staged로 올릴 때 생성된다. 그리고 이렇게 만들어진 파일들의 전체 해시 코드 값은 .git/index 파일에 파일명과 함께 보관된다. (이전까지 설명했던 objects 폴더에 들어가는 파일들의 예시는 전부 blob object에 해당된다.) 또한 추가적으로 blob object는 작업한 파일만 해당된다는 것을 명심하자. working directory 내부에 있는 하위 폴더는 tree object로 생성되기 때문이다.

추가적으로 알아야 할 사항이 있다면, blob object를 기반으로 만들어진 해시 값은 파일의 데이터를 기반으로 하기 때문에, 같은 데이터를 가진 파일들이 working directory에 여러 개 있어도 blob object는 한 개밖에 생성되지 않는다는 점이다.

자세한 실습을 통해서 살펴보자.

<img width="588" alt="15" src="https://github.com/user-attachments/assets/7010b1fc-62ab-4bc2-aa1b-b62ea7ba30d8">


현재 test 폴더에는 git init을 통해서 git 저장소를 초기화 한 상태이다. 이 상태에서 touch 명령어를 통해서 내용이 빈 파일인 test1, test2, test3를 생성하고 git status 명령어를 실행해 보자.

<img width="571" alt="16" src="https://github.com/user-attachments/assets/1b6d3cbd-42a1-4379-8a53-56f597bc2e19">


여기서 Untracked files에서 test1, test2, test3가 표시된다. 여기서 만약 git add를 하고 index 파일을 조회해 보면 어떻게 될까? 다른 해시값을 가지게 될까?

<img width="518" alt="17" src="https://github.com/user-attachments/assets/19be34d3-a6c1-4b33-8a32-e4c94c403bcc">


index 파일에는 test1, test2, test3에 대한 파일이 스테이지에 올라와 있다는 것을 확인할 수 있지만. 해당 파일들을 기반으로 만들어진 sha-1 해시 값은 전부 다 같은 것을 알 수 있다. 이를 통해 git에서 sha-1 해시 값은 각 파일의 데이터를 기반으로 생성된다는 것을 알 수 있다.

## commit object

commit object는 blob object와 생성 규칙이 동일하며, git commit 명령어를 통해서 commit 시점에 생성되는 object로 tree object가 생성된 이후 생성된다.

commit object는 이전 commit object와 현재 commit에 대한 tree object 해시 값, commit한 시간, commit 메시지, 누가 해당 commit을 하였는지에 대한 정보 등을 보관하고 있는다.

<img width="747" alt="18" src="https://github.com/user-attachments/assets/5f900f9e-e89b-41a1-bae9-190cbb618b9e">


## tree object

stage 환경에 있는 파일들 또는 하위 디렉토리에 대한 정보를 나타내기 위해서 만들어지는 object이다. tree object는 git commit 명령어를 입력한 시점에서 생성되며 tree object의 생성 규칙은 blob object의 생성 규칙과 동일하며, tree object의 내용을 살펴보면, 해당 폴더가 어떠한 파일을 가지고 있는지, 해당 파일에 대한 blob object 해시 값은 무엇인지에 대한 것이 명시되어 있다. tree object 안에는 tree object가 존재할 수 있으며, 이런 경우에는 하위 폴더가 있다는 것으로 판단할 수 있다.

자세한 실습을 통해서 살펴보자.

wow라는 폴더를 만들고 그 안에 test4라는 파일을 만들어 보자.

<img width="540" alt="19" src="https://github.com/user-attachments/assets/1c1927e0-de9e-4867-81a7-b14daaa29184">


여기서 git add 명령어를 통해서 stage 환경에 올리고 index 파일을 확인해 보면, wow 폴더 내부에 있는 test4 파일에 대해서 기존에 만들어두었던 test1, test2, test3 와 같은 sha-1 해시 값을 공유하고 있다는 것을 알 수 있다. 그러므로 git add를 통해서 stage 환경으로 올라올 때 git add 명령어는 파일들만 해당 환경에 올리고 해당 파일 이름 앞에 자신이 위치한 폴더의 경로를 표시만 하는 방식으로 구성된다는 것을 확인할 수 있다.

여기서 git commit을 통해 tree object가 생성된다는 것도 확인해보자.

<img width="693" alt="20" src="https://github.com/user-attachments/assets/2baf1526-6e5a-4db2-abb6-b6b50660d85b">

<img width="747" alt="21" src="https://github.com/user-attachments/assets/d2c6e3c3-5300-4619-92ee-9117024b6471">

<img width="697" alt="22" src="https://github.com/user-attachments/assets/50a9928c-f6ed-45ab-8c85-1fe84d8a6472">


여기서 commit object 내부에 있는 tree object를 조회할 경우, wow라는 tree object가 생성되어 있는 것을 확인할 수 있다.

<img width="712" alt="27" src="https://github.com/user-attachments/assets/c2f7bcd5-a1ba-4d09-9e53-2e8ded941593">


해당 wow tree object에는 아까 만들어둔 test4가 존재한다.

총정리 하자면, tree object는 다음과 같은 방식으로 생성된다.

git add를 하면서 index에 파일 목록이 추가된다. 이때 파일이 하위 폴더에 있는 경우 폴더는 추가하지 않고 파일 이름에 하위 폴더 경로/파일 이름 방식으로 index에 추가된다.

git commit 명령어를 통해서 commit을 하는 시점에 파일 이름을 만날 때까지 해당 폴더에 대한 tree object가 생성된다. 예를 들어 wow/test3/test4라는 것이 존재한다면,

1. wow 라는 tree object를 생성한다.
2. wow 내부에 test3 라는 tree object를 생성한다.
3. test3 tree object 안에 test4에 대응되는 파일 데이터를 가진 blob object를 목록에 추가한다.

# Git stash

working directory에서 modified와 staged 된 파일들을 임시적으로 보관하기 위해서 사용하는 명령어.

git stash 명령어를 사용할 경우, .git/refs/stash 경로에(만약 stash 폴더가 없다면 새로 생성된다.) commit object가 만들어지고 해당 commit object는 stash 했을 떄 시점에 modified와 staged된 파일들을 기반으로 만든 tree object의 해시 값을 보관한다.

또한 .git/logs/refs/stash에 stash에 관한 로그가 추가된다.

저장된 stash 는 git stash list를 통해서 확인할 수 있으며, git stash pop(가장 최신 stash를 불러와서 해당 내용을 적용하고 stash list에서 제거한다.) 또는 git stash apply(pop이랑 비슷하지만 stash list에서 제거하진 않는다.)를 사용하여 복구할 수 있다. 

# Git branch

git branch <브랜치 이름> 을 통해서 브랜치 이름에 해당되는 브랜치를 만들 수 있다.

git branch 명령어를 사용할 경우 .git/refs/head/<브랜치 이름> 경로에 해당되는 파일이 추가되고 해당 파일은 파생되기 전 브랜치의 최신 commit object를 가지고 있게 된다.

추가적으로 git checkout <브랜치 이름>을 통해서 <브랜치 이름> 브랜치로 변경할 경우 .git/HEAD는 변경된 브랜치의 경로를 가지게 된다.

<img width="633" alt="26" src="https://github.com/user-attachments/assets/4cc044b9-b091-4d60-8778-1be40bd3c947">


# Git reset

git reset은 이전 커밋 해시를 통해서 이전 커밋 시점으로 돌아갈 수 있는 명령어이다.

git reset 명령어는 세 가지 옵션이 존재한다.(여기서 HEAD는 이전에 설명하였다.)

- —soft: working directory와 staging area을 제외한 HEAD만을 해당 커밋 시점으로 되돌린다.
- —mixed: working directory을 제외한 staging area와 HEAD만을 해당 커밋 시점으로 되돌린다.
- —hard: working directory, staging area, HEAD 전부 해당 커밋 시점으로 되돌린다.

# 2-way merge, 3-way merge

master 브랜치에서 파생된 다른 a 브랜치를 다시 master 브랜치가 병합하려고 할 때, a 브랜치가 파생되는 커밋 지점을 base, master 브랜치의 최신 커밋 지점을 local a 브랜치의 최신 커밋 지점을 remote라고 했을 때 병합 방식은 2-way merge 또는 3-way merge 방식이 존재한다.

## 2-way merge

local과 remote 지점만을 비교하여 merge 하는 방식

- local과 remote 지점에 파일 이름이 똑같고 내용도 동일할 경우 ⇒ 보존
- local과 remote 지점에 파일 이름이 똑같고 내용이 다를 경우 ⇒ conflict 발생, 사용자가 어떻게 병합을 할 것인지 지정해야 한다.
- local과 remote 지점 어느 한 쪽에만 파일이 존재할 경우 ⇒ conflict 발생, 사용자가 어떻게 병합을 할 것인지 지정해야 한다.

## 3-way merge

base를 참고로 하여, local과 remote의 차이점을 이용하여 merge 하는 방식이다.

보통 git에서는 3-way merge 방식을 많이 사용한다.

3-way merge는 base 또한 추가적으로 비교하기 때문에 2-way merge 보다 conflict 되는 상황이 적다는 장점이 있다.

- base와 local 지점에 파일 이름이 똑같고 내용도 동일하지만 remote 지점에서 해당 파일이 존재하지 않을 경우 ⇒  remote 지점의 내용이 채택됨(merge가 되면 local에 있었던 파일은 삭제된다.)
- base와 local과 remote 지점에 파일 이름이 똑같고 내용이 동일 할 경우 ⇒ 보존
- base와 local과 remote 지점에 파일 이름이 똑같은 파일 내용이 모두 다를 경우 ⇒ conflict 발생, 사용자가 어떻게 병합을 할 것인지 지정해야 한다.
- base와 remote 지점에 파일 이름이 똑같고 내용도 동일하지만 local 지점에서 해당 파일이 존재하지 않을 경우 ⇒ local 지점의 내용이 채택됨(merge가 되면, remote에 있었던 파일은 삭제된다.)[https://git-scm.com/book/ko/v2/Git의-기초-수정하고-저장소에-저장하기](https://git-scm.com/book/ko/v2/Git%EC%9D%98-%EA%B8%B0%EC%B4%88-%EC%88%98%EC%A0%95%ED%95%98%EA%B3%A0-%EC%A0%80%EC%9E%A5%EC%86%8C%EC%97%90-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0)

# 출처

https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F

[Git - 수정하고 저장소에 저장하기](https://git-scm.com/book/ko/v2/Git의-기초-수정하고-저장소에-저장하기)

[지옥에서 온 Git - stash](https://www.youtube.com/watch?v=ArNd5sSwD04&list=PLuHgQVnccGMA8iwZwrGyNXCGy2LAAsTXk&index=25)


