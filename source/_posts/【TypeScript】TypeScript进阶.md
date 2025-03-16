---
title: 【TypeScript】TypeScript进阶
toc: true
date: 2022-12-26 13:42:33
tags: typescript javascript 前端
categories: 前端
---
​点击阅读更多查看文章内容<!--more-->

# 函数式编程

**函数作为“一等公民”**

- 变量类型可以是函数，值（literal）可以是函数
以下代码推荐写成lambda表达式的形式（在JavaScript/TypeScript中也被称为**箭头函数**）
`const compareNumber = (a: number, b: number) => a-b`
箭头函数：用箭头指向函数的结果，也可以用箭头指向{函数体}
```typescript
// lambda 表达式， JavaScript/TypeScript：箭头函数
//const compareNumber = (a: number, b: number) => a-b
const compareNumber = function (a:number,b:number) {
    return a-b   
}
let a = [5,2,1,6,8,10,5,25,16,23,11]
a.sort(compareNumber)
```

- 对象的字段可以是函数
```typescript
const emp1 = {
    name: 'john',
    salary: 8000,
    increaseSalary: function (p: number) {
        this.salary *= p
    }
}
```
- 函数的参数可以是函数
```typescript
function compareNumber(a:number,b:number) {
    return a-b   
}
let a = [5,2,1,6,8,10,5,25,16,23,11]
a.sort(compareNumber)
```
- 函数的返回值可以是函数
这里的createComparer函数中具有boolean类型的参数，如果直接传入false的话不知道false具体有什么作用，所以这里使用了对象类型参数在调用函数时可以很清晰的看出false的变量为smallerFirst
```typescript
function createComparer(p: {smallerFirst: boolean}){
    if (p.smallerFirst) {
        return (a: number, b: number) => a-b
    } else {
        return (a: number, b: number) => b-a
    }
}
let a = [5,2,1,6,8,10,5,25,16,23,11]
a.sort(createComparer({smallerFirst: false}))
```

---

**高阶函数**

函数的参数可以是函数
函数的返回值可以是函数
以上两种都属于高阶函数

loggingComparer()：函数的参数和返回值都是函数
```typescript
function loggingComparer(comp: (a: number, b: number) => number){
    return (a: number, b: number) => {
        console.log('comparing', a, b)
        return comp(a, b)
    }
}

function createComparer(p: {smallerFirst: boolean}){
    if (p.smallerFirst) {
        return (a: number, b: number) => a-b
    } else {
        return (a: number, b: number) => b-a
    }
}
let a = [5,2,1,6,8,10,5,25,16,23,11]
const comp = createComparer({smallerFirst: true})
a.sort(loggingComparer(comp))
```

---

**闭包**
使局部变量的生命周期延长

```typescript
function loggingComparer(
    logger: (a: number, b: number) => void,
    comp: (a: number, b: number) => number) {
    return (a: number, b: number) => {
        logger(a,b)
        return comp(a, b)
    }
}

function createComparer(p: {smallerFirst: boolean}){
    if (p.smallerFirst) {
        return (a: number, b: number) => a-b
    } else {
        return (a: number, b: number) => b-a
    }
}

function processArray(a: number[] ){
    let compCount = 0

    // logger：闭包
    const logger = (a: number, b: number) => {
        console.log('comparing', a, b)
        compCount++ //自由变量
    }
    const comp = createComparer({smallerFirst: true})
    a.sort(loggingComparer(logger, comp))
    return compCount
}
let a = [5,2,1,6,8,10,5,25,16,23,11]
const compCount = processArray(a)
console.log(a,compCount)
```

---

**部分应用函数**

下例中isGoodNumber有两个参数，我们希望先用一个已知的goodFactor，使用这个goodFactor去判断我们收到的每个v

即我们现在只有一个参数goodFactor，我们想先将goodFactor传入函数，这样isGoodNumber就只有一个参数，此时只有一个参数的isGoodNumber就可以作为filterArray的参数f传入filterArray

>解决方法：使用闭包将goodFactor放入函数体中

方法一（推荐使用）：
```typescript
function isGoodNumber(goodFactor: number, v: number){
    return v % goodFactor === 0
}

function filterArray(a: number[], f: (v: number) => boolean) {
    return a.filter(f)
}

const GOOD_FACTOR = 2
const a = [1, 2, 3, 4, 5, 6, 7, 8, 9]
console.log(filterArray(a, (v) => isGoodNumber(GOOD_FACTOR, v)))
```

方法二（加深印象）：
```typescript
function isGoodNumber(goodFactor: number, v: number){
    return v % goodFactor === 0
}

function filterArray(a: number[], f: (v: number) => boolean) {
    return a.filter(f)
}

function partiallyApply(
    f: (a: number, b: number) => boolean,
    a: number) {
        return (b: number) => f(a, b)
    }

const GOOD_FACTOR = 2
const a = [1, 2, 3, 4, 5, 6, 7, 8, 9]
console.log(filterArray(a, partiallyApply(isGoodNumber, GOOD_FACTOR)))
```


# Promise
回调函数会引起 “callback hell” 问题，如下例只能通过嵌套的方法计算2+3+4，会导致多层嵌套
```typescript
function add (a: number, b: number, callback: (res: number) => void): void{
    setTimeout(()=>{
        callback(a+b)
    },2000)
}

add(2,3,res => {
    console.log('2+3',res)
    add(res,4,res2=>{
        console.log('2+3+4',res2)
    })
})
```

改写为promise，代码更加清晰
new Promise是Promise的构造函数，它的参数
(resolve,reject)=>{
        setTimeout(()=>{
            resolve(a+b)
        },2000)
    }
    也是函数，这个函数的参数resolve和reject也是函数，如果运行成功就调resolve，如果失败就调reject
    
**resolve**
```typescript
function add (a: number, b: number): Promise<number>{
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            resolve(a+b)
        },2000)
    })
}

add(2,3).then(res => {
    console.log('2+3',res)
    return add(res,4)
}).then(res => {
    console.log('2+3+4',res)
    return add(res,6)
}).then(res =>{
    console.log('2+3+4+6',res)
})
```
**reject**

```typescript
function add (a: number, b: number): Promise<number>{
    return new Promise((resolve,reject)=>{
        if (b % 17 === 0){
            reject(`bad number: ${b}`)
        }
        setTimeout(()=>{
            resolve(a+b)
        },2000)
    })
}

add(2,17).then(res => {
    console.log('2+17',res)
    return add(res,4)
}).catch(err => {
    console.log('caught error',err)
})
// caught error bad number: 17
```

同时等待多个Promise
Promise.all
等待任意一个Promise
Promise.race
```typescript
function add (a: number, b: number): Promise<number>{
    return new Promise((resolve,reject)=>{
        if (b % 17 === 0){
            reject(`bad number: ${b}`)
        }
        setTimeout(()=>{
            resolve(a+b)
        },2000)
    })
}

function mul(a: number,b: number): Promise<number>{
    return new Promise((resolve,reject) => {
        setTimeout(() => {
            resolve(a*b)
        }, 3000)
    })
}

// (2+3)*(4+5)
Promise.all([add(2,3), add(4,5)]).then(res => {
    const [a,b] = res
    return mul(a,b)
}).then(res => {
    console.log(res)
})


Promise.race([add(2,3), add(4,5)]).then(res => {
    console.log(res)
})
```
Promise通过串联就可以解决callback hell

**async-await 语法糖**
promise可以写成async-await的形式
await是异步等待，只能在async function中调用
以下calc函数返回类型为Promise\<number>
```typescript
async function calc() {
    const a = await add(2, 3)
    const b = await add(4, 5)
    const c = await mul(a, b)
    return c     
}

calc().then(res => {
    console.log(res)
})
```

# 接口

- **作用**：描述一个类型

```typescript
interface Employee {
    name: string
    salary: number
    bonus?: number
}


const emp1: Employee = {
    name: 'john',
    salary: 8000,
}

const emp2: Employee = {
    name: '张三',
    salary: 10000,
    bonus: 20000,
}
```

- 可选字段串联
`e.name?.first?.startsWith('AAA')`
如果e.name为undefined则整个表达式都为undefined（如果不加? 则由于name可能为空编译器会报错）

```typescript
interface Employee {
    name?: {
        first?: string
        last: string
    }
    salary: number
    bonus?: number
}

function hasBadName(e: Employee){
    // 如果e.name为undefined则整个表达式都为undefined
    return e.name?.first?.startsWith('AAA')
}
```
- 非空断言
`e.name!.first!.startsWith('AAA')`
断言e.name e.name.first不为undefined，此时编译器不会报错（如果不加! 则由于name可能为空编译器会报错），若其值为undefined则后果自负（程序运行出错）
```typescript
function hasBadName(e: Employee){
    return e.name!.first!.startsWith('AAA')
}
```

- 接口扩展
extends：将已有接口拼成一个更大的接口
```typescript
interface Employee extends HasName{
    salary: number
    bonus?: number
}
interface HasName {
    name?: {
        first?: string
        last: string
    }
}
```
- 接口的并集
` e: WxButton|WxImage` e只能调用他们共有的属性visible
```typescript
interface WxButton {
    visible: boolean
    enabled: boolean
    onClick(): void
}

interface WxImage {
    visible: boolean
    src: string
    width: number
    height: number
}

function processElement(e: WxButton|WxImage) {
    e.visible
}
```
- 类型断言
`e as WxButton`，得到的类型为WxButton
判断e是Button还是Image，由于ts在转为js时没有interface，所以要根据Button和Image的属性来判断，具有onClick方法的就是Button
```typescript
function processElement(e: WxButton|WxImage) {
	// 判断e是Button还是Image
    if ((e as any).onClick) {
        const btn = e as WxButton
        btn.onClick()
    } else {
        const img = e as WxImage
        console.log(img.src)
    }
}
```
- 类型判断
`(e as WxButton).onClick !== undefined`：要想调用其他属性需要首先判断e的类型，这里不能使用typeof判断，因为在js中看不到自己定义的接口，需要通过判断e是否具有某个属性来判断它的类型。
返回值为e is WxButton，编译器可以判断传入的e为WxButton
```typescript
function processElement(e: WxButton|WxImage) {
    if (isButton(e)) {
        e.onClick()
    } else {
        console.log(e.src)
    }
}

function isButton(e: WxButton|WxImage): e is WxButton {
    return (e as WxButton).onClick !== undefined
}
```
# 类
- 类在定义时必须初始化
可以通过构造函数/赋初值的方式
不加private默认为public
```typescript
class Employee {
    name: string
    salary: number
    private bonus: number = 0
    constructor(name: string, salary: number){
        this.name = name
        this.salary = salary
    }
}
```
- 可以在构造函数的参数前添加public/private，相当于定义属性
可选参数不用初始化
```typescript
class Employee {
    private bonus?: number
    constructor(public name: string, public salary: number){
        this.name = name
        this.salary = salary
    }
    updateBonus() {
        if (!this.bonus){
            this.bonus = 20000
        }
    }
}
```

- getter/setter
（allocatedBonus是private不能直接修改）
调用时看不出是一个方法，相当于一个属性进行调用

```typescript
class Employee {
    private allocatedBonus?: number
    constructor(public name: string,public salary: number){
        this.name = name
        this.salary = salary
    }

    // getter/setter
    set bonus(v: number){
        this.allocatedBonus = v
    }
    get bonus(){
        return this.allocatedBonus || 0
    }
}

const emp1 = new Employee('john',8000)
emp1.bonus = 20000
console.log(emp1.bonus) // 20000 
console.log(emp1)
// Employee: {
//   "name": "john",
//   "salary": 8000,
//   "allocatedBonus": 20000
// } 
```
- 类的继承
extends
通过super调用基类的构造函数（不加构造函数的话会默认使用基类的构造函数）
```typescript
class Employee {
    private allocatedBonus?: number
    constructor(public name: string,public salary: number){
        this.name = name
        this.salary = salary
    }

    // getter/setter
    set bonus(v: number){
        this.allocatedBonus = v
    }
    get bonus(){
        return this.allocatedBonus || 0
    }
}

class Manager extends Employee {
    private reporters: Employee[]
    constructor(name: string, salary: number) {
        super(name, salary)
        this.reporters = []
    }
    addReporter(e: Employee){
        this.reporters.push(e)
    }
}
```

- 类实现接口
隐式实现：不用implements实现，在定义类的时候不会检查属性，在将类赋值给接口时，会挨个比较属性，如果属性不全则会报错
显示实现：使用implements，类在定义的时候需要定义接口的全部属性

```typescript
interface Employee {
    name: string
    salary: number
    bonus?: number
}


class EmplImpl implements Employee{
    private allocatedBonus?: number
    constructor(public name: string,public salary: number){
        this.name = name
        this.salary = salary
    }

    // getter/setter
    set bonus(v: number){
        this.allocatedBonus = v
    }
    get bonus(){
        return this.allocatedBonus || 0
    }
}

class Manager extends EmplImpl {
    private reporters: EmplImpl[] = []

    addReporter(e: EmplImpl){
        this.reporters.push(e)
    }
}

const empImpl = new EmplImpl('john', 8000)
const emp1: Employee = empImpl
```
- 定义方法选择 显示定义
定义者=实现者（不推荐）
在使用的时候会有很多多余的方法
```typescript
// 定义接口
interface Service {
    login(): void
    getTrips(): string
    getLic(): string
    startTrip(): void
    updateLic(lic: string): void
}

// 实现接口
class RPCService implements Service {
    login(): void {
        throw new Error("Method not implemented.")
    }
    getTrips(): string {
        throw new Error("Method not implemented.")
    }
    getLic(): string {
        throw new Error("Method not implemented.")
    }
    startTrip(): void {
        throw new Error("Method not implemented.")
    }
    updateLic(lic: string): void {
        throw new Error("Method not implemented.")
    }
}

const page = {
    service: new RPCService() as Service,
    onLoginButtonClicked() {
        //使用接口
        this.service.login()
    }
}
```
- 定义方法选择 隐示定义
定义者=使用者（推荐）
由使用的人根据自己的需要定义接口，每个接口都很小比较好维护，实现解耦
只能使用隐式实现，显示实现的话在实现时会implements很多不同的接口，每个使用者都有一个接口

```typescript
// 实现接口
class RPCService {
    login(): void {
        throw new Error("Method not implemented.")
    }
    getTrips(): string {
        throw new Error("Method not implemented.")
    }
    getLic(): string {
        throw new Error("Method not implemented.")
    }
    startTrip(): void {
        throw new Error("Method not implemented.")
    }
    updateLic(lic: string): void {
        throw new Error("Method not implemented.")
    }
}

// 定义接口
interface LoginService {
    login(): void
}

const page = {
    service: new RPCService() as LoginService,
    onLoginButtonClicked() {
        //使用接口
        this.service.login()
    }
}
```

# 泛型
用来约束参数类型
`const a: number[] = []`等价于`const a: Array<number> = []`

- 定义
在定义类型时需要表明泛型类型
在调用函数时可以不必表明类型，编译器可以根据参数自己判断类型
```typescript
class MyArray<T> {
    date: T[] = []
    add(t: T) {
        this.date.push(t)
    }
    map<U>(f: (v: T) => U): U[] {
        return this.date.map(f)
    }
    print() {
        console.log(this.date)
    }
}


const a = new MyArray<number>()
a.add(1)
a.add(2000)
a.add(30000)
//(method) MyArray<number>.map<string>(f: (v: number) => string): string[]
console.log(a.map(v => v.toExponential()))
```

- 使用接口约束泛型类型
T 可以 . 出内容
```typescript
interface HasWeight {
    weight: number
}
class MyArray<T extends HasWeight> {
    date: T[] = []
    add(t: T) {
        this.date.push(t)
    }
    map<U>(f: (v: T) => U): U[] {
        return this.date.map(f)
    }
    print() {
        console.log(this.date)
    }
    sortByWeight() {
        this.date.sort((a, b) => a.weight - b.weight)
    }
}

class WeightedNumber {
    constructor(public weight: number) { }
}
const a = new MyArray<WeightedNumber>()
a.add(new WeightedNumber(10000))
a.add(new WeightedNumber(200))
a.add(new WeightedNumber(30000))
a.sortByWeight()
console.log(a)
```
