---
layout: post
title: Javascript 제너레이터와 async/await
date: 2022-08-22
categories: [Javascript]
tag: [javascript]
---

## **제너레이터**

제너레이터는 코드 블록의 실행을 일시 중지했다가 필요한 시점에 재개할 수 있는 특수한 함수다. 제너레이터의 특징은 다음과 같다.

> **1. 제너레이터함수는 함수 호출자에게 함수 실행의 제어권을 양도할 수 있다.**
> 일반 함수를 호출하면 제어권이 함수에게 넘어가고 함수 코드를 일관 실행한다. 함수 호출자는 함수를 호출한 이후 함수 실행을 제어할 수 없다. 반면 제너레이터 함수는 함수 실행을 함수 호출자가 제어할 수 있다. 다시 말해, 함수 호출자가 함수 실행을 일시 중지시키거나 재개시킬 수 있다.
> 
> **2. 제너레이터 함수는 함수 호출자와 함수의 상태를 주고받을 수 있다.**
> 제너레이터 함수는 함수 호출자와 양방향으로 함수의 상태를 주고받을 수 있다. 제너레이터 함수는 함수 호출자에게 상태를 전달할 수 있고 함수 호출자로부터 상태를 전달받을 수도 있다.
>
> **3. 제너레이터 함수를 호출하면 제너레이터 객체를 반환한다.**
> 제너레이터 함수를 호출하면 함수 코드를 실행하는 것이 아니라 이터러블이면서 동시에 이터레이터인 제너레이터 객체를 반환한다.


## **제너레이터 함수의 정의**

```javascript

    // 제너레이터 함수 선언문
    function* genDecFunc() {
        yield 1;
    }

    // 제너레이터 함수 표현식
    const getExpFunc() {
        yield 1;
    }

    // 제너레이터 메서드
    const obj = {
        * genObjMethod() {
            yield 1;
        }
    };

    // 제너레이터 클래스 메서드
    class MyClass {
        * genClsMethod() {
            yield 1;
        }
    } 

```

제너레이터 함수는 화살표 함수로 정의할 수 없다. 그리고 new 연산자와 함께 생성자로 호출할 수 없다. 

## **제너레이터 객체**

***제너레이터 함수를 호출하면 일반 함수처럼 함수 코드 블럭을 실행하는 것이 아니라 제너레이터 객체를 생성해 반환한다. 제너레이터 함수가 반환한 제너레이터 객체는 이터러블이면서 동시에 이터레이터다.*** 제너레이터 객체는 next 메서드를 가지는 이터레이터이므로 Symbol.iterator 메서드를 호출해서 별도로 이터레이터를 생성할 필요가 없다.

제너레이터 객체는 next 메서드를 갖는 이터레이터이지만 이터레이터에는 없는 return, throw 메서드를 갖는다. 해당 메서드는 다음과 같이 동작한다.

> next 메서드는 제너레이터 함수의 yield 표현식까지 코드 블록을 실행하고 yield 된 값을 value 프로퍼티 값으로, false를 done 프로퍼티 값으로 갖는 이터레이터 리절트 객체를 반환한다.
>
> return 메서드는 인수로 전달받은 값을 value 프로퍼티 값으로, true를 done 프로퍼티 값으로 갖는 이터레이터 리절트 객체를 반환한다.
>
> throw 메서드는 인수로 전달받은 에러를 발생시키고 undefined를 value 프로퍼티 값으로, true를 done 프로퍼티 값으로 갖는 이터레이터 리절트 객체를 반환한다.


## **제너레이터 일시 중지와 재개**

제너레이터는 yield 키워드와 next 메서드를 통해 실행을 일시 중지했다가 필요한 시점에 다시 재개할 수 있다.***yield 키워드는 제너레이터 함수의 실행을 일시 중지시키거나 yield 키워드 뒤에 오는 표현식의 평가 결과를 제너레이터 함수 호출자에게 반환한다.***

```javascript

    function* genFunc() {
        yield 1;
        yield 2;
        yield 3;
    }

    const generator = genFunc();

    console.log(generator.next());  // { value: 1, done: false }
    console.log(generator.next());  // { value: 2, done: false }
    console.log(generator.next());  // { value: 3, done: false }

    consoel.log(generator.next());  // { value: undefined, done: true }

```

***제너레이터 객체의 next 메서드를 호출하면 yield 표현식까지 실행되고 일시 중지 된다. 이때 함수의 제어권이 호출자로 양도(yield)된다. 제너레이터 객체의 next 메서드는 value, done 프로퍼티를 갖는 이터레이터 리절트 객체를 반환한다. 이때 반환된 이터레이터 리절트 객체의 value 프로퍼티에는 yield 표현식에서 yield된 값이 할당되고 done 프로퍼티에는 제너레이터 함수가 끝까지 실행되었는지를 나타내는 불리언 값이 할당된다.***

이터레이터의 next 메서드와 달리 제너레이터 객체의 next 메서드에는 인수를 전달할 수 있다. 제너레이터 객체의 next 메서드에 전달한 인수는 제너레이터 함수의 yield 표현식을 할당받는 변수에 할당된다. yield 표현식을 할당받은 변수에 yield 표현식의 평가 결과가 할당되지 않는 것에 주의하자.

```javascript

    function* genFunc() {
        const x = yield 1;
        const y = yidel (x + 10);
        return x + y;
    }

    const generator = genFunc(0);

    // 처음 호출하는 next 메서드에는 인수를 전달하지 않는다. 인수를 전달해도 무시된다.
    let res = generator.next();
    console.log(res);       // { value: 1, done: false }

    // next 메서드에 인수로 전달한 10은 genFunc 함수의 x 변수에 할당된다.
    let res = generator.next(10);
    console.log(res);       // { value: 20, done: false }

    // next 메서드에 인수로 전달한 20은 genFunc 함수의 y 변수에 할당된다.
    let res = generator.next(20);
    console.log(res);       // { value: 30, done: false }


```

## **제너레이터의 활용**

제너레이터 함수는 next 메서드와 yield 표현식을 통해 함수 호출자와 함수의 상태를 주고받을 수 있다. 이러한 특성을 통해 프로미스를 통한 비동기 처리를 동기 처리처럼 구현할 수 있다.

```javascript

    const async = generatorFunc => {
        const generator = generatorFunc();

        const onResolved = arg => {
            const result = generator.next(arg);

            return result.done ? result.value : result.value.then(res => onResolved(res));
        };

        return onResolved;
    };

    (async(function* fetchTodo() {
        const url = 'https://jsonplaceholder.typicode.com/todos/1';

        const response = yield fetch(url);
        const todo = yield response.json();
        console.log(todo);  // {userId: 1, id: 1, title: ... }
    })());

```

위 코드는 제너레이터 실행기를 이해하기 위한 코드이다. 우선 즉시 실행 함수인 fetchTodo가 실행되면서 제너레이터 객체를 생성하고 onResolved 함수를 반환한다. 여기서 onResolved 함수는 상위 스코프의 generator 변수를 기억하는 클로저다. async 함수가 반환한 onResolved 함수를 즉시 호출하여 제너레이터 객체의 next 메서드를 처음 호출한다.

next 메서드가 처음 호출된 후 fetchTodo의 첫 번째 yield 문 까지 실행된다. 이때, 첫 번째 yield 된 fetch 함수가 반환한 프로미스가 resolve 한 Response 객체를 onResolved 함수에 인수로 전달하고 재귀 호출을 한다. 

onResolved 함수에 인수로 전달된 Response 객체를 next 메서드에 인수로 전달하면서 next 메서드를 두 번재로 호출한다. 

next 메서드가 반환한 이터레이터 리절트 객체의 done 프로퍼티 값이 false이므로, respone.json 메서드가 반환한 프로미스가 resolve한 todo 객체를 onResolved 함수에 인수로 전달하면서 다시 재귀 호출을 한다.

onResolved 함수에 인수로 전달된 todo 객체를 next 메서드에 인수로 전달하면서 next 메서드를 세 번째로 호출 한다. 

next 메서드가 반환한 이터레이터 리절트 객체의 done 프로퍼티가 true로 바뀌면서 fetchTodo 반환값인 undefined를 반환하고 모든 처리가 종료된다.

## **async/await**

ES8에서는 제너레이터보다 간단하고 가독성 좋게 비동기 처리르 동기 처리처럼 동작하도록 구현할 수 있는 async/await가 도입되었다. async/await는 프로미스를 기반으로 동작한다.

```javascript

    async function fetchTodo() {
        const url = 'https://jsonplaceholder.typicode.com/todos/1';

        const response = await fetch(url);
        const todo = await response.json();

        console.log(todo);
    }

    fetchTodo();

```

#### **async 함수**

await 키워드는 반드시 async 함수 내부에서 사용해야 한다. async 함수는 언제나 프로미스를 반환하며, 명시적으로 프로미스를 반환하지 않더라도 암묵적으로 반환값을 resolve 하는 프로미스를 반환한다.

```javascript

    async function foo(n) { return n; }
    foo(1).then(v => console.log(v));   // 1

    const bar = async function(n) { return n; }
    bar(2).then(v => console.log(v));   // 2

    const baz  = async n => n;
    baz(3).then(v => console.log(v));   // 3

    const obj = {
        async foo(n) { return n; }
    };
    obj.foo(4).then(v => console.log(v));   // 4

    class MyClass {
        async bar(n) { return n; }
    }
    const myClass = new MyClass();
    myClass.bar(5).then(v => console.log(v));   // 5

```

클래스의 constructor 메서드는 async 메서드가 될 수 없다. 

#### **await 키워드**

await 키워드는 프로미스가 settled 상태가 될 때까지 대기하다가 settled 상태가 되면 프로미스가 resolve한 처리 결과를 반환한다. ***await 키워드는 반드시 프로미스 앞에서 사용해야 한다.***

```javascript

    const getData = async id => {
        const res = await fetch(`https://api/users/${id}`);
        const { name } = await res.json();
        console.log(name);
    }

    getData(1);

```

fetch 함수가 수행한 HTTP 요청에 대한 응답이 도착해서 fetch 함수가 반환한 프로미스가 settled 상태가 될때까지 첫 번째 await는 대기한다. 이후 프로미가 settled 상태가 되면 프로미스가 resolve한 처리 결과가 res 변수에 할당된다.

이러한 async/await 특성은 비동기 처리를 순차적으로 진행해야할 때 사용할 수 있다.

#### **에러 처리**

async/await에서는 try...catch문으로 에러 처리를 할 수 있다. 콜백 함수를 인수로 전달받는 비동기 함수와 달리 프로미스를 반환하는 비동기 함수는 명시적으로 호출할 수 있기 때누에 호출자가 명확하기 때문이다.

```javascript

    const foo = async() => {
        try {
            const wrongUrl = 'https://wrong.url';

            const response = await fetch(wrongUrl);
            const data = await response.json();
            console.log(data);
        } catch (err) {
            console.error(err);
        }
    };

    foo();

```

async 함수 내에서 catch 문을 사용해서 에러 처리를 하지 않으면 async 함수는 발생한 에러를 reject하는 프로미스를 반환한다. 따라서 async 함수를 호출하고 Promise.prototype.catch 후속 처리 메서드를 사용해 에러를 캐치할 수도 있다.

```javascript

    const foo = async() => {
        cosnt wrongUrl = 'https://wrong.url';

        const response = await fetch(wrongUrl);
        const data = await response.json();
        return data;
    };

    foo()
    .then(console.log)
    .catch(console.error);

```