---
highlight: a11y-dark
---
# vue3(基于vite)

# 配置环境

```
npm init vite@latest

yarn create vite

pnpm init vite@latest
```




# vite配置

**配置路径别名**

`npm install @types/node -D //安装node变量声明`

```
---vite.config.js----
import { defineConfig } from 'vite'
import path from 'path'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  resolve: {
    alias: [{ find: '@', replacement: path.resolve(__dirname, 'src') }]
  },
  server: {
    host: true, // host设置为true才可以使用network的形式，以ip访问项目
    port: 8080, // 端口号
    open: true, // 自动打开浏览器
    cors: true, // 跨域设置允许
    strictPort: true, // 如果端口已占用直接退出
    // 接口代理
    proxy: {
      "/api": {
        // 本地 8000 前端代码的接口 代理到 8888 的服务端口
        target: "http://localhost:8888/",
        changeOrigin: true, // 允许跨域
        rewrite: (path) => path.replace("/api/", "/"),
      },
    },
  },
  plugins: [vue()]
})

```

## ts.config.json中配置路径

    tsconfig.json
    
    {
      "compilerOptions": {
        "target": "ESNext",
        "useDefineForClassFields": true,
        "module": "ESNext",
        "moduleResolution": "node",
        "strict": true,
        "jsx": "preserve",
        "resolveJsonModule": true,
        "isolatedModules": true,
        "esModuleInterop": true,
        "lib": [
          "ESNext",
          "DOM"
        ],
        "skipLibCheck": true,
        "noEmit": true,
        "baseUrl": "./",
        "paths": {
          "@/*": [
            "src/*"
          ]
        }
      },
      "include": [
        "src/**/*.ts",
        "src/**/*.d.ts",
        "src/**/*.tsx",
        "src/**/*.vue"
      ],
      "references": [
        {
          "path": "./tsconfig.node.json"
        }
      ]
    }
    //注意：为了防止未使用导入模块的警告
    需要将
     "noUnusedLocals": false,
     //防止TS中通过变量存储key值读取对象的属性值时报错
     "suppressImplicitAnyIndexErrors":true,


## vite-demo项目框架

`public:` 存放静态资源，特点不会被vite编译

`src/assets:` 也可以存放静态资源，如图片，但是可以通过配置将其打包编译成base64，这样不浪费资源

`vite-env.d.ts:` 声明文件的扩充

注意：**自己实现一个vue-cli的时候也需要这个声明文件**

    declare module "*.vue" {
      import { DefineComponent } from "vue"
      const component: DefineComponent<{}, {}, any>
      export default component
    }

`index.html:` webpack是通过js来作为入口文件的,index.js。但是vite是通过html作为入口文件，然后通过sctip type="module"进行导入

## 强制使用pnpm包管理工具

团队开发项目的时候，需要统一包管理器工具,因为不同包管理器工具下载同一个依赖,可能版本不一样,

导致项目出现bug问题,因此包管理器工具需要统一管理！！！

在根目录创建`scripts/preinstall.js`文件，添加下面的内容

```js
if (!/pnpm/.test(process.env.npm_execpath || '')) {
  console.warn(
    `\u001b[33mThis repository must using pnpm as the package manager ` +
    ` for scripts to work properly.\u001b[39m\n`,
  )
  process.exit(1)
}
```

配置命令

    "scripts": {
        "preinstall": "node ./scripts/preinstall.js"
    }

**当我们使用npm或者yarn来安装包的时候，就会报错了。原理就是在install的时候会触发preinstall（npm提供的生命周期钩子）这个文件里面的代码。**

## vite环境变量的配置

创建三个文件

    .env.development
    .env.production
    .env.test

<!---->

    # 变量必须以 VITE_ 为前缀才能暴露给外部读取
    NODE_ENV = 'development'
    VITE_APP_TITLE = '硅谷甄选运营平台'
    VITE_APP_BASE_API = '/dev-api'

**使用环境变量**
`import.meta.env`

## vite的优势：

①冷启动。vite启动时无需打包编译，不需要分析各个模块的依赖关系，可以直接启动服务器。只有浏览器请求相应模块时才会对相应模块进行打包编译。而webpack启动时，首先需要确定入口，从entry入口开始，递归解析所有的模块文件，调用loader进行加载处理，如果当前模块依赖于其他模块还需要对被依赖模块进行加载处理。处理完毕后得到转化内容和依赖关系，然后组装成一个个代码块chunk。之后再把chunk转化成文件添加到输出列表中

②热更新时，vite只需要对相应模块进行重编译，而webpack需要对当前模块和当前模块所依赖的模块进行重编译。

# svg图标的封装

## 安装插件

`npm install vite-plugin-svg-icons -D`

`npm install fast-glob`

vite.config.js中配置插件

    import { createSvgIconsPlugin } from 'vite-plugin-svg-icons'
    import path from 'path'
    export default () => {
      return {
        plugins: [
          createSvgIconsPlugin({
            // Specify the icon folder to be cached
            iconDirs: [path.resolve(process.cwd(), 'src/assets/icons')],
            // Specify symbolId format
            symbolId: 'icon-[dir]-[name]',
          }),
        ],
      }
    }

main.ts中配置

    import 'virtual:svg-icons-register'

## 使用

基础使用

```js
<template>
  <div>
    <h1>svg测试</h1>
    <!-- 图表外层的容器，内部需要与use结合使用,通过style设置svg的样式 -->
    <svg style="width: 30px; height: 30px">
      <!-- xlink:href执行用哪一个图表,属性值务必#icon-图标名字  fill可以设置图标的颜色 -->
      <use xlink:href="#icon-vue" fill="yellow"></use>
    </svg>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed, onMounted, watch } from 'vue'
</script>

<style scoped></style>

```

# 虚拟DOM

虚拟DOM就是通过js生成一个抽象语法树AST。比如说ts-->js也需要 es6-->es5的babel-plugin也需要。 babel原理①解析 ②转换，对AST进行操作，babel-generator ③生成

diff算法： 如果没有key，那么新旧Vnode就会进行依次比对，判断是否相同(type和key)，相同就比较下一个，不同就删除，重新新增。没法复用，所以性能很差。 有了key，采用的是从两端到中间的比较策略，首先进行一个前序对比算法，从头部开始依次比对是否是同一节点。如果不同的话就跳出，继续进行尾序比对算法，如果不同，跳出。(vue2还会头尾交叉再比，vue3取消了,vue3使用了最长递增子序列的优化)，最后，如果新节点多了就新增，旧节点多了就删除。

# vue2和vue3区别

①响应式原理不同。vue2采用的是数据劫持和发布订阅者模式实现响应式。首先设置一个Observer观察者，遍历data中的所有属性，然后通过Object.defineProeprty为其添加set和get方法，如果data对象中含有子对象，那么需要递归遍历子对象。

②设置一个compile模板解析器，解析模板中的变量并完成初始化渲染。当变量被调用的时候，就会触发get方法，同时会new一个watcher订阅者实例，然后将指令对应节点的更新函数绑定到watcher订阅者实例的update方法里，再将订阅者实例推到dep中

③dep。存放所有的订阅者，当变量被修改时，就会通过dep.notify通知调度中心，然后调度中心就会寻找到对应的订阅者触发update方法。

vue3采用的是proxy，但对于基本类型的数据，还是通过Object.defineProperty，对于引用类型的数据采用的是proxy。即reactive的原理。记住用Reflect，保证规范上下文完整。

```
export const reactive = <T extends object>(target: T) => {
  return new Proxy(target, {
    get(target, key, receiver) {
      //为了保证上下文正确
      let res = Reflect.get(target, key, receiver)
    },
    set(target, key, newValue, receiver) {
      return Reflect.set(target, key, newValue, receiver)
    }
  })
}

```

```
创建一个副作用函数
let activeEffect
export const effect = (fn: Function) => {
  const _effect = function () {
    activeEffect = _effect
  }
}
//调度中心targetMap
const targetMap = new WeakMap()
//订阅  只不过多加了一个映射，即target 和depsMap  key才是type
export const track = (target, key) => {
  let depsMap = targetMap.get(target)
  //target是传入的对象，depsMap是对应对象的名称和订阅者数组的映射
  if (!depsMap) {
    depsMap = new Map()
    targetMap.set(target, depsMap)
  }
  //找到对应的订阅者数组集合，如果没有的话就创建
  let deps = depsMap.get(key)
  if (!deps) {
    deps = new Set()
    depsMap.set(key, deps)
  }
  //收集副作用函数
  deps.add(activeEffect)
}

//发布
export const trigger = (target, key) => {
  const depsMap = targetMap.get(target)
  //找到订阅者数组
  const deps = depsMap.get(key)
  deps.forEach(item => {
    item()
  })
}

class Dep {
  watchers: Object
  constructor() {
    this.watchers = {}
  }
  on(type, callback) {
    let depMap = this.watchers[type]
    if (!depMap) {
      this.watchers[type] = true
    }
    this.watchers[type].push(callback)
  }
  emit(type, ...args) {
    if (!type) {
      this.watchers[type] = []
    }
    this.watchers[type].forEach(item => {
      item.call(null, ...args)
    })
  }
}

```

## vue3的新特性

①重写双向数据绑定 v-model 用proxy替换object.defineProperty。通过遍历对象中的每个属性，然后通过Object.deinfePropery为对象的属性添加set和get方法，从而实现对象劫持。缺点：对象中新增的属性是没有响应式的，直接操作数组是没有响应式的，只能重写数组方法。

为什么需要reflect：① reflect可以修正proxy中的this指向问题。②reflect是为了规范返回的结果。

②composition api setup语法糖形式

③VDOM的优化。添加了静态标记，假如class类名是静态的，就添加静态标记，如果是变量就添加动态标记

④Tree-shaking的支持。在保持代码的正常运行下，去除掉冗余的代码。

⑤Fragment vue2中代码必须包含在根节点中，vue3支持fragment，可以有多个节点。底层原理其实就是多加了一个虚拟节点。

# ref和reactive

## ref全家桶

**ref创建响应式变量，有两种定义类型的方法**

```
① 直接定义好类型或者是接口
const Man=ref<M>({name:'zds'})

②用Ref官方提供的
import type { Ref } from 'vue'
const Man: Ref<M> = ref({ name: 'zds' })

```

### **isRef用来判断是否是ref变量**

### shallowRef：

**用于浅层响应式，`修改其属性是非响应式的`。**

**只有到.value的时候是响应式的 如果m.value.name这样赋值就不是响应式的了。**

**ref可用于深层响应式。**

`但是可以这么写`

    const Man: Ref<M> = ref({ name: 'zds' })
    const Man2: Ref<M> = shallowRef({ name: 'xqq' })
    function change() {
      Man2.value = { name: 'zds' }  //这是响应式的
      Man2.value.name='zds' //这是非响应式的
    }

### **customRef**

让用户自己实现一个ref响应式 可以用来实现一个简单的防抖

    function myRef<T>(value: T) {
      let timer: any
      return customRef((track, trigger) => {
        return {
          get() {
            console.log('掉用了')
            track()
            return value
          },
          set(newValue) {
            //这里就可以使用防抖
            clearTimeout(timer)
            timer = setTimeout(() => {
              console.log('触发了')
              value = newValue
              trigger()
              timer = null
            }, 1000)
          }
        }
      })
    }
    const obj = myRef<string>('xiaoman')
    function change() {
      obj.value = '小满'
    }

## reactive全家桶

ref支持所有类型，reactive只支持引用类型。ref源码里对引用类型数据添加响应式，其实底层也是调用reactive

**ref取值和赋值都需要＋value 而reactive是不需要的**

### 数组异步赋值问题

这是因为reactive proxy不能直接赋值，不然会破坏响应式对象

![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

解决方案：①纯属组直接用push方法，配合解构

    function add() {
      setTimeout(() => {
        let res = ['edg', 'rng', 'jdg']
        list.push(...res)
      }, 1000)

②包裹一层对象，这样的话只是修改了内部的属性，而不是破坏响应式。

    type List = {
      arr: string[]
    }
    let list = reactive<List>({
      arr: []
    })
    function add() {
      setTimeout(() => {
        let res = ['edg', 'rng', 'jdg']
        list.arr = res
      }, 1000)
    }

### readonly

> 其实就是把每个变量变成只读的

![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

### shallowReactive

只有`浅层的数据`有响应式，即第一层

```
const newObj = reactive({
  a: {
    b: {
      c: 'e'
    }
  }
})
newObj.a.b.c = '132' //没有响应式
newObj.a = { b: { c: 'x' } } //有响应式

```

# 获取组件实例

`import {getCurrentInstance} from 'vue'`

    const instance = getCurrentInstance()
    //instance会返回一个组件实例对象，vnode

# toRef toRefs

**作用：`配合props的解构赋值。`**

在父传子中，一般通过props接受，可以配合{}进行解构，因为一般传递的都是reactive或者是ref响应式变量，如果直接解构会丢失响应式。

## toRef

`对于非响应式的对象，无论咋操作毫无用处，所以只能用于响应式对象。将响应式对象的某个属性提出来，然后通过.value能响应式的修改变量`

    import { ref, reactive, toRef } from 'vue'
    const man = { name: 'zds', age: 20 }
    
    //toRef只能修改响应式对象的值，非响应式毫无变化
    //const man = reactive({ name: 'zds', age: 20 }) 这样即可
    //说白了就是将某个属性取出来变成ref对象
    const age = toRef(man, 'age')
    function change() {
      age.value = 12
      console.log(man) //试图毫无变化，因为man是非响应式的
    }

**使用场景** hook(value) hook接受reactive响应式对象的某个属性，就可以通过toRef将某个属性值提出，单独修改，不需要传入整个对象了。

## toRefs

其实，就是遍历对象中的所有属性，把每个属性都变成ref对象，最后返回。一般会配合解构使用。

    原理：
    const toRefs = <T extends object>(object: T) => {
      const map: any = {}
      for (let key in object) {
        map[key] = toRef(object, key)
      }
      return map
    }

如果不＋toRefs，解构的话，name，age是没有响应式的，但是可以通过man.xxx来修改

![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

## toRaw

将响应式对象转化为普通对象

# computed计算属性

只有computed依赖的值发生改变，它才重新计算，不然使用的是原来缓存的值

`语法: const xx=computed(()=>{ return xx })`

写法②

    computed({
      get() {
        return 'xx'
      },
      set() {}
    })

# watch侦听器

> 语法: watch(\[v1,v2],(newValue,oldValue)=>{},{deep:true})

    watch(message, (newValue, oldValue) => {
      console.log(newValue, oldValue)
    })

## 侦听多个数据

    let message = ref<string>('小满11')
    let message2 = ref<string>('打满')
    watch([message, message2], (newValue, oldValue) => {
      console.log(newValue, oldValue)
    })

返回结果是数组形式

![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

## 侦听对象 监听对象深层次数据需要开启深度监听

`注意：`**监听到的newValue和oldValue是相同的。如果监听的是reactive数据，不需要开启deep:true,因为默认开启了**

    let message2 = ref({
    foo: {
        bar: {
          name: 'zds'
        }
      }})
    watch(
      [message, message2],
      (newValue, oldValue) => {
        console.log(newValue)
      },
      {
        deep: true //开启深度监听
      }
    )

## 只监听对象深层次某一属性 需要用回调函数方式

`oldValue和newValue不同！！`

    watch(
      () => message.foo.bar.name,
      (newValue, oldValue) => {
        console.log(newValue, oldValue)
      }
    )

## options 配置项

**deep:boolean 是否开启深度监听，ref定义的引用类型需要开启，reactive默认帮忙开启**

**immediate:boolean 是否开启立即监听，true就立刻触发一次**

**flush:"pre" 组件更新前调用| "sync" 同步执行 "post" 组件更新后执行**

# watchEffect 高级侦听器

> 语法：watchEffect((before)=>{before()},{flush:'post'})

①`非惰性的`，即会立即执行一次，watch需要手动配置immediate

② 它不需要手动传入依赖对象，会自动收集函数内的数据作为依赖对象，然后当依赖变化后会重新执行回调函数。`回调函数形参里还有个onInvalidate函数，当依赖变化后，它会优先执行它的回调。`

    watchEffect(oninvalidate => {
      console.log(message.value)
      oninvalidate(() => {
        //这个回调被优先执行，比如可以清除副作用  就会优先执行一次before。但初始化的时候不会调用
        console.log('before')
      })
    })
    
    结果：  飞机  //首先会立即执行一次log
    切换后  飞机 //会优先执行oninvalidate 输出 before  飞机

③ 可以停止监听 const stop=stopWatch()

    const stop = watchEffect(oninvalidate => {
      console.log(message.value)
      oninvalidate(() => {
        //这个回调被优先执行，比如可以清除副作用  就会优先执行一次before。但初始化的时候不会调用
        console.log('before')
      })
    })
    function stopWatch() {
      stop()
    }

④一般 flush再watchEffect中使用，因为它flush:'post'，可以在页面渲染结束后调用，可以获取到最新的DOM元素

![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

# vue3生命周期

setup语法糖中：没有beforeCreate和created生命周期，但可以用setup来代替，且setup只会执行一次。setup优先于beforeCreated,created

setup onBeforeMount onMounted onBeforeUpdate onUpdated onBeforeUnmount onUnmounted

| 生命周期            |
| ------------------- |
| 父组件setup         |
| 父组件onBeforeMount |
| 子组件setup         |
| 子组件onBeforeMount |
| 子组件onMounted     |
| 父组件onMounted     |

# 组件通信

## 父传子：

**通过v-bind 传递，子组件通过defineProps接受**

①无ts模式

`deinfeProps(['xx1','xx2'])`

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12dfd2f291d14864b212d9be60fcf630~tplv-k3u1fbpfcp-watermark.image?)

`②ts模式`

`携带默认值 withDefault(defineProps<Iprops>(),{属性:默认值})`

注意：复杂类型需要用函数形式

    withDefaults(defineProps<Iprops>(), {
      score: 0,
      data: () => ({ name: "dzd", age: 12 }),
    });

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de130bae704a4d369e500c624f5cbd3e~tplv-k3u1fbpfcp-watermark.image?)

## 子传父：

 **通过事件触发，然后父组件中定义自定义事件接受**

    ----简单写法-----
    const emit = defineEmits(['onCCClick'])
    function send() {
      emit('onCCClick', '子传父')
    }



## defineExpose

    //获取组件实例，定义的ref变量必须和ref的值同名
    <Son ref='sonRef'>
    const sonRef=ref()

`但是，组件内部数据对外是关闭的，如果想暴露给外部访问，需要通过defineExpose方法`

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9f9faaa83c7442382cd22ad26858e51~tplv-k3u1fbpfcp-watermark.image?) 

***

父组件通过定义ref变量接受 获取子组件实例，然后通过.value获取值。注意只有在onMounted中可以获取到子组件实例，数据是响应式的。

    //ts含类型写法
     <water-fall ref="waterFallRef" />
     import waterFall from './components/water-fall.vue'
      const waterFallRef = ref<InstanceType<typeof waterFall>>()
      
       onMounted(() => {
      console.log(waterFallRef.value?.list)
    })

## v-model

(可以实现父子组件数据同步,其实说白了还是通过props传递值，通过自定义事件修改父组件的值，它帮你直接定义好了一个update:modelValue的自定义事件)



![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46c8a84e13fe49409eadadbd5fcbab17~tplv-k3u1fbpfcp-watermark.image?)

    ①：相当于给子组件传递props[modelValue]=1000
    ②：同时给子组件绑定了一个@update=‘updateModelValue’，用来更新modelValue,
    所以只需要$emit('update:modelValue',原值改变)即可

写法二：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc29dff178ff4a14a331a4a772cdb9dd~tplv-k3u1fbpfcp-watermark.image?)

这种写法写，子组件\$emits('update:pageNo',props.pageNo+1000)即可

## useAttrs

`可以用来接受父组件传递的属性和属性值。但是如果props接受后，$attrs就接受不到了`

    import { ref, reactive, useAttrs } from "vue";
    let $attrs = useAttrs();
    //可以获取到props的数据 比如props.value=1   $attrs.value就获取不到了。

## 兄弟组件通信

①状态提升的方式，A组件想要将数据传递给B组件，可以先通过自定义事件将数据给父组件，然后由父组件分发给B组件
②使用mitt 本质上就是一个发布订阅。

## 模拟全局事件总线

vue3.0中，取消了vue的原型对象，改用了createApp()的方式创造组件实例，因此没有了this。如果想要实现类似功能，需要使用mitt插件

`npm install mitt`

[mitt使用文档](https://www.npmjs.com/package/mitt)

## pinia

全局状态管理，缺点在于无法持久性保存。
[pinia使用文档](https://pinia.web3doc.top/core-concepts/)

# 全局组件 局部组件

> 语法：app.component('组件名',Com)

局部组件import导入，就是不需要额外注册了 全局组件在main.ts中导入，注意：需要通过链式调用。因为每次都会返回app

# 递归组件，常用于树型结构。通过v-if来结束递归

注意：递归调用时直接使用组件名即可，点击时要注意停止冒泡动作

```
<template>
  <div @click.stop="add" v-for="item in data" :key="item.name" class="tree">
    <input type="checkbox" v-model="item.checked" /><span>{{ item.name }}</span>
    <Tree v-if="item?.children?.length" :data="item.children" />
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from "vue";
interface Tree {
  name: string;
  checked: boolean;
  children?: Tree[];
}
interface Iprop {
  data: Tree[];
}
const props = defineProps<Iprop>();
</script>

<style scoped>
.tree {
  margin-left: 10px;
}
</style>

```

效果：

![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

# 动态组件

动态组件：让多个组件使用同一个挂载点，并进行动态切换。常用于tab切换

`语法：<component :is='组件'>`

**注意：①vue2的时候是靠组件名称切换的，vue3是靠组件实例进行切换的，所以直接导入对应组件**

**②如果把组件实例直接放到reactive对象中，那么会报警告**。这是因为reactive会对传入的对象进行代理，但是组件代理毫无用处，所以可以使用markRaw跳过代理

![image-20230408135311825转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

    <template>
      <div>
        <button @click="change(item)" v-for="item in data" :key="item.name">
          {{ item.name }}
        </button>
        <component :is="isCom"></component>
      </div>
    </template>
    
    <script setup lang="ts">
    import { ref, reactive, computed,markRaw } from "vue";
    import A from "./components/A.vue";
    import B from "./components/B.vue";
    const isCom = reactive(markRaw(A));
    const data = reactive([
      {
        name: "A",
        com: markRaw(A), //这样不会有警告，会跳过proxy
      },
      {
        name: "B",
        com: B,//这样会有警告
      },
    ]);
    function change(item: { name: string; com: any }) {
      console.log(item.com);
      isCom.value = item.com;
    }
    </script>

# 插槽

**插槽其实就是子组件中提供给父组件的一个占位符，父组件可以往里面填充任何内容，替代** slot /slot **的位置**

匿名插槽和具名插槽的语法:

     <Dialog>
          <template v-slot:header>我被插入到head中</template>
          <template v-slot> 我被插入到main中 </template>
          <template #footer> 我被插入到footer中</template>
        </Dialog>
        
     <header class="header">
          <slot name="header" />
        </header>
        <main class="main">
          <slot></slot>
        </main>
        <footer class="footer">
          <slot name="footer" />
        </footer>   

作用域插槽，实现子传父

通过#slot='变量名' 父组件就可以获取到对应的值 ![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

动态插槽

![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

# 异步组件 代码分包 Suspense

(suspense和telepot一样是vue3新增的内置组件) suspense可以用来配合异步组件做骨架屏

## await

在es7 await可以直接在模块的最外层使用，这让整个模块就变成一个巨大的async函数。但也可以通过async包裹一个立即执行函数

## Suspense

需要配合异步组件来使用，有两个插槽，default是展示异步加载的组件，fallback用来展示骨架屏

    <template>
      <div>
        <Suspense>
          <template #default>
            <Async />
          </template>
          <template #fallback> 加载的骨架屏 </template>
        </Suspense>
      </div>
    </template>
    
    <script setup lang="ts">
    import { ref, reactive, computed, defineAsyncComponent } from "vue";
    const Async = defineAsyncComponent(() => import("./components/Async/index.vue"));
    </script>
    
    <style scoped></style>

## 优化打包

如果是直接import 同步导入组件的方式，那么打包出来的js文件会很大，即首次加载的白屏时间也很长。

如果用异步组件的方式，打包时会把没有用到的异步组件先拆出来，暂时不会打包到主包中，只有等用到该组件的时候才会加载，放到主包里。和路由懒加载差不多，都是用来优化首屏加载速度过慢的问题

# Teleport传送组件 类似于react的portal

它和Suspense一样，也是vue的内置组件。 作用：能够将定义的模板挂载到指定的DOM节点，不受父级style，v-show等属性影响，但data，prop数据依旧能够共用。

使用方法：

    <Teleport to="body">
    
        <Loading></Loading>
    
    </Teleport>

# keep-alive缓存组件

使用场景：不希望组件被重新渲染，或者处于性能考虑，避免多次重复渲染导致性能损耗。 当组件被keep-alive包裹的时候，那么初次进入onMounted-->onActived

退出后：deactived 再次进入后只会触发onActived

include:需要缓存的组件 exclude:不缓存的组件

# 自定义指令

`需要注意的是，必须以vName开头，这样才能使用` ![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

**vue2.0中还是通过bind insered实现，vue3.0中改成了类似生命周期**

## 局部自定义指令

import {Directive} from 'vue'

     <A v-move:aaa.xiaoman="{ background: 'red' }" />
     对象形式写法：即类似于生命周期函数
    const vMove: Directive = {
      created() {
        console.log('元素初始化调用')
      },
      beforeMount() {
        console.log('指令绑定到元素后调用，只调用一次')
      },
       mounted(el: HTMLElement, dir: DirectiveBinding) {
        console.log(el)
        console.log(dir)
        console.log('元素插入父级DOM调用')
        //绑定的值在value里
        el.style.background = dir.value.background
      },
      beforeUnmount() {
        console.log('元素被移除后调用')
      },
      updated(){},//自定义传入的值发生改变就触发
      unmounted() {
        console.log('指令被移除后调用')
      }
    }

> 常用的就3个 mounted(){} ,updated(){}只要自定义传入的值发生改变就会触发，还有个卸载时beforeUnmounted

**函数形式写法，相对来说更简单**

     函数形式简写 vMove:Directive=(el,dir)=>{}

## 案例 封装权限按钮组件是否展示的自定义指令

    <template>
      <div>
        <button v-has-show="'shop:create'">创建</button>
    
        <button v-has-show="'shop:edit'">编辑</button>
    
        <button v-has-show="'shop:delete'">删除</button>
      </div>
    </template>
    
    <script setup lang="ts">
    import { ref, reactive, computed, Directive } from 'vue'
    //自定义指令实现不同权限下，按钮的隐藏
    
    localStorage.setItem('userId', 'zds')
    /* mock后端发回的权限数据 permission数组 */
    const permission = ['zds:shop:edit', 'zds:shop:create', 'zds:shop:delete']
    const userId = localStorage.getItem('userId') as string
    const vHasShow: Directive = (el: HTMLElement, dir) => {
      if (!permission.includes(userId + ':' + dir.value)) {
        el.style.display = 'none'
      }
    }
    </script>
    
    <style scoped></style>

# 自定义hooks

**本质上就是一个函数，暴露一个函数给组件进行使用。可以直接导入生命周期，在函数里使用。类似于setup函数**

> vue2.0中代码复用的方式主要是通过mixins混入，这样可以直接使用混入里面的变量和方法，但这会让代码逻辑看起来很混乱，变量来源不明确。

> vue3官方常用的hooks import {useAttrs} from 'vue'。 这样就可以再组件中直接调用let attrs=useAttrs() 就可以获取到该组件拥有的属性值

**案例：转化为base64**

    import { onMounted } from 'vue'
    type Options = {
      el: string
    }
    export default function (options: Options): Promise<{ baseURL: string }> {
      //return new Promise其实主要是为了让外界可以通过.then传递数据
      return new Promise(resolve => {
        onMounted(() => {
          console.log('自定义hook里的mounted')
          let img: HTMLImageElement = document.querySelector(options.el) as HTMLImageElement
          //需要等图片加载完毕后再转换
          img.onload = () => {
            resolve({ baseURL: base64(img) })
          }
          const base64 = (el: HTMLImageElement) => {
            const canvas = document.createElement('canvas')
            const ctx = canvas.getContext('2d')
            canvas.width = el.width
            canvas.height = el.height
            //目标元素  开始x轴位置  y轴位置 宽度 高度
            ctx?.drawImage(el, 0, 0, canvas.width, canvas.height)
            //基于img绘制canvas，然后通过toDataURL返回base64
            return canvas.toDataURL('image/png')
          }
        })
      })
    }

# 全局函数和变量

> vue2.0中全局变量是直接挂载在vue的原型上。类似vue.prototype.\$http=axios

> vue3.0中使用app.config.globalProperties代替 然后定义变量和函数

![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

**使用** 可以在任意位置直接使用\$env

![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

`注意：这里会报警告，需要声明文件`

```

type Filter = {
    format<T>(str: T): string
}
 
// 声明要扩充@vue/runtime-core包的声明.
// 这里扩充"ComponentCustomProperties"接口, 因为他是vue3中实例的属性的类型.
declare module 'vue' {
    export interface ComponentCustomProperties {
        $filters: Filter
    }
}

在setup js中获取其值
1.  import {ref,reactive,getCurrentInstance} from 'vue'
1.  const app = getCurrentInstance()
1.  console.log(app?.proxy?.$filters.format('js'))
```

# vue自定义插件

**插件是自包含的代码，通常向 Vue 添加全局级功能。你如果是一个对象需要有install方法Vue会帮你自动注入到install 方法 你如果是function 就直接当install 方法去使用**

> 有两种形式函数和对象，常用函数，必须实现install方法，install的参数会传递一个app根组件。注意导入的组件需要通过creatVNode转化为虚拟节点，再挂载

    //定义插件有两种形式，一种对象，一种函数
    /* 
    对象形式必须调用install 函数，它会回传一个app 即全局的那个app
    */
    import type { App, VNode } from 'vue'
    import { createVNode, render } from 'vue'
    import Loading from './index.vue'
    export default {
      install(app: App) {
        //将loading组件转化为vNode
        const Vnode: VNode = createVNode(Loading)
        render(Vnode, document.body)
        //将vNode暴露出来的属性和方法直接挂载到全局的app里
        app.config.globalProperties.$loading = {
          show: Vnode.component?.exposed?.show,
          hide: Vnode.component?.exposed?.hide
        }
      }
    }
    
    调用
    /* 
    获取当前组件的实例
    */
    const instance = getCurrentInstance()
    instance?.proxy?.$loading.show()

**案例：自定义一个myUse 传入不同插件调用**

> 原理其实很简单，需要传入plugin，且plugin必须要有install方法，然后进行去重，最后调用plugin的install方法即可。

    import type { App } from 'vue'
    interface Use {
      install: (app: App, ...options: any[]) => void
    }
    //传入的插件必须要有install方法，所以进行泛型约束
    import { app } from '../../main'
    const installList = new Set()
    export function myUse<T extends Use>(plugin: T, ...options: any[]) {
      if (installList.has(plugin)) {
        console.error('不好意思，已经注册', plugin)
      }
      plugin.install(app, options)
      installList.add(plugin)
    }

# Vue transition 动画组件 实现过渡动画

> 语法: 自定义 transition 过度效果，你需要对`transition`组件的`name`属性自定义。并在css中写入对应的样式

    <transition name='fade' > </transition>
    v-enter-from：定义进入过渡的开始状态。在元素被插入之前生效，在元素被插入之后的下一帧移除。
    
    v-enter-active：定义进入过渡生效时的状态。在整个进入过渡的阶段中应用，在元素被插入之前生效，在过渡/动画完成之后移除。这个类可以被用来定义进入过渡的过程时间，延迟和曲线函数。
    
    v-enter-to：定义进入过渡的结束状态。在元素被插入之后下一帧生效 (与此同时 v-enter-from 被移除)，在过渡/动画完成之后移除。
    
    v-leave-from：定义离开过渡的开始状态。在离开过渡被触发时立刻生效，下一帧被移除。
    
    v-leave-active：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡/动画完成之后移除。这个类可以被用来定义离开过渡的过程时间，延迟和曲线函数。
    
    v-leave-to：离开过渡的结束状态。在离开过渡被触发之后下一帧生效 (与此同时 v-leave-from 被移除)，在过渡/动画完成之后移除。

**transition生命周期有八个**

      @before-enter="beforeEnter" //对应enter-from
      @enter="enter"//对应enter-active
      @after-enter="afterEnter"//对应enter-to
      @enter-cancelled="enterCancelled"//显示过度打断
      @before-leave="beforeLeave"//对应leave-from
      @leave="leave"//对应enter-active
      @after-leave="afterLeave"//对应leave-to
      @leave-cancelled="leaveCancelled"//离开过度打断

当只用 JavaScript 过渡的时候，在 **`enter` 和 `leave` 钩子中必须使用 `done` 进行回调**

# css3完整新特性

## 插槽选择器。想要修改插槽内容



## 动态CSS

**可以通过v-bind( 变量)添加动态css**

    <template>
      <div>
        <button class="div" @click="change">鼎泰css</button>
      </div>
    </template>
    
    <script setup lang="ts">
    import { ref, reactive, computed } from 'vue'
    const style = ref<string>('red')
    const change = () => {
      myStyle.color = 'yellow'
    }
    //也可以用对象形式，但是需要加""
    const myStyle = reactive({
      color: 'red'
    })
    </script>
    
    <style scoped>
    .div {
      color: v-bind('myStyle.color');
    }
    </style>

## cssmodule

这种写法主要是为了写tsx，类似于react的时候使用



    <template>
      <div :class="zds.div">
        123
        <div :class="[zds.div, zds.bodder]">123</div>
      </div>
    </template>
    
    <script setup lang="ts">
    import { ref, reactive, computed } from 'vue'
    </script>
    
    <style module="zds">
    div {
      color: red;
    }
    bodder {
      border: 1px solid red;
    }
    </style>

# vue环境变量

> 作用：让开发者区分不同的运行环境，来实现兼容开发和生产 npm run dev:开发环境 npm run build:生产环境

**步骤** ①创建.env.development文件。自定义名称



# **webpack相关**

##  	异步组件代码分包

> 默认的打包情况下，在构建整个组件树的过程种，组件和组件是`通过模块化直接导入`的(import  xx from '../xx.vue'),那么 `webpack在打包时就会将组件模块打包到一个js文件`（如app.js）

如果打包时，**代码分包（比如定义异步组件，vue3通过defineAsyncComponent(()=>import('../xx.vue'))）或者直接通过import()函数导入(它会返回一个promise)**都会进行 `分包`，会被单独打包到一个js文件中

所以，对于一些`首页加载时不需要立即使用的组件`，可以单独对它们进行拆分，拆分成一些小的代码块chunk.js。

这些chunk.js会在`被需要使用时`，才`会进行加载运行`



# 跨域

跨域是因为浏览器的同源策略引起的，浏览器的同源策略限制了非同源的url向另一个url发起ajax请求，访问其DOM元素等等。

解决跨域的方式：

**①：JSON跨域没因为script标签的src不会受到浏览器的同源策略的限制，将src替换成服务器的url同时传去一个callback函数名称，服务器返回对应的函数调用结果**

![image.png转存失败，建议直接上传图片文件](<转存失败，建议直接上传图片文件 >)

**②：cors跨域资源共享** 一般是后端设置，Access-Control-Allow-Origin:origin

**③poxy 反向代理**

    server: {
    
        proxy: {
        //它会截取/api之前的地址，替换成target
    
          '/api': {
    
            target: 'http://localhost:9999'
          }
        }
      }

# vue-router

## 配置

**注意：需要导入RouteRecordRaw这个类型，它必须包含path和component还有name**

    import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router'
    
    const routes: Array<RouteRecordRaw> = [{ path: '/', component: () => import('./components/Login.vue') }]
    const router = createRouter({
      history: createWebHistory(),
      routes
    })
    
    export default router

## 任意路径匹配404
**解决history路由404问题**

```
//其实就是参数配合正则
  {
    path: '/:pathMatch(.*)*',
    redirect: '/404',
    name: 'any',
  },
```


## 路由跳转

### router-link

**必须含有name属性，不然会报错，同时to改为对象形式了**



### 编程式路由导航

    import { useRouter } from 'vue-router'
    const router = useRouter()
    router.push({ path: '/home' })

## 路由传参

**①query参数** 跳转时，直接携带即可

    router.push({path:'/home',query:{}})

**②params参数**

注意：params参数不能用path了，因为path里携带了变量，需要用name

     router.push({ name: 'Home', params: { id: 'zds' } })