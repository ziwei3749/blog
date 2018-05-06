# Vuex

- state
- getter
- mutation
- action
- module


## Vuex使用流程

- state通过computed渲染vue组件
- 通过this.$state.store.dispatch提交action,可以处理异步操作，处理后commit一个mutation
- mutation里可以获得store和参数，然后修改store.

## redux和mobx的区别

我个人感觉Vuex更简单，但是可能它做的封装也更多一些，相对redux，因为redux好像只有几百行代码
