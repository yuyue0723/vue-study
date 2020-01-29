谈谈你对vue组件之间通信的理解
1. props和$emit
父组件向子组件传递数据是通过props传递的，子组件传递数据给父组件是通过$emit出发事件来做到的。
```js
Vue.component('child',{
  data() {
    return {
      mymessage: this.message
    }
  },
  template: `
    <div>
      <input type="text" v-model="mymessage" @input="passData(mymessage)">
    </div>
  `,
  props: ['message'], // 得到父组件传递过来的数据
  methods: {
    passData(val) {
      // 触发父组件中的事件
      this.$emit('getChildData', val)
    }
  }
})

Vue.component('parent', {
  template: `
    <div>
      <p>this is parent component</p>
      <child :message="message" v-on:getChildData="getChildData"></child>
    </div>
  `,
  data() {
    return {
      message: 'hello'
    }
  },
  methods: {
    // 执行子组件触发的事件
    getChildData(val) {
      console.log(val)
    }
  }
})
var app = new Vue({
  el: '#app',
  template: `
    <div>
      <parent></parent>
    </div>
  `
})
```
上面的例子中，有父组件parent和子组件child。
- 父组件传递了message数据给子组件，并且通过v-on绑定了一个getChildData事件来监听子组件的触发事件
- 子组件通过props得到相关的message数据，最后通过this.$emit触发了getChildData事件。

2. attrs和listeners
第一种方式处理父子组件之间的数据传输有一个问题：如果父组件A下面有子组件B，组件B下面有组件C，这时如果组件A想传递数据给组件C怎么办呢？
如果采用第一种方法，我们必须让组件A通过props传递消息给组件B，组件B在通过props传递消息给组件C ，要是组件A和C之间有更多的组件，那采用这种方式就很复杂了。Vue 2.4开始提供attrs和listeners来解决这个问题，能够让组件A传递消息给组件C。
```js
  Vue.component('C', {
    template: `
      <div>
        <input type="text" v-model="$attrs.messagec" @input="passsCData($attrs.messagec)">
      </div>
    `,
    methods: {
      passCData(val) {
        // 触发父组件A中的事件
        this.$emit('getData', val)
      }
    }
  })
  Vue.component('B', {
    data() {
      return {
        mymessage: this.message
      }
    },
    template: `
      <div>
        <input type="text" v-model="mymessage" @input="passData(mymessage)">
        <!-- C组件中能直接触发getCData的原因在于 B组件调用C组件时 使用 v-on 绑定了$listeners 属性 -->
        <!-- 通过v-bind 绑定$attrs属性，C组件可以直接获取到A组件中传递下来的props（除了B组件中props声明的） -->
        <C v-bind="$attrs" v-on="$listeners"></C>
      </div>
    `,
    props: ['message'], // 得到父组件传递过来的数据
    methods: {
      passData(val) {
        // 触发父组件中的事件
        this.$emit('getChildData', val)
      }
    }
  })
  Vue.component('A', {
    template: `
      <div>
        <p>this is parent component</p>
        <B :messagec="messagec" :message="message" v-on:getCData="getCData" v-on:getChildData="getChildData(message)></B>
      </div>
    `,
    data() {
      return {
        message: 'hello',
        messagec: 'hello c' // 传递给c组建的数据
      }
    },
    methods: {
      getChildData(val) {
        console.log('这是来自B组件的数据')
      },
      // 执行C子组件触发的事件
      getCData(val) {
        console.log("这是来自C组件的数据："+ val)
      }
    }
  })
  var app = new Vue({
    el: '#app',
    template: `
      <div>
        <A></A>
      </div>
    `
  })
```

3. 中央事件总线
上面两种方式处理的都是父子组件之间的数据传递，而如果两个组件不是父子关系呢？这种情况下可以使用中央事件总线的方式。新建一个Vue事件bus对象，然后通过bus.emit触发事件，bus.on监听触发的事件。
```js
  Vue.component('brother1', {
    data(){
      return {
        mymessage: 'hello brother1'
      }
    },
    template: `
      <div>
        <p>this is brother1 component</p>
        <input type="text" v-model="mymessage" @input="passData(mymessage)">
      </div>
    `,
    methods: {
      passData(val) {
        // 触发全局事件globalEvent
        bus.$emit('globalEvent', val)
      }
    }
  })

  Vue.component('brother2', {
    template: `
      <div>
        <p>this is brother2 component</p>
        <p>brother1传递过来的数据:{{brothermessage}}</p>
      </div>
    `,
    data() {
      return {
        mymessage: 'hello borther2',
        brothermessage:''
      }
    },
    mounted(){
      // 绑定全局事件globalEvent
      bus.$on('globalEvent', val => {
        this.brothermessage = val
      })
    }
  })

  // 中央事件总线
  var bus = new Vue();
  var app = new Vue({
    el: '#app',
    template: `
      <div>
        <brother1></brother1>
        <brother2></brother2>
      </div>
    `
  })
```
4. provide和inject
父组件中通过provide来提供变量，然后在子组件中通过inject来注入变量。不论子组件有多深，只要调用了inject那么就可以注入provide中的数据。而不是局限于只能从当前父组件的prop属性来获取数据，只要在父组件的生命中周期内，子组件都可以调用。
```js
  Vue.component('child', {
    inject:['for'], // 得到父组件传递过来的数据
    data(){
      return {
        mymessage: this.for
      }
    },
    template: `
      <div>
        <input type="tet" v-model="mymessage">
      </div>
    `,
  })

  Vue.component('parent', {
    template: `
      <div>
        <p>this is parent component</p>
        <child></child>
      </div>
    `,
    provide: {
      for: 'test'
    },
    data(){
      return {
        message: 'hello'
      }
    }
  })
  var app = new Vue({
    el: '#app',
    template: `
      <div>
       <parent></parent>
      </div>
    `
  })
```

5. v-model
父组件通过v-model传递值给子组件时，会自动传递一个value的prop属性，在子组件中通过this.$emit('input',val)自动修改v-model绑定的值
```js
  Vue.component('child', {
    props: {
      value: String, // v-model会自动传递一个字段为value的prop属性
    },
    data() {
      return {
        mymessage: this.value
      }
    },
    methods: {
      changeValue() {
        this.$emit('input', this.mymessage); // 通过如此调用可以改变父组件上v-model绑定的值
      }
    },
    template: `
      <div>
        <input type="text" v-model="mymessage" @change="changeValue">
      </div>
    `
  })

  Vue.component('parent', {
    template: `
      <div>
        <p>this is parent component</p>
        <p>{{message}}</p>
        <child v-model="message"></child>
      </div>
    `,
    data() {
      return {
        message: 'hello'
      }
    }
  })

  var app = new Vue({
    el: '#app',
    template: `
      <div>
        <parent></parent>
      </div>
    `
  })
```

6. parent 和 children
```js
  Vue.component('child', {
    props: {
      value: String, // v-model会自动传递一个字段为value的props属性
    },
    data() {
      return {
        mymessage: this.value
      }
    },
    methods: {
      changeValue() {
        this.$parent.message = this.mymessage; // 通过如此调用可以改变父组件的值
      }
    },
    template: `
      <di>
        <input type="text" v-model="mymessage" @change="changeValue">
      </di>
    `
  })

  Vue.component('parent', {
    template: `
      <div>
        <p>this is parent component</p>
        <button @click="changeChildValue">test</button>
        <child></child>
      </div>
    `,
    methods: {
      changeChildValue() {
        this.$children[0].mymessage = 'hello';
      }
    },
    data() {
      return {
        message: 'hello'
      }
    }
  })
  var app = new Vue({
    el: '#app',
    template: `
      <div>
        <parent></parent>
      </div>
    `
  })
```

7. boradcast和dispatch
vue 1.0中提供了这种方式，但vue 2.0中没有，但很多开源软件都封装了这种方式，比如minui、elememt ui和iview等。
比如如下代码，一般都作为一个mixins去使用，broadcast是想特定的父组件，触发事件，dispatch是想特定的子组件触发事件，本质上这种方式还是on和on和emit的封装，但在一些基础组件中却很实用。
```js
  function broadcast(componentName, eventName,    params) {
    this.$children.forEach(child => {
      var name = child.$options.componentName;
      if (name === componentName) {
        child.$emit.apply(child, [eventName].concat(params));
      } else {
        broadcast.apply(child, [componentName, eventName].concat(params));
      };
    })
  }

  export default {
    methods: {
      dispatch(componentName, eventName, params) {
        var parent = this.$parent;
        var name = parent.$options.componentName;
        while (parent && (!name || name !== componentName)) {
          parent = parent.$parent;
          if (parent) {
            name = parent.$options.componentName;
          }
          if (parent) {
            parent.$emit.apply(parent, [eventName].concat(params));
          }
        },
        broadcast(componentName, eventName, params) {
          broadcast.call(this, componentName, eventName, params)
        }
      }
    }
  }
```

8. vuex处理组件之间的数据交互
如果业务逻辑复杂，很多组件之间需要同时处理一些公共的数据，这个时候才有上面这一些方法可能不利于项目的维护，vuex的做法就是讲这一些公共的数据抽离出来，然后其他组件就可以对这个公共数据进行读写操作，这样达到了解耦的目的。
