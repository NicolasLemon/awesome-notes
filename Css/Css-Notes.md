**CSS入坑记录**

- **作者：** Nicolas·Lemon
- **修改：** Nicolas·Lemon
- **创建日期：** 2022.08.09
- **修改日期：** 2022.08.09

# 三角形

## 原理

![](Css-Notes.assets/2022-08-09-14-02-13-image.png)

```html
<div class="box"></div>
```

```css
.box {
  /** 宽高必须为0 */
  width: 0;
  height: 0;
  /** 兼容性 */
  line-height: 0;
  font-size: 0;
  /** 上右下左 */
  border-color: pink blue yellow green;
  border-style: solid;
  border-width: 50px;
}
```

## 基础三角

### 等腰直角三角形

#### triangle1

![](Css-Notes.assets/2022-08-09-14-08-52-image.png)

```html
<div class="box"></div>
```

```css
.box {
  /** 宽高必须为0 */
  width: 0;
  height: 0;
  /** 兼容性 */
  line-height: 0;
  font-size: 0;
  /** 上右下左，需要原理中的哪块就显示哪块 */
  border-color: pink transparent transparent transparent;
  border-style: solid;
  /** 其互补面的宽度设为0 上->下 */
  border-width: 50px 50px 0 50px;
}
```

#### triangle2

![](Css-Notes.assets/2022-08-09-14-26-37-image.png)

```html
<div class="box"></div>
```

```css
.box {
  /** 宽高必须为0 */
  width: 0;
  height: 0;
  /** 兼容性 */
  line-height: 0;
  font-size: 0;
  /** 上右下左，需要原理中的哪块就显示哪块 */
  border-color: pink transparent transparent transparent;
  border-style: solid;
  border-width: 50px 50px 0 0;
}
```

### 等腰三角形

![](Css-Notes.assets/2022-08-09-14-21-34-image.png)

```html
<div class="box"></div>
```

```css
.box {
  /** 宽高必须为0 */
  width: 0;
  height: 0;
  /** 兼容性 */
  line-height: 0;
  font-size: 0;
  /** 上右下左，需要原理中的哪块就显示哪块 */
  border-color: pink transparent transparent transparent;
  border-style: solid;
  /** 其互补面的宽度设为0 上->下 */
  /** 上：控制高，右和左：间接控制腰（腰是作为原理中的斜边） */
  border-width: 100px 20px 0 20px;
}
```

### 直角三角形

![](Css-Notes.assets/2022-08-09-14-29-50-image.png)

```html
<div class="box"></div>
```

```css
.box {
  /** 宽高必须为0 */
  width: 0;
  height: 0;
  /** 兼容性 */
  line-height: 0;
  font-size: 0;
  /** 上右下左，需要原理中的哪块就显示哪块 */
  border-color: pink transparent transparent transparent;
  border-style: solid;
  border-width: 100px 70px 0 0;
}
```

### 一般三角形

![](Css-Notes.assets/2022-08-09-14-31-47-image.png)

```html
<div class="box"></div>
```

```css
.box {
  /** 宽高必须为0 */
  width: 0;
  height: 0;
  /** 兼容性 */
  line-height: 0;
  font-size: 0;
  /** 上右下左，需要原理中的哪块就显示哪块 */
  border-color: pink transparent transparent transparent;
  border-style: solid;
  border-width: 100px 20px 30px 50px;
}
```

## 综合应用

### 案例1

实现斜边框带`Border`效果的

![](Css-Notes.assets/2022-08-09-14-54-08-image.png)

#### 分析

实际上是由两个斜边框堆叠而成，而一个斜边框又可以由一个正常边框+三角形构成

![](Css-Notes.assets/2022-08-09-14-58-15-image.png)

#### 实现

```html
<div class="bottom">
    <div class="top"></div>
</div>
```

1. 底图层
   
   ```css
   .bottom {
     /** 子绝父相定位 */
     position: relative;
     width: 696px;
     height: 80px;
     background-color: #22c3ed;
     /** 自身after伪元素 */
     &::after {
       /** 子绝父相定位 */
       position: absolute;
       /** 伪元素必须要这个值 */
       content: "";
       /** 定位到底层父容器的最右边 */
       top: 0;
       right: -80px;
       /** 宽高必须为0 */
       width: 0;
       height: 0;
       /** 兼容性 */
       line-height: 0;
       font-size: 0;
       /** 上右下左 */
       border-color: #22c3ed transparent transparent transparent;
       border-style: solid;
       border-width: 80px 80px 0 0;
     }
   }
   ```

2. 顶图层
   
   ```css
   .bottom .top {
     /** 子绝父相定位 */
     position: relative;
     width: 100%;
     height: 77px;
     background-color: pink;
     /** 自身after伪元素 */
     &::after {
       /** 子绝父相定位 */
       position: absolute;
       /** 伪元素必须要这个值 */
       content: "";
       /** 显示在最上层 */
       z-index: 1;
       /** 定位到顶层父容器的最右边 */
       top: 0;
       right: -77px;
       /** 宽高必须为0 */
       width: 0;
       height: 0;
       /** 兼容性 */
       line-height: 0;
       font-size: 0;
       /** 上右下左 */
       border-color: pink transparent transparent transparent;
       border-style: solid;
       border-width: 77px 77px 0 0;
     }
   }
   ```

可以发现顶层和底层只是和颜色不同，高度差即为视觉`Border`的宽度，所以可以提取公共css，进行样式优化，减少重复代码

#### 优化

提取出公共css部分

```html
<div class="box bottom">
    <div class="box top"></div>
</div>
```

```css
.box {
  /** 子绝父相定位 */
  position: relative;
  /** 父容器的宽度 */
  /** h(底图层) - h(顶图层) = BorderWidth */
  width: 696px;
  /** 自身after伪元素 */
  &::after {
    /** 子绝父相定位 */
    position: absolute;
    /** 伪元素必须要这个值 */
    content: "";
    /** 定位贴到父容器的顶部 */
    top: 0;
    /** 宽高必须为0 */
    width: 0;
    height: 0;
    /** 兼容性 */
    line-height: 0;
    font-size: 0;
    border-style: solid;
  }
}
.bottom {
  /** 底图层的高度 */
  height: 80px;
  background-color: #22c3ed;
  /** 自身after伪元素 */
  &::after {
    /** 定位到底层父容器的最右边 */
    right: -80px;
    /** 上右下左 */
    border-color: #22c3ed transparent transparent transparent;
    border-width: 80px 80px 0 0;
  }
}
.bottom .top {
  width: 100%;
  /** 顶图层的高度 */
  height: 77px;
  background-color: pink;
  /** 自身after伪元素 */
  &::after {
    /** 显示在最上层 */
    z-index: 1;
    /** 定位到顶层父容器的最右边 */
    right: -77px;
    /** 上右下左 */
    border-color: pink transparent transparent transparent;
    border-width: 77px 77px 0 0;
  }
}
```

### 案例2

突出边框的

![](Css-Notes.assets/2022-08-09-16-02-18-image.png)

#### 分析

实际上可由一个正常的盒子加上两个三角形组成

![](Css-Notes.assets/2022-08-09-16-05-00-image.png)

#### 实现

##### 方式1

利用伪元素拼凑出两个三角形

```html
<div class="border">
    <div class="top"></div>
    <div class="bottom"></div>
</div>
```

```css
.border {
  /** 整个边框的宽高 */
  width: 160px;
  height: 87px;
}
.border .top {
  /** 上面部分的样式 */
  width: 100%;
  height: 53px;
  text-align: center;
  line-height: 53px;
  font-size: 35px;
  font-weight: 800;
  border: 2px solid #ff9c00;
  border-bottom: none;
}
.border .bottom {
  /** 子绝父相定位 */
  position: relative;
  width: 100%;
  height: 31px;
  background-color: #ff9c00;
  /** 自身before伪元素 */
  &::before {
    /** 子绝父相定位 */
    position: absolute;
    /** 伪元素必要 */
    content: "";
    /** 显示在最下层避免压到中间的内容 */
    z-index: -1;
    // 定位到父容器的最左上
    left: -1px;
    width: 0;
    height: 0;
    border-color: #ff9c00 transparent transparent transparent;
    border-style: solid;
    border-width: 40px 40px 0 0;
  }
  /** 自身after伪元素 */
  &::after {
    position: absolute;
    content: "";
    z-index: -1;
    // 定位到父容器的最右上
    right: -1px;
    width: 0;
    height: 0;
    border-color: #ff9c00 transparent transparent transparent;
    border-style: solid;
    border-width: 40px 0 0 40px;
  }
}
```

可以看出`::before`和`::after`拼凑出来的三角形相当于做了镜像翻转的，因此也可以采用方式2

##### 方式2

只实现左边的三角形，然后对它做一个沿`Y轴翻转`的动作

![](Css-Notes.assets/2022-08-09-16-35-11-image.png)

```html
<div class="border">
    <div class="top"></div>
    <div class="bottom">
        <i class="triangle triangle-left"></i>
        <i class="triangle triangle-right"></i>
    </div>
</div>
```

```css
.border {
  /** 整个边框的宽高 */
  width: 160px;
  height: 87px;
}
.border .top {
  /** 上面部分的样式 */
  width: 100%;
  height: 53px;
  text-align: center;
  line-height: 53px;
  font-size: 35px;
  font-weight: 800;
  border: 2px solid #ff9c00;
  border-bottom: none;
}
.border .bottom {
  /** 底部整体的样式 */
  position: relative;
  width: 100%;
  height: 31px;
  background-color: #ff9c00;
}
.border .bottom .triangle {
  position: absolute;
  /** i元素是行内元素，需要转换一下 */
  display: inline-block;
  /** 让其显示到下层，避免压到中间的内容 */
  z-index: -1;
  width: 0;
  height: 0;
  border-color: #ff9c00 transparent transparent transparent;
  border-style: solid;
  border-width: 40px 40px 0 0;
}

.border .bottom .triangle-left {
  /** 控制左边的三角形 */
  left: -1px;
}
.border .bottom .triangle-right {
  /** 控制右边的三角形 */
  right: -1px;
  /** 沿Y轴翻转 */
  transform: rotateY(180deg);
}
```

### 案例3

![](Css-Notes.assets/2022-08-09-16-44-42-image.png)

#### 实现

```html
<div class="border"></div>
```

```css
.border {
  position: relative;
  width: 122px;
  height: 53px;
  /** 边框的样式 */
  background: linear-gradient(to left, #457d9c, #457d9c) left top no-repeat,
    linear-gradient(to bottom, #457d9c, #457d9c) left top no-repeat,
    linear-gradient(to left, #457d9c, #457d9c) right top no-repeat,
    linear-gradient(to bottom, #457d9c, #457d9c) right top no-repeat,
    linear-gradient(to left, #457d9c, #457d9c) left bottom no-repeat,
    linear-gradient(to bottom, #457d9c, #457d9c) left bottom no-repeat,
    linear-gradient(to left, #457d9c, #457d9c) right bottom no-repeat,
    linear-gradient(to left, #457d9c, #457d9c) right bottom no-repeat;
  background-size: 3px 11px, 11px 3px, 3px 11px, 11px 3px;
  &::after {
    position: absolute;
    content: "";
    /** 定位到容器的最中间和贴近底部 */
    left: 50%;
    margin-left: -6px;
    bottom: 0;
    width: 0;
    height: 0;
    line-height: 0;
    font-size: 0;
    border: 6px solid transparent;
    border-bottom-color: #457d9c;
  }
}
```

# 渐变色边框

![](Css-Notes.assets/2022-08-09-20-38-23-image.png)

## 实现

```html
<div class="border">
    <div class="title">内容测试</div>
</div>
```

```css
.border {
  position: relative;
  width: 460px;
  height: 243px;
  color: #feffff;
  border: 1px solid #264c67;
  background-image: linear-gradient(
    to bottom,
    RGB(14, 48, 75, 0.8) 5%,
    RGB(10, 33, 65, 0.8) 99%
  );
  &::before {
    position: absolute;
    content: "";
    width: 100%;
    right: 0;
    top: -3px;
    border-top: 3px solid;
    border-image: linear-gradient(
        to right,
        rgba(255, 255, 255, 0) 0%,
        #2fb7e2 50%,
        rgba(255, 255, 255, 0) 99%
      )
      2 2 2 2;
    background: linear-gradient(
      to right,
      rgba(255, 255, 255, 0) 0%,
      #2fb7e2 50%,
      rgba(255, 255, 255, 0) 99%
    );
  }
}
.border .title {
  position: absolute;
  font-size: 18px;
  font-weight: 800;
  top: 10px;
  left: 26px;
  z-index: 2;
  &::after {
    position: absolute;
    content: "";
    z-index: -1;
    left: -4px;
    top: 15px;
    width: 115px;
    height: 11px;
    transform: skewX(-30deg);
    background-image: linear-gradient(
      to right,
      RGB(29, 146, 177, 0.7) 5%,
      RGB(22, 97, 123, 1) 30%,
      RGB(15, 46, 71, 0.7) 99%
    );
  }
}
```
