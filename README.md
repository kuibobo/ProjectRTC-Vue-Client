# ProjectRTC-Vue-Client

基于 [ProjectRTC](https://github.com/pchab/ProjectRTC) 项目修改, 这是Vue2版本

将代码合并到您的项目中,您只要将WebRtc.vue, webrtc-client.js 复制到您的Vue项目中,并修改项目的main.js

```
new Vue({
  render: h => h(App),
  data: {
    eventHub: new Vue() // 新增加这这里
  }
}).$mount('#app')
```

项目添加socket.io-client依赖, 版本在1.0.6下通过测试通过

```
yarn add socket.io-client@1.0.6
```

