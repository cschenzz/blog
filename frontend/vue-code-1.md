## vue3项目修改启动端口
```javascript
import { defineConfig } from 'vite'
import path from 'path'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
  ],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src')
    },
  },
  server: {
    port: 3000,
    host: true,
    open: false,
    proxy: {
      // 开发环境启用代理 
      '/dev-api': {
        target: 'http://localhost:9999',
        changeOrigin: true,
        rewrite: (p) => p.replace(/^\/dev-api/, '')
      }
    }
  }
})
```

## vue常用代码
```javascript
const props = defineProps({
    title: String,
    user: Object
})

const __$_name = computed(() => {
    return 'computed:' + props.user.name
})

// onMounted(() => __init())

onUnmounted(() => {
    console.log(1, '--onUnmounted')
})

// 函数
// 会生成一个指定长度的随机字符串，包含字母和数字
const __generateRandomString = (length) => {
    var result = ''
    var characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'
    var charactersLength = characters.length
    for (var i = 0; i < length; i++) {
        result += characters.charAt(Math.floor(Math.random() * charactersLength))
    }
    return result
}
```


## js基础
```javascript
// 定义字符串常量
const companyName = "Tech Innovations Inc.";

// 定义数字常量
const MAX_USERS = 100;

// 定义布尔常量
const DEBUG_MODE = false;

// 注意：尽管数组和对象可以用const定义，但要注意的是，它们的内容（如果是可变对象或数组）是可以修改的。
const daysOfWeek = ["Monday", "Tuesday", "Wednesday"]; // 数组内容可以修改
const settings = {}; // 对象属性可以添加或修改

const PI = 3.14159;

let count = 0;
// 使用const定义常量时，尝试修改其值会导致运行时错误。不过，对于数组和对象而言，const只是阻止重新分配引用，但不会阻止修改内部结构
```

## js函数
```javascript
// 函数定义
// 1.声明函数:
// 使用function关键字定义函数，后面跟着函数名、参数列表（在圆括号内）和函数体（在花括号内）。
function sayHello(name) {
  console.log("Hello, " + name);
}

// 2.匿名函数:
// 不给函数命名，常用于赋值给变量或作为其他函数的参数。
const greet = function(name) {
  console.log("Hi there, " + name);
};

// 3.箭头函数:
// 更简洁的函数表达形式，适用于不需要this绑定改变或不需要arguments对象的场景。
const greet = (name) => {
  console.log(`Hello, ${name}`);
};


// 函数调用
// 1.直接调用:
// 直接使用函数名加上括号来调用函数，如果有参数则放在括号内。
sayHello("Alice");

// 2.作为对象的方法调用:
// 函数可以作为对象的一个属性（即方法）被调用。
const person = {
  name: "Bob",
  sayHello: function() {
    console.log("Hello, " + this.name);
  }
};
person.sayHello();
```


## 异步请求方法
```js
import axios from 'axios'

const __$_request_data = async () => {
  const response = await axios.get('http://localhost:9900/cc/public/help')
  // 使用 JSON.stringify() 方法可以将 JavaScript 对象转换为 JSON 字符串
  // 将 JSON 字符串转换为 JavaScript 对象
  console.log(1, '---', JSON.stringify(response, null, 2))
  console.log(2, '---', JSON.stringify(response.data))
  console.log(3, '---', response.data)
}

// onMounted(__$_request_data)

// onMounted 的回调函数必须是一个 async 函数，这样你才能在里面使用 await
onMounted(async () => {
  const response = await axios.get('http://localhost:9900/cc/public/help')
  console.log(3, '---', response.data)
})
```

---------------------
- [Vue3 English](https://vuejs.org/)