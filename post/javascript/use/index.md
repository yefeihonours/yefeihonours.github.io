* 数字金额转化大写
* 计算字符串日期对应星期几
* 字符串格式化[四舍五入]
* 日期格式化
* 连字符转驼峰-驼峰转连字符

<!--more-->

```
/**
 * 数字金额转化大写
 * @param {*金额} n 
 */
const digitUppercase = (n) => {
  const fraction = ['角', '分'];
  const digit = ['零', '壹', '贰', '叁', '肆', '伍', '陆', '柒', '捌', '玖'];
  const unit = [
    ['元', '万', '亿'],
    ['', '拾', '佰', '仟'],
  ];
  let num = Math.abs(n);
  let s = '';
  fraction.forEach((item, index) => {
    s += (digit[Math.floor(num * 10 * (10 ** index)) % 10] + item).replace(/零./, '');
  });
  s = s || '整';
  num = Math.floor(num);
  for (let i = 0; i < unit[0].length && num > 0; i += 1) {
    let p = '';
    for (let j = 0; j < unit[1].length && num > 0; j += 1) {
      p = digit[num % 10] + unit[1][j] + p;
      num = Math.floor(num / 10);
    }
    s = p.replace(/(零.)*零$/, '').replace(/^$/, '零') + unit[0][i] + s;
  }

  return s.replace(/(零.)*零元/, '元').replace(/(零.)+/g, '零').replace(/^整$/, '零元整');
}
```
----------
```

/**
 * 计算字符串日期对应星期几
 * @param {"2018-01-01"} day 
 */
const getWeekDay = (day) => {
  let week = new Date().getDay();
  if ( day !== undefined ) {
    week = new Date(day).getDay();
  }
  let week_str = "周";
  switch (week) {
    case 0 :
      week_str += "日";
      break;
    case 1 :
      week_str += "一";
      break;
    case 2 :
      week_str += "二";
      break;
    case 3 :
      week_str += "三";
      break;
    case 4 :
      week_str += "四";
      break;
    case 5 :
      week_str += "五";
      break;
    case 6 :
      week_str += "六";
      break;
  }
  return week_str
}
```
----------

```
/**
 * 字符串格式化
 * [四舍五入]
 */
 
var formats = {
    '%': function(val) { return '%'; },
    'b': function(val) { return parseInt(val, 10).toString(2); },
    'c': function(val) { return String.fromCharCode(parseInt(val, 10)); },
    'd': function(val) { return parseInt(val, 10) ? parseInt(val, 10) : 0; },
    'u': function(val) { return Math.abs(val); },
    'f': function(val, p) { return (p > -1) ? Math.round(parseFloat(val) * Math.pow(10, p)) / Math.pow(10, p) : parseFloat(val); },
    'o': function(val) { return parseInt(val, 10).toString(8); },
    's': function(val) { return val; },
    'S': function(val, p) { var len = p - val.toString().length; for (i = 0; i < len; i++) val = '0' + val; return val; },
    'x': function(val) { return ('' + parseInt(val, 10).toString(16)).toLowerCase(); },
    'X': function(val) { return ('' + parseInt(val, 10).toString(16)).toUpperCase(); }
};

var re = /%(?:(\d+)?(?:\.(\d+))?|\(([^)]+)\))([%bcdufosSxX])/g;

var dispatch = function(data) {
    if (data.length == 1 && typeof data[0] == 'object') {
        data = data[0];
        return function(match, w, p, lbl, fmt, off, str) {
            return formats[fmt](data[lbl]);
        };
    } else {
        var idx = 0;
        return function(match, w, p, lbl, fmt, off, str) {
            return formats[fmt](data[idx++], p);
        };
    }
};

String.prototype.sprintf = function() {
    //var argv = Array.apply(null, arguments);
    return this.replace(re, dispatch(arguments));
}
String.prototype.vsprintf = function(data) {
    return this.replace(re, dispatch(data));
}

///////
let value = "Value(%d, %.2f, %s)".sprintf(12, 5.6666, "Hello");

```
----------

```
/**
 * 日期格式化
 */
Date.prototype.format = function (format) {
  const o = {
    'M+': this.getMonth() + 1,
    'd+': this.getDate(),
    'h+': this.getHours(),
    'H+': this.getHours(),
    'm+': this.getMinutes(),
    's+': this.getSeconds(),
    'q+': Math.floor((this.getMonth() + 3) / 3),
    S: this.getMilliseconds(),
  }
  if (/(y+)/.test(format)) {
    format = format.replace(RegExp.$1, `${this.getFullYear()}`.substr(4 - RegExp.$1.length))
  }
  for (let k in o) {
    if (new RegExp(`(${k})`).test(format)) {
      format = format.replace(RegExp.$1, RegExp.$1.length === 1 ? o[k] : (`00${o[k]}`).substr(`${o[k]}`.length))
    }
  }
  return format
}

//////
let date_string = new Date().format("yyyy-MM-dd")

```
----------

```

// 连字符转驼峰
String.prototype.hyphenToHump = function () {
  return this.replace(/_(\w)/g, (...args) => {
    return args[1].toUpperCase()
  })
}

// 驼峰转连字符
String.prototype.humpToHyphen = function () {
  return this.replace(/([A-Z])/g, '_$1').toLowerCase()
}

```