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



## Header Component

![image-20220608151719241](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116058.png)

* Using Declarative Route Navigation vs Programmatic Route Navigation  

* Fix a bug in programmatic route navigation  

* The programmatic route jumps to the current route (parameters remain unchanged), and a warning error of Navigation Duplicated will be thrown. How to solve it?  

* Fix the push and replace methods on the Vue prototype  

```js
// Cache the push function on the prototype  
const originPush = VueRouter.prototype.push
const originReplace = VueRouter.prototype.replace
// Assign new function to push on prototype object  
VueRouter.prototype.push = function (location, onComplete, onAbort) {
  // Determine if no callback function is specified, call the original function and use catch to handle the error  
  if (onComplete===undefined && onAbort===undefined) {
    return originPush.call(this, location, onComplete, onAbort).catch(() => {})
  } else { // If any callback function is specified, call the original push function through call for processing  
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



```js
{
  path: '/register',
  component: Register,
  meta: { // Add this configuration to the route that needs to hide the footer    
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
//Add <Footer v-show="!$route.meta.isHideFooter"/> to the component
```

## Home Component

![image-20220608152939577](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116060.png)

### Subcomponent   

![image-20220608153006927](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116061.png)

## Encapsulate the ajax request module

```js
/* 
Secondary packaging of axios
* Configure a common base path and timeout  
* Display the request progress bar  
* The data which returned successfully isthe response body data（response.data）   
* Unified dispose all request errors, for the specific requests  which can also be processed or not processed  
*/
import axios from 'axios'
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'

//Configure hide the rotation progress bar in the upper right corner, but only the request progress bar   
NProgress.configure({ showSpinner: false }) 

const service = axios.create({
  baseURL: "/api", // basic path
  timeout: 15000   //  Timeout of the connection request  
})

service.interceptors.request.use((config) => {
  // display the request progress bar 
  NProgress.start()

  // return config
  return config
})

service.interceptors.response.use((response) => {
  // Hide  progress bar  
  NProgress.done()
  // return response body data
  return response.data
}, (error) => {
  //Hide  progress bar  
  NProgress.done()

  //  Unified dispose all request errors
  alert( `request errors: ${error.message||'unknown error'}`)

  // for the specific requests  which can also be processed or not processed later
  return Promise.reject(error)
})

export default service
```

## Configure proxy services

```js
devServer: {
  proxy: {
    '/api': { // Proxy forwarding only for requests whose request routes start with /api  
      target: 'http://182.92.128.115', // the target of repost：url
      changeOrigin: true // support cross-domain   
    }
  }
},
```

## Vuex

* The situation of data interaction is quite complicated based on the project size. Thus, we choose vuex module to manage data.  
* Vuex features:state、actions、mutations、getters、modules

## Mock/ Analog data interface  

Mockjs: Intercept ajax requests and generate random data to return

mock/mockServer.js

```js
// import mock module

import Mock from 'mockjs'
//import Jason data  
//webpack exposes images and JSoN data formats by default  
import banner from './banner.json'
import floor from './floor.json'
//mock data：the first parameter is the request address， the second one is the request data
Mock.mock("/mock/banner", { code: 200, data: banner }) // Simulate data of the carousel from homepage
Mock.mock("/mock/floor", { code: 200, data: floor }
```

api/ajaxMock.js

```js
/* 

*/
import axios from 'axios'

const mockAjax = axios.create({
  baseURL: "/mock", 
  timeout: 10000 
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

* Prepare search and query condition parameters  
* Display dynamic data of component  
* Search by category and keyword  
* Search by brands  
* Search by attributes  
* Sort Search  
* Custom paging components  

![image-20220608155022983](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116062.png)

![image-20220608155044667](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116063.png)

## Detail Route

* Partially enlarging picture  
* Carousel figure  

![image-20220608155212247](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116064.png)

## AddCartSuccess Route

Distinguish sessionStorage from localStorage

![image-20220608155413517](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116065.png)

## ShopCart Route

* Processing of user temporary ID
* Management of shopping cart data
* Monitor user input without using v-model
* Application of async / await / Promise.all() 

![image-20220608155635581](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116066.png)

## Registration and login router
* Response processing of components after registration/login request.     
* Automatically carry token data after logging in.  

![image-20220608155847993](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116067.png)

![image-20220608155904866](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116068.png)

Test account and password:

* Account: 13700000000  
* Password: 111111  

## Navigation and Route Guards

* Only after logging in, you can view the transaction/payment/personal center page.  
* You can view the login page only if you are not logged in.  
* Only when there is skuInfo data in the carried skuNum and sessionStorage,  you can view the page of adding to the shopping cart successfully.  
* You can only jump to the transaction page from the shopping cart page.  
* You can only jump to the payment page from the transaction  page.  
* Only from the payment page,  you can jumps to the successful payment page.  

## Order and Payment

* Submit the order  
* Pay with QR code  
* Get order status  

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

* When the target image didn't finish loading, the page will display the loading image during the process.
* The requested target image will only be loaded  when the <img> enters the visible range.

## Router Lazy Loading

* When packaging and building applications, the JS package could become very large, which will affect page loading. If we can split the components corresponding to different routers into different code blocks, and then load the corresponding components when the routers are accessed, it will be more efficient.  
* Essentially, Vue's asynchronous components are applied to routing components. which need to use the dynamic import syntax.

## Form Data Validation

* There are some register/login forms in the project, which need to be validated before submitting the request.  
* The request will be sent only after the forestage form is verified successfully.  
* If the validation fails, it will be prompted in the form of red text on the interface instead of alert.  
* The timing of validation is not only when clicking Submit, but also during the input process.  

![image-20220609110707175](https://150-9155-1312350958.cos.ap-chengdu.myqcloud.com/img202206091116077.png)
