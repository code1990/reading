#### 69用户画像之vuejs+nodejs构建前端项目讲解

安装nodejs

安装vue-cli

install webpack –g
npm install --global vue-cli
--registry=https://registry.npm.taobao.org

Npm install   --registry=https://registry.npm.taobao.org
```java

```
#### 70用户画像之vuejs+highcharts构建图表代码编写

npm install highcharts --save



```javascript
charts.vue
<template>
  <div class="x-bar">
    <div :id="id"  :option="option"></div>
  </div>
</template>
<script>
  import HighCharts from 'highcharts'
  export default {
    // 验证类型
    props: {
      id: {
        type: String
      },
      option: {
        type: Object
      }
    },
    watch: {
      option () {
        HighCharts.chart(this.id,this.option);
      }
    },
    mounted() {
      HighCharts.chart(this.id,this.option)
    }
  }
</script>
//=======================
highcharts
<template>
<div>
  <x-chart id="high" class="high" :option="option1"></x-chart>
 </div>
</template>
<script>
  // 导入chart组件
  var myvue = {};
  import XChart from './charts'
  export default {
    data() {
      return {
        option1:{
          chart: {
            type: 'column'
          },
          title: {
            text: '月平均降雨量'
          },
          subtitle: {
            text: '数据来源: WorldClimate.com'
          },
          xAxis: {
            categories: [
              '一月','二月','三月','四月','五月','六月','七月','八月','九月','十月','十一月','十二月'
            ],
            crosshair: true
          },
          yAxis: {
            min: 0,
            title: {
              text: '降雨量 (mm)'
            }
          },
          tooltip: {
            // head + 每个 point + footer 拼接成完整的 table
            headerFormat: '<span style="font-size:10px">{point.key}</span><table>',
            pointFormat: '<tr><td style="color:{series.color};padding:0">{series.name}: </td>' +
              '<td style="padding:0"><b>{point.y:.1f} mm</b></td></tr>',
            footerFormat: '</table>',
            shared: true,
            useHTML: true
          },
          plotOptions: {
            column: {
              borderWidth: 0
            }
          },
          series: [{
            name: '东京',
            data: [49.9, 71.5, 106.4, 129.2, 144.0, 176.0, 135.6, 148.5,500, 194.1, 95.6, 54.4]
          }]
        },
      }
    },
    beforeCreate:function(){
      myvue = this;
    },
    mounted:function(){
      myvue.other.title.text = '2010 ~ 2016 年太阳能行业就业人员发展情况';
      myvue.other.subtitle.text = '数据来源：thesolarfoundation.com';
      myvue.other.series = myvue.data;//数据
      myvue.other.yAxis.title.text = '就业人数'; //数据
      myvue.option = myvue.other;
    },
    components: {
      XChart
    }
  }
</script>
```
#### 71用户画像之vuejs+hightcharts构建图表效果演示
```javascript
//前端配置路由 index.js
// router
import Vue from 'vue'
import VueRouter from 'vue-router'
import home from '../components/home.vue'
import index from '../index.vue'

import store from '../store/index.js'
import VueResource from 'vue-resource'
import highcharts from '../components/highcharts.vue'
import baseyear from '../components/baseyear.vue'
import brandlike from '../components/brandlike.vue'
import carrier from '../components/carrier.vue'
import chaoManAndWomen from '../components/chaoManAndWomen.vue'
import consumptionlevel from '../components/consumptionlevel.vue'
import email from '../components/email.vue'
import usetype from '../components/usetype.vue'
import label from '../components/label.vue'


//highcharts的引入
Vue.use(VueResource)
Vue.use(VueRouter)

/* eslint-disable no-new */
// new Vue({
//   el: '#app',
//   render: h => h(App)
// })


// 0. 如果使用模块化机制编程， 要调用 Vue.use(VueRouter)

// 1. 定义（路由）组件。
// 可以从其他文件 import 进来
// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
// 我们晚点在讨论嵌套路由。
const routes = [
  { path: '/',name:"home",component: home},
  { path: '/highcharts',name:"highcharts",component: highcharts},
  { path: '/baseyear',name:"baseyear",component: baseyear},
  { path: '/brandlike',name:"brandlike",component: brandlike},
  { path: '/carrier',name:"carrier",component: carrier},
  { path: '/chaoManAndWomen',name:"chaoManAndWomen",component: chaoManAndWomen},
  { path: '/consumptionlevel',name:"consumptionlevel",component: consumptionlevel},
  { path: '/email',name:"email",component: email},
  { path: '/usetype',name:"usetype",component:usetype},
  { path: '/label',name:"label",component:label}
]

// 3. 创建 router 实例，然后传 `routes` 配置
// 你还可以传别的配置参数, 不过先这么简单着吧。
const router = new VueRouter({
  mode: 'history',
  routes // （缩写）相当于 routes: routes
})

// 4. 创建和挂载根实例。
// 记得要通过 router 配置参数注入路由，
// 从而让整个应用都有路由功能
const app = new Vue({
  store,
  router,
  render: h => h(index)
}).$mount('#app')

// 现在，应用已经启动了！

```

#### 77用户画像之vuejs整合前端查询接口代码编写
```java
//baseyear.vue
<template>
    <div>
      <x-chart id="high" class="high" :option="option"></x-chart>
    </div>
</template>

<script>
  // 导入chart组件
  var myvue = {};
  import XChart from './charts'
  export default {
    data() {
      return {
        option:{
          chart: {
            type: 'column'
          },
          title: {
            text: '月平均降雨量'
          },
          subtitle: {
            text: '数据来源: WorldClimate.com'
          },
          xAxis: {
            categories: [
              '一月','二月','三月','四月','五月','六月','七月','八月','九月','十月','十一月','十二月'
            ],
            crosshair: true
          },
          yAxis: {
            min: 0,
            title: {
              text: '降雨量 (mm)'
            }
          },
          series: [{
            name: '东京',
            data: [49.9, 71.5, 106.4, 129.2, 144.0, 176.0, 135.6, 148.5,500, 194.1, 95.6, 54.4]
          }]
        },
      }
    },
    beforeCreate:function(){
      myvue = this;
    },
    created() {
      //新增用户
      this.$http.post('http://127.0.0.1:8764/mongoData/resultinfoView',{
          "type": "yearBase"
        }).then((response) => {
          this.option = {
                        chart: {
                          type: 'column'
                        },
                        title: {
                          text: '年代趋势'
                        },
                        xAxis: {
                          categories: response.body.infolist,
                          crosshair: true
                        },
                        yAxis: {
                          min: 0,
                          title: {
                            text: '数量'
                          }
                        },
                        series: [{
                          name: '年代',
                          data: response.body.countlist
                        }]
          };
      });
    },
    components: {
      XChart
    }
  }
</script>

<style scoped>

</style>

```
#### 78用户画像之vuejs整合前段查询接口之跨域问题解决
```java

```
#### 79用户画像之前端查询接口进一步封装代码编写
```java

```
#### 80用户画像之接口重构代码编写
```java

```
#### 81用户画像之前端查询接口重用改造代码编写
```java

```
#### 82用户画像vuejs完善剩余图表代码编写1
```java

```
#### 83用户画像之vuejs完善剩余图表代码编写2
```java

```
#### 84用户画像之vuejs完善剩余图表代码编写3
```java

```
#### 85用户画像之vuejs配置路由代码编写
```java

```
#### 86用户画像之接口服务前端查询服务以及前端展示服务联调以及效果展示.zip
```java

```
#### 87用户画像之TF-IDF通俗讲解
```java

```
#### 88用户画像之分词工具ik讲解以及代码编写.zip
```java

```
#### 89用户画像之java实现TF-IDF代码编写1
```java

```
#### 90用户画像之java实现TF-IDF代码编写2
```java

```
#### 91用户画像之flink实现分布式TF-IDF代码编写1
```java

```
#### 92用户画像之flink实现分布式TF-IDF代码编写2、
```java

```
#### 93用户画像之fink分布式TF-IDF实现用户年度、月度，季度商品关键词代码编写1
```java

```
#### 94用户画像之fink分布式TF-IDF实现用户年度、月度，季度商品关键词代码编写2
```java

```
#### 95用户画像之fink分布式TF-IDF实现用户年度、月度，季度商品关键词代码编写3
```java

```
#### 96用户画像之fink分布式TF-IDF实现用户年度、月度，季度商品关键词代码编写4
```java

```
#### 97用户画像之标签接口之败家指数接口代码编写
```java

```
#### 98用户画像之全部标签接口代码编写
```java

```
#### 99用户画像之前端标签查询服务代码编写
```java

```
#### 100用户画像之vue.js标签显示代码编写1
```java

```
#### 101用户画像之vue.js标签显示代码编写2以及效果演示
```java

```
