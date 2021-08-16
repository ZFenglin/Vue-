# Vuex

集中式，可预测的(利于同构开发)，实现统一管理和稳定的全局状态管理

## 注册全局的$store

使用mixin给全局的vue添加$store，注意不同于router，install方法不在Store上

```js
class Store {
    constructor(options = {}) {}
}

function install(_vue) {
    Vue = _vue
    Vue.mixin({
        beforeCreate() {
            if (this.$options.store) {
                Vue.prototype.$store = this.$options.store
            }
        }
    })
}

export default {
    Store,
    install
};
```

## State

出于响应式，将State放在一个Vue实例的data中，并通过get与set方式处理

```js
class Store {
    constructor(options = {}) {
        this._vm = newVue({
            data: {
                $$state: options.state //$$表示vue不会代理
            }
        });
    }
    set state(v) {
        console.log('禁止直接修改state');
    }
    get state() {
        returnthis._vm._data.$$state;
    }
}
```

## Mutations和Actions

Mutations可以直接修改State，Actions则是通过Mutations间接控制

保存传入的配置

```js
class Store {
    constructor(options = {}) {
        //保存用户配置的mutations选项
        this._mutations = options.mutations || {}
        //保存用户编写的actions选项
        this._actions = options.actions || {}
        //保存用户编写的getters选项
        this._getters = options.getters || {}
        //...
    }
```

commit通过传入的key获取options.mutations中的方法entry，传入state并调用方法，dispatch则是将this（store类）作为参数传入

```js
class Store {
    constructor(options = {}) {
        conststore = this
        //...
    }
    commit(method, payload) {
        //this._mutations[method].apply(this,[this.state,payload])
        //获取type对应的mutation
        const entry = this._mutations[method]
        if (!entry) {
            console.error(`unknownmutationtype:${method}`);
            return
        }
        entry(this.state, payload);
    }
    dispatch(type, payload) {
        //获取用户编写的type对应的action
        const entry = this._actions[type]
        if (!entry) {
            console.error(`unknownactiontype:${type}`);
            return
        }
        //异步结果处理常常需要返回Promise
        return entry(this, payload);
    }

}
```

注意commit和dispatch的数据绑定，必须在Store下执行

```js
letVue;

class Store {
    constructor(options = {}) {
        //...
        const store = this
        const {
            commit,
            dispatch
        } = store
        this.commit = function boundCommit(type, payload) {
            commit.call(store, type, payload)
        }
        this.dispatch = function boundAction(type, payload) {
            return dispatch.call(store, type, payload)
        }
    }
    //...
}
```

## Getters

getters类似与state，使用到了vue实例中的computed

创建一个computed，将getters的方法外部包裹一层函数转变为computed并赋值给this.computed

创建一个getters，并将computed赋值到vue实例中

```js
let Vue;

class Store {
    constructor(options = {}) {
        // 保存用户编写的getters选项
        this._getters = options.getters || {}

        const store = this

        // 定义computed
        const computed = {}
        this.getters = {}
        Object.keys(this._getters).forEach(key => {
            const fn = store._getters[key]
            // 将getters的方法外部包裹一层函数
            computed[key] = function() {
                return fn(store.state)
            }
            // 设定为不可更改
            Object.defineProperty(store.getters, key, {
                get: () => store._vm[key]
            })
        })

        this._vm = new Vue({
            data: {
                // state: options.state
                $$state: options.state // $$表示vue不会代理
            },
            computed
        });

    }
}
```
