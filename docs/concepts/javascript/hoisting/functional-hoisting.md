## Function Hoisting

함수 선언식(function declaration)은 호이스팅이 되어 선언, 초기화, 할당이 일어난다. 반대로 함수 표현식(function expression)은 변수와 동일하게 취급된다.

```javascript
console.log(func_declaration(2)); // 2

func_declaration(value) {
	return value * value;
}

console.log(func_expression(2)); // ReferenceError

const func_expression = function (value){
	return value * value;
}
```
