- timeline可以通过add将子timeline添加到timeline中，但是如果多次add，会改变timeline的位置
	```typescript
	starTl.addLabel('start', 0);

	if (star1.value && star2.value) {
	
		starTl.add(star1.value.createTl(), 'start');
		
		starTl.add(star1.value.createTl(), 'start+=2.5');
		
		starTl.add(star2.value.createTl(), 'start+=0.25');
		
		starTl.add(star2.value.createTl(), 'start+=2.1');
	
	}
	```
- vue中的ref存储timeline之后会导致timeline失效（可能也只是添加到timeline中时失效），需要使用shallowRef
- timeline的duration与内部添加的动画的duration及插入相应的位置相关
