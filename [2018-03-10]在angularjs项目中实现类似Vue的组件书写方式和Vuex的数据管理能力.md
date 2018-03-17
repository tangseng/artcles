#### 最近开发了一个库：[angularjs-x](https://github.com/tangseng/angularjs-x)

- 库包含两块能力：TS 和 TSX；
- TS：模仿Vue的组件书写及定义方式，定义了相似的API：data、methods、mixins等；
- TSX：实现简单的类似Vuex的数据管理和响应式能力，从而能够在非相关的组件间实现数据共享和交互，也定义了相似的API，包括state、actions、mutations等函数签名及使用方式；
- 目前只用在angularjs工程中；
- 暂时还没有提交npm registry；

> 库本身的代码不多，但保证了基本功能和目标，之所以开发它，就得好好描述下了，在“很久很久以前...”  
> 言归正传。  
> 从2016年下半年开始，自己已经转向使用Vue技术栈做一些项目的前端开发了。  
> 那时还在负责带团队，为了保持团队活力和提升效率的考虑，需要对团队使用的技术进行决策和迭代，自己在学习研究和试用比较了React、Vue，以及还未正式发布的Angular后，决定团队新的项目的开始优先使用Vue及其生态。  
> 去年下半年来新团队后，自己最先开发的项目（双十一大屏），以及平时协助其他同事开发的项目也都是基于Vue的。  
> 总结起来说，就是自己对于Vue的使用很熟练了，API、相关场景解决方案、实现原理，也包括其生态相关库等。也习惯了使用Vue。习惯不是个好习惯，呵呵呵...  
> 但是但是...  
> 自己目前主要负责的中间件几个项目都是基于angularjs的老项目了，从接手的时候，几个项目已经经过了多人多次的功能升级迭代，代码那个酸爽啊。  
> “天降大任于斯人也，必先...”  
> 背景说完了，继续言归正传。  
> 其实用什么框架和库并不是困难或限制，在实现业务方面，他们没有区别的（当然其他方面还是稍微有一点的，比如开源生态、相关类库、部分情景下的开发效率等）。  
> 只不过是自己比较“欣赏”和“习惯”Vue的书写方式，因此诞生了“TS & TSX”，即angularjs-x。  


#### 下面就展示下在angularjs中使用TS和TSX前后的代码比较。

**先看TSX**

> TSX实现简单的数据共享和响应式能力，angularjs中如果两个非相关的组件（两者的scope没有嵌套或者兄弟关系，单纯通过属性传递很繁琐的情况），这时候我们一般会使用$rootScope进行广播和监听，以达到组件间的数据及时通知，但是有些时候有可能本组件把事件广播出去了，而另外的组件是异步的，错过了首次监听，尤其是页面刚初始化的时候最会出现这样的情况；又或者通过一个共享service来进行数据共享，但是又缺少及时响应的能力；

使用TSX前：
```javascript
//组件1
angular.module('test').directive('testComponent1', ($rootScope) => {
  'ngInject'

  return {
    restrict: 'AE',
    scope: {},
    template: ``,
    link(scope) {
      $rootScope.$broadcast('test', 'meeee....')
    }
  }
})

//组件2
angular.module('test').directive('testComponent2', ($rootScope) => {
  'ngInject'

  return {
    restrict: 'AE',
    scope: {},
    template: ``,
    link(scope) {
      $rootScope.$on('test', (event, message) => {
        //如果component2在dom层级位于component1之后，或者是异步初始化完成，那就可能无法得到message
        console.log(message)
      })
    }
  }
})
```

使用TSX后：
```javascript
//建立共享数据的模块：test模块
//模块中的数据不是仅仅适用于如下展示的这么简单场景，是可以用于多组件，多路由之间的数据共享的
const testModule = {
  state: {
    message: null
  },
  mutations: {
    mutationMessage(state, message) {
      state.message = message
    }
  },
  actions: {
    updateMessage({ commit }, message) {
      commit('mutationMessage', message)
    }
  }
}

//在TSX中注册test模块
TSX.registerModule('test', testModule)

//组件1
angular.module('test').directive('testComponent1', (TSX) => {
  'ngInject'

  return {
    restrict: 'AE',
    scope: {},
    template: `<button ng-click="click()"></button>`,
    link(scope) {
      TSX.mapActionAuto(scope, {
        updateMessage: 'test.updateMessage'
      })

      scope.updateMessage('meeee...')

      //之后的点击触发，其他组件也是可以及时得到数据改变响应的
      scope.click = () => {
        scope.updateMessage('meeee...meeee...')
      }
    }
  }
})

//组件2
angular.module('test').directive('testComponent2', (TSX) => {
  'ngInject'

  return {
    restrict: 'AE',
    scope: {},
    template: `<span>{{message}}</span>`,
    link(scope) {
      TSX.mapStateAuto(scope, {
        message: 'test.message'
      })

      //还可以这样
      // TSX.mapStateAuto(scope, {
      //   message: {
      //     name: 'test.message',
      //     compute(val) {
      //       return val + '加后缀...'
      //     }
      //   }
      // })

      console.log(scope.message)
    }
  }
})
```

**再看TS**

> TS实现了类似Vue组件的书写方式，通过创建一个TS实例来代理scope，来让相关逻辑编写清晰明了、且很Vue化；

使用TS前：
```javascript
angular.module('test').directive('testComponent', (TS) => {
  'ngInject'

  return {
    restrict: 'AE',
    scope: {},
    template: `
      <div>
        <button ng-click="click()">第{{guan}}关打怪</button>
      </div>
    `,
    link(scope, element, attrs) {
      const vm = scope
      const tangseng = {
        name: '唐僧',
        jineng() {
          console.log('念经')
          return true
        }
      }
      const sunwukong = {
        name: '孙悟空',
        jineng() {
          console.log('七十二变')
          return true
        }
      }
      const zhubaijie = {
        name: '猪八戒',
        jineng() {
          console.log('吃')
          return Math.random() > .5
        }
      }
      const shaheshang = {
        name: '沙和尚',
        jineng() {
          console.log('干活')
          return Math.random() > .5
        }
      }
      
      
      const xiyouji = vm.xiyouji = [tangseng, sunwukong, zhubaijie, shaheshang]
      const lastGuan = 81
      const lastCount = 10
      vm.guan = 1
      vm.count = 0
      vm.jieguo = true

      //打怪
      function daguai(who) {
        return Promise.resolve(who.jineng())
      }

      async function toQujing() {
        //每一关选取一个人打怪，最后一关是tangseng，其他的关随机取其他人
        let who
        if (vm.guan == lastGuan) {
          who = tangseng
        } else {
          who = xiyouji[Math.floor(1 + Math.random() * (xiyouji.length - 1))] 
        }
        vm.jieguo = await daguai(who)
        //如果jieguo为true，表示进入下一关
        if (vm.jieguo) {
          vm.guan++
          vm.$apply()
        }
      }

      //页面有按钮，绑定ng-click="click()"，开始取经打怪
      vm.click = () => {
        toQujing()
      }

      //watch每关的结果状态，记录失败次数，如果超过10次，取经失败
      vm.$watch('jieguo', (val) => {
        if (vm.guan >= lastGuan) {
          console.log('取经成功')
        } 
        if (!val) {
          vm.count++
          if (vm.count > lastCount) {
            console.log('取经失败')
          }
        }
      })
    }
  }
})
```

使用TS后：
```javascript
angular.module('test').directive('testComponent', (TS) => {
  'ngInject'

  return {
    restrict: 'AE',
    scope: {},
    template: `
      <div>
        <button ng-click="click()">第{{guan}}关打怪</button>
      </div>
    `,
    link(scope, element, attrs) {
      const tangseng = {...} //省略
      const sunwukong = {...} 
      const zhubaijie = {...}
      const shaheshang = {...}

      const lastGuan = 81
      const lastCount = 10
      //实例化TS，其中data不需要函数，这跟vue组件会可能被多次实例有区别
      const ts = new TS(scope, {
        data: {
          xiyouji: [tangseng, sunwukong, zhubaijie, shaheshang],
          guan: 1,
          count: 0,
          jieguo: true
        },

        watchs: {
          jieguo(val) {
            if (this.guan >= lastGuan) {
              console.log('取经成功')
            }
            if (!val) {
              this.count++
              if (this.count > lastCount) {
                console.log('取经失败')
              }
            }
          }
        },

        methods: {
          async toQujing() {
            let who
            if (this.guan == lastGuan) {
              who = tangseng
            } else {
              who = this.xiyouji[Math.floor(1 + Math.random() * (this.xiyouji.length - 1))] 
            }
            this.jieguo = await this.daguai(who)
            if (this.jieguo) {
              this.guan++
              this.$$apply()
            }
          },

          daguai(who) {
            return Promise.resolve(who.jineng())
          }
        },

        scopeMethods: {
          click() {
            this.toQujing()
          }
        }
      })
    }
  }
})
```

**最后看TS和TSX的结合**

> TS中实现了结合使用TSX的语法糖：computed来绑定state 和 methods中可以绑定action

```javascript
//建立共享数据的模块 test.js
const testModule = {
  state: {
    message: ''
  },
  mutations: {
    mutationMessage(state, message) {
      state.message = message
    }
  },
  actions: {
    updateMessage({ commit }, message) {
      commit('mutationMessage', message)
    }
  }
}

//在TSX中注册test模块
TSX.registerModule('test', testModule)

//组件
angular.module('test').directive('testComponent', (TS) => {
  'ngInject'

  return {
    restrict: 'AE',
    scope: {},
    template: `
      <div>
        <span>{{message}}</span>
        <button ng-click="click()"></button>
      </div>
    `,
    link(scope) {
      const component = new TS(scope, {
        computed: {
          message: 'test.message'
        },
        methods: {
          updateMessage: 'test.updateMessage'
        },
        scopeMethods: {
          click() {
            this.updateMessage('hehe...')
          }
        }
      })
    }
  }
})
```

目前TS & TSX已经在其中一个项目的新功能页面开发上使用了，目标效果还是杠杠的，也同时“满足”了习惯，呵呵呵...