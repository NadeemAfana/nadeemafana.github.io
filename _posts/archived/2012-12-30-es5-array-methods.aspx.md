---
layout: post
title: ECMAScript 5 Array methods
redirect_from:
- "/post/es5-array-methods.aspx/"
tags: [es5, javascript]
---
ES5 has nice array methods. In case you didn't hear about them, I will show their usage with some examples.

### `.forEach()`
`.forEach` invokes a function for each element in an array. For example,
```javascript
// Sum all the numbers in an array
var numbers = [1, 2, 5, 7, 3, 10];
var sum = 0;
 
numbers.forEach(function(n){
    sum += n; 
});
 
console.log(sum); // 28 is the total sum
```

The invoked function can take 3 arguments:
*    Value of the element
*    Index of the element
*    The Array itself

Another example that doubles the numbers in an array

```javascript
// Double all the numbers in an array
var numbers = [1, 2, 5, 7, 3, 10];
 
numbers.forEach(function(n, i, ar){ 
    ar[i] = 2*n; 
});
 
// numbers =  [2, 4, 10, 14, 6, 20]
```

### `.map()`
The function returns a value for each element in an array
```javascript
// Square all the numbers in an array
var numbers = [1, 2, 5, 7, 3, 10];
 
var squares = numbers.map(function(n){
 
    return n * n;
});
 
// numbers = [1, 4, 25, 49, 9, 100]
```

### `.filter()`
`.filter()` returns only the elements that match a predicate. The predicate function returns either `true` or `false`

```javascript
var numbers = [1, 2, 5, 7, 3, 10];
 
// Filter even integers
var evens = numbers.filter(function(n){    
     return n % 2 === 0; // is n even ?
});
 
console.log(evens); // evens = [2, 10]

var numbers = [1, 2, 5, 7, 3, 10];
 
function isPrime(n)
{
    if (n <= 1) return false;
             
    for(var i = 2; i < n; i++) {
        if (n % i === 0) return false;
     }
     
    return true;
}
 
// Filter prime integers
var primes = numbers.filter(function(n){    
     return isPrime(n); // is n prime ?
});
     
console.log(primes); // primes = [2, 5, 7, 3]
```
Another example with objects

```javascript
var people = [{
    name: "Helen",
    age: 24},
    {
    name: "Adam",
    age: 27},
    {
    name: "Sam",
    age: 19},
    {
    name: "Jordan",
    age: 30
    }];
 
var under25 = people.filter(function(p){
    return p.age < 25; // is person under 25 ?
});
 
console.log(under25); // under25 = [{name:"Helen", age:24}, {name:"Sam", age:19}]
 
// Find names of people that end in "m".
var names = people
    .map(function(p){return p.name;})
    .filter(RegExp.prototype.test, /m$/i);
 
console.log(names); // names = ["Adam", "Sam"]
```

### `.every()`
Returns true if all the elements in an array match a predicate. Otherwise, it returns false.

```javascript
var numbers = [1, 2, 5, 7, 3, 10];
 
var allEvens = numbers.every(function(n) {
    return n % 2 === 0; // is n even ?
});
 
var allSmall = numbers.every(function(n) {
    return n < 100; // is n less than 100?
});
 
console.log(allEvens); // false
console.log(allSmall); // true
```

### `.some()`
`.some()` returns true if at least one element matches the predicate, and it returns false if none of the elements matches the predicate
```javascript
// Does the array contain the integer 5 ? 
[1, 2, 5, 7, 3, 10].some(function(n) {
    return n === 5;
})
 
// true
```

### `.reduce()` and `.reduceRight()`
They both combine elements to return one single value. `.reduce()` processes elements from left to right, whereas `.reduceRight()` processes the elements from right to left.
```javascript
ar numbers = [1, 2, 5, 7, 3, 10];
 
var sum = numbers.reduce(function(a, b){
 
    return a + b;    
});
 
console.log(sum); // sum = 28
 
var max = numbers.reduce(function(a, b){
     
    return a > b ? a : b;    // or simply Math.max(a, b)   
});
 
console.log(max); // max = 10
 
// NOTE: You can also use .apply() to obtain the maximum or minimum from Math class
var maximum = Math.max.apply(null, numbers); // 10
var minimum = Math.min.apply(null, numbers); // 1
```

`.reduce()` accepts a function, and this function takes two values and will reduce these values to return only one single value. In the examples above, it reduced a and b by adding them together, and by choosing the largest of the two. It starts by invoking the predicate for the first two values: 1 and 2. It takes the result 3 and invokes the function again with 3 and 5, and so on. In fact, The first argument, a, is the accumulated result of the previous invocation, and the second argument b is the next value to accumulate.

`.reduceRight()` works in the same way as .reduce() except that it starts processing from the right of the array.

```javascript
var numbers = [1, 2, 5, 10];
 
var result1 = numbers.reduceRight(function(a, b){  
  
    return  a / b;    
});
 
console.log(result1); // result1 = 10 / 5 / 2 / 1 = 1
 
 
var result2 = numbers.reduce(function(a, b){  
     
    return  a / b;    
});
 
console.log(result2); // result2 = 1 / 2 / 5 / 10 = 0.01
```

Example using objects
```javascript
var people = [{
    name: "Helen",
    age: 24},
    {
    name: "Adam",
    age: 27},
    {
    name: "Sam",
    age: 19},
    {
    name: "Jordan",
    age: 30
    }];
 
// Find the youngest person
var youngest = people.reduce(function(p1, p2){
    return p1.age < p2.age ? p1 : p2; // NOTE: you cannot use Math.min(p1.age, p2.age) here
});
 
 
console.log("The youngest person is " + youngest.name + " and he/she is " + youngest.age + " years old"); 
// The youngest person is Sam and he/she is 19 years old
```

### `.indexOf()` and `.lastIndexOf()`
hey search an array for a specified value and return the index of the first element or -1 if no such value is found. `.lastIndexOf()` starts searching from right to left.

```javascript
var numbers = [1, 2, 5, 7, 3, 10, 5];
 
numbers.indexOf(5); // 2
numbers.indexOf(5, 3); // start searching at index 3. Result is  6
numbers.indexOf(8); // -1
```

**Note** that both of these functions do not take a function as an argument. However, they take a value to search for. They also accept a second optional argument that specifies the starting index where the search begins.
