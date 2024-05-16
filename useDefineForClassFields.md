---
tags:
  - "#typescript"
  - tsconfig
---
Default: [[target|target]] 为`ES2022`或更高(包括`ESNext`)时为`true`, 否则为`false`
用于应用于tc39的标准，因为typescript class fields先出的，这个为true就是和现有版本的js的行为一致

````ad-note
collapse: open

class fields的EMCA标准出来后
```javascript
class C {

foo = 100;

bar: string;

}
```
de-sugar会变为
```js
class C {

constructor() {

Object.defineProperty(this, "foo", {

enumerable: true,

configurable: true,

writable: true,

value: 100,

});

Object.defineProperty(this, "bar", {

enumerable: true,

configurable: true,

writable: true,

value: void 0,

});

}

}
```
而不是ts一开始设想的
```js
class C {

constructor() {

this.foo = 100;

}

}
```
````

有两个需要注意的点：

- fields由`Object.defineProperty`声明
- fields 总是初始化为undefined，即使没有initializer

可能导致一些问题，比如：
- 祖先类的`set` 节点可能不会触发，因为被完全覆盖了
````ad-example
collapse: open
```typescript
class Base {

set data(value: string) {

console.log("data changed to " + value);

}

}

class Derived extends Base {

// No longer triggers a 'console.log'

// when using 'useDefineForClassFields'.

data = 10;

}
```
````
-  使用class fields 标明 根class的属性也不会成功
  
````ad-example
collapse: open
```typescript
interface Animal {

animalStuff: any;

}

interface Dog extends Animal {

dogStuff: any;

}

class AnimalHouse {

resident: Animal;

constructor(animal: Animal) {

this.resident = animal;

}

}

class DogHouse extends AnimalHouse {

// Initializes 'resident' to 'undefined'

// after the call to 'super()' when

// using 'useDefineForClassFields'!

resident: Dog;

constructor(dog: Dog) {

super(dog);

}

}
```
````

总结来说就是和祖先类的属性混合会造成问题，会重新声明没有initializer的属性
````ad-example
collapse: close
```typescript
interface Animal {

animalStuff: any;

}

interface Dog extends Animal {

dogStuff: any;

}

class AnimalHouse {

resident: Animal;

constructor(animal: Animal) {

console.log('before super set resident');

this.resident = animal;

console.log('super set resident')

}

}

class DogHouse extends AnimalHouse {

// Initializes 'resident' to 'undefined'

// after the call to 'super()' when

// using 'useDefineForClassFields'!

resident: Dog = (() => {

console.log('fields');

return 1

})();

constructor(dog: Dog) {

console.log('before super')

super(dog);

console.log('after super');

}

}

  

const a = new DogHouse({

dogStuff: 12,

});

console.log(a.resident);
```
````

