# Vue核心

## 设计思想

1. 数据驱动应用
2. MVVM，即View（DOM）+ ViewModel（Vue） + Model（Data）

### MVVM理解

#### VM组成

1. DOM Listeners
2. Data Bindings

#### MVVM框架三要素

1. 响应式（数据绑定）
2. 模板引擎（模板编写和解析）
3. 渲染（模板转化为html元素）

#### 处理流程

1. 对Vue绑定ID的元素转化为虚拟DOM
2. 监听数据， 修改虚拟DOM
3. 将修改后的虚拟DOM转化为元素在页面展示
