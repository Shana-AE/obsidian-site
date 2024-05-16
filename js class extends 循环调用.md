```javascript
class A {
    test1() {
        console.log('A test1');
    }
    test2() {
        console.log('A test2');
        this.test1();
    }
}

class B extends A {
    test1() {
        console.log('B test1');
        super.test2()
    }
    test2() {
        console.log('B test2');
    }
}

const b = new B();
b.test1(); // 会导致循环调用
```