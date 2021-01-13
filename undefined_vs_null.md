# undefined vs null

### undefined

선언 이후 값을 할당하지 않은 변수는 `undefined` 값을 가진다.

즉, **선언은 되었지만 값을 할당하지 않은 변수에 접근하거나**

**존재하지 않는 객체 프로퍼티에 접근할 경우 undefined가 반환된다.**

이는 변수 선언에 의해 확보된 메모리 공간을 처음 할당이 이루어질 때까지 빈 상태로 내버려두지 않고 

자바스크립트 엔진이 undefined로 초기화하기 때문이다.

```jsx
var foo;
console.log(foo); // undefined
```

이처럼 **undefined는 개발자가 의도적으로 할당한 값이 아니라 자바스크립트 엔진에 의해 초기화된 값**이다.

변수를 참조했을 때 undefined가 반환된다면 참조한 변수가 선언 이후 값이 할당된 적인 없는 변수라는 것을 개발자는 간파할 수 있다.

**그렇다면 개발자가 의도적으로 undefined를 할당해야하는 경우가 있을까?**

자바스크립트 엔진이 변수 초기화에 사용하는 이 값을 만약 개발자가 마음대로 할당한다면 undefined의 본래의 취지와 어긋날 뿐더러 혼란을 줄 수 있으므로 권장하지 않는다.

**그럼 변수의 값이 없다는 것을 명시하고 싶은 경우 어떻게 하면 좋을까?**

**그런 경우는 undefined를 할당하는 것이 아니라 null을 할당한다.**

---

### null

null 타입의 값은 `null`이 유일하다. 자바스크립트는 대소문자를 구별하므로 `null`은 Null, NULL등과 다르다.

**프로그래밍 언어에서 `null`은 의도적으로 변수에 값이 없다는 것을 명시할 때 사용한다.**

이는 변수가 기억하는 메모리 어드레스의 참조 정보를 제거하는 것을 의미하며 자바스크립트 엔진은 누구도 참조하지 않는 메모리 영역에 대해 [가비지 콜렉션](https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management)을 수행할 것이다.

```jsx
var foo = 'Lee';
foo = null;  // 참조 정보가 제거됨
```

또는 **함수가 호출되었으나 유효한 값을 반환할 수 없는 경우, 명시적으로 null을 반환하기도 한다**.

예를 들어, 조건에 부합하는 HTML 요소를 검색해 반환하는 Document.querySelector()는 조건에 부합하는 HTML 요소를 검색할 수 없는 경우, null을 반환한다.

```jsx
var element = document.querySelector('.myElem');
// HTML 문서에 myElem 클래스를 갖는 요소가 없다면 null을 반환한다.
console.log(element); // null
```

타입을 나타내는 문자열을 반환하는 `typeof` 연산자로 null 값을 연산해 보면 null이 아닌 object가 나온다.

이는 자바스크립트의 설계상의 오류이다.

따라서 **null 타입을 확인할 때 typeof 연산자를 사용하면 안되고 일치 연산자(===)를 사용하여야 한다.**

```jsx
var foo = null;
console.log(typeof foo); // object
console.log(typeof foo === null); // false
console.log(foo === null);        // true
``
