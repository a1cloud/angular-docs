#手把手教您制作一个Demo

##创建controller

* 首先创建demo目录
``` bash
cd src/app/controllers
mkdir demo

```
* 添加demo模块的默认主文件(npm模块默认使用index.js作为主文件，如果想修改具体请参考npm package.json相关文档)

在index.js文件中添加模块相关代码
``` js
export default angular.module('app.controllers.demo')
  .name;
  
```

* 在demo目录下创建main.js
``` js
export default class MainController {

  constructor() {}
}

```

* 在index.js中配置controller
```js

export default angular.module('app.controllers.demo')
  .controller('demo.main', require('./main'))
  .name;

```

* 在上一级(../index.js中)注册demo模块
```js

import demo from './demo';

export angular.module('app.controllers', [home,products,orders,demo]).name;

```

##创建view
* 首先创建demo目录
``` bash
cd src/app/views
mkdir demo

```

* 在src/app/views/demo下创建main.html

##创建service



