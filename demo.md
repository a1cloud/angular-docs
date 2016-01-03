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
* 首先在src/app/services下创建demo.js
``` js

/**
* Demo服务
*/
export default class DemoService {
  constructor($http) {
    this.$http = $http;
  }

  findAll(params){
    return this.$http({
      method:'GET',
      params: params,
      url:'/api/Product/GetProducts'
    });
  }
}

DemoService.$inject=['$http'];

```

_注意：这里面使用到了$inject，这里使用的是依赖注入，一定要写上，这样程序才能明白$http是外表进行注入_

* 在当前目录的index.js注册服务以供controller依赖注入使用
```js

export default angular.module('app.services', [])
  .service('DemoService',require('./demo'))
  .name;

```

##创建ui路由配置

_注意：路由我们使用的是angular-ui-router(https://github.com/angular-ui/ui-router)具体使用请参考_

*创建demo的路由文件 （目前项目根据大模块路由进行单独配置）

```js

export default function routes($stateProvider, $urlRouterProvider) {

  $stateProvider
    .state('app.demo', {
      url: '/demo',
      abstract: true,
      template: '<ui-view/>'
    });

  $stateProvider
    .state('app.demo.main', {
      url: '',
      controller: 'demo.main',
      template: require('../views/demo/main.html'),
      controllerAs: 'vm',
      authenticate: false
    });
}

```

* 在同目录下的index.js文件中进行路由配置
```js

export default 'app.routes';

angular.module('app.routes', []).config(function($stateProvider, $urlRouterProvider) {
  require('./app')($stateProvider, $urlRouterProvider);
  require('./products')($stateProvider, $urlRouterProvider);
  require('./orders')($stateProvider, $urlRouterProvider);
  require('./store')($stateProvider, $urlRouterProvider);
  require('./demo')($stateProvider, $urlRouterProvider);
});


```

_说明：到目前为止，使用gulp启动程序就可以通过 /#/demo当问您的程序了_


## 添加点业务测试一下

* 首先打开src/app/controllers/demo/main.js
```js

export default class MainController {

  constructor(DemoService,$location) {
    this.DemoService = DemoService;
    this.$location=$location;
    this.loading = true;
    this.data = {};
    let {pageNumber,codeOrName} = $location.search();

    this.query = {
      pageSize :10,
      pageNumber:pageNumber||1,
      codeOrName :codeOrName||''
    };

    this.loadData();
  }

  loadData() {
    this.fetchData(this.query);
  }

  go(pageNumber){

    this.query.pageNumber = pageNumber;
    this.$location.search(this.query);
    this.fetchData(this.query);
    return;
  }

  fetchData(params) {
    this.loading=true;
    this.DemoService.findAll(params).then((resp) => {
      let data = resp.data.data || {};
      this.data = {
        items: data.Data || [],
        total: data.TotalRecords || 0,
        page: data.PageNumber || 1
      };
      this.loading = false;
    }, (err) => {
      this.errorMessage = err.data.Message;
      this.loading = false;
    });
  }
}

MainController.$inject = ['DemoService','$location'];



```

_说明：这里我们同$inject的注入方式使用DemoService进行获取数据，然后赋值到this.data中, 我们在路由里面使用了controllerAs:'vm',我们后面就可以在页面里面使用vm.data获取服务提供的数据了

##修改界面，src/app/views/demo/main.html

```html


<section class="panel panel-default content-box">
  <header class="panel-heading"> Demo列表 </header>
  <div class="table-responsive">
    <table class="table table-striped b-t b-light">
      <thead>
        <tr>
          <th width="100">商品编码</th>
          <th>商品名称</th>
          <th width="100">单价</th>
        </tr>
      </thead>

      <!--loading-->
      <tbody ng-if="vm.loading">
        <tr >
          <td colspan='6'>
          <a1-loader/>
          </td>
        </tr>
      </tbody>
      <!--not found-->
      <tbody ng-if="vm.data.items && vm.data.items.length===0 && vm.loading==false && !vm.errorMessage">
        <tr>
          <td colspan='6' height="200" style="vertical-align: middle;text-align:center">
            <i class='fa fa-umbrella fa-5x'></i>
            <div>没有找到相关商品</div>
          </td>
        </tr>
      </tbody>
      <!--list items-->
      <tbody ng-if="vm.data.items && vm.data.items.length>0 && vm.loading==false">
        <tr ng-repeat="item in vm.data.items">
          <td>{{item.Code}} </td>
          <td style="text-align:left;padding-left:10px;">{{item.Name}}</td>
          <td>  {{item.UnitPrice|currency:"￥"}}</td>
        </tr>
      </tbody>
    </table>
  </div>
  <!--page-->
  <footer class="panel-footer" ng-if="vm.data.items && vm.data.items.length>0 && vm.loading==false">
    <div class="row">
      <div class="col-sm-4 hidden-xs">
        <!--button-->
      </div>
      <div class="col-sm-8 text-right text-center-xs">
        <a1-paging
          page="vm.query.pageNumber"
          ul-class="pagination pagination-sm m-t-none m-b-none"
          page-size="10"
          total="vm.data.total"
          show-prev-next=true
          paging-action="vm.go(page)">
          </div>
      </div>

    </div>
  </footer>
</section>



```

到此为止一个通过服务获取远程访问，然后显示数据的demo就制作完成了。







