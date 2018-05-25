# 180524-Review

## 首屏加载技术

> 页面同步加载过程: 

  * 视觉预期: 入口loading组件, 页头progressBar, 增加首屏视觉预期
  * SSR: node服务端渲染, 前端组件作为字符串模板在后端ssr(前后端同构)
  * 组件懒加载: '火车票', '机票', 这两个tab, 不是首屏默认选中tab, 可以异步加载(webpack import + magic Comments), 若使用React,由于样式和js文件分离, 可以将tab[index].css与tab[index].js打到一个bundle里, 保证加载组件时样式及时输出
  * 组件下拉加载: '更多旅行'组件由于在页面底部, 不在初渲染范围内, 此组件可异步加载(监听window的scroll事件,获取dom.getBoundingClientRect().top与window.innerHeight做对比)
  * 组件复用: '国内酒店','国际酒店','钟点房',可封装为一个组件用props控制复用
  * 默认数据占位: 设置需填充的默认数据值占位(比如地理信息), 在第一次用户进入页面时显示默认占位数据
  * 组件占位 1: '一大波优惠正在来袭'组件占位
  * 组件占位 2: 轮播图底部用一个img组件占位, 设置宽高, 防止抖动, src设置base64(webpack url-loader);
  * 组件占位 3: '查找酒店'上显示一个动画效果组件, 增加视觉预期

> 同步加载完成后(DomContentLoaded事件)的异步加载过程: 

  * ajax获取活动内容, 用活动组件(webpack import + magicComments)替换'一大波优惠正在来袭'组件
  * 浏览器存储: 从localStorage获取地理位置信息, 替换页面默认数据占位, 并存入Redux或Vuex(方便组件间地理位置共享), 将'查找酒店'button上的动画组件卸载
  * 用cookie带着sessionid请求服务端session判断身份(因为不需数据,可用head请求只获取报文首部), 异步加载我的酒店组件
  * ajax获取轮播图图片地址数组, 异步加载轮播图组件(图片从cdn加载)
  
> 工程构建:

  * 减少http请求数
    * 合并js文件
    * 2 * 4 图标区块的所有图标打进一个sprite图中(webpack postcss-sprites) 
    * 提供base64 (url-loader)
    * webpack构建时将manifest部分打入页面与ssr一同输出 (HtmlWebpackPlugin + HtmlWebpackInlineSourcePlugin)
  * 减少请求体积:
    * js/css/html Uglify
    * 生产环境上线后服务器开启gzip压缩
    * webpack import + magicComments, 非核心代码异步加载
  * 其他:
    * 预解析dns: 用meta标签和link标签设置隐式dns预解析和显式dns预解析, 为之后加载不同域下的页面进行dns预解析
    * 利用缓存: 在webpack打包时使用chunkhash代替hash (NamedModulesPlugin),使其在内容不更改时可利用浏览器缓存
    * 使用cdn: 将资源放在cdn上
      
## 一面代码Review
    
``` javascript
// 1 实现符合特定需求的setInterval
// 总结: 第一问没问题, 第二问(添加clearTimeout功能)光想了怎么return setid, 换个角度用思考问题, 用闭包解决
function _setIterval(callback, time) {
  var _setid
  _setid = setTimeout(function cb() {
    callback()
    _setid = setTimeout(cb, time)
  }, time)
  return function() {
    clearTimeout(_setid)
  }
}

// 2 顺序拼接异步接收的字符串
// 总结: Promise.all虽学过但没用过, 第一次用

// html结构
// <input type="text" id="input">
// <button id="btn">点我发送请求</button>

// 模拟http接口,响应异步随机返回
function httpFeedBack() {
  var i = Math.random() * 10
  var ret = (Math.random() * 10000000000000 + '').slice(1, 4)
  console.log(ret)
  return new Promise(function(resolve) {
    setTimeout(function() {
      resolve(ret)
    }, 200 * i)
  })
}

var input = document.getElementById('input')

var getAll = (function() {
  // 设置promise数组
  var promiseAll = []
  // return 闭包, 操作promiseAll
  return function() {
    var value = ''
    promiseAll.push(httpFeedBack())
    Promise.all(promiseAll)
      .then(function(datas) {
        datas.forEach(function(item, index) {
          value += item
        })
      })
      .then(function() {
        input.value = value
      })
  }
})()

document.getElementById('btn').onclick = getAll

// 3 'ababbbccc'规定只能用正则表达式去重
// 总结: 正则表达式'\num'知识盲点
function deRepeat(str) {
  str = str.replace(/(.)(\1)+/g, '$1')
  return str
}
console.log(deRepeat('ababbbccc')) // ababc
```
