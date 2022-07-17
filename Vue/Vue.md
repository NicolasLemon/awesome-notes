**Vue-Study**

- **作者：** Nicolas·Lemon
- **修改：** Nicolas·Lemon
- **创建日期：** 2022.07.17
- **修改日期：** 2022.07.17

# Vue2.0

## 前端环境

### 版本

* NodeJs：`v12.22.0`

* @vue/cli：`v4.5.17`

<img title="" src="Vue.assets/2022-07-17-16-01-14-image.png" alt="" data-align="inline">

### 配置

#### Node地址

**安装目录：** `D:\Program Files (x86)\nodejs`

* 在安装目录下新建`node_cache`与`node_global`两个文件夹
  
  ![](Vue.assets/2022-07-17-16-15-48-image.png)

* 设置node地址
  
  ```bash
  # 设置相关
  npm config set prefix "D:\Program Files (x86)\nodejs\node_global"
  npm config set cache "D:\Program Files (x86)\nodejs\node_cache"
  
  # 查看相关
  npm root -g
  npm config get prefix
  npm config get cache
  ```
  
  ![](Vue.assets/2022-07-17-16-22-16-image.png)

* 添加环境变量
  
  ```textile
  D:\Program Files (x86)\nodejs\
  
  D:\Program Files (x86)\nodejs\node_global
  ```
  
  ![](Vue.assets/2022-07-17-16-25-51-image.png)

#### 淘宝镜像

```bash
# 查看当前npm仓库镜像地址
npm config get registry
# 设置当前npm仓库镜像地址为淘宝镜像
npm config set registry http://registry.npm.taobao.org/
```

![](Vue.assets/2022-07-17-16-09-46-image.png)

#### Vue脚手架

```bash
npm install -g @vue/cli@4.5.17
```

![](Vue.assets/2022-07-17-16-28-22-image.png)

## 响应式布局

### 安装插件

- **flexible**
  
  阿里团队研发的自适应响应式布局插件
  
  ```bash
  npm install --save lib-flexible
  ```

- **postcss-plugin-px2rem**
  
  自动将代码中的`px`转换成`rem`，这样在写代码的时候，仍然可以写成`px`的单位
  
  ```bash
  npm install --save postcss-plugin-px2rem
  ```

- **cssrem** （可选）
  
  vs code插件，写代码时，可以提示将`px`转换成`rem`

### flexible

#### 引入

- 在`src/main.js`中import导入
  
  ```v
  // 引用 flexible 插件
  import "lib-flexible/flexible.js";
  ```
  
  <img title="" src="Vue.assets/2022-07-17-16-37-02-image.png" alt="" data-align="inline">

#### 配置

- 修改`flexible.js`
  
  找到`node_modules\lib-flexible\flexible.js`，修改`refreshRem()`函数
  
  ```js
  function refreshRem() {
      var width = docEl.getBoundingClientRect().width;
      // if (width / dpr > 540) {
      //     width = 540 * dpr;
      // }
  
      // 修改 最大值：2560   最小值：400
      if (width / dpr < 400) {
          width = 400 * dpr;
      } else if (width / dpr > 2560) {
          width = 2560 * dpr;
      }
      var rem = width / 10;
  
      docEl.style.fontSize = rem + 'px';
      flexible.rem = win.rem = rem;
  }
  ```
  
  ![](Vue.assets/2022-07-17-16-36-05-image.png)

### postcss-plugin-px2rem

- 修改`vue.config.js`
  
  在`css.loaderOptions`选项后添加，其中根据自身项目去确定换算基数`rootValue`的值
  
  ```js
  css: {
      loaderOptions: {
          // ......
          postcss: {
              plugins: [
                  require("postcss-plugin-px2rem")({
                      // 换算基数， 默认100  ，这样的话把根标签的字体规定为1rem为50px,这样就可以从设计稿上量出多少个px直接在代码中写多上px了。
                      rootValue: 207,
                      // 默认false，可以（reg）利用正则表达式排除某些文件夹的方法，例如/(node_module)/ 。如果想把前端UI框架内的px也转换成rem，请把此属性设为默认值
                      exclude: /(node_module|other)/,
                      //（布尔值）允许在媒体查询中转换px。
                      mediaQuery: false,
                      // 设置要替换的最小像素值(3px会被转rem)。 默认 0
                      minPixelValue: 0,
                      /// 允许REM单位增长到的十进制数字。
                      // unitPrecision: 5, 
                      /// 默认值是一个空数组，这意味着禁用白名单并启用所有属性。
                      // propWhiteList: [],  
                      // 黑名单
                      // propBlackList: [], 
                      /// 要忽略并保留为px的选择器
                      // selectorBlackList: [], 
                      ///（boolean/string）忽略单个属性的方法，启用ignoreidentifier后，replace将自动设置为true。
                      // ignoreIdentifier: false,
                      /// （布尔值）替换包含REM的规则，而不是添加回退。
                      // replace: true
                  })
              ]
          }
      }
  }
  ```
  
  ![](Vue.assets/2022-07-17-16-34-54-image.png)
