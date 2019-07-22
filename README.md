# vue-router-dispense

一个基于 vue-router 的扩展， vue-router 原有功能及配置完全保留，主要功能有以下几点：

1. 指定路由配置模块，统一加载，无需单个 import 进来
2. 可开启路由访问记录和路由面包屑导航信息
3. 可以指定路由过滤器函数，自由定义路由权限

##### 使用 vue-router-dispense 之前路由的配置项是这样的

```
export default new Router({
  mode: 'history',
  base: process.env.BASE_URL,
  scrollBehavior: () => ({ y: 0 }),
  routes: [
    {
      path: '/login',
      name: 'login',
      component: () => import('../views/login.vue'),
      meta: {
        title: '登录'
      }
    },
    {
      path: '/form',
      name: 'form',
      component: () => import('@/views/index'),
      redirect: '/form/basic-form',
      children: [
        {
          path: '/form/basic-form',
          name: 'basicForm',
          component: () => import('./basic.vue')
        },
        {
          path: '/form/step-form',
          name: 'stepForm',
          component: () => import('./step.vue')
        }
      ]
    },
    {
      path: '',
      name: 'home',
      component: () => import('@/views/index'),
      redirect: '/dashboard',
      children: [
        {
          path: '/dashboard',
          name: 'dashboard',
          component: () => import('./index.vue')
        }
      ]
    }
  ]
})
```

##### 或者是这样的

```
import form from '../router/form.js'
import dashboard from '../router/dashboard.js'

export default new Router({
  mode: 'history',
  base: process.env.BASE_URL,
  scrollBehavior: () => ({ y: 0 }),
  routes: [
    {
      path: '/login',
      name: 'login',
      component: () => import('../views/login.vue'),
      meta: {
        title: '登录'
      }
    },
    ...form,
    ...dashboard
  ]
})
```

##### 使用 vue-router-dispense 之后

```
export default new Router({
  mode: 'history',
  base: process.env.BASE_URL,
  scrollBehavior: () => ({ y: 0 }),
  routes: [
    {
      path: '/login',
      name: 'login',
      component: () => import('../views/login.vue'),
      meta: {
        title: '登录'
      }
    }
  ], // layout 静态路由
  modules: require.context('../views', true, /router\.js/), // 指定定义模块路由的文件路径并加载
  record: true, // 是否保存历史访问记录
  breadcrumb: true, // 保存面包屑记录
  filter (routes) { // 路由过滤函数，多用于权限控制
    // console.log(routes)
    return routes
  },
  beforeEach (to, from, next) { // 前置钩子
    // console.log('before each')
    next()
  },
  afterEach (to, from) { // 后置钩子
    // console.log('after each')
  }
})
```

#### Options

- routes（Array）

静态路由，通常也是白名单路由

- modules（File Module）

配置路由模块的文件模块，示例中配合 webpack 的 require.context 统一加载

- record（Boolean）

是否开启路由访问记录，在 vue 组件中可以通过 this.$router.records 访问

```
this.$router.records
// [
//    {
//        fullPath: "/form/basic-form"
//        hash: ""
//        matched: (2) [{…}, {…}]
//        meta: {}
//        name: "basicForm"
//        params: {}
//        path: "/form/basic-form",
//        query: {}
//    },
//    {
//        fullPath: "/form/step-form"
//        hash: ""
//        matched: (2) [{…}, {…}]
//        meta: {}
//        name: "stepForm"
//        params: {}
//        path: "/form/step-form",
//        query: {}
//    },
//    {
//        fullPath: "/dashboard"
//        hash: ""
//        matched: (2) [{…}, {…}]
//        meta: {}
//        name: "dashboard"
//        params: {}
//        path: "/dashboard"
//        query: {}
//        redirectedFrom: "/"
//    }
//]
```

- breadcrumb（Boolean）

是否开启面包屑记录，在 vue 组件中可以通过 this.$router.breadcrumbs 访问

```
this.$router.breadcrumbs
//[
//    {
//        component: ƒ component()
//        name: "home"
//        path: ""
//        redirect: "/dashboard"
//    },
//    {
//        beforeEnter: undefined
//        components: {default: {…}}
//        instances: {default: VueComponent}
//        matchAs: undefined
//        meta: {}
//        name: "form"
//        parent: undefined
//        path: "/form"
//        props: {}
//        redirect: "/form/basic-form"
//        regex: /^\/form(?:\/(?=$))?$/i
//    },
//    {
//        beforeEnter: undefined
//        components: {default: {…}}
//        instances: {default: VueComponent}
//        matchAs: undefined
//        meta: {}
//        name: "stepForm"
//        parent: {path: "/form", regex: /^\/form(?:\/(?=$))?$/i, components: {…}, instances: {…}, name: "form", …}
//        path: "/form/step-form"
//        props: {}
//        redirect: undefined
//        regex: /^\/form\/step-form(?:\/(?=$))?$/i
//    }
//]
```

- filter（Function(routes)）

路由配置项的过滤器函数，函数接收一个参数 routes，是过滤前的路由配置信息，函数需要返回一个数组（Array）作为新的路由配置项。该函数通常可以用来做权限控制。

- beforeEach

路由前置钩子函数，同 vue-router

- afterEach

路由后置钩子函数，同 vue-router
