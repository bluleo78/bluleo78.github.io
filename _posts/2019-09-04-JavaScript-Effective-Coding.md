---
title: JavaScript 코딩
---

Array/Object Destructuring
---
### Array Destructuring
```
let [x, y] = [1, 2, 3, 4];
x; // 1
let {z} = {x:1, y:2, z:3};
z; // 3
```
### Object Destructuring
```
let {x} = {x:1, y:2};
x; // 1
let {y:y1} = {x:1, y:2};
y1; // 3
let {z = 3, x:x1 = 1} = {};
z; // 3
x1; // 1
```
### Object Destructuring - 함수 호출
```
function myfunc({x, y=1}={}) {
  return [x,y]
}
myfunc({x:1}); // [x:1, y:1]
```

Array Rest/Spread Property
---
배열을 이용하여 배열을 생성하기 쉽게 지원
### Rest
```
let [x, y, ...z] = [1, 2, 3, 4];
z; // [3, 4]
```

### Spread
```
let n = [1, 2, ...z];
n; // [1, 2, 3, 4]
```

### Spread - 함수 호출
```
function myfunc(a, b, c, d) {
  return a;
}
myfunc(...n) // 1
```

Object Rest/Spread Property
---
오브젝트를 이용하여 오브젝트를 생성하기 쉽게 지원
### Rest
```
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
z; // { a: 3, b: 4 }
```
### Spread
```
let n = { x, y, ...z };
n; // { x: 1, y: 2, a: 3, b: 4 }
```

참고
---
* <https://github.com/tc39/proposal-object-rest-spread>
