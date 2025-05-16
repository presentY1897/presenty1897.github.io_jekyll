## Class Hoisting

function hoisting과 달리 클래스 선언식(class declaration)은 호이스팅되어도 선언까지만 진행하고 TDZ 제약을 따른다.

```javascript
new ClassDeclaration(); // ReferenceError

class ClassDeclaration {}

new ClassExpression(); // ReferenceError

const ClassExpression = class {};
```
