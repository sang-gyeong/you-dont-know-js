```
_SUMMARY_

**클로저란 어떤 함수에서 선언한 변수를 참조하는 내부함수를 외부로 전달한 경우,
함수의 실행 컨텍스트가 종료된 후에도 해당 변수가 사라지지 않는 현상이다.**

내부함수를 외부로 전달하는 방법에는 함수를 return하는 경우뿐 아니라 콜백으로 전달하는 경우도 포함된다.

클로저는 그 본질이 메모리를 계속 차지하는 개념이므로 더는 사용하지 않게 된 클로저에 대해서는
메모리를 차지하지 않도록 관리해줄 필요가 있다.

클로저는 다양한 곳에서 활용할 수 있는 중요한 개념이다.
(콜백함수 내부에서 외부함수 사용, 정보 은닉, 부분 적용 함수, 커링함수 등)
```

## 클로저의 의미 및 원리 이해

클로저는 여러 함수형 프로그래밍 언어에서 등장하는 보편적인 특성이다.

다양한 서적에서 소개하는 클로저의 개념은 다음과 같다.

- 함수를 선언할 때 만들어지는 유효범위가 사라진 후에도 호출할 수 있는 함수
- 이미 생명 주기상 끝난 외부 함수의 변수를 참조하는 함수
- 자신이 생성될 때의 스코프에서 알 수 있었던 변수들 중 언젠가 자신이 실행될 때 사용할 변수들만을 기억하여 유지시키는 함수
- 함수와 함수가 선언될 당시의 lexical environment의 상호관계에 따른 현상

클로저 개념을 이해하기 위해

외부 함수에서 변수를 선언하고 내부함수에서 해당 변수를 참조하는 형태의 간단한 코드를 작성해볼 것.

```jsx
var outer = function () {
	var a = 1;
	var inner = function () {
		console.log(++a);
	};
	inner();
};
outer();   //2
```

- outer 함수에서 변수 a를 선언했고, outer의 내부함수인 inner함수에서 a의 값을 1만큼 증가시킨 다음 출력한다.
- inner함수 내부에서는 a를 선언하지 않았기 때문에 environmentRecord에서 값을 찾지 못하므로 outerEnvironmentReference에 지정된 상위 컨텍스트인 outer의 LexicalEnvironment에 접근해서 다시 a를 찾는다. (그러므로 2가 출력된다)
- outer 함수의 실행 컨텍스트가 종료되면 LexicalEnvironment에 저장된 식별자들(a, inner)에 대한 참조를 지운다. 그러면 각 주소에 저장돼 있던 값들은 자신을 참조하는 변수가 하나도 없게 되므로 가비지 컬렉터의 수집 대상이 될 것이다.

이번엔 조금 바뀐 예시를 들어보자.

```jsx
// 예제 2 : 외부 함수의 변수를 참조하는 내부함수
var outer = function() {
	var a = 1;
	var inner = function () {
		return ++a;
	};
	return inner();
};
var outer2 = outer();
console.log(outer2);   //2
```

- 이번에도 inner 함수 내부에서 외부변수인 a를 사용했다. 그런데 6번째 줄에서는 inner함수를 실행한 결과를 리턴하고 있으므로 결과적으로 outer함수의 실행 컨텍스트가 종료된 시점에는 a 변수를 참조하는 대상이 없어진다.
- 앞선 예시와 마찬가지로 a, inner 변수의 값들은 언젠가 가비지 컬렉터에 의해 소멸될 것이다.
- 역시 일반적인 함수 및 내부함수에서의 동작과 차이가 없다.

앞선 두가지의 예시는 outer 함수의 실행 컨텍스트가 종료되기 이전에 inner 함수의 실행 컨텍스트가 종료돼 있으며, 이후 별도로 inner 함수를 호출할 수 없다는 공통점이 있다.

그렇다면 outer의 실행 컨텍스트가 종료된 후에도 inner 함수를 호출할 수 있게 만들면 어떨까?

```jsx
// 예제 3 : 외부 함수의 변수를 참조하는 내부함수

var outer = function() {
	var a = 1;
	var inner = function () {
		return ++a;
	};
	return inner;         //****
};
var outer2 = outer();
console.log(outer2());  // 2
console.log(outer2());  // 3
```

- 이번에는 6번째 줄에서 inner 함수의 실행 결과가 아닌 **inner 함수 자체를 반환했다.**
- 그러면 outer 함수의 실행 컨텍스트가 종료될때 outer2 변수는 outer의 실행 결과인 inner 함수를 참조하게 될 것이다. 이후 9번째에서 outer2를 호출하면 앞서 반환된 함수인 inner가 실행될 것이다.
- inner 함수의 실행 컨텍스트의 environmentRecord에는 수집할 정보가 없다. outer-EnvironmentReference에는 inner함수가 선언된 위치의 LexicalEnvironment가 참조복사된다. inner함수는 outer함수 내부에서 선언됐으므로, outer 함수의 LexicalEnvironment가 담길 것이다.
- 이제 스코프 체이닝에 따라 outer에서 선언한 변수 a에 접근해서 1만큼 증가시킨 후 그 값인 2를 반환하고, inner 함수의 실행 컨텍스트가 종료된다. 10번째 줄에서 다시 outer2를 호출하면 같은 방식으로 a의 값을 2에서 3으로 1 증가시킨 후 3을 반환한다.

그런데 이상한 점이 있다. 

inner 함수의 실행 시점에는 outer 함수는 이미 실행이 종료된 상태인데 outer 함수의 LexicalEnvironment에 어떻게 접근할 수 있는 것인가? **이는 가비지 컬렉터의 동작 방식 때문이다.**

**가비지 컬렉터는 어떤 값을 참조하는 변수가 하나라도 있다면 그 값은 수집 대상에 포함시키지 않는다.**

- 위의 예제의 outer함수는 실행 종료 시점에 inner 함수를 반환한다.
- 외부 함수인 outer의 실행이 종료되더라도 내부 함수인 inner함수는 언젠가 outer2를 실행함으로써 호출될 가능성이 열린 것.
- 언젠가 inner 함수의 실행 컨텍스트가 활성화되면 outerEnvironmentReference가 outer함수의 LexicalEnvironment를 필요로 할 것이므로 수집대항에서 제외된다.
- 그 덕에 inner함수가 이 변수에 접근할 수 있는 것.

클로저는 **어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상**이라고 했다.

- 예제 1과 2에서는 일반적인 함수의 경우와 마찬가지로 outer의 LexicalEnvironment에 속하는 변수가 모두 가지비 컬렉팅 대상에 포함된 반면, 예제 3의 경우 변수 a가 대상에서 제외된다.
- 이처럼 함수의 실행 컨텍스트가 종료된 후에도 LexicalEnvironment가 가비지 컬렉터의 수집 대상에서 제외되는 경우는 3과 같이 지역변수를 참조하는 내부함수가 외부로 전달된 경우가 유일하다.
- 그러니까 "어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상"이란 "외부 함수의 LexicalEnvironment가 가비지 컬렉팅 되지 않는 현상"을 말하는 것이다.

이를 바탕으로 클로저의 정의를 내리면 다음과 같다.

> **클로저란 어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우,  A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상을 말한다.**

여기서 한가지 주의할 점이 있다. 바로 '외부로 전달'이 곧 return만을 의미하는 것은 아니라는 것이다.

아래의 코드를 통해 확인해보자.

- **return 없이도 클로저가 발생하는 다양한 경우**

```jsx
// (1) setInterval / setTimeout

(function () {
	var a = 0;
	var intervalId = null;
	var inner = function () {
		if (++a >= 10) {
			clearInterval(intervalId);
		};
		console.log(a);
	};
	intervalId = setInterval(inner, 1000);
})();
```

```jsx
// (2) eventListener

(function () {
	var count = 0;
	var button = document.createElement('button');
	button.innerText = 'click';
	button.addEventListener('click', function () {
		console.log(++count, 'times clicked');
	});
	document.body.appendChild(button);
})();
```

(1)은 별도의 외부객체인 window의 메서드(setTimeout 또는 setInterval)에 전달할 콜백 함수 내부에서 지역변수를 참조한다.

(2)는 별도의 외부객체인 DOM의 메서드(addEventListener)에 등록할 handler 함수 내부에서 지역변수를 참조한다. 

두 상황 모두 지역변수를 참조하는 내부함수를 외부에 전달했기 때문에 클로저이다.

## 클로저와 메모리 관리

클로저는 객체지향과 함수형 모두를 아우르는 중요한 개념이다.

메모리 누수의 위험을 이유로 클로저 사용을 조심해야 한다거나 심지어 지양해야 한다고 주장하는 사람들도 있지만 **메모리 소모는 클로저의 본질적인 특성일 뿐이다. 오히려 이러한 특성을 정확히 이해하고 잘 활용하도록 노력해야 한다.**

'메모리 누수'라는 표현은 개발자의 의도와 달리 어떤 값의 참조 카운터가 0이 되지 않아 GC의 수거 대상이 되지 않는 경우에는 맞는 표현이지만, 개발자가 의도적으로 참조 카운트를 0이 되지 않게 설계한 경우에는 '누수'라고 할 순 없다. 의도대로 설계한 '메모리 소모'에 대한 관리법만 잘 파악해서 적용하는 것이 중요하다.

관리법은 간단하다.

클로저는 어떤 필요에 의해 의도적으로 함수의 지역변수를 메모리를 소모하도록 함으로써 발생한다.

그렇다면 그 필요성이 사라진 시점에는 더는 메모리를 소모하지 않게 해주면 된다.

참조 카운터를 0으로 만들면 언젠가는 GC가 수거해갈 것이고, 이때 소모됐던 메모리가 회수될 것이다.

참조 카운트를 0으로 만드는 방법은 ? 

**→ 식별자에 참조형이 아닌 기본형 데이터(보통 null이나 undefined)를 할당하면 된다.**

```jsx
// (1) return에 의한 클로저의 메모리 해제

var outer = (function() {
	var a = 1;
	var inner = function () {
		return ++a;
	};
	return inner;
})();
console.log(outer());
console.log(outer());
outer = null; // outer 식별자의 inner 함수 참조를 끊음.
```

```jsx
// (2) setInterval에 의한 클로저의 메모리 해제

(function () {
	var a = 0;
	var intervalId = null;
	var inner = function () {
		if (++a >= 10) {
			clearInterval(intervalId);
			inner = null;  // inner 식별자의 함수 참조를 끊음.
		}
		console.log(a);
	};
	intervalId = setInterval(inner, 1000);
})();
```

```jsx
// (3) eventListener에 의한 클로저의 메모리 해제
(function () {
	var count = 0;
	var button = document.createElement('button');
	button.innerText = 'click';

	var clickHandler = function () {
		console.log(++count, 'times clicked');
		if (count >= 10) {
			button.removeEventListener('click', clickHandler);
			clickHandler = null;
		}
	};
	button.addEventListener('click', clickHandler);
	document.body.appendChild(button);
})();
```

## 클로저 활용 사례

### 1. 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때

- **콜백 함수를 내부함수로 선언해서 외부변수를 직접 참조하는 방법**

```jsx
var fruits = ['apple', 'banana', 'peach'];
var $ul = document.createElement('ul');       // (공통 코드)

fruits.forEach(function (fruit) {             // (A)
	var $li = document.createElement('li');
	$li.innerText = fruit;
	$li.addEventListener('click', function () { // (B)
		alert('your choice is ' + fruit);
	});
	$ul.appendChild($li);
});
document.body.appendChild($ul);
```

4번째 줄의 forEach메서드에 넘겨준 익명의 콜백함수 (A)는 그 내부에서 외부 변수를 사용하지 않고 있으므로 클로저가 없지만, 7번째 줄의 addEventListener에 넘겨준 콜백 함수(B)에는 fruit이라는 외부 변수를 참조하고 있으므로 클로저가 있다.

(A)는 fruits의 개수만큼 실행되며, 그때마다 새로운 실행 컨텍스트가 활성화될 것이다.

A의 실행 종료 여부와 무관하게 클릭 이벤트에 의해 각 컨텍스트의 (B)가 실행될 때는 (B)의 outerEnvironmentReference가 (A)의 LexicalEnvironment를 참조하게 될 것이다.

따라서 최소한 (B) 함수가 참조할 예정인 변수 fruit에 대해서는 (A)가 종료된 후에도 GC대상에서 제외되어 계속 참조 가능할 것이다.

그런데 (B) 함수의 쓰임새가 콜백 함수에 국한되지 않는 경우라면 반복을 줄이기 위해 (B)를 외부로 분리하는 편이 나을수 있을 것이다. 즉 fruit를 인자로 받아 출력하는 형태를 말한다. 그렇게 바꾸어보자.

```jsx
...

var alertFruit = function (fruit) {
	alert('your choice is ' + fruit);
};

fruits.forEach(function (fruit) {             
	var $li = document.createElement('li');
	$li.innerText = fruit;
	$li.addEventListener('click', alertFruit);
	$ul.appendChild($li);
});
document.body.appendChild($ul);
alertFruit(fruits[1]);
```

위의 예제에서는 공통 함수로 쓰고자 콜백 함수를 외부로 꺼내어 alertFruit라는 변수에 담았다.

이제 alertFruit을 직접 실행할 수 있다. 또한 14번째 줄에서는 정상적으로 'banana'에 대한 얼럿이 실행된다.

그런데 각 li를 클릭하면 클릭한 대상의 과일명이 아닌 [object MouseEvent]라는 값이 출력된다. 콜백 함수의 인자에 대한 제어권을 addEventListener가 가진 상태이며, addEventListener는 콜백 함수를 호출할 때 첫 번째 인자에 '이벤트 객체'를 주입하기 때문이다. 이 문제는 bind 메서드를 활용하면 손쉽게 해결할 수 있다.

- **bind를 활용**

bind 메서드로 값을 직접 넘겨준 덕분에 클로저는 발생하지 않게된 반면 여러가지 제약사항이 따르게 됨.

```jsx
...

fruits.forEach(function (fruit) {             
	var $li = document.createElement('li');
	$li.innerText = fruit;
	$li.addEventListener('click', alertFruit.bind(null, fruit));
	$ul.appendChild($li);
});
...
```

다만 이렇게 하면 이벤트 객체가 인자로 넘어오는 순서가 바뀌는 점 및 함수 내부에서의 this가 원래의 그것과 달라지는 점은 감안해야 한다. 이런 변경사항이 발생하지 않게끔 하면서 이슈를 해결하기 위해서는 bind메서드가 아닌 다른 방식으로 풀어내야만 한다. 여기서 다른 방식이란 고차함수를 활용하는 것으로, 함수형 프로그래밍에서 자주 쓰이는 방식이기도 하다.

- **고차함수 활용 -** **콜백 함수를 고차함수로 바꿔서 클로저를 적극적으로 활용한 방안 ⭐️⭐️⭐️**

```jsx
...

var alertFruitBuilder = function (fruit) {
	return function () {
		alert('your choice is ' + fruit);
	};
};

fruits.forEach(function (fruit) {             
	var $li = document.createElement('li');
	$li.innerText = fruit;
	$li.addEventListener('click', alertFruitBuilder(fruit));
	$ul.appendChild($li);
});

...
```

- alertFruit 함수 대신 alertFruitBuilder라는 이름의 함수를 작성했다.
- 함수 내부에서는 다시 익명함수를 반환하는데, 이 익명함수가 바로 기존의 alertFruit함수이다.
- 그리고 alertFruitBuilder 함수를 실행하면서 fruit 값을 인자로 전달했다.
- 그러면 이 함수의 실행 결과가 다시 함수가 되며, 이렇게 반환된 함수를 리스너에 콜백 함수로서 전달할 것이다.
- 이후 언젠가 클릭 이벤트가 발생하면 비로소 이 함수의 실행 컨텍스트가 열리면서 alertFruitBuilder의 인자로 넘어온 fruit를 outerEnvironmentReference에 의해 참조할 수 있을 것이다.
- 즉 alertFruitBuilder의 실행 결과로 반환된 함수에는 클로저가 존재한다.

위 세 방법의 장단점을 각기 파악하고 구체적인 상황에 따라 어떤 방법을 도입하는 것이 가장 효과적인지 고민해보기 바란다. 

### 2. 접근 권한 제어(정보 은닉)

정보 은닉은 어떤 모듈의 내부 로직에 대해 외부로의 노출을 최소화해서

모듈간의 결합도를 낮추고 유연성을 높이고자 하는 현대 프로그래밍 언어의 중요한 개념 중 하나이다.

흔히 접근 권한에는 public, private, protected 세 종류가 있다. 

자바스크립트는 기본적으로 변수 자체에 이러한 접근 권한을 직접 부여하도록 설계되어 있지 않다.

그렇다고 접근 권한 제어가 불가능한 것은 아니다.

클로저를 이용하면 함수 차원에서 public한 값과 private한 값을 구분하는 것이 가능하다.

```jsx
var outer = function () {
	var a = 1;
	var inner = function () {
		return ++a;
	};
	return inner;
};
var outer2 = outer();
console.log(outer2());
console.log(outer2());
```

outer 함수를 종료할 때 inner 함수를 반환함으로써 outer함수의 지역변수인 a의 값을 외부에서도 읽을 수 있게 되었다.

이처럼 클로저를 활용하면 외부 스코프에서 함수 내부의 변수들 중 선택적으로 일부의 변수에 대해 접근 권한을 부여할 수 있다. 바로 return을 통해서!!

outer함수는 외부(전역 스코프)로부터 철저하게 격리된 닫힌 공간이다. 외부에서는 외부 공간에 노출돼있는 outer 변수를 통해 outer 함수를 실행할 수는 있지만, outer 함수 내부에는 어떠한 개입도 할 수 없다. 외부에서는 오직 outer 함수가 return한 정보에만 접근할 수 있다. return 값이 외부에 정보를 제공하는 유일한 수단인 셈.

그러니까 외부에 제공하고자 하는 정보들을 모아서 return하고, 내부에서만 사용할 정보들은 return하지 않는 것으로 접근 권한 제어가 가능한 것. return한 변수들은 공개멤버가 되고, 그렇지 않은 변수들은 비공개 멤버가 되는 것.

1. 함수에서 지역변수 및 내부함수 등을 생성한다.
2. 외부에 접근권한을 주고자 하는 대상들로 구성된 참조형 데이터(대상이 여럿일 때는 객체 또는 배열, 하나일 때는 함수)를 return 한다.

    → return한 변수들은 공개 멤버가 되고, 그렇지 않은 변수들은 비공개 멤버가 된다.

### 3. 부분 적용 함수

**n개의 인자를 받는 함수에 따라 미리 m개의 인자만 넘겨 기억시켰다가,**

**나중에 (n-m)개의 인자를 넘기면 비로소 원래 함수의 실행 결과를 얻을 수 있게끔 하는 함수.**

this를 바인딩해야 하는 점을 제외하면 앞서 살펴본 bind 메서드의 실행결과가 바로 부분 적용 함수이다.

- **bind 메서드를 활용한 부분 적용 함수**

```jsx
var add = function () {
	var result = 0;
	for (var i = 0; i < arguments.length; i++) {
		result += arguments[i];
	}
	return result;
};
var addPartial = add.bind(null, 1, 2, 3, 4, 5);
console.log(addPartial(6, 7, 8, 9, 10);
```

addPartial 함수는 인자 5개를 미리 적용하고, 추후 추가적으로 인자들을 전달하면 모든 인자를 모아 원래의 함수가 실행되는 부분 적용 함수이다.

add함수는 this를 사용하지 않으므로, bind 메서드만으로도 문제없이 구현되었다. 그러나 this의 값을 변경할 수 밖에 없기 때문에 메서드에서는 사용할 수 없을 것. this에 관여하지 않는 별도의 부분 적용 함수가 있다면 범용성 측면에서 더욱 좋을 것.

- 부분 적용 함수 구현(1)

```jsx
var partial = function() {
	var originalPartialArgs = arguments;
	var func = originalPartialArgs[0];
	if (typeof func !== 'function') {
		throw new Error('첫 번째 인자가 함수가 아닙니다.');
	}
	return function () {
		var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
		var restArgs = Array.prototype.slice.call(arguments);
		return func.apply(this, partialArgs.concat(restArgs);
```

### 4. 커링 함수

여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수로 나눠서 순차적으로 호출될 수 있게 체인 형태로 구성한 것.

앞서 살펴본 부분 적용 함수와 기본적인 맥락은 일치하지만 몇가지 다른점이 있다.

커링은 한번에 하나의 인자만 전달하는 것을 원칙으로 한다. 또한 중간 과정상의 함수를 실행한 결과는 그다음 인자를 받기 위해 대기만 할 뿐으로, 마지막 인자가 전달되기 전까지는 원본 함수가 실행되지 않는다. (부분 적용 함수는 여러 개의 인자를 전달할 수 있고, 실행 결과를 재실행할때 원본 함수가 무조건 실행된다.)

---

# **클로저(Closure)**

**함수가 속한 렉시컬 스코프를 기억하여**

**함수가 렉시컬 스코프 밖에서 실행될 때도 그 스코프에 접근할 수 있게 하는 기능**을 말한다.

```jsx
function outer() {
  var a = 2;
  function inner() {
    console.log(a);
  }
  return inner;
}

var func = outer();
func(); // 2
```

여기서 GC(Garbage Collector)가 `outer()` 의 참조를 없앨 것 같지만

내부함수인 `inner()` 가 해당 스코프의 변수인 a를 참조하고 있기 때문에 없애지 않는다.

따라서 스코프 외부에서 `inner()` 가 실행되도 해당 스코프를 기억하기 때문에 2를 출력하게 된다.

즉, 여기서 클로저는 `inner()` 가 되며 `func` 에 담겨 밖에서도 실행되고 렉시컬 스코프를 기억한다.

### **예제**

클로저를 사용하는 대표적인 예제는 역시 "반복문 클로저"이다.

```jsx
function func() {
  for (var i=1; i<5; i++) {
    setTimeout(function() { console.log(i); }, i*500);
  }
}
func(); // 5 5 5 5
```

코드의 의도한 바는 1부터 4까지 간격을 두고 출력하는 것이었지만 5가 4번 출력된다. 왜 이렇게 되는 것일까?

`setTimeout()` 을 반복문 안에서 돌리면 콜백함수가 계속해서 task queue에 쌓이게 되고

반복문이 끝나고 나서 call stack으로 돌아와서 실행된다.

콜백함수는 클로저이기 때문에 상위 스코프에게 `i` 의 값을 물어보고

상위 스코프인 `func` 의 스코프에선 `i` 가 5까지 증가했기 때문에 5가 4번 출력된다.

### **해결방법**

위 문제를 해결하기 위해선 2가지 방법이 있다.

- **새로운 함수 스코프로 해결하기**

```jsx
function func() {
  for (var i=1; i<5; i++) {
    (function (j) { 
      setTimeout(function() { console.log(j); }, j*500);
    })(i);
  }
}
func(); // 1 2 3 4
```

`setTimeout()` 을 IIFE(Immediately Invoked Function Expression, 즉시실행함수 표현식)로 감싸게 되면, 새로운 스코프를 형성하고 나중에 콜백함수가 `j` 를 참조할 때 그 시점의 `i` 값을 갖기 때문에

원하는 결과를 얻을 수 있게 된다.

- **블록 스코프로 해결하기**

```jsx
function func() {
  for (let i=1; i<5; i++) {
    setTimeout(function() { console.log(i); }, i*500);
  }
}
func(); // 1 2 3 4
```

함수 스코프가 아닌 블록 스코프를 갖는 `let` 을 사용하면 `for` 문 내의 새로운 스코프를 갖기 때문에

매 반복마다 새로운 `i` 가 선언되고 반복이 끝난 이후의 값으로 초기화가 된다.

따라서, `setTimeout()` 의 클로저인 콜백함수가 `i` 를 참조하기 위해 상위 스코프를 검색할 때

블록 스코프에서 매 반복마다 선언 및 초기화 된 `i` 를 참조하는 것이다.

실제 클로저를 사용하는 예를 알고 싶다면 [Stackoverflow의 답변](https://stackoverflow.com/a/39045098/11789111) 을 참고하자.

**참고**

- [TOAST, 자바스크립트의 스코프와 클로저](https://meetup.toast.com/posts/86)
- 카일 심슨, 『You don't know JS - 타입과 문법, 스코프와 클로저』, 최병현, 한빛미디어(2017)
- [https://poiemaweb.com/js-closure](https://poiemaweb.com/js-closure)