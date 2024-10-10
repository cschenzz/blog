在 JavaScript 中，对对象数组进行遍历、过滤和 `reduce` 操作是非常常见的需求。下面分别介绍如何使用 `forEach`、`filter` 和 `reduce` 方法来处理对象数组。

### 示例对象数组

假设我们有一个包含用户信息的对象数组：

```javascript
const users = [
  { id: 1, name: 'Alice', age: 25, isActive: true },
  { id: 2, name: 'Bob', age: 30, isActive: false },
  { id: 3, name: 'Charlie', age: 35, isActive: true },
  { id: 4, name: 'David', age: 40, isActive: false }
];
```

### 1. 遍历对象数组

使用 `forEach` 方法遍历对象数组，并对每个对象执行操作：

```javascript
users.forEach(user => {
  console.log(`Name: ${user.name}, Age: ${user.age}`);
});
```

### 2. 过滤对象数组

使用 `filter` 方法过滤出符合条件的对象数组。例如，过滤出所有活跃的用户：

```javascript
const activeUsers = users.filter(user => user.isActive);
console.log(activeUsers);
// 输出:
// [
//   { id: 1, name: 'Alice', age: 25, isActive: true },
//   { id: 3, name: 'Charlie', age: 35, isActive: true }
// ]
```

### 3. 使用 `reduce` 进行累积操作

使用 `reduce` 方法对对象数组进行累积操作。例如，计算所有用户的总年龄：

```javascript
const totalAge = users.reduce((accumulator, user) => {
  return accumulator + user.age;
}, 0);

console.log(totalAge); // 输出: 130
```

### 更复杂的 `reduce` 示例

假设我们需要创建一个对象，其中键是用户的名称，值是用户的年龄：

```javascript
const nameToAgeMap = users.reduce((accumulator, user) => {
  accumulator[user.name] = user.age;
  return accumulator;
}, {});

console.log(nameToAgeMap);
// 输出:
// {
//   Alice: 25,
//   Bob: 30,
//   Charlie: 35,
//   David: 40
// }
```

### 组合使用 `filter` 和 `reduce`

假设我们需要计算所有活跃用户的总年龄：

```javascript
const totalActiveAge = users
  .filter(user => user.isActive)
  .reduce((accumulator, user) => accumulator + user.age, 0);

console.log(totalActiveAge); // 输出: 60
```

### 总结

- **`forEach`**：用于遍历数组中的每个元素，并对每个元素执行一个提供的函数。
- **`filter`**：用于创建一个新数组，其结果是通过提供的测试函数的元素。
- **`reduce`**：用于对数组中的每个元素执行一个提供的累加器函数，最终返回一个单一的值。

通过这些方法，你可以灵活地处理对象数组，实现各种复杂的需求。