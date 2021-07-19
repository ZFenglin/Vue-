# VueRouter

vue采取哈希路由，这种路由变化不会导致URL变化

监听路由变化 => routes中寻找对应的路由 => 从查找到的路由上获取组件并渲染

## 全局注册$router

使用mixin给全局的vue添加$router

``` js
class ZRouter {
    constructor(options) {
        this.$options = options;
    }
}
ZRouter.install = function(_Vue) {
    // 绑定实例
    Vue = _Vue;
    Vue.mixin({
        beforeCreate() {
            // 只有根组件拥有router选项
            // 在Vue的$root.$options中可以找到创建实例的传参
            if (this.$options.router) {
                // vm.$router
                Vue.prototype.$router = this.$options.router;
            }
        }
    });
}
```

## 监听current

使用Vue.util.defineReactive注册响应式数据，被注册的数据变动会触发vue的渲染函数render()，使视图更新

注意函数运行环境

``` js
class ZRouter {

    constructor(options) {
        this.$options = options;
        const initial = window.location.hash.slice(1) || '/'
        // 添加响应式current
        Vue.util.defineReactive(this, 'current', initial);
        // 监听浏览器数据改变
        window.addEventListener('hashchange', this.changeCurrent.bind(this));
        window.addEventListener('reload', this.changeCurrent.bind(this));

    }
    // 更换current
    changeCurrent() {
        this.current = window.location.hash.slice(1)
    }

}
```

## router-link 和 router-view

ZRouter中注册组件，并添加路由的映射关系

``` js
class ZRouter {
    constructor(options) {
        // 缓存path和route映射关系
        this.routeMap = {}
        this.$options.routes.forEach(route => {
            this.routeMap[route.path] = route
        });
        //...
    }
    // 路由link组件
    Vue.component('router-link', ZLink);
    // 路由view组件
    Vue.component('router-view', ZView);
}
```

``` js
// router-link
export default {
    // components参数在对象中直接使用
    props: {
        to: {
            type: String,
            required: true
        },
    },
    render: function(h) {
        return h(
            'a', {
                attrs: {
                    href: '#' + this.to
                },
            },
            [this.$slots.default] // 获取组件的slot
        );
    }
};
```

``` js
// router-view
export default {
    render(h) {
        const {
            routeMap,
            current
        } = this.$router
        const component = routeMap[current].component;
        return h(component);
    }
}
```

## 嵌套路由

嵌套路由主要是由记录深度的depth和记录路由层级的matched列表处理

首先将router-view标记为routerView组件并获取当前路由深度

``` js
export default {
    render(h) {
        // 标记为routerView组件
        this.$vnode.data.routerView = true;

        // 循环获取当前路由深度
        let depth = 0
        let parent = this.$parent
        while (parent) {
            const vnodeData = parent.$vnode && parent.$vnode.data
            if (vnodeData && vnodeData.routerView) {
                // 说明当前parent是一个router-view
                depth++
            }
            parent = parent.$parent
        }
        // ...
    }
}
```

在ZRouter中创建matched数组，取消current响应和缓存path和route映射关系，获取matched

``` js
class ZRouter {
    constructor(options) {
        this.$options = options;

        // 取消current响应和缓存path和route映射关系，获取matched
        this.current = window.location.hash.slice(1) || '/'
        Vue.util.defineReactive(this, 'matched', []);

        // 监听浏览器数据改变
        window.addEventListener('hashchange', this.onHashchange.bind(this));
        window.addEventListener('reload', this.onHashchange.bind(this));
    }

    // 更换current并重置matched
    onHashchange() {
        this.current = window.location.hash.slice(1)
        this.matched = []
        this.match()
    }

    // 获取match方法
    match(routes) {
        routes = routes || this.$options.routes
        for (const route of routes) {
            // 当当前路由为根路径
            if (route.path === '/' && this.current === '/') {
                this.matched.push(route)
            }
            // 当前路由为页面路由的子路由
            if (route.path !== '/' && this.current.indexOf(route.path) != -1) {
                this.matched.push(route)
                if (route.children) {
                    this.match(route.children)
                }
            }
        }
    }

}
```

在routerView中依照depth和matched获取component并渲染

``` js
// 依照depth和matched获取component并渲染
let component = null;
const route = this.$router.matched[depth]
if (route) {
    component = route.component
}
return h(component);
```
