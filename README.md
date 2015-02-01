![alt text](https://github.com/imskojs/UnderComp/blob/master/ins1.PNG "Instruction 1") 

`commit`에 노트저장함

![alt text](https://github.com/imskojs/UnderComp/blob/master/ins2.PNG "Instruction 2") 



 ```js
 var X = {
     apple: 0,
     pineapple: 0,
     addPineappleChain: function (num) {
         this.pineapple = num + this.pineapple;
         return this;
     },
     addAppleChain: function (num) {
         this.apple = num + this.apple;
         return this;
     }
 }
 X.addPineappleChain(1).addAppleChain(1).addPineappleChain(1).addAppleChain(1);
 console.log(X)
 ```
 문제점은 `return this`만으로는 `function`이 Object안에 있어야 한다는것. Chaining을 Object안이 아닌곳에서 쓰기위해서는 window에다가 새로운 object을 만들어서 거기에 저장을 하던가 `new` function을 사용하여 `this`를 assigning하는 variable에 바인딩 하는수 밖에 없다.
 언더스코는;
 ```js
 var wrapper = function(obj) {this._wrapped = obj; };
 wrapper("window에 _.wrapped저장")
 
 var _ = function(obj) { return new wrapper(obj); };
 _("window에 저장하지 않음. 어디에도 저장되있지않음."); 
 var y = _("y에 _wrapped저장");
 
 console.log(window._wrapped);
 console.log(y._wrapped);
 
 new wrapper(55); //이렇게 하면 어디에 _wrapped가 생기는거지? 
  ```
  `new wrapper(55)`는 `[1, 2, 3, 4]`을 처주는것과 같다. `var`에다가 assign을 안하면 `window`에 `return`하고 바로 없어진다. (`temp`). 
  ```
  [1, 2, 3, 4].pop().push(4).slice();
  new wrapper(55).pop().push(4).slice();
  ```
  이런느낌?
  
  이 wrapper때문에 `_`는 object가 아닌 function 이다. `typeof _ === 'function'`
  
  

## `_.wrap`
언더스코어 예제를 보면;

```js
var hello = function(name) { return "hello: " + name; };
hello = _.wrap(hello, function(func) {
  return "before, " + func("moe") + ", after";
});
hello();
//=> 'before, hello: moe, after'
```
별로 좋은 예가 아닌것 같다.

그리고 `_.wrap`이라는 이름도 이 함수를 잘못생각 하게 만드는것 같다.

이름으로 `_.cook`이 어떨까요? 
```js
_.wrap = function(func, wrapper) {
  return function() {
    var args = [func].concat(_.toArray(arguments));
    return wrapper.apply(wrapper, args);
  };
};
```

`_.wrap`은 `wrapper`안으로 들어오는 arguments인 `func`함수와 나중에 콜되는 arguments를 재료로 더 맛있는 (좋은) 함수를 만든다!

여기서 wrapper는 언더스코어안에서 쓰이는 wrapper와는 아무 상관없음. 그냥 이름임. 

`wrapper`가 요리사 `func`가 매인 재료, 그리고 나중에 콜할때 들어오는 arguments들이 양념인 샘이다..-.-;;

```js
function add(x, y) {
  return x + y;
}

function addAfterMult(originalAdd, a, b, c, d) {
  var x = a * b;
  var y = c * d;
  return originalAdd(x, y);
}

var addAfter = _.wrap(add, addAfterMult);

addAfter(1,2,3,4);

// 1*2 + 3*4 = 14
```
