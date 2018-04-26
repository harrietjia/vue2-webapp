# webapp vue2.x仿饿了么app项目总结

  项目地址：https://github.com/harrietjia/vue2webapp.git

  *如果觉得对您有帮助，希望可以在右上角给我个star支持一下，谢谢！*

## 实现功能：
```
•	Goods、Ratings、Seller组件视图均可上下滚动
•	商品页 点击左侧menu，右侧list对应跳转到相应位置
•	点击list查看商品详情页，父子组件的通信
•	评论内容够可以筛选查看
•	购物车组件，包括添加删除商品及动效，购物控件与购物车组件之间非父子组件通信，点击购物车图标，展示选择的商品列表
•	商家实景图片可以左右滑动
•	loaclStorage缓存商家信息（id、name）
```
## 项目结构分析：

    common/---- 文件夹存放的是通用的js,css和fonts
    components/----文件夹用来存放我们的 Vue 组件
    build/----文件是 webpack 的打包编译配置文件
    config/----文件夹存放的是一些配置项，比如我们服务器访问的端口配置等
    dist/----该文件夹一开始是不存在，在我们的项目经过 build 之后才会产出
    prod.server.js----该文件是测试是模拟的服务器配置，用来运行dist里面的文件，在config/index.js中,build对象中添加一条端口设置port：9000，
    App.vue----根组件，所有的子组件都将在这里被引用，eventHub空实例是用来组件间通信的中央数据总线作用，主要连接购买控件和购物车组件之间的数据通信
    index.html----整个项目的入口文件，将会引用我们的根组件 App.vue
    main.js----入口文件的 js 逻辑，在 webpack 打包之后将被注入到 index.html 中

## 各组件之间的关系：
```
├──APP.vue
  │  ├──Header.vue--头部组件
  │  │  ├──star.vue--星星评分组件
  │  ├──goods.vue--商品组件
  │  │  ├──shopcart.vue--购物车组件,包括小球飞入购物车动画,使用this.\$root.$on('cart.add', this.drop)接收，并给drop方法使用,这里特别说明`$root`，官方解释：表示当前组建树的根实例，如果根实例没有父实例，次实例将会是自己
  │  │  ├──cartcontrol.vue--购买控件--选中数量返回给父组件goods，goods响应后，重新计算选中数量，并用this.\$root.$emit（'name',event.target）将数据发送给购物车组件，
  │  │  ├──food.vue--商品详情页
  │  │  │  ├──ratingselect.vue--评价内容筛选组件
  │  ├──ratings.vue--评论组件
  │  │  ├──ratingselect.vue--评价内容筛选组件
  │  ├──seller.vue--商家组件

独立组件
  ├──split.vue--关于分割线组件
  ├──ratingselect.vue--评价内容筛选组件
```
## 开发过程问题和技巧汇总：

### 1、分开设置css样式：
图标icon.css--文字图标样式，通过icommon.io网站 将svg图片转成文字图标样式
公共base.css--处理设备像素比的一些样式,针对1px border的实现：主要是利用伪类和通过设置class针对DPR的一个y轴的缩放
工具mixin.css--设置border-1px样式和背景样式
css书写规范: 1、先设置布局 (display,position)
     2、设置宽高 (触发重绘的)
     3、设置字体、颜色 (可被继承的)

### 2、sticky-footer布局
header组件的详情页采用sticky-footer布局，主要特点是如果内容不够长，页脚部分也会贴在视窗底部，内容足够长，就会将页脚推到内容底部，父级position:fixed,内容设为padding-bottom:64px，页脚相对定位，margin-top:-64px

### 3、要求宽度自适应的布局，
采用flex布局,父级display:fixed,子元素flex:放大倍数，缩小倍数，基准值

### 4、transition过渡

在购买控件中使用transition过渡效果，实现添加减少按钮的动效，和小球飞入购物车的动效（模仿贝塞尔曲线的效果）

vue2.x里面定义了transition过渡状态，
name - string, 用于自动生成 CSS 过渡类名。

    例如：name: 'fade' 将自动拓展为.fade-enter，.fade-enter-active等。默认类名为 "v"

    fade-enter
    fade-enter-active
    fade-leave
    fade-leave-active

     例子：<transition name="fade"></transition>

          opacity: 1
          background: rgba(7, 17, 27, .8)
          &.fade-enter-active, &.fade-leave-active
            transition: all .5s
          &.fade-enter, &.fade-leave-active
            opacity: 0
            background: rgba(7, 17, 27, 0)

包括transition过渡的钩子函数

    before-enter
    before-leave
    before-appear
    enter
    leave
    appear
    after-enter
    after-leave
    after-appear
    enter-cancelled
    leave-cancelled (v-show only)
    appear-cancelled

### 5、seller组件：

#### 问题一：seller页面中商品商家实景图片横向滚动
解决方案：每个li要display：inline-block，因为width不会自动撑开父级ul，所以需要计算ul的width，**（每一张图片的width+margin）*图片数量-一个margin**，因为最后一张图片没有margin
同时new BScroll里面要设置`scrollX: true,eventPassthrough: 'vertical',// 滚动方向横向`

#### 问题二：打开seller页面，无法滚动
问题分析：出现这种现象是因为better-scroll插件是严格基于DOM的，数据是采用异步传输的，页面刚打开，DOM并没有被渲染，所以，要确保DOM渲染了，才能使用better-scroll，
解决方案：用到`mounted`钩子函数，同时搭配`this.$nextTick()`

#### 问题三：在seller页面，刷新后，无法滚动
问题分析：出现这种情况是因为mounted函数在整个生命周期中只会只行一次
解决方案：使用watch方法监控数据变化，并执行滚动函数 `this._initScroll();this._initPicScroll();`

## 6、goods,ratings,seller组件之间切换时会重新渲染
解决方案：在app.vue内使用`keep-alive`，保留各组件状态，避免重新渲染

```
<keep-alive>
    <router-view :seller="seller"></router-view>
</keep-alive>
```

### 7、food页面中图片的设置
pading-top设置为100%时，会根据宽度来计算padding，所以width:100%; height: 0;padding-top: 100%会是一个正方形的盒子

### 8、better-scroll的设置
better-scroll依赖于文档，所以最好在mounted中对其进行初始化
在初始时要对文档中的内容进行引用，最好放在vue.$nextTick()中进行，这样可以保证该引用已经存在于文档流中


## 总结

    将相同样式或功能的区块单独提出来，作为一个组件。

另外组件中用到的图片等资源就近维护，即可以考虑在组件文件夹中新建images文件夹。

抽离组件遵循原则：
*要尽量遵循单一职责原则，复用性更高，不要设置额外的margin等影响布局的东西*

### -css预处理器--stylus
全局安装，安装之前你需要你安装 nodejs

    $ npm install stylus -g
index.styl是stylus文件的入口文件，里面使用@import 引入各种styl文件

### vue2相较vue1有很多地方改动
比如

>*  v-for 中的隐式变量$index,$key已经取消，可以自己定义，例如 v-for="(itemClass,index) key="index"
>*  transition书写格式不在是在元素标签上写，而是作为一个标签`<transition></transition>`将目标元素包起来,过渡状态变为4种状态
>*  v-el 和 v-ref 都弃用，改为使用`ref`属性为元素或组件添加标记，通过`$refs`获取
>*  只允许一个根元素
>*  $dispatch废除，vue2.0中的事件用vue.$emit, vue.$on, $vue.off来派发，响应和解除(增加按钮案例)
>*  vue2.0中props中的数据只可以父组件来改变，当子组件试图对该属性进行改变之后vue的编译器会发出警告信息，我们一般不建议子组件对props中的数据进行改变，
    当必需时，可以用vue.$emit()方法将这个改变的事件派发出去，在你组件中进行接收并进行相应的改变
>*  生命周期的钩子函数
    beforeCreate
    created
    beforeMount
    mounted
    beforeUpdate
    updated
    beforeDestroy
    destroyed

### 项目运行
```
克隆项目到本地
git clone https://github.com/harrietjia/vue2webapp.git

安装依赖
npm install

本地开发，开启服务器，浏览器访问http://localhost:8888
npm run dev

构建生产
npm run build

运行打包文件
node prod.server.js

会看到 Listening at http://localhost:9000 在浏览器中打开即可

```
