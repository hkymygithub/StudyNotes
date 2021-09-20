# WebApi

WebApi：浏览器提供的一套操作浏览器功能和页面元素的API（BOM和DOM）。

MDN文档：https://develouer.mozilla.org/zh-CN/docs/Web/API

### 获取元素

#### 根据ID获取

```javascript
document.getElementById(id)	//返回元素
```

#### 根据标签名获取

```javascript
document.getElementByTagName(name) //元素列表
```

#### 根据类名获取

```javascript
document.getElementByClassName(classNmae) //元素列表
```

#### 根据选择器获取

```javascript
document.querySelector() //返回第一个对象
```

```
document.querySelectorAll() //返回所有对象
```

 ### 获取特殊元素

```javascript
document.body	//获取body
document.documentElement	//获取html
```

### 事件

| 鼠标事件    | 触发条件     |
| ----------- | ------------ |
| onclick     | 鼠标左键点击 |
| onmouseover | 鼠标经过     |
| onmousout   | 鼠标离开     |
| onfocus     | 获取焦点     |
| onblur      | 失去焦点     |
| onmousemove | 鼠标移动     |
| onmouseup   | 鼠标弹起     |
| onmousedown | 鼠标按下     |

### 操作元素

#### 修改标签中的值

-  innerText：修改标签中的值，不识别html，去除空格和换行。
- innerHTML：修改标签中的值，识别html，不去除空格和和换行。

#### 修改元素属性

- src
- href
- title
- id
- alt

#### 修改表单属性

- type
- value：修改表单中的值。
- checked
- selected
- disabled：表单禁用。

#### 修改样式属性style

- backgroundColor
- width

### 自定义属性

#### 设置

```javascript
element.setAttribute('属性','值')
```

#### 获取

```javascript
element.getAttribute('属性')
```

#### 删除

```
element.removeAttribute('属性')
```

