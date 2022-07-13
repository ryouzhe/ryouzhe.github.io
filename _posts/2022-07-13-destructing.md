---
layout: post
title: Javascript 구조 분해 할당
date: 2022-07-13
categories: [Javascript]
tag: [javascript]
---

디스트럭처링 할당은 구조 분해 할당이다. ***이는 배열과 같은 이터러블 또는 객체를 비구조화 하여 1개 이상의 변수에 개별적으로 할당하는 것을 말한다.*** 이터러블 또는 객체 리터럴에서 필요한 값만 추출할 때 유용하다.

## 배열 구조 분해 할당

***배열 구조 분해 할당의 대상은 이터러블이여야 하며, 할당의 기준은 배열의 인덱스이다.***

```javascript

    const arr = [1, 2, 3];
    const [one, two, three] = arr;

    console.log(one);       // 1
    console.log(two);       // 2
    console.log(three);     // 3

```

배열 구조 분해 할당에서 우변에 이터러블을 할당하지 않으면 에러가 발생한다.

```javascript

    const [x, y];       // SyntaxError: Missing initializer in destructuring declaration
    const [a, b] = {};  // TypeError: {} is not iterable

```

배열 구조 분해 할당에서 변수의 개수와 이터러블의 요소 개수가 반드시 일치할 필요는 없다.

```javascript

    const [a, b] = [1, 2];
    console.log(a, b);          // 1 2

    const [c, d] = [1];
    console.log(c, d);          // 1 undefined

    const [e, f] = [1, 2, 3];   
    console.log(e, f);          // 1 2

    const [g, , h] = [1, 2, 3];
    console.log(g, h);          // 1 3

```

배열 구조 분해 할당에서는 변수의 기본값을 설정할 수 있다. 하지만 기본값보다 할당값이 우선한다.

```javascript

    const [a, b, c = 3] = [1, 2];
    console.log(a, b, c);           // 1 2 3

    const [e, f = 10, g = 20] = [1, 2, 3];
    console.log(e, f, g);           // 1 2 3

```

배열 구조 분해 할당을 위한 변수에서는 Rest 요소를 사용할 수 있으며, 이 때 Rest 요소는 마지막에 위치해야 한다.

```javascript

    const [x, ...y] = [1, 2, 3];
    console.log(x, y);              // 1 [2, 3]

```

## 객체 구조 분해 할당

ES6 객체 구조 분해 할당은 객체의 각 프로퍼티를 객체로 추출하여 1개 이상의 변수에 할당한다. 이때 객체 구조 분해 할당의 대상은 객체이여야 하며, 할당 기준은 프로퍼티 키다. ***즉, 순서는 의미가 없으며 선언된 변수 이름과 프로퍼티 키가 일치해야 한다.***

```javascript

    const user = { id: 1, nick: 'Lee' };
    const { nick, id } = user;

    console.log(id, nick);   // 1 'Lee'

```

겍체 구조 분해 할당에서 객체 리터럴 형태로 선언한 변수는 프로퍼티의 축약 표현이다. 아래 두 코드는 동일한 의미이다.

```javascript

    const { id, nick } = user;
    const { id: id, nick: nick } = user;

```

따라서 객체의 프로퍼티 키와 다른 변수 이름으로 프로퍼티 값을 할당받을 수 있다.

```javascript

    const user = { id: 1, nick: 'Lee' };
    const { id: id_num, nick: name } = user;

    console.log(id_num, name);  // 1 'Lee'

```

***객체 구조 분해 할당은 객체에서 필요한 프로퍼티 값만 추출할 때 유용하다.***

```javascript

    const data = { id: 1, content: 'AAA' };
    const { id } = data;

    console.log(id);    // 1

```

***객체 구조 분해 할당은 객체를 인수로 전달받는 함수의 매개변수에서도 유용하다.***

```javascript

    function printData({ id, content }) {
        console.log(`ID: ${id}, CONTENT: ${content}`);
    }

```

중첩 객체의 경우 프로퍼티 키로 객체를 추출한 후 해당 객체의 프로퍼티 키로 값을 추출할 수 있다.

```javascript

    const user = {
        neme: 'Lee',
        address: {
            city: 'A'
        }
    };

    const { address: { city } } = user;
    console.log(city);      // A 

```
