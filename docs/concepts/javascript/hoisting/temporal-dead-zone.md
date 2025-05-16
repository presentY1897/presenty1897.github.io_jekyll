## Temporal Dead Zone

`let`, `const`, `class`는 변수가 선언과 초기화(initialize) 된 코드에 도달하기 전까지 TDZ(Temporal Dead Zone)에서 선언된다.

TDZ에 있는 변수를 접근하면 언제나 ReferenceError가 반환된다. 변수가 선언 및 초기화된 코드에 도달하면 변수에 값을 초기화하고 값이 있으면 할당(assignment)하며 TDZ에서 벗어난다.

예외적으로 `var`는 선언과 동시에 `undefined`로 초기화되며 scope내에서 언제나 접근가능하다.

Javascript는 변수를 Compile하면서 선언, 초기화, 할당의 과정을 거친다.
