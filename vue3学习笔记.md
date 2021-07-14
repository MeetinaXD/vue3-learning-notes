## Vue 3学习笔记
Last edit on `Jul 14, 2021`.
By `MeetinaXD`.
Visit my blog: [MeetinaXD' blog](http://meetinaxd.ltiex.com)
难免有遗漏错误之处，如有疑惑请及时查阅官方文档。

---
## 项目起步
需要的环境:
- @vue/cli
- @vite
- node
- npm / yarn

### 开始之前
需要自行安装node环境（这个过程会安装好npm）

### 安装开发工具
**vue**
`npm install -g @vue/cli create-vite-app`
**vite**
`npm init @vitejs/app`
or
`create-vite-app vue3-demo`
`cva vue3-demo`
上面两个命令等价

## vue3的新增内容
 - 组合式API `composition-api`
 - 异步组件 `Suspend`
 - 瞬移标签 `Teleport`
 - 组件的触发事件选项`emits`
 - 组件支持多根节点（片段）
 - css变量支持绑定组件数据（style内的`v-bind()`）
 - 语法糖`script setup`标签

详情见：[值得注意的新特性 - vue官网迁移指南](https://v3.cn.vuejs.org/guide/migration/introduction.html#%E6%A6%82%E8%A7%88)

## vue2和vue3的差异
 - ⚠️ `filters`选项被废除
 - ⚠️ 事件的`.native`修饰符被废除
 - 全局函数`set/$set`和`delete/$delete`被废除
 - ⚠️ 父组件不再支持`$on()`方法接收事件，请使用事件传入
 - 新的`v-model`，不再需要`v-bind.sync`
 - ⚠️ 组件的`emit`事件都需要在`emits`事件中声明（如同props）
 - `$attrs`变为setup的第二个参数内的属性，且包含`class`和`style`（style会是一个对象）
 - `mixin`已变成浅合并（只合并根级属性且以`data`为先）
 - `data`必须是一个函数
 - `props`选项内不能再访问上下文
 - 内置的一些transition效果名称被更改
 - ⚠️ 不再支持基于vue实例中`$emit`实现的`eventBus`，请使用[mitt](https://github.com/developit/mitt)

详情见：[非兼容的变更 - vue官网迁移指南](https://v3.cn.vuejs.org/guide/migration/introduction.html#%E9%9D%9E%E5%85%BC%E5%AE%B9%E7%9A%84%E5%8F%98%E6%9B%B4)

## 代码起步
文档示例使用`Typescript`开发语言进行开发。

### ♻️ 生命周期
在vue3的`composition-api`中，可以从vue引入并使用以下生命周期钩子：
- setup
- onBeforeMounted
- onMounted
- onBeforeUpdate
- onUpdate (更新之后触发)
- onBeforeUnmounted
- onUnmounted
- onActivated (仅`keep-alive`标签内的组件可用)
- onDeactivated (仅`keep-alive`标签内的组件可用)
- onErrorCaptured (捕获当前组件内产生的所有错误)

#### onErrorCaptured(callback(e: Error): Boolean)
⚠️ 在当前页面发生错误时，钩子执行回调函数并传入错误。
回调函数需要返回一个`Boolean`值，当值为`true`时，代表向上传递错误；当值为`false`时，不向上传递错误。

> 需要注意的是，在option中定义生命周期的用法仍然适用

详细用法见: [Lifecycle Hooks](https://v3.vuejs.org/guide/composition-api-lifecycle-hooks.html)

### ⌛ Suspend异步加载组件
使用异步组件时，显示加载态和加载完毕态

####父组件
**模版**
```HTML
<Suspense>
  <!-- 加载完毕后显示的东西 -->
  <template #default>
    <Async />
  </template>

  <!-- 正在加载显示的东西 -->
  <template #fallback>
    <h1>Loading...</h1>
  </template>
</Suspense>
```
**代码**
```typescript
import Async form '@/component/AsyncLoad.vue'

export default {
  components :{
    Async
  },
  setup(){
    return { }
  }
}
```

#### 子组件
**模版**
```HTML
<template>
  <span>{{result}}</span>
</template>
```

**代码**
```typescript
import { defineComponent } form 'vue'
export default defineComponent({
  setup: () => new Promise(async (resolve, reject) => {
    const data = await (axios.get('xxxx')).data

    // resolve其实就是setup的返回
    resolve({ result: data })
  })
})
```
当然也可以这样：
```typescript
import { defineComponent } form 'vue'
export default defineComponent({
  setup: async () => {
    const data = await (axios.get('xxxx')).data

    // resolve其实就是setup的返回
    resolve({ result: data })
  })
})
```

> 以上示例将会在axios的get请求成功后，页面显示的Loading...变为请求返回的内容。

### ⚠️ setup函数与Volar
在vue3的`composition-api`中，`setup`是一个先于组件渲染执行的函数。
换言之，`setup`先于`onBeforeMounted`执行。
一般情况下，`setup`函数形式如下：
```html
<script lang="ts">
import { ref } from 'vue'
export default {
  name: 'something',
  setup(){
    const counter = ref(0)
    const addCounter = ():void => {
      counter.value++
    }
    return { counter, addCounter }
  }
}
</script>
```
上面的例子在`setup`函数中暴露了一个名为`counter`的响应式变量，以及一个使其数值加一的方法。
与vue2时代中常用的`option-api`一样，`counter`以及`addCounter`都能够在模版中直接使用。

#### Volar
`Volar`是vue3的一个语法糖，可以在html标签内直接定义setup函数。
要使用volar，需要在`script`标签内添加`setup`属性
```html
<script lang="ts" setup>
import { ref } from 'vue'
const counter = ref(0)
const addCounter = ():void => {
  counter.value++
}
</script>
```

这个示例的效果与上面的示例一样。

#### 不同之处
volar将暴露所有在标签内的对象，setup可根据需要自行控制。

### ⚠️ `watch`与`watchEffect`
> 需要自行引入

`watch与watchEffect`与`watch`类似，但不需要显式指定监测的变量，而且不需要等到数据变化后执行。在第一次数据赋值时，`watchEffect`也会被执行。

下面使用一个例子来分别说明`watch`以及`watchEffect`的区别。
假如我们需要根据一个id值来从api获取用户的数据，下面实现这个接口:
```ts
// @/api/user.ts
interface UserInfo {
  id: number
  name: string
  imageURI: string
}

const userlist: { [id: number]: UserInfo } = {
  1: { id: 1, name: 'Tenma', imageURI: 'img.com/img1' },
  2: { id: 2, name: 'Jeans', imageURI: 'img.com/img2' },
  3: { id: 3, name: 'Tom', imageURI: 'img.com/img3' },
}

const getUserInfo = (
  id: number
): Promise<UserInfo> => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(userlist[id])
    }, 1000)
  })
}

export default getUserInfo;
```

现在，接口已经实现, 下面来实现自动根据id的变化获取用户数据并输出。
#### 在watch中实现
```ts
import { ref, watch } from 'vue'
import getUserInfo from '@/apis/user'
export default {
  setup(){
    const id = ref(1)
    const getInfo = async (id:number) => {
      const userinfo: UserInfo = await getUserInfo(nowId)
      console.log(userinfo)
    }
    // 初始化时获取用户数据
    getInfo(id)
    watch(id, nowId => getInfo(nowId))
    // 暴露方法
    return { id, getInfo }
  }
}
```

下面来看看使用`watchEffect`如何实现同样的效果
#### 在watchEffect中实现
```ts
import { ref, watchEffect } from 'vue'
import getUserInfo from '@/apis/user'
export default {
  setup(){
    const id = ref(0)
    const getInfo = async (id:number) => {
      const userinfo: UserInfo = await getUserInfo(nowId)
      console.log(userinfo)
    }
    // 并不需要指定检测变量及手动在初始化时获取数据
    watchEffect(() => getInfo(id))
    // 暴露方法
    return { id, getInfo }
  }
}
```
现在，以上两个应该都工作正常，并能在初始加载及id变动时自动加载用户数据并输出。

#### ⚠️ 进一步的问题
考虑实际情况，api的获取并不是在相同的时间内完成的，视乎网络环境而定。同时，**id的变化也许会在api的获取中产生**。
让我们来改写api来贴合实际情况：
```ts
// time代表api获取所需要的时间
// 返回值为一个数组，第一个是包含用户信息的promise对象，第二个是cancel(reject)方法
const getUserInfo = (
  id: number,
  time: number = 1000
): [Promise<UserInfo>, () => void] => {
  let _cancel: (reason: any) => void
  const _promise = new Promise((resolve, reject) => {
    _cancel = reject
    setTimeout(() => {
      resolve(userlist[id])
    }, time)
  })

  // _cancel实际上是reject
  return [_promise, () => _cancel("abort")]
}
```

下面让我们尝试在watch中模拟这一情况
##### 🎯 watch
``` ts
// ...
const id = ref(0)
// 这个函数在获取id更大的用户信息时，更加缓慢
const getInfo = async (id:number) => {
  const [promise, cancel] = getUserInfo(nowId, nowId * 1000)
  const userinfo = await promise
  console.log(userinfo)
}
watch(id, nowId => getInfo(nowId))
// 模拟id被快速地切换了
id.value = 2
id.value = 1
```
以上例子模拟了用户快速切换id，而且由于id较大的用户所需的调用时间较长，**较先变化**的id的用户信息晚于**较后变化**的id的用户信息先得到。

因此，输出将是这样的：
``` json
{ id: 1, name: 'Tom', imageURI: 'img.com/img3' }
{ id: 2, name: 'Jeans', imageURI: 'img.com/img2' }
```
额外地，id的变化通常认为只有最后一个是有意义的，由此实际上我们只需要`id为1`的用户信息。

为了解决这些问题，我们可以使用`watchEffect`中传入的`注册失效回调函数(onInvalidate)`。

##### 🎯 watchEffect
``` ts
// ...
const id = ref(0)
// 这个函数在获取id更大的用户信息时，更加缓慢
const getInfo = async (id: number, onInvalidate: Function) => {
  onInvalidate(() => {
    cancel && cancel()
  })
  const [promise, cancel] = getUserInfo(nowId, nowId * 1000)
  const userinfo = await promise
  console.log(userinfo)
}
watchEffect(onInvalidate => {
  getInfo(id, onInvalidate)
})
// 模拟id被快速地切换了
id.value = 2
id.value = 1
```
在id快速地由2切换为1时，`onInvalidate`将被触发，并在`getInfo`内执行Promise请求的reject方法，请求将会终止。输出将是这样的：
``` json
{ id: 1, name: 'Tom', imageURI: 'img.com/img3' }
```

#### 🔖 额外部分
如果监听的数据在一个对象的深处，在vue2我们可以使用字符串的形式(如`data.user.name.firstname`)指明被watch的属性，但现在它已经被删除了。

可以通过一个函数返回值或是计算属性的方法来代替。
```ts
watch(() => data.user.name.firstname, newVal => {
  /* do something */
})
```
or
```ts
const firstname = computed(() => data.user.name.firstname)
watch(firstname, newVal => {
  /* do something */
})
```
如果你愿意，甚至可以为computed添加`getter`和`setter`:
```ts
const firstname = computed(() => {
  get: () => data.user.name.firstname
  set: val => data.user.name.firstname = val
})
```

#### 其他关于`watch`或`watchEffect`的更改
对数组使用watch：[Watch on Arrays](https://v3.cn.vuejs.org/guide/migration/watch.html#frontmatter-title)
### `emits`属性
在vue2的`option-api`中的`props`对象用于对传入的属性进行类型和默认值的约束。而vue3中的`emits`则是对调用`emit()`时传出数据的约束。
```ts
emits: {
  onUpdate(id: number, name: string){
    if (id < 0 || name.length === 0){
      console.error("id must over 0 or name cannot be empty")
      return false
    }
    return true
  }
},
setup(props, content){
  // ...
  const { emit } = content
  emit("onUpdate", 0, "Tom")
  emit("onUpdate", -1, "Tom") // 报错
}
```

> vue3中emit已经不再使用`this.$emit`，而是在setup的第二个参数`content`中获取

### 全局方法挂载
在vue2中，可以通过以下方式挂载一个方法：
```js
import axios from '@/plugins/axios'
Vue.prototype.$axios = axios
```

在vue3中，要实现类似的挂载需要使用一个新的属性`globalProperties`
``` ts
import axios from '@/plugins/axios'
import { createApp } from 'vue';
const app = createApp({ /* ... */ });
app.config.globalProperties.$axios = axios
```

然后在`setup`内部使用`$axios`（当前App示例）
```ts
setup() {
  const {
    appContext: {
      config: {
        globalProperties: { $axios }
      }
    }
  } = getCurrentInstance()
}
```

### `props`和`attrs`属性
假设我们编写了一个组件，并在外部使用它。
``` html
<my-component
  id="component"
  style="color: red"
  class="my-style"
  name="Tom"
  gender="man"
  :age="12"
  @myevent="e => e + 1"
/>
```
像这样，通常情况下我们会认为`name`和`age`是待传入的属性，那么我们应该在`props`选项中定义它。
#### 🎯 `props`
``` ts
{
  props: {
    name: {
      type: string,
      default: ""
    },
    age: {
      type: number,
      default: 0
    }
  }
}
```
注意到我们还传入了`class`和`style`，以及额外的`gender`属性吗？
它在渲染后看上去是这样的：
```html
<div id="component" class="my-style" style="color: red" gender="man">
  <!--  -->
</div>
```
这是因为**在默认情况下**，`props`属性中已定义的属性不会在渲染后的节点中显示。而且，未声明的属性不能在props中使用。
```ts
props.gender //这是非法的
```

#### 🎯 `attrs`
要使用未声明的属性，可以使用`attrs`。
实际上，任何传入的**非`props`声明的属性**都会出现在`attrs`中。
> ⚠️ 甚至，`attrs`中还包含传入的事件。

注意到我们在上面的组件中还传入了`@myevent`，让我们看看`attrs`中有什么东西。
```ts
{
  //
  setup(props, { attrs }){
    console.log(attrs)
  }
}
```
以上代码将输出
```ts
Proxy {id: "component", style: {color: "red"}, class: "mystyle", gender: "man", onMyevent: ƒ}
```
> vue2行为：`attrs`的用法是`this.$attrs`

#### 🔖 差异总结
 - `props`需要先声明才能使用，而`attrs`不需要
 - `props`不包含事件，而`attrs`包含 (⚠️ 但在`props`定义的事件依然会出现在`props`中而不是`attrs`中)
 - `props`允许传入非string类型的值，而`attrs`只能传入string(但style传入一个`object`)

#### 🔖 其他需要注意的特性
 - 在`props`中定义的所有值，`attrs`中都不会出现(⚠️ 包括`class`、`style`和`id`)
 - `attrs`可以在传入的`content`参数中解构获取

### `inheritAttrs`属性（禁用 Attribute 继承）
> ⚠️ **警告**
设置该属性为`false`时，请确保样式仍然符合预期

该内容可参考：[禁用 Attribute 继承](https://v3.cn.vuejs.org/guide/component-attrs.html#%E7%A6%81%E7%94%A8-attribute-%E7%BB%A7%E6%89%BF)

如果你不希望组件的根元素继承特性，你可以在组件的选项中设置 `inheritAttrs: false`（默认为true）
如果设置为`false`，这个时候就可以自由地决定`attrs`应该添加到哪个元素中。
```html
<template>
  <div>
    <!-- 非根元素 -->
    <div v-bind="$attrs">
    </div>
  </div>
</template>
```
上述代码会被编译为
```html
<!-- 现在属性不再被添加到根元素 -->
<div>
  <div class="my-style" style="color: red" gender="man">
    <!--  -->
  </div>
</div>
```
> 在模板中使用`attrs`的方法是`$attrs`

> vue2行为：`class`和`style` 不属于`attrs`，仍然会应用到组件的根元素

### ⚠️ slot插槽
> 这并不是vue3的新内容

关于`slot插槽`详细的介绍见：[组件插槽 - vuejs.org](https://cn.vuejs.org/v2/guide/components-slots.html)

#### 🎯 具名插槽
当需要使用多个插槽的时候，可以在slot内提供一个名为`name`的属性
如对于一个名为`foo`的组件：
```html
<div>
  <header>
    <slot name="header" />
  </header>

  <main>
    <slot />
  </main>

  <footer>
    <slot name="footer" />
  </footer>
</div>
```
那么在父组件中使用时：
```html
<div>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</div>
```
同时，类似于`v-bind`和`v-on`，`v-slot`也有自己的缩写: `#`
于是，上述代码可以改写为：
```html
<div>
  <template #header>
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template #footer>
    <p>Here's some contact info</p>
  </template>
</div>
```

#### 🎯 作用域插槽
考虑下面的情况，一个名为`user-info`的组件具有以下的内容：
```html
<template>
  <div>
    <slot>{{ user.lastName }}</slot>
  </div>
</template>
```
它也许会拥有这样的数据：
```ts
{
  setup(){
    const data = reactive({
      user: {
        firstName: "Swift",
        lastName: "Taylor"
      }
    })

    return { ...toRefs(user) }
  }
}
```
在父组件使用的时候，我们可能需要用`firstName`替换掉默认的内容。但在使用时，我们并不能访问到子组件`user-info`中的数据。
因此，我们需要用到`作用域插槽`，来让父组件能够访问到组件内的数据。
让我们来分别改写父子组件。

子组件：
```html
<template>
  <div>
    <!-- 这里的data指数据将在父组件中暴露的名称 -->
    <slot v-bind:data="user">{{ user.lastName }}</slot>
    <!-- 或者可以这么用 -->
    <slot :data="user">{{ user.lastName }}</slot>
  </div>
</template>
```

父组件：
```html
<user-info>
  <!-- 从默认插槽中获得数据，并挂载到名为userProp的变量内 -->
  <template v-slot:default="userProp">
    {{ userProp.data.firstName }}
  </template>
  <!-- 使用缩写和解构语法 -->
  <template #="{ data }">
    {{ data.firstName }}
  </template>
</user-info>
```
如果进一步考虑`独占默认插槽`的缩写语法，父组件还可以写成：
```html
<user-info #="{ data }">
    {{ data.firstName }}
</user-info>
```
这是一个很棒的特性，可以减少不必要的嵌套，详见[独占默认插槽](#独占默认插槽)

上述实例将渲染出：
```html
<div>
  Swift
</div>
```
##### 🔖 更详细的解释
在子组件向父组件提供数据时，实际上的用法就如同向元素提供`props`一般。使用`v-bind:暴露出去提供给父组件的数据名="子组件作用域内的数据名"`类似的语法。
如`v-bind:mydata="user"`或是`:mydata="user"`。

而在父组件使用这些数据时，需要在`template`或者`组件标签内`（仅当只有一个插槽的时候可以这么做）使用`v-slot:插槽的名称="接收数据到哪里（一个变量的名称）"`或是`#插槽的名称="接收数据到哪里"`类似的语法。
如`v-slot:default="nameProp"`或是`#="nameProp"`

**⚠️ 从子组件内引入的数据将被包装在一个对象内放入到父组件所指定的变量名内。**

如上述例子，子组件向父组件提供了`mydata`（实际上是`user`的数据），同时父组件将其放入到了`nameProp`内。

`mydata`会拥有的数据：
```ts
{
  firstName: "Swift",
  lastName: "Taylor"
}
```
`nameProp`会拥有的数据：
```ts
{
  mydata: {
    firstName: "Swift",
    lastName: "Taylor"
  }
}
```
这也是为什么可以使用解构语法的原因。

#### 🎯 独占默认插槽
假如组件仅有一个未命名的`默认插槽`，那么引用组件数据时，可以将`v-slot`指令直接放在组件标签内，就像这样：
```html
<user-info v-slot:default="{ data }">
<!-- 或者是 -->
<user-info #="{ data }">
```
**⚠️ 只要出现多个插槽，请始终为所有的插槽使用完整的基于 `<template>`的语法**

### 组件的多根节点（片段）
vue3中支持在组件`template`内放置多个根节点
```html
<!-- Layout.vue -->
<template>
  <header>...</header>
  <main v-bind="$attrs">...</main>
  <footer>...</footer>
</template>
```
你也许已经留意到了，`main`标签被绑定了`$attrs`。
> **⚠️ 在使用多个根节点时，组件不具有自动 attribute 回退行为。请始终显式地定义attribute应该绑定到哪个根节点内，如果未显式绑定 $attrs，将发出运行时警告。**

## ✨ 项目配置
### `@绝对目录`的配置
`@绝对目录`在vue2似乎是默认已经配置好的，而在vue3中并不能直接使用，需要自行进行一些配置。

#### 🎯 在`tsconfig.json`中配置
```json
// 未定义baseUrl时不允许使用paths
"baseUrl": "./",
"paths": {
  "@/*": ["src/*"]
}
```
⭐ 关于tsconfig.json的设置，还有更多的玩法。如为特定模块或组件设置别名。
见：[路径设置 - typescript](https://www.typescriptlang.org/zh/tsconfig#paths)

#### 🎯 在`vite.config.ts`中配置
```ts
// 如果报错：找不到模块 “path” 或其相对应的类型声明，则需要安装一下类型提示
// npm install @types/node --save-dev
import path from "path"
export default defineConfig({
  /* ... */
  base: "./",
  resolve: {
    alias: {
      // map '@' to './src'
      "@": path.resolve(__dirname, "./src")
    }
  }
})
```
这里参考了：[如何进行vue的@别名设置](https://www.5axxw.com/questions/content/s0uf0x)

#### 🎯 配合插件使用
> 使用编辑器vscode

虽然上面完成了对`@绝对目录`的定义，但在类似于vscode的编辑器中编写时，输入'@'并不会出现目录的语法提示。

为了产生语法提示，可以安装插件`Path Intellisense`。

 - 在设置中找到`扩展 - Path Intellisense`
 - 找到`Mappings`自定义项，点击`在settings.json中编辑`
 - 加入以下配置
```json
"path-intellisense.mappings": {
  "@": "${workspaceRoot}/src"
}
```

### `proxy`本地代理
本地代理需要在`vite.config.ts`中设置。
相关配置字段在`server.proxy`中，使用了[http-proxy](https://github.com/http-party/node-http-proxy)模块。
```ts
{
  // 字符串简写写法
  '/foo': 'http://localhost:4567/foo',
  // 选项写法
  '/api': {
    target: 'http://jsonplaceholder.typicode.com',
    changeOrigin: true,
    rewrite: (path) => path.replace(/^\/api/, '')
  },
  // 正则表达式写法
  '^/fallback/.*': {
    target: 'http://jsonplaceholder.typicode.com',
    changeOrigin: true,
    rewrite: (path) => path.replace(/^\/fallback/, '')
  },
  // 使用 proxy 实例
  '/api': {
    target: 'http://jsonplaceholder.typicode.com',
    changeOrigin: true,
    configure: (proxy, options) => {
      // proxy 是 'http-proxy' 的实例
    },
  }
}
```
> ⚠️ **https代理**
如果你需要代理到https服务器，需要提供目标地址的证书才能继续。
详见：[Using Https](https://github.com/http-party/node-http-proxy#using-https)

如果对正则表达式的使用不太明白，可以参考：[nginx正则表达式](https://www.cnblogs.com/bethal/p/5514557.html)
配置来源：[Server Proxy - Vite 官方文档](https://cn.vitejs.dev/config/#server-proxy)

## ✨ TypeScript笔记
### 交叉类型和联合类型
#### 语法
- 交叉类型: `Type1 & Type2`
- 联合类型: `Type1 | Type2`

#### 交叉类型
交叉类型取所有类型的**并集**，拥有所有类型的所有属性，下面使用一个官方的例子解释。
```ts
function extend<T, U>(first: T, second: U): T & U {
  // result 是要返回结果，类型断言为 T & U
  let result = {} as T & U
  for(let id in first){
    // 不能将类型“T”分配给类型“T & U”，所以需使用断言
    result[id] = first[id] as any
  }
  for(let id in second){
    if(!result.hasOwnProperty(id)){
      // 不能将类型“U”分配给类型“T & U”，同样需要断言
      result[id] = second[id] as any
    }
  }

  // 返回结果，类型是 T & U
  return result
}

class Person{
  constructor(public name: string) {}
}

interface Loggable {
  log(): void
}

class ConsoleLogger implements Loggable {
  log(){
    console.log("do log")
  }
}

// 使用 extend 方法合并两个类的实例，返回的是交叉类型，所以可以访问 name 和 log()
let tom = extend(new Person('Tom'), new ConsoleLogger())
let n = tom.name //Tom
tom.log() // 输出do log
```
例子中的结果可以看到，交叉类型取的是并集，拥有两个类型成员的所有属性。

#### 联合类型
交叉类型取所有类型的**交集**，只能选择多个属性中的其中一个，下面还是使用一个官方的例子解释。
```ts
function PadLeft(value: string, padding: any){
    // 如果是 number 类型，则在 value 前填充对用空格
    if(typeof padding === 'number'){
        return Array(padding + 1).join(' ') + value
    }
    // 如果是 string 类型，则直接拼接 value 和 padding
    if(typeof padding === 'string'){
        return padding + value
    }

    // 如果不是 string 和 number，抛出错误
    throw new Error(`Expected string or number, got '${padding}'.`)
}

PadLeft("hello world", '   ') // '   hello world'
PadLeft("hello world", 4)     // '    hello world'
```
看上去一切都好，但如果传入了一个既不是`string`也不是`number`类型的数据时。但错误并不是在编写时抛出的，而是运行时报错的。
```ts
padLeft("Hello world", true); // 编译阶段通过，运行时报错
```
要在编译阶段发现问题，这时可以使用联合类型来替代 `any`类型。
```ts
function PadLeft(value: string, padding: string | number){
    // ...
}

PadLeft("hello world", true) // 编辑器报错，true不能赋给类型string | number的参数。
```

**⚠️ 需要注意：** 联合类型只能访问所有类型中共有的属性。
```ts
function getLength(value: string | number) :number{
  return value.length // 属性'length'不存在于'number'类型
}
```

### 索引类型
索引类型规定了类型中未定义的成员的`属性名类型`和`属性类型`。
举个例子：
```ts
interface Person {
  name: String;
  age: Number;
  phone: String;
}

const p: Person = {
  name: '9527',
  age: 18,
  phone: '18000000000'
}
```
如果这个时候，我们需要添加成员到Person中
```ts
p.info = 'xxxxx' //Property 'info' does not exist on type 'People'.
```
由于`Person`中没有`info`成员，因此不能通过编译。
此时可以使用**索引签名（Index Signature）**。
```ts
interface Person {
  name: string;
  age: number;
  status: boolean; // <-- error! boolean not assignable to string | number
  [key: string]: string | number;
}
```
此时，你应该可以获得一个可以自由添加成员的`Person类型`对象，但编译器仍会给出一个报错，原因在`status: boolean`。
⚠️ 因为，**索引签名**定义了**所有属性**及其返回值类型，因此使用时请确保所有的**值类型**与索引签名相同。
> **外文回答**
If you add an `index signature`, it **must not** conflict with any other properties.
The index signature `[k: string]: string | number` means "if you read a property from `Person` with any key of type `string`, you will get a value of type `string | number`." The `name` property is compatible, because the key `"name"` is a `string`, and the value type `string` is assignable to `string | number`. The age property is compatible, because the key `"age"` is a `number`, and the value type `number` is assignable to `string | number`. But `status` is in error. The key `"status"` is a `boolean`, but it violates the index signature; `boolean` is not assignable to `string | number`.
You cannot use index signatures like the above to say "well, the property at key `"status"` is a `boolean` but every other `string-keyed` property has a value of type `string | number`. It would be nice to have a way to say that, (see [microsoft/TypeScript#17687](https://github.com/microsoft/TypeScript/issues/17867) for a request for this) but index signatures don't work that way.

#### 将字面量联合类型用作索引
索引签名可以通过使用`映射类型`，将字面量联合类型中的成员用作索引名。
> An index signature can require that index strings be members of a union of literal strings by using Mapped Types

##### 映射类型
使用`keyof`关键字，我们可以创建一个所谓的`映射类型`，它将原始类型的所有属性映射到一个新的类型。
> In combination with keyof we can use it to create a so called mapped type, which re-maps all properties of the original type.

```ts
type Index = 'a' | 'b' | 'c'
type FromIndex = { [k in Index]?: number }

const good: FromIndex = {b:1, c:2}

// Error:
// Type '{ b: number; c: number; d: number; }' is not assignable to type 'FromIndex'.
// 对象字面量只能使用定义属性，'d'并不在范围内
const bad: FromIndex = {b:1, c:2, d:3};
```
甚至，你可以编写一个工具函数来实现这个功能。
```ts
type TypeFromIndex<K extends string, T> = { [key in K]?: T }

type Index = 'name' | 'gender' | 'phone'
const FromIndex: TypeFromIndex<Index, string> = {
  name: 'Tom',
  gender: 'female'
  // phone: '18000000000'
}
```
#### 其他值得关注的问题
有一种十分罕见的用法，是同时声明`string`和`number`索引签名。
```ts
interface ArrStr {
  [key: string]: string | number; // Must accommodate all members
  [index: number]: string; // Can be a subset of string indexer
  // Just an example member
  length: number;
}
```
这里也提到了一点：`索引签名`对于`string`索引的限制比`number`索引的限制严格得多，这是由于js对象并没有真正意义上的`number`索引导致的。如:`obj[123]`等价于`obj['123']`.
实际上，对象内的`key`只有`string`和`symbol`两种类型。
看一个**Javascript**的例子：
```js
let obj = { message:'Hello' }
let foo = {};

foo[obj] = 'World';

// Here is where you actually stored it!
console.log(foo["[object Object]"]); // World
```
在`v8引擎`中，对象的key在传入时会调用默认的`toString`方法，因此也解释了在对象内`number`就是`string`的原因。

✨ 这里给出了很好的解释：[Multiple indexer in Indexable types in TypeScript](https://stackoverflow.com/questions/65483822/multiple-indexer-in-indexable-types-in-typescript)
#### `keyof`, 索引类型查询操作符
> ⚠️ 需要明确一点，`keyof`不考虑值类型。

`keyof`操作符具有两个作用：**映射`对象类型`为它的成员名称所组成的联合类型**、**映射具有`索引签名`的类型为它的索引签名的类型**
这听起来很拗口，需要先了解`对象类型`和`索引类型`。
##### 对`对象类型`使用
```ts
type Person = {
  name: string;
  age: number;
};
type P = keyof Person;
```
P将得到`"name" | "age"`。这是由Person这个对象类型所具有的成员名称所组成的联合类型。
也就是会得到由所有key组成的联合类型。
> **外文回答：** For any type `T`, `keyof T` is the union of known, public property names of `T`.

##### 对`索引类型`使用
```ts
type A = {
  name: string;
  [p: number]: unknown;
};
type P = keyof A
```
P将得到`"name" | number`。
在这里，key由`'name'`以及`[p: number]`组成，换言之，由`'name'`以及`number`组成。（注意`'name'`在这里是有引号的，是一个字符串）
但，还有一个例子：
```ts
type A = {
  name: string;
  [p: string]: unknown;
};
type P = keyof A
```
P将得到`string | number`。
在这里，key由`'name'`以及`[p: string]`组成，换言之，由`'name'`以及`string`组成。而`'name'`属于`string`类型，因此实际上只存在`string`。

> ⚠️ **为什么是`string | number`？**
This is because JavaScript object keys are always coerced to a string, so `obj[0]` is always the same as `obj["0"]`.
详见：[Keyof Type Operator](https://www.typescriptlang.org/docs/handbook/2/keyof-types.html)

##### ⚠️ `extends keyof`和`in keyof`的区别
- `extends keyof`用于约束参数类型，见**索引访问操作符**
- `in keyof`用于定义索引签名
这是一个使用`in keyof`的例子。
```ts
interface Person {
  age: number;
  name: string;
}

type Optional<T> = {
  [K in keyof T]?: T[K]
};

const person: Optional<Person> = {
  name: "Tobias"
  // 注意这里我没有写'age'属性，这是故意的。
};
```
`keyof`其他用法可以见[映射类型](#映射类型)
该内容参考自：[In TypeScript, what do “extends keyof” and “in keyof” mean?](https://stackoverflow.com/questions/57337598/in-typescript-what-do-extends-keyof-and-in-keyof-mean/57338105)

#### `T[K]`, 索引访问操作符
这是一种使用类型语法来反映表达式语法的方式。
这有些抽象，我们使用一个例子来说明。
例如编写一个获取对象成员的方法：
```ts
type Person = {
  name: string;
  age: number;
}

const p: Person = {
  name: '9527',
  age: 18
}

// 获取单个成员
function getProperty<T, K extends keyof T>(o: T, key: K): T[K] {
    return o[key]; // o[name] is of type T[K]
}

// 获取多个成员
function pick<T, K extends keyof T>(o: T, keys: K[]):T[K][]{
  return keys.map(key => o[key])
}

console.log(getProperty(p, 'name')) // 9527
console.log(pick(p, ['name', 'age'])) // 9527 18
console.log(pick(p, ['name', 'age', 'dog'])) // Error: Argument of type '"dog"' is not assignable to parameter of type 'keyof Person'.
```
其中，`K extends keyof T`使得`K`继承了T的所有成员，在这里是`name`和`age`。
这里出现的`T[K]`，是**函数的返回类型**。函数需要返回对象内的任一成员的值，因此`T[K]`涵盖了所有在对象中出现的值类型。实际上，它代表了`p[key]`，也就是`Person[key]`。
例如，`p['key']`具有类型`Person['name']`。
因此，使用这种方式，不仅可以约束了返回类型只能是对象内出现的值类型。同时，也约束了成员的名称"K"必须出现在`Person`内。

### `infer`类型推断
#### 解释
作用是在类型表达式中，通过extends来声明一个不确定/待推断的变量类型。

#### 通过`ReturnType`理解`infer`
参考Typescript内置的工具类型`ReturnType`，可以尝试写一些代码：
```ts
const add = (x:number, y:number) => x + y
type fn = typeof add
type t1 = ReturnType<typeof add> // type t = number
type t2 = ReturnType<fn> // type t = number
```

- `ReturnType<T>` - 根据**函数返回值**获取类型
```ts
/**
 * Obtain the return type of a function type
 */
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any
```
`infer` 的作用是让 TypeScript 自己推断，并将推断的结果存储到一个类型变量中，`infer` 只能用于 `extends` 语句中。

再来看 ReturnType 的实现：如果 T 满足约束条件`(...args: any) => any`（**即等号左边**）。 并且能够赋值给 `(...args: any) => infer R`，则返回类型为 `R`（**三目表达式**），否则为 `any`类型。

继续看几个例子：
```ts
type T0 = ReturnType<() => string>        // string
type T1 = ReturnType<(s: string) => void> // void
type T2 = ReturnType<<T>() => T>          // unknown
```
##### 代码解释：
分别可以得到 `type T0 = string`， `type T1 = void`， `type T2 = unknown`，只要满足约束条件 `(...args: any) => any`，TypeScript 推断出函数的返回值，并借助 `infer` 关键字将其储存在类型变量 `R` 中，那么最终得到返回类型 `R`。

#### 通过`Parameters`理解`infer`
这也是`Typescript`的内置工具类型
```ts
type T0 = Parameters<() => string>;  // []
type T1 = Parameters<(s: string) => void>;  // [string]
type T2 = Parameters<(<T>(arg: T) => T)>;  // [unknown]
```
- `Parameters<T>`根据**函数参数**获取类型
```ts
/**
 * Obtain the return type of a function arguments
 */
type Parameters<T> = T extends (...args: infer R) => any ? R : any;
```
##### 代码解释：
如果泛型参数`T`能够赋值给`(...args: infer R) => any`（**请忽略`infer R`**），那么TypeScript 推断出函数的返回值，并借助 `infer` 关键字将其储存在类型变量 `R` 中，那么最终得到返回类型 `R`

#### 使用`infer`: 实现元组转联合类型
借助 infer 可以实现元组转联合类型，如：`[string, number] -> string | number`
```ts
type Flatten<T> = T extends Array<infer U> ? U : never

type T0 = [string, number]
type T1 = Flatten<T0> // string | number
```

##### 代码解释：

第 1 行，如果泛型参数 T 满足约束条件 Array<infer U>，那么就返回这个类型变量 U。
第 3 行，元组类型在一定条件下，是可以赋值给数组类型，满足条件:
```ts
type TypeTuple = [string, number]
type TypeArray = Array<string | number>

type B0 = TypeTuple extends TypeArray ? true : false // true
```
第 4 行，就可以得到 `type T1 = string | number`。
简而言之，上述的`infer U`实现了类型推断，并将其储存到变量`U`中，变为`string | number`，与上面的解释相似。

该内容参考自：[TypeScript infer 关键字](https://www.wenjiangs.com/doc/typescript-infer)
> ⚠️ **`infer`属于高级类型用法**
其他用法解释及相关题目见：[【typescript】infer的理解与使用](https://blog.csdn.net/yehuozhili/article/details/108253532)


### 其他~~姿势~~ 知识点
还没来得及归类，但是值得一看。
- [TypeScript简介与文档](https://www.wenjiangs.com/doc/typescript-tsintroduction)
- [TypeScript给拍平后的Object加类型](https://zhuanlan.zhihu.com/p/91144493)
- [TypeScript官方中文文档](https://www.tslang.cn/docs/home.html)
- [TypeScript官方原文文档](https://www.typescriptlang.org/docs/handbook/intro.html)
- [Type和Interface的区别](https://blog.csdn.net/dtbk123/article/details/107013672)
- [✨ never类型的作用](https://www.zhihu.com/question/354601204)