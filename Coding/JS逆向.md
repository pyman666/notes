
- [webpack](#webpack)
- [jsdom](#jsdom)
- [proxy](#proxy)

## webpack

## **jsdom**

jsdom是一个纯粹由 javascript 实现的一系列 web标准，特别是dom和html标准，用于在nodejs中使用。大体上来说，该项目的目标是模拟足够的Web浏览器子集，以便用于测试和挖掘真实世界的Web应用程序

环境安装：
```bash
npm install jsdom --save
```
基本使用
```jsx
const jsdom = require("jsdom");
const { JSDOM } = jsdom;//导入jsdom模块
const html = "<!DOCTYPE html><p>逆向有你</p>";
const resourceLoader = new jsdom.ResourceLoader({
    userAgent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36",
});
const dom = new JSDOM(html, {
    url: "https://www.toutiao.com",
    referrer: "https://www.toutiao.com",
    contentType: "text/html",
    resources: resourceLoader,
})

window = global
document = dom.window.document
const params = {
    location: {
        host: "www.toutiao.com",
        href: "https://www.toutiao.com",
        origin: "https://www.toutiao.com",
        search: "",
    },
    navigator: {
        appCodeName: "Mozilla",
        appName: "Netscape",
        appVersion: "5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36",
    }
};
Object.assign(global,params)
```
## proxy
```jsx
//代理普通对象
var person = {
    name: 14,
    age: 444,
    hobbies: [
        '泡妞',
        '看妹子'
    ]
}
var proxy = new Proxy(window, {
    get(target, property){
        // console.log('调用了target:', target)
        console.log('调用了property: window的', property)
        // Reflect.get(target, property)
        return target[property]
    },
    set(target, property, value){
        console.log('设置对象window', value)
        target[property] = value
    }
})
proxy.age

//代理window
window = global
window.a = '4444';
window.b = {
    name: 'xiaosheng',
    height: '200cm',
    hobbies: [
        '钓鱼',
        '养鱼'
    ]
}

var proxy = new Proxy(window, {
    get(target, property){
        // console.log('调用了target:', target)
        console.log('调用了property: window的', property)
        // Reflect.get(target, property)
        return target[property]
    },
    set(target, property, value){
        console.log('设置对象window', value)
        target[property] = value
    }
})
proxy.a = 45
// console.log(b.length)
proxy.b.length

// ->设置对象window 45
// ->调用了property: window的 b
```
# 吐环境

todo
