---
title: 【TypeScript】TypeScript基础
toc: true
date: 2022-12-16 17:58:26
tags: typescript javascript 前端
categories: 前端
---

​点击阅读更多查看文章内容<!--more-->

# 数据类型
- const
常量，不能改变，**类型可以通过初始化的值由编译器推断，也可以自己以 变量名 : 数据类型 的方式指定**

```typescript
const a = 'abc'
const a : string = 'abc'
const a : number = 'abc' //报错：Type 'string' is not assignable to type 'number'.
```

- number
整型，浮点型，表示所有数字类型

```typescript
let a = 123

let b : number = 123.456
```
- boolean
布尔型，true/false

```typescript
let a : boolean = true
```

- string
字符串，可以用单引号也可以用双引号

```typescript
let a : string = 'abc'
let b : string = "abc"
```

- literal类型
把变量的所有可能取值都列出来

```typescript
let httpStatus : 200 | 404 | 500 | '200' | '404' | '500'  = '200'
```

- 类型的并集
使变量可以支持多种类型

```typescript
function f(s: 200 | 404 | 500 | '200' | '404' | '500'){
    let status : string | number = s
}
```

- any类型
相当于JavaScript，可以对变量进行任何操作，编译器都不会报错（运行可能出错）

```typescript
let a : any = 'abc'
a = 123 // 不会报错，a为123
a.name = 'John' //运行出错
```

- undefined类型
定义为undefined类型，值也只能为undefined

```typescript
let a : undefined = undefined
```

# 逻辑控制

## if/else
等于一律使用：=\=\=
不等于一律使用：!==
undefined作为条件为false
```typescript
function processHttpStatus(s : 200 | 404 | 500 | '200' | '404' | '500') { 
    // 一律使用=== 以及 !==
    if (s === 200) {
        console.log('ok')
    } else if (s === 404) {
        console.log('not found')
    } else if (s === 500){
        console.log('internal server error')
    } else if (s === '200') {
        console.log('ok')
    } else if (s === '404') {
        console.log('not found')
    } else if (s === '500'){
        console.log('internal server error')
    }
    //无需再判断其他情况，因为传入的参数只有以上六种情况
}
```
问号表达式：`typeof s === 'string' ? parseInt(s) : s`
a?b|c，条件a为true返回b，否则返回c


```typescript
function processHttpStatus(s : 200 | 404 | 500 | '200' | '404' | '500') { 
    // 将 string 转为对应的 number
    const statusNum = typeof s === 'string' ? parseInt(s) : s
    // 一律使用=== 以及 !==
    if (statusNum === 200) {
        console.log('ok')
    } else if (statusNum === 404) {
        console.log('not found')
    } else if (statusNum === 500){
        console.log('internal server error')
    } 
}
```

---

## switch

```typescript
function processHttpStatus(s : 200 | 404 | 500 | '200' | '404' | '500') { 
    // string 转 number
    const statusNum = typeof s === 'string' ? parseInt(s) : s
    switch (statusNum){
        case 200:
            console.log('ok')
            break
        case 404:
            console.log('not found')
            break
        case 500:
            console.log('internal server error')
            break
        default:
            console.log('internal server error')
            break
        //default实际上不需要，这里只是展示switch的使用方法
    }
}
```

---

## 循环语句

跟C语言一样

- for
```typescript
let sum = 0
for (let i = 0; i < 100; i++){
    sum += i
}
console.log(sum)
```

- while
```typescript
let sum = 0
let i = 0
while (i < 100) {
    sum += i
    i++
}
console.log(sum)
```

## try-catch
throw抛出错误，注意throw后面的字符串包含${}取值所以不能使用引号，要使用 `

抛出错误后程序依然会继续执行，sum输出结果为4950
```typescript
let sum = 0
for (let i = 0; i < 100; i++){
    try {
        sum += i 
        if (i % 17 === 0) {
            throw `bad number ${i}`
        }
    } catch (err) {
        console.error(err)
    }
}
console.log(sum)
// 输出：
// [ERR]: "bad number 0" 
// [ERR]: "bad number 17" 
// [ERR]: "bad number 34" 
// [ERR]: "bad number 51" 
// [ERR]: "bad number 68" 
// [ERR]: "bad number 85" 
// [LOG]: 4950 
```

若不使用try-catch，在抛出错误后程序会直接终止

```typescript
let sum = 0
for (let i = 0; i < 100; i++){
    sum += i 
    if (i % 17 === 0) {
        throw `bad number ${i}`
     }
}
console.log(sum)
// 输出:
// [ERR]: "执行 JavaScript 失败:" 
// [ERR]: "bad number 0" 
```

# 枚举类型
typescript特有，JavaScript没有枚举类型
如果不对枚举类型赋值的话就从0开始递增赋值
HTTPStatus[s]可以直接打印出s，而不是其对应的枚举值
```typescript
enum HTTPStatus{
    OK = 200,
    NOT_FOUND = 404 ,
    INTERNAL_SERVER_ERROR = 500,
}
function processHTTPStatus(s:HTTPStatus){
    if (s === HTTPStatus.OK){
        console.log('good response')
    } else {
        console.log('bad response')
    }
    console.log(s)
    
    // 打印出s对应的值
    console.log(HTTPStatus[s])
}
processHTTPStatus(HTTPStatus.INTERNAL_SERVER_ERROR)
// 输出:
// [LOG]: "bad response" 
// [LOG]: 500 
// [LOG]: "INTERNAL_SERVER_ERROR" 
```

---

# 数组

- 定义
以下两种方法都可以，第二个是泛型类型
let a : number[] = [1,2,3]
let b : Array<number> = [1,2,3]
```typescript
let a = [1,2,3]

let a : number[] = [1,2,3]

let a = [1,2,3,'a']

let a : (string|number)[] = [1,2,3,'a']

let b : Array<number> = [1,2,3]
```

- 获取数组长度及元素，**数组越界不会报错会获得undefined**
Ps：空数组判断 ： a.length===0，不能直接写if(a)

```typescript
let a : number[] = [1,2,3,4]
let b : string[] = ['a','b','c']
console.log(a.length,b.length)
console.log(a[3],b[1])
console.log(a[4],b[-1])
// 输出
// [LOG]: 4,  3 
// [LOG]: 4,  "b" 
// [LOG]: undefined,  undefined
```
- 增删元素
	- push/pop 从数组右边增加/删除元素
	- unshift/shift 从数组左边增加/删除元素
	- **这里的数组可以用const定义，const表示a始终指向这个数组，数组的内容可以发生变化**
```typescript
const a : number[] = []
a.push(1)   // [1] 
a.push(2)   // [1, 2] 
a.push(3)   // [1, 2, 3] 
a.pop()     // [1, 2] 
a.push(4)   // [1, 2, 4] 
a.shift()   // [2, 4]
a.unshift(1)// [1, 2, 4]
// a = [1, 2, 3] 报错 ， a为const不能指向其他数组
```

- slice函数
a.slice(start,end)：截取子数组a[start,end)，前闭后开
越界不会报错，到边界就会结束
a.slice(start)：从start取到数组结尾
```typescript
const a = [0, 1, 2, 3, 4, 5, 6, 7]

console.log(a.slice(2, 5), a.slice(5, 10)) // [2, 3, 4],  [5, 6, 7] 
```
- splice函数
a.splice(start, datacount, ...items)：从start开始删除datacount个元素，并在start后插入items（items是可选的）

```typescript
const a = [0, 1, 2, 3, 4, 5, 6, 7]
console.log(a.splice(3, 2, 10, 11, 12, 13)) // [3, 4] 
console.log(a) // [0, 1, 2, 10, 11, 12, 13, 5, 6, 7] 
```
- indexOf函数
a.indexOf(searchElement, fromIndex)：从下标fromIndex开始查找，返回searchElement第一次出现的位置，（fromIndex可选）
a.lastIndexOf()：从后往前找

```typescript
const a = [0, 1, 2, 3, 4, 5, 6, 7, 11]
console.log(a.splice(3, 2, 10, 11, 12, 13)) // [3, 4] 
console.log(a) // [0, 1, 2, 10, 11, 12, 13, 5, 6, 7, 11] 

console.log(a.indexOf(11)) // 4
console.log(a.indexOf(11, 5)) // 10
console.log(a.lastIndexOf(11)) // 10
```
- sort函数
排序，注意sort的排序是按照字典序排的，实际上对我们排序数字没有用，更多的是考虑排序一些单词，排序数字需要给sort传递一个compareFn参数。

```typescript
const a = [0, 1, 2, 3, 4, 5, 6, 7, 11]
console.log(a.splice(3, 2, 10, 11, 12, 13)) // [3, 4] 
console.log(a) // [0, 1, 2, 10, 11, 12, 13, 5, 6, 7, 11] 

a.sort()
console.log(a) // [0, 1, 10, 11, 11, 12, 13, 2, 5, 6, 7] 
```
- 元组
 没有特别的元组语法，数组就可以当元组使用
```typescript
// 元组 tuple
const a = [1, 2, 3]
const [a1, a2, a3, a4] = a
console.log(a1, a2, a3, a4) // 1,  2,  3,  undefined 
```

- split/join
split()：使用指定的分隔符将字符串拆分为子字符串，并将它们作为数组返回。
join()：将数组的所有元素添加到字符串中，以指定的分隔符字符串分隔。
```typescript
// split/join
console.log('a,b,c,1,2,3'.split(',')) // ["a", "b", "c", "1", "2", "3"] 
console.log([1,2,3,4].join(',')) // "1,2,3,4" 
```
- forEach
遍历数组，也可以使用传统for循环遍历整个数组，推荐使用forEach
```go
const a = [1,2,3,4]
a.forEach(v => {
    console.log(v)
})
```

# 对象类型
不需要有类就可以直接声明

- 定义

```typescript
const emp1 = {
    name: {
        first: '三',
        last: '张'
    },
    gender: 'male' as 'male' | 'female' | 'other' | 'unknown', //规定gender的类型
    salary: 8000,
    bonus: undefined as number|undefined,
    performance: 3.5,
    badges: ['优秀员工', '迟到王'],
}
console.log(emp1)
// 输出:
// [LOG]: {
//   "name": {
//     "first": "三",
//     "last": "张"
//   },
//   "gender": "male",
//   "salary": 8000,
//   "bonus": undefined,
//   "performance": 3.5,
//   "badges": [
//     "优秀员工",
//     "迟到王"
//   ]
// }
```
输出结果为JSON对象 （JSON：JavaScript Object Notation）

- JSON

可以直接将输出结果赋值给新对象（字段名用不用引号效果是一样的）
这里的新对象的类型就只有从结果中读出的类型，没有emp1中额外加的类型
```typescript
const emp2 = {
  "name": {
    "first": "三",
    "last": "张"
  },
  "gender": "male",
  "salary": 8000,
  "bonus": undefined,
  "performance": 3.5,
  "badges": [
    "优秀员工",
    "迟到王"
  ]
} 
```

JSON对象转为JSON字符串
`JSON.stringify(）`
```typescript
const s : string = JSON.stringify(emp1)
console.log(s)
// [LOG]: "{"name":{"first":"三","last":"张"},"gender":"male","salary":8000,"bonus":28000,"performance":3.5,"badges":["优秀员工","迟到王"]}" 
```

JSON字符串转为JSON对象
`JSON.parse()`，这里JSON.parse()的返回值为any类型与直接输出emp1得到的结果相同（any类型不能直接通过.获取它的内在元素）
```typescript
const s : string = JSON.stringify(emp1)
const emp2 = JSON.parse(s)
console.log(emp2)
// [LOG]: {
//   "name": {
//     "first": "三",
//     "last": "张"
//   },
//   "gender": "male",
//   "salary": 8000,
//   "bonus": 28000,
//   "performance": 3.5,
//   "badges": [
//     "优秀员工",
//     "迟到王"
//   ]
// } 
```

>**对象类型不能直接比较 emp1 === emp2 返回值为false**


# 函数
- 定义
返回值可写可不写，不写的话可以函数的返回值类型由return语句决定，没有return语句的话返回值为void
```typescript
function add(a: number,b: number): number{
    return a+b
}
```
- 可选参数
?表示可选参数，如果没有输入的话，值为undefined
c||0：表示若c不为undefined的话返回值为c，否则返回值为0
```typescript
function add(a: number, b: number, c?: number): number{
    return c ? a + b + c : a + b
    // return a + b + (c||0)
}
```
- 参数默认值
可选参数后的参数必须是可选参数或者带有默认值
```typescript
function add(a: number, b: number, c?: number, d: number=0): number{
    return a + b + (c||0) + d
}
```
- 可变参数列表
...表示可变参数列表：可以传入任意多个参数，当做数组使用
若想将数组传入函数，可以在数组前加...，将数组展开传入函数
```typescript
function add(
    a: number, 
    b: number, 
    c?: number, 
    d: number=0,
    ...e: number[]): number{
    let sum =  a + b + (c||0) + d
    for (let i = 0; i < e.length; i++){
        sum += e[i]
    }
    return sum
}
console.log(add(1, 2, 3, 4, 5, 6, 7, 8, 9, 10))
const numbers = [5, 6, 7, 8, 9, 10]
console.log(add(1, 2, 3, 4, ...numbers))
```
- 重载
**不建议使用函数重载**
函数重载通过声明函数实现

```typescript
function add(a: number, b: number): number
function add(
    a: number, 
    b: number, 
    ...e: number[]): number

function add(
    a: number, 
    b: number, 
    c?: number, 
    d: number=0,
    ...e: number[]): number{
    let sum =  a + b + (c||0) + d
    for (let i = 0; i < e.length; i++){
        sum += e[i]
    }
    return sum
}
```
- 对象类型参数
参数列表过多或者具有boolean类型的参数时，可以定义对象类型的参数，这样函数调用者可以清楚的理解每个参数的具体作用（如果不使用对象类型直接传入一堆参数很难理解每个参数的意义，使用对象类型在调用的时候会表明对应的参数名便于理解）
```typescript
function sendRequest(params:{
    url: string,
    method: 'GET'|'POST'|'PUT',
    header: object,
    data?: string,
    requireAuth: boolean,
    retry: boolean,
    retryTimeout?: number,
}){}


sendRequest({
    url: 'https://www.test.com',
    method: 'GET',
    header: {
        contentType: 'application/json',
    },
    data: '{}',
    requireAuth: false,
    retry: true,
    retryTimeout: 30000,
})
```
- 为对象定义方法
不需要加function，其他的与普通的函数定义相同
函数中的变量会从全局搜索，可以使用this引用自身的字段和方法
```typescript
const emp1 = {
    name: 'john',
    salary: 8000,
    bonus: undefined as number|undefined,
    performance: 3.5,
    updateBonus() {
        if (!this.bonus) {
            this.bonus = this.salary * this.performance
        }
    },
}
emp1.updateBonus()
```

