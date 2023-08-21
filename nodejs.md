# Node.js
[掘金笔记链接](https://juejin.cn/flash-note/list?note_id=7214005409902755895&from=6)
`nodejs是什么`

nodejs是一个支持js运行的一个运行环境



## nodejs注意点

nodejs中是不能使用DOM和BOM的api的。

但是**console**和**定时器**是可以使用的。



## 顶级对象 global 

nodejs中的顶级对象是**global**，类似于`window`对象，但是nodejs环境下是没有window的





# Buffer

buffer是缓冲区，类似于Array的对象。换句话说，它是一段固定长度的内存空间，用来处理二进制数据。



# 进程与线程

进程：是资源分配的最小单位。运行中的程序可以理解为进程，可以通过任务管理器查看进程。

线程：是cpu调度的最小单位，也是实际的工作单位。进程是由一个或多个线程组成的。进程中的任意一个线程崩溃都会导致整个进程的崩溃。进程中的所有线程共享进程中的数据。进程之间相互独立互不干扰。



# fs模块  



## 文件写入

| 方法             | 说明                             |
| ---------------- | -------------------------------- |
| writeFile        | 异步写入                         |
| writeFileSync    | 同步写入                         |
| createWriteSteam | 流式写入(建立连接，进行连续写入) |



###  writeFile 异步写入 (宏任务)

语法：`fs.writeFile(file,data,options,callback)`

- file 文件(要写入的文件路径，如果不存在会创建)
- data待写入的数据
- options配置选项
- callback写入的回调   //写入完成后会触发  写入成功参数是null，失败是个err对象

```javascript
/* 
创建一个文件，写入内容
*/
/* 导入fs模块 */
const fs = require('fs')

/* 
写入文件
*/
fs.writeFile('./座右铭.txt', '三人行', {}, err => {
  if (!err) {
    console.log('写入成功')
  } else {
    console.log('写入失败')
  }
})
console.log(1)   //输出 1 
                 //   写入成功
```



### writeFileSync 同步写入

**它会先进行写入文件，然后再输出‘12’**

```js
const fs = require('fs')

const res=fs.writeFileSync('./座右铭.txt', '帅气的同步任务', {})
//返回值res是undefined
console.log('12')

```



### appendFile/appendFileSync 追加写入

**appendFile作用是再文件尾部追加内容，语法与writeFile完全相同**

`fs.appendFile('文件路径','追加内容',options,callback)`



### createWriteStream 流式写入

语法: `const ws = fs.createWriteStream(path,options)`

- path 文件路径
- options 选项配置(可选)

**适用于频繁写入的情景，writeFile是一次性写入的。**

```js
const fs = require('fs')

//创建写入流对象，建立连接
const ws = fs.createWriteStream('./座右铭.txt')

//通过write进行写入
ws.write('帅气的郑大顺\r\n')
ws.write('很无语呀\r\n')

//关闭连接  (它会自动关闭的所以写不写都行)
ws.close()

```





## 文件读取

| 方法               | 说明                             |                           |
| ------------------ | -------------------------------- | ------------------------- |
| readFile(err,data) | 异步读取                         |                           |
| readFileSync       | 同步读取                         |                           |
| createReadSteam    | 流式读取(建立连接，进行连续读取) | rs.on('data',(chunk)=>{}) |



### readFile 异步读取

语法:`fs.readFile(path,options,callback)`

- path 文件路径
- options配置选项
- callback回调函数   (错误信息，读取到的内容)=>{}

返回值：`undefined`

```js
const fs = require('fs')

//1.异步读取
fs.readFile('./座右铭.txt', {}, (err, data) => {
    //data是buffer序列，需要通过toString转化
  console.log(data.toString())
})

```



### readFileSync 同步读取

语法： `let data=readFileSync('路径')`

**它只需要传入路径即可，返回值是读取到的buffer序列**

```js
let data = fs.readFileSync('./座右铭.txt')
console.log(data)
```



### createReadSteam 流式读取

**建立读取流对象，通过on绑定data事件，每次读取64KB的数据**

```js
//创建读取流对象，建立连接
const rs = fs.createReadStream('./座右铭.txt')

//绑定data事件，需要用on方法
rs.on('data', chunk => {
  //从文件中读取出来一块文件后，就会执行  每次会读取64KB的数据
  console.log(chunk.length)
})
//end是可选事件 结束读取时触发
rs.on('end',()=>{})
```





## 案例：复制文件

```js
方式①：
//复制座右铭.txt
const fs = require('fs')
//读取文件内容
const data = fs.readFileSync('./座右铭.txt')
//重新写入一个新文件
const res=fs.writeFileSync('./复制文件.js', data, {})
console.log(res)
console.log('12')

----
方式②：流式读取和写入
const fs = require('fs')
//创建读取流对象
const rs = fs.createReadStream('./座右铭.txt')
//创建写入流对象
const ws = fs.createWriteStream('./复制文件.txt')
//为读取流绑定data事件,读一部分就写一部分
rs.on('data', chunk => {
  ws.write(chunk)
})


```



## 文件重命名和移动

语法: `fs.rename(oldPath,newPath,callback)`

​		  `fs.renameSync(oldPath,newPath)`



## 文件删除

语法： `fs.unlink(path,callback)`

​			 `fs.unlinkSync(path)`

```js
const fs = require('fs')

fs.unlink('./复制文件.txt', err => {
  if (!err) {
    console.log('删除成功')
  }
})

```



第二种方式: `fs.rm(path,callback)`

​					 `fs.rmSync(path)`

```js
/* 调用rm方法 */  node 14版本新增的
fs.rm('./a.txt', err => {
  if (!err) {
    console.log('删除成功')
  }
})

```



## 文件夹操作

| 操作       | 异步                               | 同步                    |
| ---------- | ---------------------------------- | ----------------------- |
| 创建文件夹 | mkdir(path,options,callback)       | mkdirSync(path,options) |
| 读取文件夹 | readdir(path,(err，data)=>{})      | readdirSync(path)       |
| 删除文件夹 | rmdir(path,callback)//需要目录为空 | rmdirSync(path)         |

### 创建文件夹

```js
const fs = require('fs')

fs.mkdir('./文件夹', {}, err => {
  if (!err) console.log('文件夹创建成功')
})

```

#### 递归创建文件夹

**需要配置 `{recursive:true}`**

```js
const fs = require('fs')

//递归创建多个文件夹，需要配置项{recursive:true}
fs.mkdir('./a/b/c', { recursive: true }, err => {
  if (err) {
    console.log('失败')
  }
})

```

### 

### 读取文件夹

```js
const fs = require('fs')

fs.readdir('../fs模块', (err, data) => {
  if (err) return
  console.log(data)
})

```

![image-20230324154405025](C:\Users\szdrz\AppData\Roaming\Typora\typora-user-images\image-20230324154405025.png)





### 删除文件夹  

**必须目录为空才能删除**

语法：`fs.rmdir('./',(err)=>)`







## 路径补充说明



**相对路径的bug:**

如果不在对应的目录下，node  .\代码\08路径补充.js，那么相对路径就会当命令行的当前目录下创建。(即工作目录)

```js
const fs = require('fs')
//通过相对路径写入文件
fs.writeFileSync('./相对路径.txt', '相对路径', {}, () => {})
```



**__dirname (可以理解为一个''全局变量'')**

`它代表所在文件所在路径的绝对路径`

```js
const fs = require('fs')

//通过绝对路径  __dirname表示当前文件所在的绝对路径
fs.writeFileSync(__dirname + '/相对路径.txt', '相对路径', {}, () => {})

```

![image-20230324155417314](https://gitee.com/zhengdashun/pic_bed/raw/master/img/image-20230324155417314.png)



# path模块



**path模块提供了 `操作路径的功能`**

| API           | 说明                          |
| ------------- | ----------------------------- |
| path.resolve  | 拼接规范的绝对路径 `最常使用` |
| path.sep      | 获取操作系统的路径分隔符      |
| path.parse    | 解析路径并返回对象            |
| path.basename | 获取路径的基础名称            |
| path.dirname  | 获取路径的目录名              |
| path.extname  | 获取路径的扩展名              |



> 之前通过绝对路径创建文件的时候，那个绝对路径是不规范的，需要通过path.resolve进行拼接

```js
const fs = require('fs')
const path = require('path')

//resolve  拼接绝对路径
fs.writeFileSync(path.resolve(__dirname, './index.js'), '绝对路径', {}, () => {})

```



# Http

## 创建一个可接受http请求的服务器(最基础的)

```
//1、导入Http模块
const http = require('http')

//2.创建服务对象
//request:封装的请求报文对象
//response:封装的响应报文对象
const server = http.createServer((request, response) => {
  //设置响应体
  response.setHeader('content-type', 'text/html;charset=utf-8')
  //设置响应体
  response.end('你好')
})

//监听9000端口，启动服务器
server.listen(9000, () => {
  console.log('服务已经启动')
})

```



## 提取Http请求报文

想要获取请求报文信息，需要通过request这个封装好的请求报文对象

| 含义          | 语法                                                         |                                              |
| ------------- | ------------------------------------------------------------ | -------------------------------------------- |
| `请求方法`    | `request.method`                                             |                                              |
| 请求版本      | request.httpVersion                                          |                                              |
| `请求路径`    | `request.url`                                                | 只包含路径和查询参数，不包含协议、域名、端口 |
| URL路径       | require('url').parse(request.url).pathname                   | 获取完整的url                                |
| URL查询字符串 | require('url').parse(request.url,true).query                 | 获取查询参数                                 |
| `请求头`      | `request.headers`                                            | 浏览器中开头是大写，它会变成小写             |
| `请求体`      | request.on('data',function(chunk){})<br/>request.on('end',function(){}) | 请求体是一个流式对象，需要通过on事件读取     |

**注意事项**

`request.url`只能获取路径和查询字符串，无法获取URL中的内容。

提取Http报文路径和查询字符串：

```js
const url = require('url')
const server = http.createServer((request, response) => {
  let res = url.parse(request.url,true) 
  response.end('11')
})
```

request.headers将请求信息转化成一个对象，并且属性名都[**小写**]



```js
//1、导入Http模块
const http = require('http')

//2.创建服务对象
//request:封装的请求报文对象
//response:封装的响应报文对象
const server = http.createServer((request, response) => {
  //获取请求方法
  console.log(request.method)
  //获取请求的url。request.url只包含路径和查询字符串，不包含协议，域名(主机)和端口
  console.log(request.url)
  //获取请求头
  console.log(request.headers)
  //设置响应体
  response.setHeader('content-type', 'text/html;charset=utf-8')
  //设置响应体
  response.end('你好')
    

  //提取请求体
  let body = ''
  //request是一个可读流对象 createReadStream，可以每次取出一部分
  request.on('data', chunk => {
    body += chunk
  })
  request.on('end', () => {
    //可读流数据读取完毕
    console.log(body)
    response.end('hello http')
  })
})

//监听9000端口，启动服务器
server.listen(9000, () => {
  console.log('服务已经启动')
})

```





## 提取Http响应报文

| 作用             | 语法                                   |
| ---------------- | -------------------------------------- |
| 设置响应状态码   | response.statusCode                    |
| 设置相应状态描述 | response.statusMessage                 |
| 设置响应头       | response.setHeader('key','value')      |
| 设置响应头       | response.write()  <br/> response.end() |



**练习：服务器返回一个html文档给客户端**

```js
const http = require('http')
const fs = require('fs')
const server = http.createServer((request, response) => {
  //读取文件内容 ,读出来的是buffer。response.end()可以发送buffer
  fs.readFile('./index.html', {}, data => {
    response.end(data)
  })
})

server.listen(9000, () => {
  console.log('服务启动')
})

```

## 网页资源



### 静态资源与动态资源

**静态资源：内容长时间不发生改变(运行阶段不改变)，例如图片、css、js**

**动态资源：内容经常发生改变，例如首页的html文档**



### 搭建静态资源服务



> 服务器返回index.html ,html文档通过link导入css，script导入js脚本。

![image-20230327161012519](https://gitee.com/zhengdashun/pic_bed/raw/master/img/image-20230327161012519.png)

请求是发起了，但是你服务器只根据index.html这个路径返回html文档。![image-20230327161102222](C:\Users\szdrz\AppData\Roaming\Typora\typora-user-images\image-20230327161102222.png)

对于这种请求没有处理呀。要么通过if判断，每次pathname不同就返回。

缺点：每次新增一个js文件都要进行一次if判断，太繁琐

![image-20230327161130567](https://gitee.com/zhengdashun/pic_bed/raw/master/img/image-20230327161130567.png)



解决措施：文件路径和请求的pathname有关联

```js
const http = require('http')
const fs = require('fs')
const path = require('path')
const server = http.createServer((request, response) => {
  let url = new URL(request.url, 'http://127.0.0.1')
  let { pathname } = url
  //找到当前目录的pages文件夹 同时拼接传递来的pathname，从而找到对应的静态文件
  let filePath = path.resolve(__dirname, '/page' + pathname)
  //读取对应的静态文件，并返回
  fs.readFileSync(filePath, (err, data) => {
    if (err) {
      response.end('读取静态文件失败')
    } else {
      response.end(data)
    }
  })
})
server.listen(9000, () => {
  console.log('服务器启动')
})

```



## 网页中的URL

> 网页也区分绝对路径和相对路径

**绝对路径**

| 形式                   | 特点                                                        |                                                              |
| ---------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| http://atguigu.com/web | 直接向目标资源发送请求                                      | (如a标签href='https://xxx')                                  |
| //atguigu.com/web      | 与当前`页面URL`的协议拼接形成完整URL再发送请求              | 比如当前页面http://127xxx。//jd.com。就会变成http://jd.com。一般会配合重定向 |
| /web                   | 与当前网页的**协议、主机名、端口**拼接形成完整URL再发送请求 | ![image-20230330150323193](https://gitee.com/zhengdashun/pic_bed/raw/master/img/image-20230330150323193.png)<br/>会直接与当前网页的`协议、主机名、端口`进行拼接。![image-20230330150427466](C:\Users\szdrz\AppData\Roaming\Typora\typora-user-images\image-20230330150427466.png)<br/>![image-20230330150458221](https://gitee.com/zhengdashun/pic_bed/raw/master/img/image-20230330150458221.png)注意：页面路径不会被拼接上去 |

又因为设置了静态资源目录，所以就会直接找到对应资源

```js
app.use(express.static(path.join(__dirname, 'public')))
```



**相对路径**：相对路径在发送请求时，需要与当前页面URL路径进行 **计算**，得到完整URL后，再发送请求

例如当前url:http://www.atguigu.com/course/h5.html

| 形式          | 注意是文件夹            | url                                       |
| ------------- | ----------------------- | ----------------------------------------- |
| ./css/app.css | 当前`文件夹`下的资源    | http://www.atguigu.com/course/css/app.css |
| js/app.js     | 当前`文件夹`下的资源    | http://www.atguigu.com/course/js/app.js   |
| ../image/a.js | 上一个`文件夹`下的image | http://www.atguigu.com/image/a.js         |





# express 

express是封装好的`轻量的web服务器框架`，有了express不需要再使用http模块自己创建服务器并处理路径了。

## 搭建express服务

```js
const express = require('express')

//创建应用对象
const app = express()

app.get('/:id.html', (req, res) => {
  res.send('hello express')
})

app.listen(9000, () => {})

```



## 路由

路由分为了前端路由和后端路由。

前端路由：路径对应了不同的组件，无需发送请求给服务器，也不会导致页面的刷新。

> 这也就是为什么history路由，刷新页面会导致404的问题。因为用户刷新页面，会发起一个get请求，而且在history路由下，路径会被携带着发送给服务端。而这个路径只是前端用来映射组件的，并不是后端路由，（后端路由是服务端**根据不同的路径和请求方法**做出不同的处理，返回对应资源），因此对该路径不会返回资源，自然报404。而hash路由hash值并不会随着http请求发送给服务器，所以不会出现这个问题。

后端路由：服务端根据不同的路径和请求方法，返回对应的资源



## 获取报文参数



### 请求报文参数

| 方法                                     | 作用                                                       |
| ---------------------------------------- | ---------------------------------------------------------- |
| req.method                               | 获取请求方法                                               |
| req.url                                  | 只包含页面路径，不包含协议主机端口                         |
| req.headers                              | 获取请求头                                                 |
| `req.path` 等同于req.url                 | 获取页面路径 即pathname  /login                            |
| `req.query`                              | 获取query参数                                              |
| `req.ip`                                 | 获取IP                                                     |
| req.get('referer')                       | 获取refer: refer指明了该请求来自于哪个源即协议、域名和端口 |
| let url=new URL(referer)<br>url.hostname | 获取域名。通过new URL()传入refer                           |



### 获取请求体

`npm i body-parser`

`const bodyParser=require("body-parser")`

[body-parser文档](https://www.npmjs.com/package/body-parser)

```js
路由中间件方式：
const express = require('express')
const app = express()
const path = require('path')
const bodyParser = require('body-parser')

// create application/json parser
//解析JSON类型的请求体
var jsonParser = bodyParser.json()

// create application/x-www-form-urlencoded parser
/* 解析queryString格式请求体的中间件 */
var urlencodedParser = bodyParser.urlencoded({ extended: false })

//设置静态资源中间件
app.post('/login', urlencodedParser, (req, res) => {
  //直接返回html内容。
  //当中间件执行完毕后，就会往req中添加一个body属性
  console.log(req.body)
  res.sendFile(__dirname + '/pages/index.html')
})

app.listen(9000, () => {})
```



### 配置params，动态路由参数

通配ID参数。 

http://locahost:9000/1.html 2.html 都可以返回hello express

```js
const app = express()

app.get('/:id.html', (req, res) => {
  res.end('hello express')
    //获取params
   console.log(req.params.id)
})

```



### 设置响应报文参数

| 方法                   | 作用                                                    |
| ---------------------- | ------------------------------------------------------- |
| res.statusCode         | 设置状态码                                              |
| res.setHeader          | 设置响应头 （'content-type','text/html;charset=utf-8'） |
| res.end                | 设置响应体                                              |
| `res.json`             | express返回JSON类型数据                                 |
| `res.status`           | express设置的状态码                                     |
| `res.send()`           | express设置响应体，且不会乱码                           |
| `res.set`              | 设置响应头                                              |
| `res.redirect()`       | 跳转响应                                                |
| **res.sendFile(路径)** | **响应文件内容，可以返回html文档**                      |

一般来说，通过express.static搭建静态资源中间件(即静态资源服务，注意不能写/page/index.html)。后端路由想要返回html文档，可以通过res.sendFile()

## 中间件

> 中间件本质是一个回调函数，用来处理逻辑

全局中间件：每一个请求到达服务器后都会执行的回调函数。 `中间件函数`可以像路由回调一样访问 `请求对象、响应对象`

### express中间件原理

定义一个`中间件数组`，每一次`调用use就将中间件推入其中`，执行完成后再调用next继续执行下一个中间件。如果没有next就会终止。

### 全局中间件

> `每一个请求`到达服务端之后 `都会执行全局中间件函数`

**通过next()继续执行，通过server.use()调用中间件函数**

```js
//声明全局中间件函数
function recordMiddleWare(req, res, next) {
  //next():内部函数，执行以后才会继续执行
  let { url, ip } = req
  //将信息保存文件中,不断追加
  fs.appendFileSync(path.resolve(__dirname, './access.log'), `${url} ${ip}\r\n`)
  //中间件执行完毕，调用next
  next()
}
//使用中间件函数
app.use(recordMiddleWare)
```



### 路由中间件

> 在进入路由处理前进行判断处理

```js
const express = require('express')
const app = express()

//路由中间件  路由判断前进行处理
const checkCodeMiddleWare = (req, res, next) => {
  const { path } = req
  if (req.query.code == '521') {
    //满足条件，调剩余的路由回调
    next()
  } else {
    res.send('暗号错误')
  }
}

app.get('/home', checkCodeMiddleWare, (req, res) => {
  res.send('前端首页')
})

app.get('/admin', checkCodeMiddleWare, (req, res) => {})

app.get('/setting', checkCodeMiddleWare, (req, res) => {
  res.send('设置页面')
})
app.listen(9000, () => {})

```



#### 路由模块化

```js
const express = require('express')

//创建后端路由对象
const router = express.Router()

router.get('/home', (req, res) => {
  res.send('前端首页')
})

router.get('/search', (req, res) => {
  res.send('搜索页面')
})

module.exports = router


----
server.js中
const homeRouter = require('./routes/homeRouter')
app.use(homeRouter)

```















### 静态资源中间件  express.static()

> 静态资源：项目上线后，长时间不发生改变的内容。如css、js

**静态资源目录：**当请求发送到服务端后，`服务端到哪个文件夹找对应的文件`，那个文件夹就是静态资源目录  。

http模块是获取请求路径，然后`拼接文件路径`，最后`读取文件返回`。而那个抽离出来的文件夹就是静态资源目录。

> 在express中，html中通过link导入的css或者通过script导入的js，他们都是静态资源，都需要到静态资源目录下寻找相应资源，然后返回。所以link的路径，必须是个绝对路径，直接**href='/css/style.css'**。因为绝对路径会和当前网页的协议、主机、端口拼接形成一个完整的get请求发送。
>
> ![image-20230330153555297](https://gitee.com/zhengdashun/pic_bed/raw/master/img/image-20230330153555297.png)
>
> ![image-20230330152535229](C:\Users\szdrz\AppData\Roaming\Typora\typora-user-images\image-20230330152535229.png)



```js
const express = require('express')

//创建应用对象
const app = express()
//静态资源中间件设置   
app.use(express.static(__dirname+'/public'))
app.get('/home', (req, res) => {
  res.send('hello express')
})

app.listen(9000, () => {})

```

![image-20230329134231846](https://gitee.com/zhengdashun/pic_bed/raw/master/img/image-20230329134231846.png)

**优点：**不需要自己去额外配置静态资源服务。之前http模块需要根据不同的path，然后读取不同的文件，最后返回。



**注意事项：**

- index.html是默认打开的资源文件。没必要写**/public，只需要/public**。 **（写了反而报错，因为设置的是静态资源，不只是html）**。但是发起的请求是没有这个/public的

  ![image-20230330152535229](C:\Users\szdrz\AppData\Roaming\Typora\typora-user-images\image-20230330152535229.png)

- 如果静态资源与路由规则同时匹配，(比如路由规则是app.get('/'))那么谁先匹配就响应。**因为express执行是自上向下的**

- 路由响应动态资源，静态资源中间件(即静态资源服务)响应静态资源







# koa

```js
___安装配置____
npm install koa
```



## 搭建koa服务

```js
const Koa = require('koa')

const app = new Koa()

app.use(ctx => {
  ctx.body = 'hello koa2'
})

app.listen(9000, () => {})

```



## Koa和express的区别

[koa和expres的区别](https://zhuanlan.zhihu.com/p/115339314)

> koa是不支持路由的，需要导入第三方库koa-router

|                | Koa                                                          | express                       |
| -------------- | ------------------------------------------------------------ | ----------------------------- |
| 初始化         | const app=new Koa()                                          | const app=express()           |
| 实例化路由     | const Router=require('koa-router')<br/>const router=Router() | const router=express.Router() |
| 路由中间件挂载 | app.use(router.routes())                                     | app.use('/',router)           |

| koa                                                | express                                                      |
| -------------------------------------------------- | ------------------------------------------------------------ |
| 中间件支持异步操作，通过async和await进行异步处理。 | 对于中间件，exress采用的是回调函数的方式实现，它不会去等待异步完成，如果有异步操作就会出现异常。所以express中间件不支持异步 |
|                                                    |                                                              |
|                                                    |                                                              |



## 中间件

中间件：本质上就是一个回调函数，用来处理代码。分为全局中间件和路由中间件。

全局中间件：当任意一个请求到达服务器后，都会执行全局中间件函数。接受了两个参数一个ctx，next()。只有调用了next才能继续执行下一个中间件。

路由中间件：只有当特点的请求到达服务器后，才会执行对应的路由中间件，服务器返回相应的资源。



## 洋葱圈模型

koa底层使用的是递归，逐级深入，再逐层返回。

```js
//即使加了async和await也是逐级深入，逐层返回
app.use((ctx, next) => {
  console.log(1)
  next()
  console.log(2)
})
app.use((ctx, next) => {
  console.log(3)
  next()
  console.log(4)
})
app.use((ctx, next) => {
  console.log(5)
  next()
  console.log(6)
})
app.use((ctx, next) => {
  console.log(7)
  next()
  console.log(8)
})
执行结果为:
1 3 5 7 8 6 4 2
```



## 异步处理

> 如果中间件存在异步的代码，koa通过async和await进行处理。

koa的next()返回的其实是一个 **`Promise`**

```js
server.use((ctx, next) => {
  ctx.message = 'aa'
  next()
  ctx.body = ctx.message
})
server.use((ctx, next) => {
  console.log(ctx)
  ctx.message += 'bb'
  next()
})
server.use((ctx, next) => {
  //它会认为所有同步代码执行完了，没有执行异步，直接next()
  Promise.resolve('cc').then(data => {
    ctx.message += data
  })
  next()
})
//所以最后结果只是aabb   

const Koa = require('koa')
const server = new Koa()
server.use(async (ctx, next) => {
  ctx.message = 'aa'

  await next()
  ctx.body = ctx.message
})
server.use(async (ctx, next) => {
  console.log(ctx)
  ctx.message += 'bb'
  await next()
})
server.use(async (ctx, next) => {
  //它会认为所有同步代码执行完了，没有执行异步，直接next()
  const res = await Promise.resolve('cc')
  ctx.body += 'res'
  await next()
})
server.listen(9000, () => {})

```



## 路由

`npm i koa-router`

> 后端路由：根据不同的Method和URL返回不同的资源。

```js
const Koa = require('koa')
const app = new Koa()

const Router = require('koa-router')
//实例化路由对象
const router = new Router()
//编写路由规则
router.get('/', (ctx, next) => {})
//注册路由中间件
app.use(router.routes())
app.listen(9000, () => {})

```



### 路由模块化

`npm install koa-router`

```js
const router = require('./router/router')
app.use(router.routes())
```

```js
const Router = require('koa-router')
const router = new Router()
router.get('/api', (ctx, next) => {
  ctx.body = '123'
})

module.exports = router

```





## Koa返回资源

注意：如果是json类型 ctx.type='json' ,需要使用绝对路径

```js
const Router = require('koa-router')
const router = new Router()
const fs = require('fs')
const path = require('path')

router.get('/api', (ctx, next) => {
  ctx.type = 'html'
  const html = fs.readFileSync(path.resolve(__dirname, './index.html'))
  ctx.body = html

})

module.exports = router

```



## 静态资源中间件

`npm install koa-static`

![image-20230407165413779](https://gitee.com/zhengdashun/pic_bed/raw/master/img/image-20230407165413779.png)

```js
const Koa = require('koa')
const serve = require('koa-static')
const app = new Koa()

app.use(serve(__dirname + '/public'))
app.listen(9000, () => {})

```

koa静态资源中间件的优点：express搭建静态资源，html文档中引入的css必须是/的绝对路径，因为它的路径会和**当前协议、主机名、端口拼接，形成一个get请求**。而Koa不会拼接





# 模块化

> 将一个复杂文件拆分成多个文件，`每个文件都是一个单独的模块`，模块内部的数据是私有的，但是可以暴露出来给其他模块使用。

好处： 

- 防止命名冲突
- 高复用性



## 导入文件模块

可导入文件类型：

- JSON数据，可以直接导入

- js文件也可以直接导入 

- 如果导入其他类型文件，它默认`按照js文件类型导入`的

- `如果导入的是个文件夹`，它首先检测文件夹下`pacakage.json`文件中`main`属性对应的文件，如果存在导入，不存在报错。如果没有package.json,就会导入index.js或者index.json

  ![image-20230327145204026](https://gitee.com/zhengdashun/pic_bed/raw/master/img/image-20230327145204026.png)

![image-20230327145211090](https://gitee.com/zhengdashun/pic_bed/raw/master/img/image-20230327145211090.png)



## require导入模块的基本流程

1. 将相对路径转化为绝对路径，定位文件
2. 缓存检测
3. 读取目标文件代码
4. 包裹为一个函数并执行，通过 `arguments.callee.toString()`可以查看
5. 缓存模块的值
6. 返回module.exports的值



## require导入npm包

①在当前文件夹下的node_modules中寻找同名的文件夹

②如果在当前目录中没有，就会去上级目录下的node_modules中寻找同名文件夹，直至找到磁盘根目录

# 包管理工具

常见的包管理工具：npm、yarn、cnpm。

可以对包进行下载安装，更新，删除，上传等等。



## npm init 

> 将文件夹初始化问一个包，交互式的创建`package.json`文件

package.json是包的配置文件，每个包都必须要由package.json。





## 生成环境与开发环境

开发环境：程序员的电脑，只能程序员自己访问

生成环境：指正式的服务器电脑，所有用户都可以访问



| 类型     | 命令                                      | 补充                                                   |
| -------- | ----------------------------------------- | ------------------------------------------------------ |
| 生成依赖 | npm i -S uniq <br/>npm i --save uniq      | -S等同--save 包信息保存在package.json的 `dependencies` |
| 开发依赖 | npm i -D less <br/>npm i --save -dev less | -D等同于 --save -dev。包信息保存在devDependencies      |



## 环境变量的理解

①首先会在当前目录下，找对应同名的exe或者cmd文件，如果找到了直接运行

②如果找不到，就会到环境变量path里，找到对应的路径，执行对应文件



## 安装指定版本包、删除包

`安装：` npm i 包名@版本号

`删除：` npm remove 包名



## yarn

缓存了之前下载过的包，再次使用无需重复下载，所以安装速度很快。会检测安装包的完整性，很安全

| 功能         | 命令                                                         |
| ------------ | ------------------------------------------------------------ |
| 初始化       | yarn init  /  yarn init -y                                   |
| 安装包       | yarn add  less生成依赖 <br>yarn add less -dev 开发依赖 <br>yarn  global add nodemon  全局安装 |
| 删除包       | yarn remove uniq  删除项目依赖包<br> yarn global remove nodemon  全局删除包 |
| 安装项目依赖 | yarn                                                         |
| 运行命令别名 | yarn <别名>  不需要添加run  （npm中除了start都需要有run）    |



**yarn的缺点**：全局安装的依赖不能直接使用。

需要通过yarn global bin查找到对应cmd或者exe所在路径，然后自己配置环境变量



## ~npm 发布一个包~

[切换镜像地址](https://huaweicloud.csdn.net/63a562f0b878a54545945f21.html)









# 浏览器操作

## cookie  

**浏览器操作cookie使用的很少**，了解即可

| 禁用所有cookie | 浏览器设置里面搜cookie，阻止所有cookie                       |
| -------------- | ------------------------------------------------------------ |
| 删除cookie     | 也是设置里删除就行                                           |
| 查看cookie     | ![image-20230330155441075](https://gitee.com/zhengdashun/pic_bed/raw/master/img/image-20230330155441075.png) |



**express里操作cookie**

**设置cookie**

```js
res.cookie('key','value') //缺点：关闭浏览器后cookie消失
```

第一次浏览器发送请求的时候其实没有cookie的

服务器返回的响应头Set-cookie:name=xx。

浏览器看到后，就会设置cookie，之后发送的请求都会携带。

**设置cookie有效期**

```js
res.cookie('key','value',{maxAge:单位数字/毫秒})  maxAge：cookie的有效期
//但是报文里面就变成了秒了。设置的是毫秒
```



**删除cookie**

```js
res.clearCookie('name')
```

浏览器接受到该响应后，就会删除name这个cookie，下一次发送请求就会没有这个cookie



**读取cookie**

安装: `npm i cookie-parser`

它其实就是一个中间件，所以需要调用

```js
res.cookies //安装使用后，就可以直接通过res.cookies获取cookie
```

