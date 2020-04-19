---
title: js-reduce
date: 2020-04-19 01:32:32
tags: javascript
categories: [javascript]
seotitle: reduce
---

#### 常见的 `reduce` 用法是数字相加

{% codeblock  lang:javascript %}
const arr = [1, 2, 3, 4, 5]
const sum = arr.reduce((previous, next) => {
return (previous += next)
}, 0)
console.log(sum) // 15
{% endcodeblock %}

reduce 有 2 个参数， {% span blue,  第一个是回调函数， 第二个是初始值 %}

- 回调函数的每次执行后的返回值 将会赋予给下一次回调函数的 previous
- 初始值是指定第一次的回调函数的 previous 值
- 如上例，如果指定 0，则第一次执行结果 是 0 + 1， 不指定则是 1 + 2

<!-- more -->

---

_下面来个复杂点的用法_

#### 想象一个购物车的场景步骤

1. 加入购物车
2. 计算税费价格之类
3. 埋单付款
4. 清空购物车

{% codeblock  lang:javascript %}
const user = {
name: 'fate',
cart: [],
purchases: [], // 把购物车的内容存进该字段模拟付款
}

// 加入购物车
function addItemToCart(user, item) {
const updatedCart = user.cart.concat(item)
return Object.assign({}, user, { cart: updatedCart })
}
// 计算价格税费之类的
function applyTaxToItems(user) {
const { cart } = user
const taxRate = 1.5 // 税率
const updatedCart = cart.map((item) => {
return {
name: item.name,
price: item.price \* taxRate,
}
})
return Object.assign({}, user, { cart: updatedCart })
}
// 模拟下单付款
function buyItem(user) {
const itemsInCart = user.cart
return Object.assign({}, user, { purchases: itemsInCart })
}
// 清空购物车
function emptyUserCart(user) {
return Object.assign({}, user, { cart: [] })
}

// 下面让我们来调用
const pipe = (f, g) => (...args) => g(f(...args))
const purchaseItem = (...fns) => fns.reduce(pipe)
const res = purchaseItem(
addItemToCart,
applyTaxToItems,
buyItem,
emptyUserCart
)(user, { name: 'iPhone', price: 100 })
console.log(res)
// 打印结果
// {
// name: 'fate',
// cart: [],
// purchases: [ { name: 'iPhone', price: 150 } ]
// }
{% endcodeblock %}

#### 分解一下 `pipe` 和 `purchaseItem` 函数

{% codeblock  lang:javascript %}
function purchaseItem2(...fns) {
return fns.reduce(function(f, g){
console.log(f)
console.log(g)
return function(...args) {
return g(f(...args))
}
});
}

const res = purchaseItem2(
addItemToCart,
applyTaxToItems,
buyItem,
emptyUserCart,
)(user, {name: 'iphone', price: 100})
{% endcodeblock %}

下面依次打印结果

- [Function: addItemToCart]
- [Function: applyTaxToItems]
- [Function]
- [Function: buyItem]
- [Function]
- [Function: emptyUserCart]
