```
**SUMMARY**

* 콜백 함수는 다른 코드에 인자로 넘겨줌으로써 그 제어권도 함께 위임한 함수이다.

* 제어권을 넘겨받은 코드는 다음과 같은 제어권을 가진다.

	1) 콜백 함수를 호출하는 시점을 스스로 판단해서 실행한다.

	2) 콜백 함수를 호출할 때 인자로 넘겨줄 값들 및 그 순서가 정해져 있다.
		이 순서를 따르지 않고 코드를 작성하면 엉뚱한 결과를 얻게 된다.

	3) 콜백 함수의 this가 무엇을 바라보도록 할지가 정해져 있는 경우도 있다.
		정하지 않은 경우에는 전역객체를 바라본다.
		사용자 임의로 this를 바꾸고 싶을 경우 bind 메서드를 활용하면 된다.

* 어떤 함수에 인자로 메서드를 전달하더라도 이는 결국 함수로서 실행된다.

* 비동기 제어를 위해 콜백 함수를 사용하다보면 콜백 지옥에 빠지기 쉽다.
	최근의 ES에는 Promise, Generator, async/await 등
	콜백 지옥에서 벗어날 수 있는 훌륭한 방법들이 등장하고 있다.
```

콜백함수는 다른 코드의 인자로 넘겨주는 함수이다.

콜백 함수를 넘겨받은 코드는 이 콜백 함수를 필요에 따라 적절한 시점에 실행할 것이다.

콜백 함수는 '제어권'과 관련이 깊다.

callback은 '부르다', '호출하다'는 의미인 call과, '뒤돌아오다', '되돌다'는 의미인 back의 합성어로,

'되돌아 호출해달라'는 명령이다.

콜백 함수는 다른 코드(함수 또는 메서드)에게 인자를 넘겨줌으로써 그 제어권도 함께 위임한 함수.

콜백 함수를 위임받은 코드는 자체적인 내부 로직에 의해 이 콜백 함수를 적절한 시점에 실행할 것.

## 제어권

몇 가지 예제를 통해 구체적으로 살펴보자

### 호출 시점

```jsx
var count = 0;
var timer = setInterval(function () {
	console.log(count);
	if (++count > 4) clearInterval(timer);
}, 300);
```

1번째 줄에서 count 변수를 선언하고 0을 할당했다.

2번째 줄에서는 timer 변수를 선언하고 여기에 setInterval을 실행한 결과를 할당했다.

setInterval을 호출할 때 두개의 매개변수를 전달했는데, 그중 첫번째는 익명 함수이고 두번째는 300이라는 숫자.

setInterval의 구조를 살펴보면 다음과 같다.

```jsx
var intervalID = scope.setInterval(func, delay[, param1, param2, ...]);
```

우선 scope에는 Window 객체 또는 Worker의 인스턴스가 들어올 수 있습니다.

두 객체 모두 setInterval 메서드를 제공하기 때문인데, 일반적인 브라우저 환경에서는 window를 생략해서 함수처럼 사용 가능할 것

매개변수로는 func, delay 값을 반드시 전달해야하고, 세 번째 매개변수부터는 선택적이다.

func는 함수이고, delay는 밀리초 단위의 숫자이며, 나머지는 func 함수를 실행할 때 매개변수로 전달할 인자이다.

func에 넘겨준 함수는 매 delay마다 실행되며, 그 결과 어떠한 값도 리턴하지 않는다.

setInterval를 실행하면 반복적으로 실행되는 내용 자체를 특정할 수 있는 고유한 ID값이 반환된다.

이를 변수에 담는 이유는 반복 실행되는 중간에 종료할 수 있게 하기 위함이다.

```jsx
var count = 0;
var cbFunc = function () {
	console.log(count);
	if (++count > 4 ) clearInterval(timer);
};
var timer = setInterval(cbFunc, 300);

	// --- 실행 결과 --
	// 0 (0.3초)
	// 1 (0.6초)
	// 2 (0.9초)
	// 3 (1.2초)
	// 4 (1.5초)

```

timer 변수에는 setInterval의 ID 값이 담깁니다.

setInterval에 전달한 첫 번째 인자인 cbFunc 함수(이 함수가 곧 콜백 함수이다)

는 0.3초마다 자동으로 실행될 것이다.

콜백 함수 내부에서는 count 값을 출력하고, count를 1만큼 증가시킨 다음,

그 값이 4보다 크면 반복 실행을 종료하라고 한다.

[코드 실행방식과 제어권](https://www.notion.so/8342bdf3b3bf47f2bb536c5174c6ba7b)

이 코드를 실행하면 콘솔창에는 0.3초에 한 번씩 숫자가 0부터 1씩 증가하며 출력되다가

4가 출력된 이후 종료된다.

setInterval이라고 하는 '다른 코드'에 첫번째 인자로서 cbFunc 함수를 넘겨주자 제어권을 넘겨받은 setInterval이 스스로의 판단에 따라 적절한 시점에 (0.3초 마다) 이 익명 함수를 실행했습니다.

이처럼 콜백 함수의 제어권을 넘겨받은 코드는 콜백 함수 호출 시점에 대한 제어권을 가진다.

### 인자

```jsx
var newArr = [10,20,30].map(function (currentValue, index) {
	console.log(currentValue, index);
	return currentValue + 5;
});
console.log(newArr);

// --실행 결과--
// 10 0
// 20 1
// 30 2
// [10, 25, 35]
```

1번째 줄에서 newArr 변수를 선언하고 우항의 결과를 할당했다. 

5번째 줄에서 그 결과를 확인하고자 한다.

1번째 줄의 우항은 배열 [10, 20, 30]에 map 메서드를 호출하고 있다.

이때 첫번째 매개변수로 익명 함수를 전달한다. 우선 map 메서드가 어떤 방식으로 동작하는지 살펴보자

Array의 prototype에 담긴 map메서드는 다음과 같은 구조로 이루어져 있다.

```jsx
Array.prototype.map(callback[, thisArg])
callback : function(currentValue, index, array)
```

map메서드는 첫 번째 인자로 callback 함수를 받고, 생략 가능한 두 번째 인자로 콜백 함수 내부에서

this로 인식할 대상을 특정할 수 있다.

thisArg를 생략할 경우에는 일반적인 함수와 마찬가지로 전역객체가 바인딩된다.

map메서드는 메서드의 대상이 되는 배열의 모든 요소들을 처음부터 끝까지 하나씩 꺼내어 콜백 함수를 반복 호출하고, 콜백함수의 실행 결과들을 모아 새로운 배열을 만든다.

콜백 함수의 첫 번째 인자에는 배열의 요소 중 현재값이, 두 번째 인자에는 현재값의 인덱스가,

세 번째 인자에는 map 메서드의 대상이 되는 배열 자체가 담긴다.

이를 바탕으로 예제 4-3을 다시 살펴보자. 배열 [10,20,30]의 각 요소를 처음부터 하나씩 꺼내어 콜백 함수를 실행한다. 우선 첫 번째(인덱스 0)에 대한 콜백 함수는 currentValue에 10이, index에는 인덱스 0이 담긴채 실행될 것.

각 값을 출력한 다음, 15(10+5)를 반환할 것이다.

두번째에 대한 콜백 함수는 currentValue에 20에, index에는 1이 담긴체 실행될 것이므로 25를 반환한다.

같은 방식으로 세번째에 대한 콜백 함수까지 실행을 마치고 나면, 이제는 [15, 25, 30]이라는 새로운 배열이 만들어져서 변수 newArr에 담기고, 이 값이 5번째 줄에서 출력될 것이다.

제이쿼리의 메서드들은 기본적으로 첫 번째 인자에 index가, 두 번째 인자가 currentValue가 온다.

제이쿼리에 익숙한 독자라면 이런 순서가 더 자연스럽게 느껴질 수 있다.

만약 map 메서드를 제이쿼리의 방식처럼 순서를 바꾸어 사용해보면 어떨까?

- 인자의 순서를 임의로 바꾸어 사용한 경우

```jsx
var newArr2 = [10, 20, 30].map(function (index, currentValue) {
	console.log(index, currentValue);
	return currentValue + 5;
});
console.log(newArr);

// --실행 결과--
// 10 0
// 20 1
// 30 2
// [5, 6, 7]
```

사람은 'index', 'currentValue' 등을 단어로 접근하기 때문에 순서를 바꾸더라도 각 단어의 의미가 바뀌지 않으니까 문제 없을 것이라고 생각하기 쉽지만, 사실 저 단어들은 사용자가 명명한 것일 뿐이다.

컴퓨터는 그저 첫 번째, 두 번째의 순서에 의해서만 각각을 구분하고 인식할 것이다.

우리가 첫 번째 인자의 이름을 'index'로 하건, 'currentValue'로 하건 'nothing'으로 칭하건 관계없이

그냥 순회 중인 배열 중 현재 요소의 값을 배정하는 것이다.

따라서 위의 코드를 실행해보면 5번째 줄에서 [5,6,7]이라는 전혀 다른 결과가 나온다.

currentValue라고 명명한 인자의 위치가 두 번째라서 컴퓨터가 여기에 인덱스 값을 부여했기 때문이다.

map 메서드를 호출해서 원하는 배열을 얻으려면 map 메서드에 정의된 규칙에 따라 함수를 작성해야 한다.

map 메서드에 정의된 규칙에는 콜백 함수의 인자로 넘어올 값들 및 그 순서도 포함되어 있다.

콜백 함수를 호출하는 주체가 사용자가 아닌 map 메서드이므로 map 메서드가 콜백 함수를 호출할 때

인자에 어떤 값들을 어떤 순서로 넘길지가 전적으로 map 메서드에게 달린 것이다.

이처럼 콜백 함수의 제어권을 넘겨받은 코드는 콜백 함수를 호출할 때 인자에 어떤 값들을 어떤 순서로 넘길 것인지에 대한 제어권을 가진다.

### this

"콜백 함수도 함수이기 때문에 기본적으로 this가 전역객체를 참조하지만,

제어권을 넘겨받을 코드에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우에는 그 대상을 참조하게 된다"

별도의 this를 지정하는 방식 및 제어권에 대한 이해를 높이기 위해 map 메서드를 직접 구현해 보자.

어떤 값을 받아 어떤 식으로 처리하는지는 이미 소개했으니, 이런 요구사항에 부합하도록 만들면 될 것.

예외처리 내용은 모두 배재하고 동작원리를 이해하는 것을 목표로 다음 코드를 보자.

```jsx
Array.prototype.map = function (callbavck, thisArg) {
	var mappedArr = [];
	for (var i = 0 ; i<this.length; i++){
		var mappedValue = callback.call(thisArg || window, this[i], i, this);
		mappedArr[i] = mappedValue;
	}
	return mappedArr;
};
```

메서드 구현의 핵심은 call/apply 메서드에 있다. this에는 thisArg 값이 있을 경우에는 그 값을,

없는 경우에는 전역객체를 저장하고, 첫 번째 인자에는 메서드의 this가 배열을 가리킬 것이므로

배열의 i번째 요소 값을, 두 번째 인자에는 i 값을, 세 번째 인자에는 배열 자체를 지정해 호출한다.

그 결과가 변수 mappedValue에 담겨 mappedArr의 i번째 인자에 할당된다.

이제 this에 다른 값이 담기는 이유를 정확히 알 수 있다.

바로 제어권을 넘겨받을 코드에서 call/apply 메서드의 첫 번째 인자에 콜백 함수 내부에서의 this가 될 대상을 명시적으로 바인딩하기 때문이다.

```jsx
setTimeout(function () { console.log(this); }, 300);

[1, 2, 3, 4, 5].forEach(function (x) {
	console.log(this);
});

document.body.innerHTML += `<button id="a">클릭</button>`;
document.body.querySelector('#a');
		.addEventListener('click', function (e) {
			console.log(this, e);
		}
};
```

각각 콜백 함수 내에서의 this를 살펴보면, 우선 (1)의 setTimeout은 내부에서 콜백 함수를 호출할 때

call 메서드의 첫 번째 인자에 전역객체를 넘기기 때문에 콜백 함수 내부에서의 this가 전역객체를 가리킨다.

(2)의 forEach는 4-2-5절의 '별도의 인자로 this를 받는 경우'에 해당하지만 별도의 인자로 this를 넘겨주지 않았기 때문에 전역객체를 가리키게 된다.

(3)의 addEventListener는 내부에서 콜백 함수를 호출할 때 call 메서드의 첫 번째 인자에 addEventListener메서드의 this를 그대로 넘기도록 정의돼 있기 때문에 콜백 함수 내부에서의 this가 addEventListener를 호출한 주체인 HTML 엘리먼트를 가리키게 된다.

## 콜백 함수는 함수다

콜백 함수는 함수이다.

즉, 콜백 함수로 어떤 객체의 메서드를 전달하더라도 그 메서드는 메서드가 아닌 함수로서 호출된다.

```jsx
var obj = {
	vals: [1, 2, 3],
	logValues: function(v, i) {
		console.log(this, v, i);
	}
};

obj.logValues(1, 2);              // { vals: [1, 2, 3], logValues: f } 1 2
[4, 5, 6].forEach(obj.logValues); // Window { ... } 4 0
																	// Window { ... } 5 1
																	// Window { ... } 6 2
```

obj 객체의 logValues는 메서드로 정의됐다. 7번째 줄에서는 이 메서드의 이름 앞에 점이 있으니

메서드로서 호출될 것이다. 따라서 this는 obj를 가리키고, 인자로 넘어온 1, 2가 출력된다.

한편 8번째 줄에서는 이 메서드를 forEach 함수의 콜백 함수로서 전달했다.

obj를 this 로 하는 메서드를 그대로 전달한 것이 아니라, obj.logValues가 가리키는 함수만 전달한 것.

이 함수는 메서드로서 호출할 때가 아닌 한 obj와의 직접적인 연관이 없어진다.

forEach에 의해 콜백이 함수로서 호출되고, 별도로 this를 지정하는 인자를 지정하지 않았으므로

함수 내부에서의 this는 전역객체를 바라보게 된다.

그러니까 어떤 함수의 인자에 객체의 메서드를 전달하더라도 이는 결국 메서드가 아닌 함수일 뿐이다.

이 차이를 정확히 이해하는 것이 중요하다.

## 콜백 함수 내부의 this에 다른 값 바인딩하기

객체의 메서드를 콜백 함수로 전달하면 해당 객체를 this로 바라볼 수 없게 된다는 점은 이해할 것.

그럼에도 콜백 함수 내부에서 this가 객체를 바라보게 하고 싶다면 어떻게 해야 할 까?

별도의 인자로 this를 받는 함수의 경우에는 여기에 원하는 값을 넘겨주면 되지만, 그렇지 않은 경우에는 this의 제어권도 넘겨주게 되므로 사용자가 임의로 값을 바꿀수 없다.

그래서 전통적으로는 this를 다른 변수에 담아 콜백 함수로 활용할 함수에서는 this대신 그 변수를 사용하도록 하고,

이를 클로저로 만드는 방식이 많이 썼다.

- 전통적인 방식

```jsx
var obj1 = {
	name : 'obj1',
	func : function () {
		var self = this;
		return function () {
			console.log(self.name);
		};
	}
};
var callback = obj1.func();
setTimeout(callback, 1000);
```

obj1.func 메서드 내부에서 self 변수에 this를 담고, 익명 함수를 선언과 동시에 반환했다.

이제 10번째 줄에서 obj1.func를 호출하면 앞서 선언한 내부함수가 반환되어 callback 변수에 담긴다.

11번째 줄에서 이 callback을 setTimeout 함수에 인자로 전달하면 1초뒤 callback이 실행되면서 'obj1'를 출력할 것이다.

이 방식은 실제로 this를 사용하지 않을뿐더러 번거롭다.

이를 보완하기 위해 ES5에서 등장한 bind 메서드를 사용할 수 있다.

```jsx
var obj1 = {
	name : 'obj1',
	func : function () {
			console.log(this.name);
		};
	}
};

setTimeout(obj1.func.bind(obj1), 1000);

var obj2 = { name : 'obj2' };
setTimeout(obj1.func.bind(obj2), 1500);
```

## 콜백 지옥과 비동기 제어

콜백 지옥은 콜백 함수를 익명 함수로 전달하는 과정이 반복되어 코드의 들여쓰기 수준이 감당하기 힘들 정도로 깊어지는 현상으로, 자바스크립트에서 흔히 발생하는 문제이다.

주로 이벤트 처리나 서버 통신과 같이 비동기적인 작업을 수행하기 위해 이런 형태가 자주 등장하곤 하는데,

가독성이 떨어질 뿐더러 코드를 수정하기도 어렵다.

비동기는 동기의 반댓말이다. 동기적인 코드는 현재 실행 중인 코드가 완료된 후에야 다음 코드를 실행하는 방식.

반대로 비동기적인 코드는 현재 실행 중인 코드의 완료 여부와 무관하게 즉시 다음 코드로 넘어간다.

CPU의 계산에 의해 즉시 처리가 가능한 대부분의 코드는 동기적인 코드이다.

계산식이 복잡해서 CPU가 계산하는데 시간이 많이 필요한 경우라 하더라도 이는 동기적인 코드이다.

반면 사용자의 요청에 의해 특정 시간이 경과되기 전까지 어떤 함수의 실행을 보류한다거나(setTimeout),

사용자의 직접적인 개입이 있을 때 비로서 어떤 함수를 실행하도록 대기한다거나(addEventListener),

웹브라우저 자체가 아닌 별도의 대상에 무언가를 요청하고 그에 대한 응답이 왔을 때 비로소 어떤 함수를

실행하도록 대기하는 등(XMLHttpRequest), 별도의 요청, 실행 대기, 보류 등과 관련된 코드는 비동기적인 코드이다.

그런데 현대의 자바스크립트는 웹의 복잡도가 높아진 만큼

비동기적인 코드의 비중이 예전보다 훨씬 높아진 상황이다.

그와 동시에 콜백 지옥에 빠지기도 훨씬 쉬워진 셈.

지난 십수년간 자바스크립트 진영은 비동기적인 일련의 작업을 동기적으로,

혹은 동기적인 것으로 보이게끔 처리해주는 장치를 마련하고자 끊임없이 노력해왔다.

ES6에서는 Promise, Generator 등이 도입됐고, ES2017에서는 async/await가 도입되었다.

이들을 이용해 위 코드를 수정한 내용을 각각 간략하게 소개하겠다.

### Promise

```jsx
new Promise(function (resolve) {
	setTimeout(function () {
		var name = '에스프레소';
		console.log(name);
		resolve(name);
	}, 500);
}).then(function (prevName) {
	return new Promise(function (resolve) {
		setTimeout(function () {
			var name = prevName + ', 아메리카노';
			console.log(name);
			resolve(name);
		}, 500);
	});
}).then(function (prevName) {
	return new Promise(function (resolve) {
		setTimeout(function () {
			var name = prevName + ', 카페모카';
			console.log(name);
			resolve(name);
		}, 500);
	});
}).then(function (prevName) {
	return new Promise(function (resolve) {
		setTimeout(function () {
			var name = prevName + ', 카페라떼';
			console.log(name);
			resolve(name);
		}, 500);
	});
});
```

첫 번째로 ES6의 Promise를 이용한 방식이다.

new 연산자와 함께 호출한 Promise의 인자로 넘겨주는 콜백 함수는 호출할 때 바로 실행되지만

그 내부에 resolve 또는 reject 함수를 호출하는 구문이 있을 경우

둘 중 하나가 실행되기 전까지는 다음(then) 또는 오류 구문(catch)로 넘어가지 않는다.

따라서 비동기 작업이 완료될 때 비로소 resolve 또는 reject를 호출하는 방법으로 비동기 작업의 동기적 표현이 가능하다.

```jsx
var addCoffee = function (name) {
	return function (prevName) {
		return new Promise(function (resolve) {
			setTimeout(function () {
				var newName = prevName ? (prevName + ', ' + name) : name;
				console.log(newName);
				resolve(newName);
			}, 500);
		});
	};
};

addCoffee('에스프레소')()
	.then(addCoffee('아메리카노'))
	.then(addCoffee('카페모카'))
	.then(addCoffee('카페라떼'))
```