# promise

- 状态不受外界影响：拥有三种状态（Pending、Fullfilled、Rejected）。状态只取决于异步执行结果，其他过程无法改变这个状态。
- 状态改变不会再变：初始状态为Pending状态，它只可以变成Fullfill或者变成Rejected，但是一旦发生改变就不会再变。
- promise可以使得异步操作的写法像同步写法一样优雅，代码可读性更高。但是缺点就是中途无法取消promise，一旦新建就会执行，假如没有设置回调函数，Promise内部如果出现错误不会反应到外部。

**实例方法**

- then()/catch()
- all()/race()
- resolve()/reject()
- done()/finally()

# proxy

Proxy可以理解为，当你试图访问一个对象的时候必须先经过一个拦截或者代理，你才可以进行对对象的操作。这种机制的好处就是可以对外界的访问进行过滤和改写。

# ES6的新特性

- let、var、const区别
- 箭头函数
- 解构赋值
- 模板字符串
- Set、Map数据结构
- Promise对象
- Proxy