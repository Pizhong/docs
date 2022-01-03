# JS工具类函数

## 一、验证数字

```javascript
function isQuantity(*quantity*) {
  const regExp = /^[0-9]*$/
  return regExp.test(quantity)
}
```

## 二、验证颜色编码

```javascript
function isColor(color) {
  const regExp = /^#([0-9a-fA-F]{6}|[0-9a-fA-F]{3})/
  return regExp.test(color)
}
```

## 三、验证域名格式（可以是http也可以是字母开头的情况）

```javascript
function isUrl(url) {
  const reg = /^((http|https):\/\/)|([A-Za-z]\.)([\w.]+\/?)\S*/
  return reg.test(url)
}
```

## 四、检验是否是http:// 或者 https:// 开头 (必须以协议头开头的场景)

```javascript
function isHttp(url) {
  const reg = /^https?:\/\//i
  return reg.test(url)
}
```

## 五、完整域名校验

```javascript
function isCom(url) {
  const reg = /^(?=^.{3,255}$)[a-zA-Z0-9][-a-zA-Z0-9]{0,62}(\.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+$/
  return reg.test(url)
}
```

## 六、日期格式化

```javascript
function formatDate(date, format) {
  if (!date) return '-'
  if (!(date instanceof Date)) {
    date = new Date(date);
  }
  var paddNum = function (num) {
    num += "";
    return num.replace(/^(\d)$/, "0$1");
  }
  //指定格式字符
  var cfg = {
    yyyy: date.getFullYear(), //年 : 4位
    yy: date.getFullYear().toString().substring(2), //年 : 2位
    M: date.getMonth() + 1, //月 : 如果1位的时候不补0
    MM: paddNum(date.getMonth() + 1), //月 : 如果1位的时候补0
    d: date.getDate(), //日 : 如果1位的时候不补0=
    dd: paddNum(date.getDate()), //日 : 如果1位的时候补0
    hh: paddNum(date.getHours()), //时
    mm: paddNum(date.getMinutes()), //分
    ss: paddNum(date.getSeconds()) //秒
  }
  format || (format = "yyyy-MM-dd hh:mm:ss");
  return format.replace(/([a-z])(\1)*/ig, function (m) {
    return cfg[m];
  })
}
```

## 七、导出 Excel

```javascript
function exportExcel(url, param) {
  if (url.indexOf('.com') == -1) {
    url = $server + url
  }
  url = url + '?'
  for (var key in param) {
    if (param[key] !== '') {
      var oValue = param[key]
      if (typeof (oValue) != 'string') {
        oValue = JSON.stringify(oValue)
      }
      url = url + key + '=' + oValue + '&';
    }
  }
  var a = document.querySelector('#aClick')
  a.href = url
  a.click()
}
```

## 八、获取url带的参数

```javascript
function getParam(name) {
  var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)");
  //var r = window.location.search.substr(1).match(reg);
  var tmp = window.location.href.split('?');
  if (tmp.length <= 1) return null;
  var r = tmp[tmp.length - 1].match(reg);

  if (r != null) {
    return unescape(r[2]);
  } else {
    return null;
  }
}
```

## 九、生成随机或者指定位数的英文数字组合

```javascript
/**
 * 生成随机或者指定位数的英文数字组合
 * @param {boolean} randomFlag	是否是随机生成位数
 * @param {number} min 			生成随机位数的最小数
 * @param {number} max			生成随机位数的最大数
 * @return {string}				返回生成的英文数字组合
*/
function randomWord(randomFlag, min, max) {
  let str = '',
  range = min,	// 默认赋值为第二个参数，如果是随机产生位数会通过下面的if改变。
  arr = ['1', '2', '3', '4', '5', '6', '7', '8', '9','a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'];

  // 随机产生
  if (randomFlag) {
    range = Math.round(Math.random() * (max - min)) + min;
  }
  for (let i = 0; i < range; i++) {
    let index = Math.round(Math.random() * (arr.length - 1));
    str += arr[index];
  }
  return str;
}
```

## 十、树形数组平铺

```javascript
/**
 * 把深层级的数组,平铺成一个单层数组
 * @param {*} arr 需要平铺的数组
 * @param {*} childrenKey 子节点的key
 * @returns 只有单层的数组
 */
function flatDeep(arr, childrenKey) {
  return arr.reduce((acc, val) => {
    return acc.concat(
      (val[childrenKey] && val[childrenKey].length > 0) ?
        (acc.push(val) && flatDeep(val[childrenKey], childrenKey)) :
        [val]
    )
  }, [])
}
```

## 十一、邮箱验证

```javascript
function isEmail(str) {
  return /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/.test(str)
}
```

## 十二、手机号码验证

```javascript
function isMobile(n) {
  return (/^1[0-9]{10}$/).test(n)
}
```

## 十三、数组去重

```javascript
function unique(array) {
  return Array.from(new Set(array));
}
```

## 十四、根据key从数组查找对象

```javascript
function findArrElement(array,keyValue) {
  let resultArr = array.find((item) => {
    if(item.key === keyValue){
      return item;
    }
  });
  return resultArr;
}
```

## 十五、检查对象是否存在属性

```javascript
function checkKey(obj, key) {
  if (key in obj) {
      if (obj.hasOwnProperty(key)) {
          return obj[key];
      }
  }
}
```

## 十六、图片下载

```javascript
function downloadImage(src) {
  const image = new Image()
  image.setAttribute('crossOrigin', 'anonymous')
  image.onload = () => {
    const canvas = document.createElement('canvas')
    canvas.width = image.width
    canvas.height = image.height
    const ctx = canvas.getContext('2d')
    ctx.drawImage(image, 0, 0, image.width, image.height)
  }
  image.src = src
}
```

