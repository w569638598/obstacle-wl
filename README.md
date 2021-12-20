# 我自己用的。是的没错，就是给我自己用的。
# for me

##### 求大神指导，我就想让图片就是图片资源，不想转成base64，打包后图片地址就是http:localhost:8080/js/img/,百度翻译一下
###### Ask God for guidance. I just want the picture to be a picture resource. I don't want to turn it into Base64. After packaging, the picture address ishttp:localhost:8080/js/img/
---
#### [github](https://github.com/w569638598/obstacle-wl.git)
---
###### 文档都没有你们下载下来干什么？
###### 不会的就 [百度](https://www.baidu.com)
----

##### 引入方法
```javascript


import x6 from './x6'

const components = [x6]

function install(Vue){
    components.forEach(comp => {
        Vue.component(comp.name, comp)
    })
}

if(typeof window !== "undefined" && window.Vue){
    install(window.Vue)
}

export { x6 }

export default { install }
```
---- 
`javascript`

![](call.jpg)