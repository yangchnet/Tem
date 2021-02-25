---
title: Vue中的指令介绍
date: '2021-01-22T09:27:21.000Z'
draft: false
---

# Vue中的指令介绍

## 指令

* 解释：指令 \(Directives\) 是带有 v- 前缀的特殊属性
* 作用：当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM

### **v-text**

解释：更新元素的 textContent

```markup
<h1 v-text="msg"></h1>
```

### **v-html**

解释：更新元素的 innerHTML

```markup
<h1 v-html="msg"></h1>
```

### **v-bind**

作用：当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM.响应式地更新 HTML attribute： 语法：`v-bind:title="msg"` 简写：`:title="msg"`

```markup
<!-- 完整语法 -->
<a v-bind:href="url"></a>
<!-- 缩写 -->
<a :href="url"></a>
<script>
    // 2 创建 Vue 的实例对象
    var vm = new Vue({
      // el 用来指定vue挂载到页面中的元素，值是：选择器
      // 理解：用来指定vue管理的HTML区域
      el: '#app',
      // 数据对象，用来给视图中提供数据的
      data: {
        url: 'http://www.baidu.com'
      }
    })
  </script>
```

### **v-on**

作用：绑定事件 语法：`v-on:click="say" or v-on:click="say('参数', $event)"` 简写：`@click="say"` 说明：绑定的事件从methods中获取

```markup
<!-- 完整语法 -->
<a v-on:click="doSomething"></a>
<!-- 缩写 -->
<a @click="doSomething"></a>
<!-- 方法传参 -->
<a @click="doSomething（“123”）"></a>

 <script>
    // 2 创建 Vue 的实例对象
    var vm = new Vue({
      el: '#app',
      // methods属性用来给vue实例提供方法（事件）
      methods: {
        doSomething: function(str) {
          //接受参数，并输出
          console.log(str);
        }
      }
    })
  </script>
```

### **事件修饰符**

* .stop 阻止冒泡，调用 event.stopPropagation\(\)
* .prevent 阻止默认事件，调用 event.preventDefault\(\)
* .capture 添加事件侦听器时使用事件捕获模式
* .self 只当事件在该元素本身（比如不是子元素）触发时触发回调
* .once 事件只触发一次

### **v-model**

作用：在表单元素上创建双向数据绑定 说明：监听用户的输入事件以更新数据

```markup
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```

### **v-for**

作用：基于源数据多次渲染元素或模板块

```markup
<!-- 1 基础用法 -->
<div v-for="item in items">
  {{ item.text }}
</div>

<!-- item 为当前项，index 为索引 -->
<p v-for="(item, index) in list">{{item}} -- {{index}}</p>
<!-- item 为值，key 为键，index 为索引 -->
<p v-for="(item, key, index) in obj">{{item}} -- {{key}}</p>
<p v-for="item in 10">{{item}}</p>
```

### **key属性**

推荐：使用 v-for 的时候提供 key 属性，以获得性能提升。 说明：使用 key，VUE会基于 key 的变化重新排列元素顺序，并且会移除 key 不存在的元素。

```markup
<div v-for="item in items" :key="item.id">
  <!-- 内容 -->
</div>
```

### **样式处理 class和style**

说明：这两个都是HTML元素的属性，使用v-bind，只需要通过表达式计算出字符串结果即可 表达式的类型：字符串、数组、对象 语法：

```markup
<!-- 1 -->
<div v-bind:class="{ active: true }"></div> ===>
<div class="active"></div>

<!-- 2 -->
<div :class="['active', 'text-danger']"></div> ===>
<div class="active text-danger"></div>

<!-- 3 -->
<div v-bind:class="[{ active: true }, errorClass]"></div> ===>
<div class="active text-danger"></div>


--- style ---
<!-- 1 -->
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
<!-- 2 将多个 样式对象 应用到一个元素上-->
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

### **v-if 和 v-show\***

* 条件渲染
* v-if：根据表达式的值的真假条件，销毁或重建元素
* v-show：根据表达式之真假值，切换元素的 display CSS 属性

### **提升用户体验：v-cloak**

这个指令保持在元素上直到关联实例结束编译。和 CSS 规则如`[v-cloak] { display: none }` 一起用时，这个指令可以隐藏未编译的 `Mustache` 标签直到实例准备完毕。 防止刷新页面，网速慢的情况下出现`{{ message }}`等数据格式

```markup
<div v-cloak>
  {{ message }}
</div>
```

### **提升性能：v-pre**

说明：跳过这个元素和它的子元素的编译过程。可以用来显示原始 Mustache 标签。跳过大量没有指令的节点会加快编译。

```markup
<span vpre>{{ this will not be compiled }}</span>
```

### **提升性能：v-once**

说明：只渲染元素和组件一次。随后的重新渲染，元素/组件及其所有的子节点将被视为静态内容并跳过。这可以用于优化更新性能。

```markup
<span v-once>This will never change: {{msg}}</span>
```

