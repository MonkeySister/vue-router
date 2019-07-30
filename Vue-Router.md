## vue-router基础及进阶总结

### 基础

- **HTML导航**，

  <router-link to='路由路径'></router-link>，之后将组件显示在</router-view>里。其中router-link会被渲染成一个a标签。

- **JavaScript导航**，（路由基础，少不了，最常用，最好封装成路由模块）

  1. 定义路由组件

  2. 定义路由routes

  3. 导入Vue-Router，创建router实例，并配置routes参数

  4. 创建和挂载根实例

  5. 如果是單獨的路由文件，导出路由的同时在main.js中使用路由，并挂载到根实例中。

     定义路由模块index.js

  ```js
  import Vue from 'vue'
  import VueRouter from 'vue-router'
  // 路由懒加载模式，改变首屏加载过于缓慢的问题
  // 改成使用哪个组件加载哪个组件
  // 这样写后在打包 时每个组件会变成一个对应的js文件，否者会变成一个大的js文件
  Vue.use(VueRouter)
  const router = new VueRouter({
          routes: [
          {
                  name: 'home',
                  path: '/',
                  component:()=>import (`@/components/home.vue`),
              },   
              {
                  name: 'login',
                  path: '/login',
                  component: () =>import (`@/components/Login.vue`)
              }
          ]
      })
  ```

  在main.js中导入index.js并将router加入到根实例中

  ```js
  import router from './router'
  new Vue({
    el: '#app',
    router,
    components: {
      App
    },
    template: '<App/>'
  })
  ```

- **404 NOT Found**

  使用通配符 * 来匹配任意路径，但是在使用通配符时要注意路由的顺序，由于路由匹配的优先级是按照路由的定义顺序，所以含有通配符的路由要放到最后，通常用于客户端的404错误。

  ```js
  {
      path:'*',
      name:'404 NOT Found',
      component:()=>import (`@/components/404.vue`),
  }
  ```

- **命名路由**

  通过名称来标识路由，

  ```js
  const router = new VueRouter({
    routes: [
      {
        path: '/user/:userId',
        name: 'user',
        component: User
      }
    ]
  })
  ```

  路由跳转时可以直接通过name来进行跳转

  ```js
  this.$router.push('user');
  ```

- **路由嵌套**

  ```js
  <view-router><view-router></view-router></view-router>
  ```

  路由中通过child来嵌套子路由

  ```js
  routes: [{
      name: 'home',
      path: '/',
      component: Home,
      children: [{
          name: 'users',
          path: '/users',
          component: Users
      }],
  }]
  ```

- **编程式导航**

  vue中的导航除了使用<view-router></view-router>外，还可以使用 this.$router.push来进行路由的跳转。

  ```js
  router.push({ name: 'user', params: { userId }}) 
  ```

  <u>如果提供了path,params会被忽略；name配params，path配query传参</u>

- 命名视图

  在一个组件中同级显示多个组件，如果router-view没有name属性，则默认为default

  ```
  <router-view class="view one"></router-view>
  <router-view class="view two" name="a"></router-view>
  <router-view class="view three" name="b"></router-view>
  ```

  在路由定义中

  ```
  const router = new VueRouter({
    routes: [
      {
        path: '/',
        components: {
          default: Foo,//匹配one
          a: Bar,
          b: Baz
        }
      }
    ]
  })
  ```

- 重定向

  重定向就是redirect,重新定向到另一个组件，

  ```js
  routes:[{
      path:'/a',
      redirect:'/b',
  }]
  routes:[{
      path:'/a',
      redirect:{
          name:'b',
      }
  }]
  ```

- 传参

  路由传参我一般通过导航式路由放在params中或者query中然后通过this.$router.params或者通过this.$router.query中取得参数。

- HTML5 history模式

  还不是横明白，学会了再说

### 路由进阶

- 全局前置守卫router.beforeEach

  一般在router.js模块中为路由的实例化对象定义一个全局的路由守卫，来判断当访问是否携带token

  ```
  //创建一个全局的前置守卫,确保每个路由的访问都带有token
  router.beforeEach((to, from, next) => {
      if (to.name === 'login') {
          next();
      } else {
          if (!localStorage.getItem('token')) {
              Message.warning('请先登录!');
              Router.replace('login');
          } else {
              next();
          }
          // next();
      }
  });
  ```

- 全局后置钩子

  router.afterEach((to,from)=>{

  });暂时还没用到。

- 单个路由独享守卫beforeEnter,为涉及到

  ```
  const router = new VueRouter({
    routes: [
      {
        path: '/foo',
        component: Foo,
        beforeEnter: (to, from, next) => {
        }
      }
    ]
  })
  ```

- 组件内守卫

  

  ```js
  beforeRouterEnter(to,from,next){
      // 在渲染该组件的对应路由被 confirm 前调用
      // 不！能！获取组件实例 `this`
      // 因为当守卫执行前，组件实例还没被创建
      //虽然此时没有组件实例this但是可以在next()回调中通过vm来调用组件中的方法
      beforeRouteEnter (to, from, next) {
      const pathName = from.name;
          next(vm=>{
            vm.setActiveName(pathName);
          });
    	},
  };
  beforeRouterUpdate(to,from,next){
      // 在当前路由改变，但是该组件被复用时调用
      // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
      // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
      // 可以访问组件实例 `this`
  };
  //这个离开守卫通常用来禁止用户在还未保存修改前突然离开
  beforeRouterLeave(to,from,next){
      // 导航离开该组件的对应路由时调用
      // 可以访问组件实例 `this`
      const answer = window.confirm('Do you really want to leave? you have unsaved changes!')
    if (answer) {
      next()
    } else {
      next(false)
    }
  };
  ```

- 完整的导航解析流程

  1. 导航被触发
  2. 在失活的组件里调用beforeRouterLeave
  3. 离开失活的组件，调用全局守卫beforeEach
  4. 在重用的组件里调用beforeRouterUpdate守卫
  5. 在路由匹配里调用beforeEnter
  6. 解析异步路由组件
  7. 在被激活的组件里调用beforeRouterEnter
  8. 调用全局的路由解析守卫，beforeResolve
  9. 导航被确认
  10. 调用全局的后置钩子afterEach
  11. 触发DOM更新
  12. 用创建好的实例调用beforeRouterEnter

  

