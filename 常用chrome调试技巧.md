# 常用chrome调试技巧

## 一、过滤请求

网页请求服务器，有时候发起的请求太多，我们想知道哪些请求返回阻塞了。我们可以对请求的网络进行过滤，来定位问题。

![image-20211123171533511](https://raw.githubusercontent.com/Pizhong/PicGoImg/main/image-202111231715331511.png)

## 二、滚动元素到视图

在调试`DOM`元素的时候，我们已经聚焦到相关的`DOM`结构上了，但是对应的元素并没有在可视窗口上展示，那么我们可以将其快速滚动到可视窗口。

```
控制面板 => Elements => 右击选中的DOM节点 => Scroll into view
```

https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fa865c9466e4fe6908e7cf267630bde~tplv-k3u1fbpfcp-watermark.awebp

## 三、预设设备

在进行调试的时候，我们手头上没有那么多设备。特别是开发移动端的**猿儿**，在没有充足调试机的情况下，我们就靠调试工具进行模拟。那么，除了谷歌浏览器默认设备的几个值，比如`iPhone X, iPad`。我们还可以自定义自己需要的设备。



![image-20211123172358998](https://raw.githubusercontent.com/Pizhong/PicGoImg/main/image-20211123172358998.png)

CTRL + G：跳转指定行

CTRL + F ：搜索

## 四、快速清空站点缓存

有时候开发调试，我们需要清空缓存信息。

```
控制面板 => command/ctrl + shift + p => clear site data
```

## 五、**Sources** Js资源页面

当我们想不起某个方法的具体使用时候，会打开控制台随意写一些测试代码，或者想测试一下刚刚写的方法是否会出现期待的样子，但是控制台一打回车本想换行但是却执行刚写的半截代码，所以推荐使用**Sources**下面的左侧的Sinppets代码片段按钮，这时候点击创建一个新的片段文件，写完测试代码后把鼠标放在新建文件上run，再结合控制台查看相关信息（**新建了一个名叫：app.js的片段代码，在你的项目环境页面内，该片段可执行项目内的方法**）

![image-20211123173545528](https://raw.githubusercontent.com/Pizhong/PicGoImg/main/image-20211123173545528.png)

## 六、Network

可以看到所有的资源请求，包括网络请求，图片资源，html,css，js文件等请求，可以根据需求筛选请求项，一般多用于网络请求的查看和分析，分析后端接口是否正确传输，获取的数据是否准确，请求头，请求参数的查看

![image-20211123173748714](https://raw.githubusercontent.com/Pizhong/PicGoImg/main/image-20211123173748714.png)

## 七、断点调试

### 1、常见断点快捷键

`F8`: 继续执行 

`F10`: step over, 单步执行, 不进入函数 

`F11`: step into, 单步执行, 进入函数 

`shift + F11`: step out, 跳出函数

 `ctrl + o`: 打开文件

 `ctrl + shit + o`: 跳到函数定义位置

 `ctrl + shift + f`: 所有脚本中搜索

### 2、侧操作按钮含义如下：

`Pause/Resume script execution`：暂停/恢复脚本执行（程序执行到下一断点停止）。

 `Step over next function call`：执行到下一步的函数调用（跳到下一行）。

 `Step into next function call`：进入当前函数。 

`Step out of current function`：跳出当前执行函数。

 `Deactive/Active all breakpoints`：关闭/开启所有断点（不会取消）。

 `Pause on exceptions`：异常情况自动断点设置。

```javascript
var foo = function () {

    console.log('foo1');

}

foo();  // foo1

var foo = function () {

    console.log('foo2');

}

foo(); // foo2
```



## 八、console技巧

### 1、`console.log`

打印日志

### 2、`console.error`

打印错误

```javascript
console.error("Error")
```



### 3、`console.table`

打印表格

```javascript
var arr= [ 
  { num: "1"},
  { num: "2"}, 
  { num: "3" }
];
console.table(arr);

var obj= {
  a:{ num: "1"},
  b:{ num: "2"},
  c:{ num: "3" }
};
console.table(obj);
```



### 4、`console.trace`

追溯逻辑执行过程

```javascript
function d(a) { 
  console.trace();
  return a;
}
function b(a) { 
  return c(a);
}
function c(a) { 
  return d(a);
}
var a = b('123');
```



### 5、`console.clear`

`console.clear` 方法会清空控制台所有打印内容，并将光标返回第一行



## 参考

[前端chrome浏览器调试总结]: https://www.jianshu.com/p/b25c5b88baf5

