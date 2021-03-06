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


`_(obj).chain()` returns the `_(obj)` which is really a `new wrapper(obj)`
which is just an object itself with possibility of having custom prototype.methods through
`wrapper.prototype`. i.e. it is itself assigned to `_(obj)._wrapped` i.e. `_(obj)._wrapped = obj`.
`chain` and `_(obj).chain()` are totally different things. One is `Boolean` another is a `function`.
argument chain refers to `Boolean` chain.
function passed in as an argument cannot be called like a method eg) `_(obj).chain()` is not a function
passed in as an argument where as `chain()` is.

```js
var result = function(obj, chain) {
  return chain ? _(obj).chain() : obj;
};
```

chain is a `[wrapper object]`s prototype method. It just sets the `_(obj)._chain` to `true`
which will be used as a flag for future chaining methods. Later on when we want to chain
methods we will need to first call `_(obj).chain()` which will set `_chain` to `true` which in
turn are used by custom methods added with `addToWrapper` method.

```js 
wrapper.prototype.chain = function() {
  this._chain = true;
  return this;
};
 ```

`wrapper` should have been named `Wrapper` to make it explicit that it is a constructor.
`this` is used in the same way as any other constructors. (i.e. for the purpose of `new` function)
```js
var wrapper = function(obj) { this._wrapped = obj; }; 
```

`_(obj)` retruns a function with `this._wrapped = obj`.  that is `_wrapped = obj`
So far we have `_(obj)._chain` property and `_(obj)._wrapped` property.
```js
var _ = function(obj) { return new wrapper(obj); };
```

`addToWrapper` only adds to the `wrapper.prototype` chain. It does not add to `_` itself.
addToWrapper serves 2 purposes. One is to make a custom method available to all methods
that was constructed (well I guess it's not a total constructor we don't save _(obj) often) with 
`new wrapper(obj)` i.e. `_(obj)`. Another is to make such custom methods to be chainable.
```js
var addToWrapper = function(name, func) {
  wrapper.prototype[name] = function() {
    var args = _.toArray(arguments);
    unshift.call(args, this._wrapped);
    return result(func.apply(_, args), this._chain);

  };
};
```
`wrapper.prototype[name] = function() {`  

wrapper 콘스트럭터의 프로토타입에 함수 저장한다. 프로토타입에 저장하면 이미 존재하는 `_(obj)`들도 나중에
저장된 함수들을 사용할수 있다.

`var args = _.toArray(arguments);`

보통 arguments 없이 사용할때가 많다 eg) `_(obj).first()`
arguments를 주었을 시의 `arg` 값 eg) `_(obj).push(5)`에서 `arg`는 [5]

`unshift.call(args, this._wrapped);`

this._wrapped는 _(obj)._wrapped이다. 즉 `args`의 첫번째 값은 `_(obj)`에서 `obj`인것이다.

`return result(func.apply(_, args), this._chain);`

모든 `wrapper` 프로토탕비 메서드들은 감싸는 function들이다. 예);
```js
wrapper.toHanGeul = function () {
 return (function toHanGuel(){})();
}
```

`_` 에다가 apply로 context바인딩 한이유는 만약 함수가 `this`사용시 언더스코 method들을 사용하기 위한것이다.

 `this._chain` which is `_(obj)._chain` is a boolean which would have been set to `true` if `_(obj).chain()` was called before.
Note that if `this._chain` is `true`,  `result` function calls `_(obj).chain()` again which returns `_(obj)`  

with return of result(obj, boolean), it makes a OOP style method chainable. Hence it is essential to return `result(obj, boolean)`
also this part is what returns calle object i.e. `return this`

```js
_.mixin = function(obj) {
  each(_.functions(obj), function(name){
    addToWrapper(name, _[name] = obj[name]);
  });
}
```
Simple use of addToWrapper but this time makes the methods available to both `_` method and `_(obj).method` (i.e. `new wrapper` prototype method)
`_[name] = obj[name]` makes the method available to `_`


```js
// unshift.call(args, this._wrapped) part makes value to refer to returned _(obj)'s obj which is 5 as obj.chain().first() returns _(obj) as _(5)
// args = [5, 5]
// Note to make OOP methods chainable we really do need to use addToWrapper to add to wrapper. or if we do it manually we would need to do
// wrapper.prototype.rangeIndex = function() { return result(rangeIndex.apply(_, _.toArray(arguments)), true) }
_([5,2,3]).addToWrapper("rangeIndex", function(obj, x){ return (this.indexOf( this.range( obj ) ) === x) });

obj.chain()
  .first()
  .rangeIndex(4)
```


TODO: 

```js
  each(['pop', 'push', 'reverse', 'shift', 'sort', 'splice', 'unshift'], function(name) {
    var method = ArrayProto[name];
    wrapper.prototype[name] = function() {
      method.apply(this._wrapped, arguments);
      return result(this._wrapped, this._chain);
    };
  });

  // Add all accessor Array functions to the wrapper.
  each(['concat', 'join', 'slice'], function(name) {
    var method = ArrayProto[name];
    wrapper.prototype[name] = function() {
      return result(method.apply(this._wrapped, arguments), this._chain);
    };
  });
```
