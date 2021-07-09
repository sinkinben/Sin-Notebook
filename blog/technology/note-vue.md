本文章为 Web 开发的学习笔记，前端使用 Vue，后端使用 SpringBoot 框架。

## Vue

### 发送POST请求

```vue
<template>
  <div>
    Username:<input type="text" v-model="loginForm.username" placeholder="请输入用户名"/>
    <br><br>
    Password:<input type="password" v-model="loginForm.password" placeholder="请输入密码"/>
    <br><br>
    <button v-on:click="login">登录</button>
  </div>
</template>

<script>

  export default {
    name: 'Login',
    data() {
      return {
        loginForm: {
          username: '',
          password: ''
        },
        responseResult: []
      }
    },
    methods: {
      login() {
        alert(this.loginForm.username + ' ' + this.loginForm.password);
        this.$axios
          .post('/login', {
            username: this.loginForm.username,
            password: this.loginForm.password
          })
          .then(successResponse => {
            if (successResponse.data.code === 200) {
              window.location.href = 'https://www.baidu.com';
              this.$router.replace({path: '/index'})
            }
          })
          .catch(failResponse => {
          })
      }
    }
  }
</script>
```



### 项目部署相关事项

+ 怎么查看依赖的 module ?

  打开 `package.json`，查看 `dependencies` 和 `devDependencies` 的内容。



## Vue 指令

### v-if , v-else

```vue
<template>
  <div>
    <p v-if="flag === 0"> flag is zero </p>
    <p v-else> flag is one </p>
  </div>
</template>

<script>
  export default {
    name: "Test",
    data() {
      return {
        flag: 1
      }
    }
  }
</script>

<style scoped>

</style>
```

### v-for, v-bind

如果数据为字典类型：

```vue
<script>
  export default {
    name: "Test",
    data() {
      return {
        data: {
          "1": "aaa",
          "2": "bbb",
          "3": "ccc"
        }
      }
    }
  }
</script>
```

Vue 指令：

```vue
<Select style="width: 200px">
    <Option>default</Option>
    <Option v-for="(value, name) in data" v-bind:key="name">{{value}}</Option>
</Select>
```

其中，`name` 分别是 `"1", "2", "3"` 。





## HTML/CSS 基础

+ 行内元素，一个左对齐，一个右对齐

  ```html
  <span style="display: inline; float: left;"><el-checkbox>记住我</el-checkbox></span>
  <span style="display: inline; float: right;"><el-link style="color: dodgerblue;">忘记密码</el-link></span>
  ```

+ 把一个图片或者表格覆盖在一张已有图片上的任意位置

  ```html
  <div style="position:relative;">
      <img src="" width="500" height="500" />
      <div style="position:absolute; left:80px; top:50px; border:#000 solid 1px;">
          <img  src="" width="50" height="50"/>
      </div>
  </div>
  ```
  可以用里面的 `LEFT, TOP` 属性控制他的位置。 `position:absolute`属性的标签会根据他上层最近的  `position:relative` 进行浮动。

  





## JS 基础

### 获取下拉菜单 \<select\> 所选取的值

```js
// js获取select标签选中的值 

var obj = document.getElementByIdx_x(”testSelect”); //定位id
var index = obj.selectedIndex; // 选中索引
var text = obj.options[index].text; // 选中文本
var value = obj.options[index].value; // 选中值
 
// jQuery中获得选中select值
// 第一种方式
$('#testSelect option:selected').text();//选中的文本
$('#testSelect option:selected').val();//选中的值
$("#testSelect ").get(0).selectedIndex;//索引
 
// 第二种方式
$("#tesetSelect").find("option:selected").text();//选中的文本
…….val();
…….get(0).selectedIndex;
```



