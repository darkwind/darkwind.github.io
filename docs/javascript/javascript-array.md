---
layout: default
title: Array 다루기
---

# Array 다루기

배열 요소 추가/제거
===
Code
``` javascript
const jbAry1 = [ 'one', 'two', 'three' ];
jbAry1.push( 'four' );
document.write( '<p>' + jbAry1.join( ' / ' ) + '</p>' );

const jbAry2 = [ 'one', 'two', 'three' ];
jbAry2.pop();
document.write( '<p>' + jbAry2.join( ' / ' ) + '</p>' );

const jbAry3 = [ 'one', 'two', 'three' ];
jbAry3.unshift( 'zero' );
document.write( '<p>' + jbAry3.join( ' / ' ) + '</p>' );

const jbAry4 = [ 'one', 'two', 'three' ];
jbAry4.shift();
document.write( '<p>' + jbAry4.join( ' / ' ) + '</p>' );
```
Result
``` html
one / two / three / four
one / two
zero / one / two / three
two / three
```

배열 뒤집기
===
Code
``` javascript
const fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.reverse();
```
