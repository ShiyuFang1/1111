# shangpinhuishop
## Build Setup

```

# cd shangpinhuishop

# npm install

# npm run serve
```
## Technological Selection

![image-20220608150139753](https://raw.githubusercontent.com/ShiyuFang1/Shangpinhui-shopping-website/main/src/components/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220920105818.png)

## Router

![image-20220608150232527](https://raw.githubusercontent.com/ShiyuFang1/Shangpinhui-shopping-website/main/src/components/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220920114306.png)

## Menu Introduction

![image-20220608150502494](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116057.png)

## Header Component

![image-20220608151719241](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116058.png)

使用声明式路由导航与编程式路由导航

解决编程式路由导航的一个错误

编程式路由跳转到当前路由(参数不变), 会抛出NavigationDuplicated的警告错误,如何解决?

通过修正Vue原型上的push和replace方法

```js
// 缓存原型上的push函数
const originPush = VueRouter.prototype.push
const originReplace = VueRouter.prototype.replace
// 给原型对象上的push指定新函数函数
VueRouter.prototype.push = function (location, onComplete, onAbort) {
  // 判断如果没有指定回调函数, 通过call调用源函数并使用catch来处理错误
  if (onComplete===undefined && onAbort===undefined) {
    return originPush.call(this, location, onComplete, onAbort).catch(() => {})
  } else { // 如果有指定任意回调函数, 通过call调用源push函数处理
    originPush.call(this, location, onComplete, onAbort)
  }
}
VueRouter.prototype.replace = function (location, onComplete, onAbort) {
  if (onComplete===undefined && onAbort===undefined) {
    return originReplace.call(this, location, onComplete, onAbort).catch(() => {})
  } else {
    originReplace.call(this, location, onComplete, onAbort)
  }
}
```



## Footer Component

![image-20220608152337887](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116059.png)

利用路由元信息meta配置和v-show控制footer组件的显示和隐藏在

```js
{
  path: '/register',
  component: Register,
  meta: { // 需要隐藏footer的路由添加此配置
    isHideFooter: true
  }
},

{
  path: '/login',
  component: Login,
  meta: {
    isHideFooter: true
  }
},
//在组件上面添加<Footer v-show="!$route.meta.isHideFooter"/>
```

## Home Component

![image-20220608152939577](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116060.png)

子组件

![image-20220608153006927](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116061.png)

## 封装ajax请求模块

```js
/* 
对axios进行二次包装
1. 配置通用的基础路径和超时
2. 显示请求进度条
3. 成功返回的数据不再是response, 而直接是响应体数据response.data
4. 统一处理请求错误, 具体请求也可以选择处理或不处理
*/
import axios from 'axios'
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'

// 配置不显示右上角的旋转进度条, 只显示水平进度条
NProgress.configure({ showSpinner: false }) 

const service = axios.create({
  baseURL: "/api", // 基础路径
  timeout: 15000   // 连接请求超时时间
})

service.interceptors.request.use((config) => {
  // 显示请求中的水平进度条
  NProgress.start()

  // 必须返回配置对象
  return config
})

service.interceptors.response.use((response) => {
  // 隐藏进度条
  NProgress.done()
  // 返回响应体数据
  return response.data
}, (error) => {
  // 隐藏进度条
  NProgress.done()

  // 统一处理一下错误
  alert( `请求出错: ${error.message||'未知错误'}`)

  // 后面可以选择不处理或处理
  return Promise.reject(error)
})

export default service
```

## 配置代理服务器

```js
devServer: {
  proxy: {
    '/api': { // 只对请求路由以/api开头的请求进行代理转发
      target: 'http://182.92.128.115', // 转发的目标url
      changeOrigin: true // 支持跨域
    }
  }
},
```

## 使用vuex管理状态

由于项目体积比较大，向服务器发请求的接口过多，服务器返回的数据也会很多，如果还用以前的方式存储数据，导致vuex中的state数据格式比较复杂。采用vuex模块式管理数据。
Vuex核心概念:state、actions、mutations、getters、modules

## Mock/模拟数据接口

Mockjs: 用来拦截ajax请求, 生成随机数据返回

mock/mockServer.js

```js
// 先引入mockjs模块

import Mock from 'mockjs'
//把JSON数据格式引入进来[JSON数据格式根本没有对外暴露，但是可以引入]
//webpack默认对外暴露的：图片、JSoN数据格式
import banner from './banner.json'
import floor from './floor.json'
//mock数据：第一个参数请求地址第二个参数：请求数据
Mock.mock("/mock/banner", { code: 200, data: banner }) //模拟首页大的轮播图的数据
Mock.mock("/mock/floor", { code: 200, data: floor })
```

api/ajaxMock.js

```js
/* 
专门请求mock接口的axios封装
*/
import axios from 'axios'

const mockAjax = axios.create({
  baseURL: "/mock", // 路径前缀
  timeout: 10000 // 请求超时时间
})

mockAjax.interceptors.request.use((config) => {
  return config
})

mockAjax.interceptors.response.use((response) => {
  return response.data
}, (error) => {
  return Promise.reject(error)
})

export default mockAjax
```

api/index.js

```js
import mockAjax from './mockAjax'

// Get the advertisement carousel list
export const reqBanners = ()=> mockAjax.get('/banners')

// Get floor list of home page
export const reqFloors = ()=> mockAjax.get('/floors')
```

## Search Route

1. Prepare search and query condition parameters
2. Display dynamic data of component
3. Search by category and keyword
4. Search by brands
5. Search by attributes
6. Sort Search
7. Custom paging components

![image-20220608155022983](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116062.png)

![image-20220608155044667](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116063.png)

## Detail Route

1)Partially enlarging picture
2)Carousel figure

![image-20220608155212247](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116064.png)

## AddCartSuccess Route

Distinguish sessionStorage与localStorage

![image-20220608155413517](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116065.png)

## ShopCart Route

1) Processing of user temporary ID
2) Management of shopping cart data
3) Monitor user input without using v-model
4)Application of async / await / Promise.all() 

![image-20220608155635581](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116066.png)

## Registration and login router
1) Response processing of components after registration/login request. 
2) Automatically carry token data after logging in.

![image-20220608155847993](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116067.png)

![image-20220608155904866](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116068.png)

Test account and password:

Account: 13700000000
Password: 111111

## Navigation and Route Guards

a. Only after logging in, you can view the transaction/payment/personal center page.
b. You can view the login page only if you are not logged in.
c. Only when there is skuInfo data in the carried skuNum and sessionStorage,  you can view the page of adding to the shopping cart successfully.
d. You can only jump to the transaction page from the shopping cart page.
e.You can only jump to the payment page from the transaction  page.
f. Only from the payment page,  you can jumps to the successful payment page.

## Order and Payment

1) Submit the order
2) Pay with QR code
3) Get order status

![image-20220609105837105](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116069.png)

![image-20220609105953404](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116070.png)

![image-20220609110004433](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116071.png)

## Payment Component

![image-20220609110322172](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116072.png)

![image-20220609110333949](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116073.png)

## Successful Payment Component

![image-20220609110358429](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116074.png)

## Order Component

![image-20220609110438770](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116075.png)

## Image Lazy Loading

When the target image didn't finish loading, the page will display the loading image during the process.
Moreover, the requested target image will only be loaded  when the <img> enters the visible range.

## Router Lazy Loading

(1) When packaging and building applications, the JS package could become very large, which will affect page loading. If we can split the components corresponding to different routers into different code blocks, and then load the corresponding components when the routers are accessed, it will be more efficient.
(2) Essentially, Vue's asynchronous components are applied to routing components. which need to use the dynamic import syntax.

## Form Data Validation

(1) There are some register/login forms in the project, which need to be validated before submitting the request.
(2) The request will be sent only after the forestage form is verified successfully.
(3) If the validation fails, it will be prompted in the form of red text on the interface instead of alert.
(4) The timing of validation is not only when clicking Submit, but also during the input process.

![image-20220609110707175](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116077.png)
