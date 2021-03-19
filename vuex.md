# vuex

- state：用于数据的存储，是store中的唯一数据源
- getters：如vue中的计算属性一样，基于**state数据**的**二次包装**，常用于**数据的筛选**和多个数据的相关性**计算**
- mutation：类似函数，改变state数据的唯一途径，且**不能**用于**处理异步**事件
- action：类似于mutation，用于提交mutation来改变状态，而不直接变更状态，可以**包含任意异步**操作
- modules：类似于**命名空间**，用于项目中将**各个模块的状态**分开定义和操作，便于维护

![image-20210313144154080](https://cdn.jsdelivr.net/gh/rxdragon/webLearning/img/image-20210313144154080.png)

![image-20210313144519685](https://cdn.jsdelivr.net/gh/rxdragon/webLearning/img/image-20210313144519685.png)

![image-20210313144921117](https://cdn.jsdelivr.net/gh/rxdragon/webLearning/img/image-20210313144921117.png)

![image-20210313145752504](https://cdn.jsdelivr.net/gh/rxdragon/webLearning/img/image-20210313145752504.png)

![image-20210313154343413](https://cdn.jsdelivr.net/gh/rxdragon/webLearning/img/image-20210313154343413.png)

![image-20210313154420071](https://cdn.jsdelivr.net/gh/rxdragon/webLearning/img/image-20210313154420071.png)

