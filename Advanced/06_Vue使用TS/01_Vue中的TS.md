# Vue中的TS

## 基本使用

1. Vue类中的属性就是data
2. Vue类中函数就是回调函数
3. Vue类中与生命周期同名的钩子函数，就是生命周期函数
4. 多使用范型，可对接口数据进行限制处理

## 声明文件

使用ts开发时如果要使用第三方js库的同时还想利用ts诸如类型检查等特性就需要声明文件

挂载$axios到vue原型上在组件里面直接用

``` JS
// main.ts
import axios from 'axios'
Vue.prototype.$axios = axios;
```

对$axios进行声明文件

``` JS
// shims-vue.d.ts
import Vue from "vue";
import {
    AxiosInstance
} from "axios";
declare module "vue/types/vue" {
    interface Vue {
        $axios: AxiosInstance;
    }
}
```

## 装饰器

### @Component

``` JS
@Component
export default class Hello extends Vue {
    addFeature(e: KeyboardEvent) {
        // ...
    });
}
```

### @Prop

``` JS
export default class HelloWorld extends Vue {
    // Props()参数是为vue提供属性选项
    // !称为明确赋值断言,它是提供给ts的
    @Prop({
        type: String,
        required: true
    })
    private msg!: string;
}
```

### @Emit

``` JS
// 通知父类新增事件,若未指定事件名则函数名作为事件名
@Emit()
private addFeature(event: any) {
    // ...
}
```

### @Watch

``` JS
@Watch('msg')
onMsgChange(val: string, oldVal: any) {
    console.log(val, oldVal);
}
```

## 路由

组件内路由守卫，会被当成普通函数，并不会触发

想要在生命周期调用方法，在APP页面(最外层页面, 其他组件都没有创建)对component用registerHook注册hooks用于触发路由守卫

## 状态管理

### store定义

store为范型类， 使用时推荐传入自定义类型

``` typescript
export default new Vuex.Store<RootState/*自定义类型*/>({
    // ...
})

```

### module使用

声明

``` TypeScript
// store/user.ts
export const user: Module<UserState, RootState> = {
    namespace: true,
    state: {
        name: "",
        token: ""
    },
    mutations: {
        setUser(state, { name, token }) {
            state.name = name;
            state.token = token;
        }
    }
}

```

使用

``` TypeScript
// store/index.ts
import {user} from './user'
modules：{
    user
}

```

``` TypeScript
// Home.vue
import { State, Action, Mutation, namespace } from 'vuex-calss'

const userModule = namespace('user)
export default class Home extends Vue {
    @userModule.State('name')
    username!: string // 此时username等同于states中的name
    @userModule.Mutation
    setUser!: (type: string, userInfo: { name: string, token: string }) => void // 此时setUser等同于states中的setUser
}
```

## Mixin

声明

``` TypeScript
import { Compnent } from "vue-property-decorator"

@Compnent
export default class MyMixin extends Vue {
    created() {
        // ...
    }

    customMethod() {
        // ...
    }
}
```

使用

``` TypeScript
// 支持多个Mixins使用， 或者全部放入一个数组中
export default class Home extends Mixins(MyMixin) {
    // ...
}
```

## 模仿装饰器

``` ts
// 组件装饰器
function Component(options: any){
    return function(target: any){
        // 返回构造函数
        return Vue.extend(options)
    }
}
```

Component转接导出，有两个实现方分为Vue（传参形式）和VueClass（Vue扩展类），最后到了componentFactory处理，遍历属性获取名称并处理，将处理的options extend到vue实例上

其他装饰器作为decorate属性赋值到options中，之后遍历处理
