![alt text](https://github.com/imskojs/UnderComp/ins1.png "Instruction 1") 
![alt text](https://github.com/imskojs/UnderComp/ins2.png "Instruction 2") 
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
  이 wrapper때문에 `_`는 object가 아닌 function 이다. `typeof _ === 'function'`
