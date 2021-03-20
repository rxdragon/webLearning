# 双向绑定

- vue.js 则是采用数据劫持结合发布者-订阅者模式的方式，通过`Object.defineProperty()`来劫持各个属性的`setter`，`getter`，在数据变动时发布消息给订阅者，触发相应的监听回调。

- vue的双向数据绑定原理
  监听器 Observer ，用来劫持并监听所有属性（转变成setter/getter形式），如果属性发生变化，就通知订阅者
  
  订阅者 Watcher，可以收到属性的变化通知并执行相应的方法，从而更新视图
  
  解析器 Compile，可以解析每个节点的相关指令，对模板数据和订阅器进行初始化
  
  订阅器 Dep，用来收集订阅者，对监听器 Observer 和 订阅者 Watcher 进行统一管理

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200318221541823.png?x-oss-process=image)

- ## 核心

- 是通过Object.defineProperty() + addEventListener，对data的每个属性进行了get、set的拦截。其实只用Object.defineProperty()已经可以实现双向绑定，只是这样做效率很低

  ## 原理图

  ![img](https://images2017.cnblogs.com/blog/1162184/201709/1162184-20170918135341618-553576179.png)

  vue的数据双向绑定 将MVVM作为数据绑定的入口，整合Observer，Compile和Watcher三者，通过Observer来监听自己的model的数据变化，通过Compile来解析编译模板指令（vue中是用来解析 {{}}），最终利用watcher搭起observer和Compile之间的通信桥梁，达到数据变化 —>视图更新；视图交互变化（input）—>数据model变更双向绑定效果。

  - observer用来实现对每个vue中的data中定义的属性循环用Object.defineProperty()实现数据劫持，以便利用其中的setter和getter，然后通知订阅者，订阅者会触发它的update方法，对视图进行更新。

  - 我们介绍为什么要订阅者，在vue中v-model，v-name，{{}}等都可以对数据进行显示，也就是说假如一个属性都通过这三个指令了，那么每当这个属性改变的时候，相应的这个三个指令的html视图也必须改变，于是vue中就是每当有这样的可能用到双向绑定的指令，就在一个Dep中增加一个订阅者，其订阅者只是更新自己的指令对应的数据，也就是v-model='name'和{{name}}有两个对应的订阅者，各自管理自己的地方。每当属性的set方法触发，就循环更新Dep中的订阅者。

  ## observer实现

- 主要是给每个vue的属性用Object.defineProperty()，代码如下：

  ```js
  function defineReactive (obj, key, val) {
      var dep = new Dep();
          Object.defineProperty(obj, key, {
               get: function() {
                      //添加订阅者watcher到主题对象Dep
                      if(Dep.target) {
                          // JS的浏览器单线程特性，保证这个全局变量在同一时间内，只会有同一个监听器使用
                          dep.addSub(Dep.target);
                      }
                      return val;
               },
               set: function (newVal) {
                      if(newVal === val) return;
                      val = newVal;
                      console.log(val);
                      // 作为发布者发出通知
                      dep.notify();//通知后dep会循环调用各自的update方法更新视图
               }
         })
  }
          function observe(obj, vm) {
              Object.keys(obj).forEach(function(key) {
                  defineReactive(vm, key, obj[key]);
              })
          }
  ```

  ## 实现compile

  compile的目的就是解析各种指令称真正的html。

  ```js
  function Compile(node, vm) {
      if(node) {
          this.$frag = this.nodeToFragment(node, vm);
          return this.$frag;
      }
  }
  Compile.prototype = {
      nodeToFragment: function(node, vm) {
          var self = this;
          var frag = document.createDocumentFragment();
          var child;
          while(child = node.firstChild) {
              console.log([child])
              self.compileElement(child, vm);
              frag.append(child); // 将所有子节点添加到fragment中
          }
          return frag;
      },
      compileElement: function(node, vm) {
          var reg = /\{\{(.*)\}\}/;
          //节点类型为元素(input元素这里)
          if(node.nodeType === 1) {
              var attr = node.attributes;
              // 解析属性
              for(var i = 0; i < attr.length; i++ ) {
                  if(attr[i].nodeName == 'v-model') {//遍历属性节点找到v-model的属性
                      var name = attr[i].nodeValue; // 获取v-model绑定的属性名
                      node.addEventListener('input', function(e) {
                          // 给相应的data属性赋值，进而触发该属性的set方法
                          vm[name]= e.target.value;
                      });
                      new Watcher(vm, node, name, 'value');//创建新的watcher，会触发函数向对应属性的dep数组中添加订阅者，
                  }
              };
          }
          //节点类型为text
          if(node.nodeType === 3) {
              if(reg.test(node.nodeValue)) {
                  var name = RegExp.$1; // 获取匹配到的字符串
                  name = name.trim();
                  new Watcher(vm, node, name, 'nodeValue');
              }
          }
      }
  }
  ```

  ## watcher实现

  ```js
  function Watcher(vm, node, name, type) {
      Dep.target = this;
      this.name = name;
      this.node = node;
      this.vm = vm;
      this.type = type;
      this.update();
      Dep.target = null;
  }
  
  Watcher.prototype = {
      update: function() {
          this.get();
          this.node[this.type] = this.value; // 订阅者执行相应操作
      },
      // 获取data的属性值
      get: function() {
          console.log(1)
          this.value = this.vm[this.name]; //触发相应属性的get
      }
  }
  ```

- 首先我们为每个vue属性用Object.defineProperty()实现数据劫持，为每个属性分配一个订阅者集合的管理数组dep；然后在编译的时候在该属性的数组dep中添加订阅者，v-model会添加一个订阅者，{{}}也会，v-bind也会，只要用到该属性的指令理论上都会，接着为input会添加监听事件，修改值就会为该属性赋值，触发该属性的set方法，在set方法内通知订阅者数组dep，订阅者数组循环调用各订阅者的update方法更新视图。

# keep-alive

![image-20210320214349942](https://cdn.jsdelivr.net/gh/rxdragon/webLearning/img/image-20210320214349942.png)

![image-20210320214124473](https://cdn.jsdelivr.net/gh/rxdragon/webLearning/img/image-20210320214124473.png)

![image-20210320214707600](https://cdn.jsdelivr.net/gh/rxdragon/webLearning/img/image-20210320214707600.png)

![image-20210320215509142](https://cdn.jsdelivr.net/gh/rxdragon/webLearning/img/image-20210320215509142.png)

# 组件通信方式

- props / $emit 适用 父子组件通信
- ref 与 $parent / $children 适用 父子组件通信
- EventBus （$emit / $on） 适用于 父子、隔代、兄弟组件通信
- *a**t**t**r**s*/listeners 适用于 隔代组件通信

- provide / inject 适用于 隔代组件通信
- Vuex 适用于 父子、隔代、兄弟组件通信

# watch/computed的区别

- computed是计算属性，依赖其它属性值，并且 computed 的值有缓存，只有它依赖的属性值发生改变，下一次获取 computed 的值时才会重新计算 computed 的值
- watch是观察监听的作用，类似于某些数据的监听回调 ，每当监听的数据变化时都会执行回调进行后续操作
- 当我们需要进行数值计算，并且依赖于其它数据时，应该使用 computed
- 当我们需要在数据变化时执行异步或开销较大的操作时，应该使用 watch

# v-if与v-show的区别

- v-if 是真正的条件渲染，直到条件第一次变为真时，才会开始渲染
- v-show是由display样式决定，不管初始条件是什么都会渲染
- v-if 适用于不需要频繁切换条件的场景；v-show 则适用于需要非常频繁切换条件的场景

# vue中的路由模式

## history模式

- HTML5中的两个API：pushState和replaceState，改变url之后页面不会重新刷新，也不会带有#号，页面地址美观，url的改变会触发popState事件，监听该事件也可以实现根据不同的url渲染对应的页面内容
- 但是因为没有#会导致用户在刷新页面的时候，还会发送请求到服务端，为避免这种情况，需要每次url改变的时候，都将所有的路由重新定位到跟路由下

## hash模式

- url hash: http 😕/foo.com/#help
- \#后面hash值的改变，并不会重新加载页面，同时hash值的变化会触发hashchange事件，该事件可以监听，可根据不同的哈希值渲染不同的页面内容

# vue 3.0中proxy数据双向绑定

- Proxy 可以直接监听对象而非属性；
- Proxy 可以直接监听数组的变化；
- Proxy 有多达 13 种拦截方法,不限于 apply、ownKeys、deleteProperty、has 等等是 Object.defineProperty 不具备的；
- Proxy 返回的是一个新对象,我们可以只操作新的对象达到目的,而 Object.defineProperty 只能遍历对象属性直接修改；
- Proxy 作为新标准将受到浏览器厂商重点持续的性能优化，也就是传说中的新标准的性能红利；
