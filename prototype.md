# 프로토타입

자바스크립트는 프로토타입 기반 언어이다. 클래스 기반 언어에서는 '상속'을 사용하지만 프로토타입 기반 언어에서는 어떤 객체를 원형(prototype)으로 삼고, 이를 참조함으로써 상속과 비슷한 효과를 얻는다.

# 1. 프로토타입의 개념 이해

## 1-1. constructor, prototype, instance

일단 그림부터 보고 시작하자. 이 그림만 이해하면 프로토타입은 끝이다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/683a4a17-bace-4039-8a9d-dafb1447f6ad/KakaoTalk_Photo_2021-01-17-15-06-11.jpeg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/683a4a17-bace-4039-8a9d-dafb1447f6ad/KakaoTalk_Photo_2021-01-17-15-06-11.jpeg)

이 그림으로부터 전체 구조를 파악할 수 있고, 반대로 전체 구조로부터 이 그림을 도출해 낼 수 있으면 된다.

위 그림은 사실 다음의 코드 내용을 추상화 한 것이다.

```jsx
var instance = new Constructor();
```

이를 바탕으로 좀 더 구체적인 형태로 바꾸면 다음과 같다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9aee2c5f-55bb-4c87-bc33-5f375c532969/KakaoTalk_Photo_2021-01-17-15-09-37.jpeg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9aee2c5f-55bb-4c87-bc33-5f375c532969/KakaoTalk_Photo_2021-01-17-15-09-37.jpeg)

그림의 윗변(실선)의 왼쪽 꼭짓점에는 Constructor(생성자 함수)를,

오른쪽 꼭짓점에는 Constructor.prototype이라는 프로퍼티를 위치시켰다.

왼쪽 꼭짓점으로부터 아래를 향한 화살표 중간에 new가 있고, 화살표의 종점에는 instance가 있다.

오른쪽 꼭짓점으로부터 대각선 아래로 향하는 화살표의 종점에는 instance.__proto__ 이라는 프로퍼티를 위치시켰다.

두개의 그림을 번갈아보면서 흐름을 따라가보자.

- 어떤 생성자 함수(Constructor)를 new 연산자와 함께 호출하면
- Constructor에서 정의된 내용을 바탕으로 새로운 인스턴스가 생성된다.
- 이때 instance에는 __proto__라는 프로퍼티가 자동으로 부여되는데,
- 이 프로퍼티는 Constructor의 prototype이라는 프로퍼티를 참조한다.

prototype이라는 프로퍼티와 __proto__ 라는 프로퍼티가 새로 등장했는데,

이 둘의 관계가 프로퍼티 개념의 핵심.

prototype은 객체이다. 이를 참조하는 __proto__ 역시 객체.

prototype 객체 내부에는 인스턴스가 사용할 메서드를 저장한다.

그러면 인스턴스에서도 숨겨진 프로퍼티인 __proto__ 를 통해 이 메서드들에 접근할 수 있게 된다.

```
ES5.1 명세에서는 __proto__ 가 아니라 [[prototype]]이라는 명칭으로 정의돼있다.
__proto__라는 프로퍼티는 사실 브라우저들이 [[prototype]]을 구현한 대상에 지나지 않는다.

명세에는 또 instance.__proto__와 같은 방식으로 직접 접근하는 것은 허용하지 않고
오직 Object.getPrototypeOf(instance)/Refelect.getPrototypeOf(instance)를 통해서만
접근할 수 있도록 정의했었다.

그러나 이런 명세에도 불구하고 대부분의 브라우저들이 __proto__에 직접 접근하는 방식을 포기하지 않았고,
결국 ES6에서는 이를 브라우저에서 동작하는 레거시 코드에 대한 호환성 유지 차원에서 정식으로 인정하게된다.
다만 어디까지나 브라우저에서의 호환성을 고려한 지원일 뿐 권장되는 방식은 아니며,
브라우저가 아닌 다른 환경에서는 얼마든지 이 방식이 지원되지 않을 가능성이 열려있다.

여기에서는 이해의 편의를 위해 __proto__를 계속 사용하고자 한다만, 이를 학습목적으로만 이용하고
**실무에서는 가급적 __proto__를 사용하지 않기를 권한다.
대신 Object, getPrototypeOf()/Object.create()등을 이용하도록 하자.**
```

예를 들어, Person이라는 생성자 함수의 prototype에 getName이라는 메서드를 지정했다고 해보자.

```jsx
new Person = function (name) {
	this._name = name;
};
Person.prototype.getName = function() {
	return this._name;
};
```

이제 Person의 인스턴스는 __proto__ 프로퍼티를 통해 getName을 호출할 수 있다.

```jsx
var suzi = new Person('suzi');
suzi.__proto__.getName();         //undefined
```

왜냐하면 instance의 __proto__가 Constructor의 prototype 프로퍼티를 참조하므로

결국 같은 객체를 바라보기 때문.

```jsx
Person.prototype === suzi.__proto__  //true
```

메서드 호출 결과로 undefined가 나온 점에 주목해보자. 'Suzi'라는 값이 나오지 않은 것보다는

'에러가 발생하지 않았다'는 점이 우선이다.

어떤 변수를 실행해 undefined가 나왔다는 것은 이 변수가 '호출할 수 있는 함수'에 해당한다는 것을 의미한다.

만약 실행할 수 없는, 즉 함수가 아닌 다른 데이터 타입이었다면 TypeError가 발생했을 것이다.

그런데 값이 에러가 아닌 다른 값이 나왔으므로 getName이 실행됐음을 알 수 있고,

이로부터 getName이 함수라는 것이 입증되었다.

다음으로 함수 내부에서 어떤 값을 반환하는지를 살펴볼 차례이다.

this.name값을 리턴하는 내용으로 구성돼 있다. 

그렇다면 this에 의도와는 다른 값이 할당된 것은 아닐까, 하는 의심을 가질 수 있다. 이런 의심을 가지고 로그를 출력해 보거나 debugger를 지정하는 등으로 의심되는 사항을 하나하나 추적하다 보면 원인을 파악할 수 있을 것.

문제는 바로 this에 바인딩 된 대상이 잘못 지정됐다는 것이다.

어떤 함수를 '메서드로서' 호출할 때는 메서드명 바로 앞의 객체가 곧 this가 된다.

그러니까 thomas.__proto__.getName()에서 getName 함수 내부에서의 this는 thomas가 아니라

thomas.__proto__라는 객체가 되는 것. 이 객체 내부에서는 name 프로퍼티가 없으므로

'찾고자 하는 식별자가 정의돼 있지 않을 때는 Error 대신 undefined를 반환한다'라는 자바스크립트 규약에 따라

undefined가 반환된 것.

그럼 만약 __proto__ 객체에 name 프로퍼티가 있다면 어떨까?

```jsx
var suzi = new Person('Suzi');
suzi.__proto__._name = 'SUZI__proto__';
suzi.__proto__.getName();   // SUZI__proto__
```

잘 출력되는 것을 볼 수 있다. 그러니까 관건은 this이다.

this를 인스턴스로 할 수 있다면 좋을 것. 그 방법은 __proto__ 없이 인스턴스에서 곧바로 메서드를 쓰는 것이다.

```jsx
var suzi = new Person('Suzi', 28);
suzi.getName();   // Suzi
var iu = new Person('Jieun', 28);
iu.getName();     // Jieun
```

__proto__를 빼면 this는 instance가 되는게 맞지만,

이대로 메서드가 호출되고 심지어 원하는 값이 나오니 이상하다. 이상하지만 의외로 정상이다.

그 이유는 바로 **__proto__가 생략 가능한 프로퍼티이기 때문이다.**

원래부터 생략 가능하도록 정의돼 있으며,

이 정의를 바탕으로 자바스크립트의 전체 구조가 구성됐다고 해도 과언이 아니므로

__proto__가 생략 가능하다는 점만 기억하도록 하자.

```jsx
suzi.__proto__.getName
-> suzi(.__proto__).getName
-> suzi.getName
```

__proto__를 생략하지 않으면 this는 suzi.__proto__를 가리키지만, 이를 생략하면 suzi를 가리킨다.

suzi.__proto__에 있는 메서드인 getName을 실행하지만 this는 suzi를 바라보게 할 수 있게 된 것.

도식으로 보면 다음과 같다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b41176b-5eb8-43cc-8bd2-4bc07d913e61/KakaoTalk_Photo_2021-01-17-15-35-42.jpeg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b41176b-5eb8-43cc-8bd2-4bc07d913e61/KakaoTalk_Photo_2021-01-17-15-35-42.jpeg)

이제부터 프로토타입을 보는 순간 그림을 떠올리고, 각 꼭지점에 위의 글자를 떠올리고, 그로부터 문장을 만들어보는 연습을 해보기 바란다.

"new 연산자로 Constructor를 호출하면 instance가 만들어지는데,

이 instance의 생략가능한 프로퍼티인 __proto__는 Constructor의 prototype을 참조한다!"

여기까지 이해했다면 프로토타입을 이해한 것이나 마찬가지이다.

프로토타입의 개념을 좀더 상세히 설명하자면,

자바스크립트는 함수에 자동으로 객체인 prototype 프로퍼티를 생성해 놓는데

해당 함수를 생성자 함수로서 사용할 경우, 즉 new 연산자와 함께 함수를 호출할 경우

그로부터 생성된 인스턴스에는 숨겨진 프로퍼티인 __proto__가 자동으로 생성되며,

이 프로퍼티는 생성자 함수의 prototype 프로퍼티를 참조한다.

__proto__ 프로퍼티는 생략 가능하도록 구현돼 있기 때문에

**생성자 함수의 prototype에 어떤 메서드나 프로퍼티가 있다면 인스턴스에서도**

**마치 자신의 것처럼 해당 메서드나 프로퍼티에 접근할 수 있게 된다.*****

코드로 다시 살펴보자. 이번에는 크롬 개발자 도구의 콘솔 탭을 열어서 출력 결과를 살펴볼 것.

```jsx
var Constructor = function (name) {
	this.name = name;
};
Constructor.prototype.method1 = function() {};
Constructor.prototype.property1 = 'Constructor Prototype Property';

var instance = new Constructor('Instance');
console.dir(Constructor);
console.dir(instance);
```

크롬 개발자 콘솔에서 이를 실행한 결과는 다음과 같다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5fa4ea62-0425-4603-b909-743821f4772a/_2021-01-17__3.43.44.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5fa4ea62-0425-4603-b909-743821f4772a/_2021-01-17__3.43.44.png)

출력 결과의 첫줄에는 함수라는 의미의 f와 함수 이름인 Constructor, 인자 name이 보인다.

그 내부에는 옅은 색의 arguments, caller, length, name, prototype, __proto__ 등의 프로퍼티들이 보인다.

다시 prototype을 열어보면 위의 코드에서 추가한 method1, property1 등의 값이 짙은 색으로 보이고,

constructor, __proto__ 등이 옅은 색으로 보인다.

```jsx
이런 색상의 차이는 { enumerable: false } 속성이 부여된 프로퍼티인지 여부에 따른다.
짙은색은 enumerable, 즉 열거 가능한 프로퍼티임을 의미하고,
옅은 색은 innumerable, 즉 열거할 수 없는 프로퍼티임을 의미한다.
for in 등으로 객체의 프로퍼티 전체에 접근하고자 할때 접근 가능 여부를 색상으로 구분지어 표기한 것.
```

9번째 줄에서는 instance의 디렉터리 구조를 출력하려 했다. 그런데 출력 결과에는 Constructor가 나오고 있다.

어떤 생성자 함수의 인스턴스는 해당 생성자 함수의 이름을 표기함으로써 해당 함수함수의 인스턴스임을 표기하고 있다.

Constructor를 열어보면 name 프로퍼티가 짙은 색으로 보이고, __proto__ 프로퍼티가 옅은 색으로 보인다.

다시 __proto__를 열어보니 method1, property1, constructor, __proto__ 등이 보이는 것으로 봐서

Constructor의 prototype과 동일한 내용으로 구성돼 있음이 확인된다.

이번에는 대표적인 내장 생성자 함수인 Array를 바탕으로 다시 한번 살펴보자.

```jsx
var arr = [1, 2];
console.dir(arr);
console.dir(Array);
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/61b081a9-b87b-4edd-8296-9cf3f5713448/_2021-01-17__3.49.24.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/61b081a9-b87b-4edd-8296-9cf3f5713448/_2021-01-17__3.49.24.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a620d6d7-d1a0-4159-a791-9f2881bf9c5a/_2021-01-17__3.49.33.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a620d6d7-d1a0-4159-a791-9f2881bf9c5a/_2021-01-17__3.49.33.png)

왼쪽은 arr 변수를 출력한 결과이고, 오른쪽인 생성자 함수인 Array를 출력한 결과이다.

왼쪽부터 보자. 첫 줄에는 Array(2)라고 표기되고 있다. Array라는 생성자 함수를 원형으로 삼아 생성됐고,

length가 2임을 알 수 있다. 인덱스인 0, 1이 짙은 색상으로, length와 __proto__가 옅은 색상으로 표기되고있다.

__proto__를 열어보니 옅은 색상의 다양한 메서드들이 길게 펼쳐진다.

여기에는 push, pop, shift, slice, concat, sort, every, some 등등 우리가 배열에 사용하는 메서드들이 거의 모두 들어있다.

이제 오른쪽은 보자. 첫 줄에는 함수라는 의미의 f가 표시돼 있고, 둘째 줄부터는 함수의 기본적인 프로퍼티들인

arguments, caller, length, name 등이 옅은 색으로 보인다.

또한 Array 함수의 정적 메서드인 from, isArray, of 등도 보인다.

prototype을 열어보니 왼쪽의 __proto__와 완전히 동일한 내용으로 구성돼 있다.

위 출력결과를 바탕으로 도식을 더욱 구체화하면 다음과 같다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b07a1cff-f0d8-444a-aa8c-01386573f316/KakaoTalk_Photo_2021-01-17-16-05-00.jpeg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b07a1cff-f0d8-444a-aa8c-01386573f316/KakaoTalk_Photo_2021-01-17-16-05-00.jpeg)

이제 생성자 함수와 prototype, 인스턴스 사이의 관계가 명확히 보이는 듯 하다.

Array를 new 연산자와 함께 호출해서 인스턴스를 생성하든, 그냥 배열 리터럴을 생성하든,

어쨌든 instance인 [1, 2]가 만들어진다.

이 인스턴스의 __proto__은 Array.prototype을 참조하는데, __proto__가 생략 가능하도록 설계돼 있기 때문에

인스턴스가 push, pop, forEach 등의 메서드를 마치 자신의 것처럼 호출할 수 있다.

한편 Array의 prototype 프로퍼티 내부에 없는 from, isArray 등의 메서드들은 인스턴스가 직접 호출할 수 없다

이들은 Array 생성자 함수에서 직접 접근해야 실행이 가능하다.

```jsx
var arr = [1, 2];
arr.forEach(function (){});  // (0)
Array.isArray(arr);          // (0) true
arr.isArray();               // (X) TypeError : arr.isArray is not a function
```

---

---

자바스크립트에서 정말 헷갈리는 개념으로,

자바스크립트를 다루는데 있어서 굉장히 중요한 역할을 하기 때문에 반드시 이해하여야 한다.

## **정의**

자바스크립트의 모든 객체는 자신의 **"원형(Prototype)"** 이 되는 객체를 가지며 이를 프로토타입이라고 한다.

보이지 않는 속성인 `[[Prototype]]` 이 자신의 프로토타입 객체를 참조한다.

이를 `__proto__` 라는 속성으로 참조할 수 있으나,

이는 비표준이고 모든 브라우저에서 동작하는 것은 아니기 때문에 실제로 사용하는 것은 피해야 한다.

## **.prototype과 [[Prototype]]**

프로토타입이 헷갈리는 이유는 그 명명법과 연결방식에 있는데,

모든 객체는 은닉 속성인 `[[Prototype]]` 을 갖는데

특별히 **함수 객체** 는 접근할 수 있는 속성인 `prototype` 을 갖는다.

이름만 보면 같은 것으로 보이기 때문에 관계를 명확히 파악하여야 한다.

- `[[Prototype]]` : 자신의 프로토타입 객체를 참조하는 속성이다.
- `.prototype` : `new` 연산자로 자신을 생성자 함수로 사용한 경우, 그걸로 만들어진 새로운 객체의 `[[Prototype]]` 이 참조하는 값이다.

다음 코드를 보자.

```jsx
function Func() {}
var a = new Func();
```

`Func` 을 생성자로 호출하여 새로운 객체 `a` 를 생성한다. 이는 아래와 같은 구조로 프로토타입이 연결된다.

![https://github.com/baeharam/Must-Know-About-Frontend/raw/master/images/javascript/prototype1.png](https://github.com/baeharam/Must-Know-About-Frontend/raw/master/images/javascript/prototype1.png)

그림을 보면 위에서 언급하지 않은 2가지가 있는데,

`constructor` 는 모든 `.prototype` 객체의 속성에 있는 것으로 실제 객체를 참조한다.

따라서 위와 같이 `Func` 을 가리키는 것이다.

그 다음, `Func.prototype` 의 `[[Prototype]]` 이 `Object.prototype` 으로 연결되는데

이는 모든 객체의 프로토타입 객체로 마지막으로 연결되는 프로토타입 객체이다.

즉, 정리하자면 다음과 같다.

- `new` 연산자로 새로운 객체 `a` 를 생성하면, `a` 의 프로토타입 객체는 생성자 함수로 사용한 `Func` 의 속성인 `Func.prototype` 이 된다.
- `Func.prototype` 은 `constructor` 속성을 가지며 이는 실제 객체 `Func` 을 가리킨다.
- `Func.prototype` 또한 객체이므로 `[[Prototype]]` 을 가지고 이는 모든 객체의 원형이 되는 객체인 `Object.prototype` 을 가리킨다.

## **프로토타입 체인**

어떤 객체의 프로퍼티를 참조하거나 값을 할당할 때 **해당 객체에 프로퍼티가 없을 경우,**

**그 객체의 프로토타입 객체를 연쇄적으로 보면서 프로퍼티를 찾는 방식** 을 프로토타입 체인이라고 한다.

단, 참조할 때와 값을 할당할 때의 메커니즘이 다르니 기억해두어야 한다.

- **프로퍼티를 참조할 때**
    - 찾고자 하는 프로퍼티가 객체에 존재하면 사용한다.
    - 없으면 `[[Prototype]]` 링크를 타고 끝까지 올라가면서 해당 프로퍼티를 찾는다.
    - 찾으면 그 값을 사용하고 없으면 `undefined` 를 반환한다.

- **프로퍼티에 값을 할당할 때**
    - 찾고자 하는 프로퍼티가 객체에 존재하면 값을 바꾼다.
    - 프로퍼티가 없고 `[[Prototype]]` 링크를 타고 올라가서 해당 프로퍼티를 찾았을 경우
        - 그 프로퍼티가 변경가능한 값, 즉 `writable: true` 라면 새로운 직속 프로퍼티를 할당해서 상위 프로퍼티가 가려지는 현상이 발생한다.
        - 그 프로퍼티가 변경불가능한 값, 즉 `writable: false` 라면 비엄격 모드에선 무시되고 엄격 모드에선 에러가 발생한다.
        - 해당 프로퍼티가 세터(setter) 일 경우, 이 세터가 호출되고 가려짐이 발생하지 않는다.

여기서 말하는 **"가려짐"** 이란,

상위 프로토타입 객체에 동일한 이름의 프로퍼티가 있는 경우

하위 객체의 프로퍼티에 의해 가려지는 현상을 말한다.

```jsx
function Func() {}
Func.prototype.num = 2;
var a = new Func();
a.num = 1;
console.log(a.num); // 1
```

위 코드의 경우 객체 `a` 의 프로토타입 객체인 `Func.prototype` 에 `num` 이 있지만

`a.num = 1` 로 인해 가려짐 현상이 발생해서 1을 출력한다.

```jsx
function Func() {}
Object.defineProperty(Func.prototype, "num", {
  value: 2,
  writable: false
})
var a = new Func();
a.num = 1; // 무시됨
console.log(a.num); // 2
```

그러나 `defineProperty()` 를 사용해서 변경불가능한 프로퍼티로 만들면,

비엄격모드에서 무시되서 2를 출력한다.

## **관련 함수 및 연산자**

### **Object.create()**

**프로토타입 객체를 받아 연결시켜서 새 객체를 만드는 함수** 로, 프로토타입 객체를 바꾸기 때문에 원래의 `constructor` 를 잃어버린다.

```jsx
function Func1() {}
function Func2() {}

Func2.prototype = Object.create(Func1.prototype);
console.log(Func2.prototype.constructor); // Func1
```

기대하는 결과는 `Func2` 이겠지만 프로토타입 객체가 `Func1.prototype` 으로 바뀌었고 이에 따라 `constructor` 값을 잃어버렸다. 따라서, 프토토타입 체인을 통해서 `Func1.prototype` 의 `constructor` 인 `Func1` 을 출력하게 된다.

### **instanceof vs isPrototypeOf**

instanceof는 2개의 피연산자를 받는 연산자이고 isPrototypeOf는 `Object.prototype` 에 있는 함수이다. **둘 다 특정 객체의 프로토타입 체인에 찾고자 하는 객체가 있는지 검사할 때 사용** 된다.

```jsx
function A() {}
function B() {}
B.prototype = new A();
B.prototype.constructor = B;
function C() {}
C.prototype = new B();
C.prototype.constructor = C;
var c = new C();
console.log(c instanceof A, A.prototype.isPrototypeOf(c)); // true
console.log(c instanceof B, B.prototype.isPrototypeOf(c)); // true
console.log(c instanceof C, C.prototype.isPrototypeOf(c)); // true
```

### **getPrototypeOf, setPrototypeOf**

각각 특정 객체의 프로토타입 객체를 가져오고 할당하는 함수이다.

```jsx
function A() {}
function B() {}
var a = new A();
console.log(Object.getPrototypeOf(a)); // A.prototype
Object.setPrototypeOf(a, B.prototype);
console.log(Object.getPrototypeOf(a)); // B.prototype
```

**참고**

- You don't know JS, 프로토타입 (책)
- [Stackoverflow, What's the difference between isPrototypeOf and instanceof in Javascript?](https://stackoverflow.com/questions/2464426/whats-the-difference-between-isprototypeof-and-instanceof-in-javascript)
- [Poiemaweb, 프로토타입](https://poiemaweb.com/js-prototype)
- MDN 문서의 프로토타입 관련 함수들