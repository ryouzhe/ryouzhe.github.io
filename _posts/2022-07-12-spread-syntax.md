---
layout: post
title: Javascript 스프레드 문법
date: 2022-07-12
categories: [Javascript]
tag: [javascript]
---

스프레드 문법은 ES6에서 도입된 문법으로 하나로 뭉쳐 있는 여러 값들의 집합을 펼쳐서 개별적인 값들의 목록으로 만든다. 이러한 스프레드 문법은 for...of 문으로 순회할 수 있는 이터러블에 한정된다.

```javascript

    console.log(...[1, 2, 3]);  // 1, 2, 3

    console.log(...'Hello');    // H e l l o

    console.log(...{a: 1, b: 2});
    // TypeError: Found non-callable @@iterator
    // 이터러블이 아닌 일반 객체는 스프레드 문법의 대상이 될 수 없다.

```

***스프레드 문법의 결과는 값이 아니다. 이터러블의 요소들을 펼쳐서 만든 개별적인 값들의 목록이다.*** 따라서 스프레드 문법의 결과물은 값으로 사용할 수 없고, 다음과 같이 쉽표로 구분한 값의 목록을 사용하는 문맥에서만 사용할 수 있다.

**- 함수 호출문의 인수 목록**

**- 배열 리터럴의 요소 목록**

**- 객체 리터럴의 프로퍼티 목록**

## 함수 호출문의 인수 목록에서 사용하는 경우

```javascript

    const arr = [1, 2, 3];

    const max1 = Math.max(arr);     // NaN
    // Math.max(...[1, 2, 3])은 Math.max(1, 2, 3)과 같다.
    const max2 = Math.max(...arr);  // 3

```

스프레드 문법은 Rest 파라미터와 형태가 동일하여 혼동할 수 있다. ***Rest 파라미터와 스프레드 문법은 서로 반대의 개념이다.*** Rest 파라미터는 목록을 배열로 만들고, 스프레드 문법은 이터러블을 펼쳐서 값들의 목록으로 만든다.

## 배열 리터럴 내부에서 사용하는 경우

스프레드 문법을 활용하면 기존에 사용하던 배열 빌드인 함수를 대체할 수 있다.

```javascript

    // 2개의 배열을 1개의 배열로 결합하는 경우
    // ES5
    var arr1 = [1, 2].concat([3, 4]);
    console.log(arr1);      // [1, 2, 3, 4]

    // ES6
    const arr2 = [...[1, 2], ...[3, 4]];
    console.log(arr2);      // [1, 2, 3, 4]

    // 배열을 복사하는 경우
    // ES5
    var origin1 = [1, 2];
    var copy1 = origin1.slice();

    console.log(copy1);                 // [1, 2]
    console.log(copy1 === origin1);     // false

    // ES6
    const origin2 = [1, 2];
    const copy2 = [...origin2];

    console.log(copy2);                 // [1, 2]
    console.log(copy2 === origin2);     // false

```

배열을 복사하는 경우 스프레드 문법과 slice 메서드 모두 얕은 복사를 통해 새로운 복사본을 생성한다.

## 객체 리터럴 내부에서 사용하는 경우

Rest 프로퍼티와 함께 2021년 1월 현재 TC39 프로세스의 stage 4 단계에 제안되어 있는 스프레드 프로퍼티를 사용하면 객체 리러털의 프로퍼티 목록에서도 스프레드 문법을 사용할 수 있다. 스프레드 문법은 이터러블에서만 제한되었지만 스프레드 프로퍼티 제안은 일반 객체를 대상으로도 스프레드 문법의 사용을 허용한다.

```javascript

    //스프레드 프로퍼티
    const obj = { x: 1, y: 2 };
    const copy = { ...obj };
    
    console.log(copy);              // { x: 1, y: 2 }
    console.log(obj === copy);      // false

    // 객체 병합
    const merged = { x: 1, y: 2, ...{ a: 3, b: 4 } };
    console.log(merged);            // { x: 1, y: 2, a: 3, b: 4 }

```

프로퍼티가 중복되는 경우, 특정 프로퍼티의 병경 및 추가는 아래와 같다.

```javascript

    // 프로퍼티가 중복되는 경우 뒤에 위치한 프로퍼티가 우선권을 갖는다.
    const merged = { ...{ x: 1, y: 2 }, ...{ y: 10, z: 3 } };
    console.log(merged);    // { x: 1, y: 10, z: 3 }

    // 특정 프로퍼티 변경
    const changed = { ...{ x: 1, y: 2 }, y: 100 };
    console.log(changed);   // { x: 1, y: 100 }

    // 프로퍼티 추가
    const added = { ...{ x: 1, y: 2 }, z: 0 };
    console.log(added);     // { x: 1, y: 2, z: 0 }

```

