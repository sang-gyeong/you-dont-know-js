### 🔥 SUMMARY

다음 규칙은 명시적 this 바인딩이 없는 한 늘 성립한다.

원리는 바탕으로 꾸준히 다양한 상황에서 this가 무엇일지 예측해보는 연습을 해보기 바란다.

- 전역공간에서의 this는 전역객체를 참조한다.
- 어떤 함수를 메서드로서 호출한 경우 this는 메서드 호출 주체(메서드명 앞의 객체)를 참조한다.
- 어떤 함수를 함수로서 호출한 경우 this는 전역객체를 참조한다. 메서드의 내부함수에서도 같다.
- 콜백 함수 내부에서의 this는 해당 콜백 함수의 제어권을 넘겨받은 함수가 정의한 바에 따르며,

정의하지 않은 경우에는 전역객체를 참조한다.

- 생성자 함수에서의 this는 생성될 인스턴스를 참조한다.

다음은 명시적 this 바인딩이다. 위 규칙에 부합하지 않는 경우에는 다음을 바탕으로 this를 예측할 수 있다.

- call, apply 메서드는 this를 명시적으로 지정하면서 함수 또는 메서드를 호출한다.
- bind 메서드는 this 및 함수에 넘길 인수를 일부 지정해서 새로운 함수를 만든다.
- 요소를 순회하면서 콜백 함수를 반복 호출하는 내용의 일부 메서드는 별도의 인자로 this를 받기도 한다.

---

# 상황에 따라 달라지는 this

자바스크립트에서 this는 기본적으로 실행 컨텍스트가 생성될 때 함께 결정된다.

**this는 함수를 호출할 때 결정된다.** 

## 1. 전역 공간에서의 this

전역 공간에서 this는 전역 객체를 가르킨다.

개념상 전역 컨텍스트를 생성하는 주체가 바로 전역 객체이기 때문이다.

전역 객체는 자바스크립트 런타임 환경에 따라 다른 이름과 정보를 가지고 있다.

브라우저 환경에서 전역객체는 window이고 node.js환경에서는 global이다.

- 전역 공간에서의 this(브라우저 환경)

```jsx
console.log(this);            // {alert: f(), atob: f(), blur: f(), ...}
console.log(window);          // {alert: f(), atob: f(), blur: f(), ...}
console.log(this === window); // true
```

- 전역 공간에서의 this(Node.js 환경)

```jsx
console.log(this);            // {process: { title: 'node', version: 'v10.13.0', ... }}
console.log(global);          // {process: { title: 'node', version: 'v10.13.0', ... }}
console.log(this === global); // true
```

- (참고) 전역 공간에서만 발생하는 특이한 성질

## 2. 메서드로서 호출할 때 그 메서드 내부에서의 this

- **함수 vs 메서드**

어떤 함수를 실행하는 방법은 여러가지가 있는데,

가장 일반적인 방법 두가지는 함수로서 호출하는 경우와 메서드로 호출하는 경우이다.

프로그래밍 언어에서 함수와 메서드는 미리 정의한 동작을 수행하는 코드 뭉치로,

이 둘을 구분하는 유일한 차이는 **독립성**에 있다.

**함수는 그 자체로 독립적인 기능을 수행하는 반면, 메서드는 자신을 호출한 대상 객체에 관한 동작을 수행한다.**

자바스크립트는 상황별로 this 키워드에 다른 값을 부여하게 함으로써 이를 구현했다.

자바스크립트를 처음 접한다면 흔히 메서드를 '객체의 프로퍼티에 할당된 함수'로 이해하곤 한다.

이는 반은 맞고 반은 틀린 이야기이다.

**어떤 함수를 객체의 프로퍼티에 할당한다고 해서 그 자체로서 무조건 메서드가 되는 것이 아니라**

**객체의 메서드로서 호출할 경우에만 메서드로 동작하고, 그렇지 않으면 함수로 동작한다.**

```jsx
var func = function (x) {
	console.log(this, x);
};
func(1);         // Window { ... } 1

var obj = {
	method: func
};
obj.method(2);    // {method: f} 2
obj['method'](3); // {method: f} 3
```

1번째 줄에서 func라는 변수에 익명함수를 할당했다. 4번째 줄에서 func를 호출했더니 this로 전역객체 Window가 출력된다. 6번째 줄에서 obj라는 변수에 객체를 할당하는데, 그 객체의 method 프로퍼티에 앞에서 만든 func 함수를 할당했다.

이제 9번째 줄에서 obj의 method를 호출했더니, 이번에는 this가 obj라고 한다.

obj의 method 프로퍼티에 할당한 값과 func 변수에 할당한 값은 모두 1번째 줄에서 선언한 함수를 참조한다.

즉 원래의 익명함수는 그대로인데 이를 변수에 담아 호출한 경우와 obj 객체의 프로퍼티에 할당해서 호출한 경우에 this가 달라지는 것이다.

그렇다면 '함수로서 호출'과 '메서드로서 호출'을 어떻게 구분할까? 함수 앞에 점(.)이 있는지 여부만으로 간단하게 구분할 수 있다.

앞의 4번째 줄은 앞에 점이 없으니 함수로서 호출한 것이고,

9번째 줄은 method 앞에 점이 있으니 메서드로서 호출한 것이다.

진짜다!(물론 대괄호 표기법에 따른 경우에도 메서드로서 호출한 것)

다시말해 어떤 함수를 호출할 때 그 함수 이름(프로퍼티명) 앞에 객체가 명시되어 있는 경우에는 메서드로 호출한 것이고, 그렇지 않은 모든 경우에는 함수로 호출한 것이다.

### 메서드 내부에서의 this

this에는 호출한 주체에 대한 정보가 담긴다.

**어떤 함수를 메서드로서 호출하는 경우 호출 주체는 바로 함수명(프로퍼티명) 앞의 객체이다.**

점 표기법의 경우 마지막 점 앞에 명시된 객체가 곧 this가 되는 것이다.

```jsx
var obj = {
	methodA : function () { console.log(this);},
	inner: {
		methodB : function () { console.log(this); }
	}
};
obj.methodA();              // {methodA: f, inner: {...}} ( === obj)
obj['methodA']();           // {methodA: f, inner: {...}} ( === obj)

obj.inner.methodB();        // {methodB: f}               ( === obj.inner)
obj.inner['methodB']();     // {methodB: f}               ( === obj.inner)
obj['inner'].methodB();     // {methodB: f}               ( === obj.inner)
obj['inner']['methodB']();  // {methodB: f}               ( === obj.inner)
```

## 3. 함수로서 호출할 때 그 함수 내부에서의 this

### 함수 내부에서의 this

어떤 함수를 함수로서 호출할 경우에는 this가 지정되지 않는다.

this에는 호출한 주체에 대한 정보가 담기는데 함수로서 호출하는 것은 호출 주체를 명시하지 않고 개발자가 코드에 직접 관여해서 실행한 것이기 때문에 호출 주체의 정보를 알 수 없는 것이다.

2장에서 실행 컨텍스트를 활성화할 당시에 this가 지정되지 않은 경우 this는 전역 객체를 바라본다고 하였다.

따라서 함수에서의 this는 전역 객체를 가르킨다. 

더글라스 크락포드는 이를 명백한 설계상의 오류라고 지적한다. why?

### 메서드의 내부함수에서의 this

혼동이 있지만 우리는 공부했기 때문에 할 수 있을 것.

내부함수 역시 이를 함수로서 호출했는지 메서드로서 호출했는지만 파악하면 this의 값을 정확히 맞출 수 있다.

다음의 예제를 보고 맞춰보자.

```jsx
var obj = {
	outer: function () {
		console.log(this);                 // (1)
		var innerFunc = function () {
			console.log(this);               // (2) (3)
		}
		innerFunc();

		var obj2 = {
			innerMethod: innerFunc
		};
		obj2.innerMethod();
	}
};
obj1.outer();
```

(1) : obj1

(2) : 전역객체(Window)

(3) : obj2.innerMethod

즉 this 바인딩에 관해서는 함수를 실행하는 당시의 주변 환경(메서드, 함수 내부인지 등)은 중요하지 않고,

**오직 해당 함수를 호출하는 구문 앞에 점 또는 대괄호 표기가 있는지 없는지가 관건인 것이다.**

### 메서드의 내부 함수에서의 this를 우회하는 방법

이렇게 하면 this에 대한 구분은 명확히 할 수 있지만, 그 결과 this라는 단어가 주는 인상과는 사뭇 달라져버렸다.

호출 주체가 없을 때는 자동으로 전역객체를 바인딩 하지 않고, 호출 당시 주변 환경의 this를 그대로 상속받아 사용할 수 있었으면 좋겠다.

그게 훨씬 자연스러울 뿐 더러 자바스크립트 설계상 이렇게 동작하는 것이 스코프 체인과의 일관성을 지키는 설득력 있는 방식일 것이다.

변수를 검색하면 우성 가장 가까운 스코프의 L.E를 찾고 없으면 상위 스코프를 탐색하듯이,

this 역시 현재 컨텍스트에 바인딩된 대상이 없으면 직전 컨텍스트의 this를 바라보게끔 말이다.

아쉽게도 ES5까지는 자체적으로 내부 함수에 this를 상속할 방법이 없지만 다행히 이를 우회할 방법이 없진 않다.

그중 대표적인 방법은 바로 변수를 활용하는 것이다.

```jsx
var obj = {
	outer: function () {
		console.log(this);              // (1) {outer: f}
		var innerFunc1 = function () {
			console.log(this);            // (2) Window { ... }
		};
		innerFunc1();
	
		var self = this;
		var innerFunc2 = function () {
			console.log(self);             // (3) {outer: f}
		};
		innerFunc();
	}
};
obj.outer();
```

상위 스코프의 this를 저장해서 내부함수에서 활용하는 수단으로 사용되는 변수 self를 사용하였다.

허무한 방법으로 보일수 있지만 기대한 기능을 수행하는 것을 볼 수 있다.

(self가 가장 많이 쓰이는 변수명으로, 의미만 통한다면 변수명은 무엇으로 해도 무방하다)

### this를 바인딩하지 않는 함수 (Arrow Function)

ES6에서는 함수 내부에서 this가 전역객체를 바라보는 문제를 보완하고자,

this를 바인딩하지 않는 화살표함수를 새로 도입하였다.

⭐️ **화살표 함수는 실행 컨텍스트를 생성할 때 this 바인딩 과정 자체가 빠지게 되어,**

**상위 스코프의 this를 그대로 활용할 수 있다.** ⭐️ 

```jsx
var obj = {
	outer: function () {
		console.log(this);
		var innerFunc = () => {
			console.log(this);
		};
		innerFunc();
	}
};
obj.outer();
```

그 밖에도 call, apply 등의 메서드를 활용해 함수를 호출할 때 명시적으로 this를 지정하는 방법이 있다.

(이러한 방법에 대해선 뒤에서 자세히 다룰 것)

---

# 명시적으로 this를 바인드 하는 방법

## 1. call 메서드

```jsx
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```

call 메서드는 메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령이다.

이때 call 메서드의 첫 번째 인자를 call로 바인딩하고, 이후의 인자들을 호출할 함수의 매개변수로 한다.

함수를 그냥 실행하면 this는 전역객체를 참조하지만 call 메서드를 이용하면 임의의 객체를 this로 지정할 수 있다.

```jsx
var func = function (a, b, c) {
	console.log(this, a, b, c);
};

func(1, 2, 3);                 // Window{ ... } 1 2 3
func.call({ x : 1 }, 4, 5, 6); // { x : 1 } 4 5 6
```

메서드에 대해서도 마찬가지로 객체의 메서드를 그냥 호출하면 this는 객체를 참조하지만

call 메서드를 사용하면 임의의 객체를 this로 지정할 수 있다.

```jsx
var obj = {
	a : 1,
	method: function (x, y) {
				console.log(this.a, x, y);
	}
};

obj.method(2, 3);                // 1 2 3
obj.method.call({ a: 4}, 5, 6,); // 4 5 6
```

## 2. apply 메서드

```jsx
Function.prototype.apply(thisArg[, argArray])
```

apply 메서드는 call 메서드와 기능적으로 완전히 동일하다.

call 메서드는 첫 번째 인자를 제외한 나머지 모든 인자들을 호출할 함수의 매개변수로 지정하는 반면,

**apply 메서드는 두 번째 인자를 배열로 받아 그 배열의 요소들을 호출할 함수의 매개변수로 지정한다는 차이가 있다.**

```jsx
var func = function (a, b, c) {
	console.log(this, a, b, c);
};
func.apply({x:1}, [4, 5, 6])      // {x:1} 4 5 6

var obj = {
	a : 1,
	method: function (x, y) {
				console.log(this.a, x, y);
	}
};
obj.method.apply({a:4}, [5, 6]);  // 4 5 6
```

## 3. call / apply 메서드의 활용 사례

- 유사배열객체(array-like object)에 배열 메서드를 적용
- 생성자 내부에서 다른 생성자를 호출
- 여러 인수를 묶어 하나의 배열로 전달하고 싶을 때 - apply 활용

## 4. bind 메서드

```jsx
Function.prototype.bind(thisArg[, arg1[, arg2[, ...]]])
```

bind 메서드는 ES5에서 추가된 기능으로, call과 비슷하지만

**즉시 호출하지는 않고 넘겨받은 this 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드**이다.

다시 새로운 함수를 호출할 때 인수를 넘기면 그 인수들은 기존 bind 메서드를 호출할 때 전달했던 인수들의 뒤에 이어서 등록된다. 즉 bind 메서드는 함수에 this를 미리 적용하는 것과 부분 적용 함수를 구현하는 두 가지 목적을 모두 지닌다.

```jsx
var func = function (a, b, c, d) {
	console.log(this, a, b, c, d);
};
func(1, 2, 3, 4);        // Window{ ... } 1 2 3 4

var bindFunc1 = func.bind({ x : 1});
bindFunc1(5, 6, 7, 8);    // { x : 1 } 5 6 7 8

var bindFunc2 = func.bind({ x : 1}, 4, 5);
bindFunc1(6, 7);        // { x : 1 } 4 5 6 7
bindFunc1(8, 9);        // { x : 1 } 4 5 8 9
```

- **name 프로퍼티**

bind 메서드를 적용해서 새로 만든 함수는 한가지 독특한 성질이 있다.

바로 name 프로퍼티에 동사 bind의 수동태인 'bound'라는 접두어가 붙는다는 점이다.

어떤 함수의 name 프로퍼티가 'bound xxx'라면 이는 곧 함수명이 xxx인 원본 함수에 bind 메서드를 적용한 새로운 함수라는 의미가 되므로 기존의 call이나 apply보다 코드를 추적하기에 더 수월해진 면이 있다.

```jsx
var func = function (a, b, c, d) {
	console.log(this, a, b, c, d);
};
var bindFunc = func.bind({x:1}, 4, 5);
console.log(func.name);       // func
console.log(bindFunc.name);   // bound func
```

- **상위 컨텍스트의 this를 내부함수나 콜백 함수에 전달하기**

앞에서 메서드의 내부함수에서 메서드의 this를 그대로 바라보게 하기 위한 방법으로

self 등의 변수를 활용한 우회법을 소개했는데,

call, apply 또는 bind 메서드를 이용하면 더 깔끔하게 처리할 수 있다.

```jsx
var obj = {
	outer: function () {
			console.log(this);
			var innerFunc = function () {
					console.log(this);
			};
			innerFunc.call(this);
	}
};
obj.outer();
```

```jsx
var obj = {
	outer: function () {
			console.log(this);
			var innerFunc = function () {
					console.log(this);
			}.bind(this);
			innerFunc();
	}
};
obj.outer();
```

또한 콜백함수를 인자로 받는 함수나 메서드 중에서 기본적으로 콜백 함수 내에서의 this에 관여하는 함수 또는 메서드에 대해서도 bind 메서드를 사용하면 this 값을 사용자 입맛에 맞게 바꿀 수 있다.

```jsx
var obj = {
		logThis: function () {
				console.log(this);
		}.
		logThisLater1: function () {
				setTimeout(this.logThis, 500);
		},
		logThisLater2: function () {
				setTimeout(this.logThis.bind(this), 1000);
		}
};

obj.logThisLater1(); // Window {...}
obj.logThisLater2(); // obj {logThis: f, ...}
```

## 5. 화살표 함수의 예외사항

ES6에 새롭게 도입된 화살표 함수는 실행 컨텍스트 생성 시 this를 바인딩하는 과정이 제외되었다.

즉 이 함수 내부에는 this가 아예 없으며, 접근하고자 하면 스코프체인상 가장 가까운 this에 접근하게 된다.

```jsx
var obj = {
	outer: function () {
		console.log(this);
		var innerFunc = () => {
			console.log(this);
		};
		innerFunc();
	}
};
obj.outer();
```

이렇게 하면 별도의 변수로 this를 우회하거나

call, apply, bind를 적용할 필요가 없어 더욱 간결하고 편리하다.

## 6. 별도의 인자로 this를 받는 경우(콜백 함수 내에서의 this)

콜백 함수를 인자로 받는 메서드 중 일부는 추가로 this를 지정할 객체(thisArg)를 인자로 지정할 수 있는 경우가 있다.

이러한 메서드의 thisArg 값을 지정하면 콜백 함수 내부에서 this 값을 원하는 대로 변경할 수 있다.

이런 형태는 여러 내부 요소에 대해 같은 동작을 반복 수행해야 하는 **배열 메서드**에 많이 포진돼 있으며,

같은 이유로 ES6에서 새로 등장한 Set, Map 등의 메서드에도 일부 존재한다.

그중 대표적인 배열 메서드인 forEach의 예를 살펴보자.

```jsx
var report = {
		sum : 0,
		count : 0,
		add : function () {
			var args = Array.prototype.slice.call(arguments);
			args.forEach(function (entry) {
					this.sum += entry;
					++this.count;
				}, this);
		},
		average: function() {
			return this.sum / this.count;
		}
};
report.add(60, 85, 95);
console.log(report.sum, report.count, report.average()); // 240 3 80
```

report 객체에서 sum, count 프로퍼티가 있고, add, average 메서드가 있다.

5번째 줄에서 add 메서드는 arguments를 배열로 변환해서 args 변수에 담고, 

6번째 줄에서는 이 배열을 순회하면서 콜백 함수를 실행하는데, 이때 콜백 함수 내부에서의 this는 forEach 함수의 두번째 인자로 전달해준 this(9번째 줄)가 바인딩 된다.

15번째 줄에서 60, 85, 95를 인자로 삼아 add 메서드를 호출하면 이 세 인자를 배열로 만들어 forEach 메서드가 실행된다. 콜백 함수 내부에서의 this는 add 메서드에서의 this가 전달된 상태이므로 add 메서드의 this(report)를 그대로 가리키고 있다. 따라서 배열의 세 요소를 순회하면서 report.sum 값 및 report.count 값이 차례로 바뀌고, 순회를 마친 결과 report.sum에는 240이, report.count에는 3이 담기게 된다.

이 밖에도 thisArg를 인자로 받는 메서드는 많이 있다.

```jsx
Array.prototype.forEach(callback[, thisArg])
Array.prototype.map(callback[, thisArg])
Array.prototype.filter(callback[, thisArg])
Array.prototype.some(callback[, thisArg])
Array.prototype.every(callback[, thisArg])
Array.prototype.find(callback[, thisArg])
Array.prototype.findIndex(callback[, thisArg])
Array.prototype.flatMap(callback[, thisArg])
Array.prototype.from(callback[, thisArg])
Set.prototype.forEach(callback[, thisArg])
Map.prototype.forEach(callback[, thisArg])
```

                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   

## This와 바인딩

EC(Execution Context)가 생성될 때마다 this의 바인딩이 일어나며 우선순위 순으로 나열해보면 다음과 같다.

- `new` 를 사용했을 때 해당 객체로 바인딩된다.

```jsx
var name = "global";
function Func() {
  this.name = "Func";
  this.print = function f() { console.log(this.name); };
}
var a = new Func();
a.print(); // Func
```

- `call`, `apply`, `bind` 와 같은 명시적 바인딩을 사용했을 때 인자로 전달된 객체에 바인딩된다.

```jsx
function func() {
  console.log(this.name);
}

var obj = { name: 'obj name' };
func.call(obj); // obj name
func.apply(obj); // obj name
(func.bind(obj))(); // obj name
```

- 객체의 메소드로 호출할 경우 해당 객체에 바인딩된다.

```jsx
var obj = {
  name: 'obj name',
  print: function p() { console.log(this.name); }
};
obj.print(); // obj name
```

- 그 외의 경우
    - strict mode: `undefined` 로 초기화된다.
    - 일반: 브라우저라면 `window` 객체에 바인딩 된다.

**참고**

- [김정환 블로그, 자바스크립트 this 바인딩 우선순위](http://jeonghwan-kim.github.io/2017/10/22/js-context-binding.html#%EC%95%94%EC%8B%9C%EC%A0%81-%EB%B0%94%EC%9D%B8%EB%94%A9%EA%B3%BC-new-%EB%B0%94%EC%9D%B8%EB%94%A9%EC%9D%98-%EC%9A%B0%EC%84%A0%EC%88%9C%EC%9C%84)
- [How does the “this” keyword work?](https://stackoverflow.com/questions/3127429/how-does-the-this-keyword-work)