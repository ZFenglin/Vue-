# 常用API

## 数据相关

### Vue.set / vm.set

动态追加data中未声明的响应式数据

``` js
 Vue.set(target, propertyName / index, value)
```

### Vue.delete / vm.delete

动态删除， 会触发视图更新

``` js
Vue.delete(target, propertyName / index)
```

## 事件相关

### vm.$on

监听当前实例上的自定义事件，事件可以由 vm.$emit 触发，回调函数会收所有传入事件触发函数的额外参数

``` js
vm.$on('test', function(msg) {
    console.log(msg)
})
```

### vm.$emit

触发当前实例上的事件，附加参数都会传给监听器回调

``` js
vm.$emit('test', 'hi')
```

### 事件总线

通过在Vue原型上添加一个Vue实例作为事件总线, 实现组件间相互通信, 而且不受组件间关系的影响

``` js
// 这样做可以在任意组件中使用 this.$bus 访问到该Vue实例
Vue.prototype.$bus = new Vue();
```

弹窗组件

``` js
Vue.component('message', {
    template: `<message :show.sync="show" >...</message>`
    // 监听关闭事件
    mounted() {
        this.$bus.$on('message-close', () => {
            this.$emit('update:show', false)
        });
    },
})
```

派发关闭事件

``` html
<div class="toolbar">
    <button @click="$bus.$emit('message-close')">清空提示框</button>
</div>
```

### vm.$once

监听一个自定义事件, 但是只触发一次, 一旦触发之后, 监听器就会被移除

``` js
vm.$on('test', function(msg) {
    console.log(msg)
})
```

### vm.$off

移除自定义事件监听器

``` js
vm.$off() // 移除所有的事件监听器
vm.$off('test') // 移除该事件所有的监听器
vm.$off('test', callback) // 只移除这个回调的监听器
```

## 元素获取

### ref和vm.$refs

ref 被用来给元素或子组件注册引用信息，引用信息将会注册在父组件的$refs对象上

如果在普通的DOM元素上使用, 引用指向的就是DOM元素; 如果用在子组件上, 引用就指向组件

``` html
<message ref="msg">test</message>
```

``` js
// 获取组件实例
this.$refs.msg
```

注意

1. ref 是作为渲染结果被创建的, 在初始渲染时不能访问它们
2. $refs 不是响应式的, 不要试图用它在模板中做数据绑定
3. 当 v-for 用于元素或组件时, 引用信息将是包含 DOM 节点或组件实例的数组。
