# TypeScript基础

## 简介
> TypeScript 是一种由微软开发的自由和开源的编程语言。它是 JavaScript 的一个超集，TypeScript 在 JavaScript 的基础上添加了可选的静态类型和基于类的面向对象编程;其实TypeScript就是相当于JavaScript的增强版，但是最后运行时还要编译成JavaScript

## demo初始化
> 用npm安装typescript
```
npm install typescript -g //全局安装typescript  成功后使用tsc --version查看是否安装成功
```
> 初始化package.json文件
```
npm init -y // 使用 -y 标记表示你能接受 package.json 文件的一堆默认值
```

> 生成tsconfig.json文件
```
tsc --init //tsconfig.json初始化命令
```
> 安装@types/node,使用npm install @types/node --dev-save进行安装。这个主要是解决模块的声明文件问题

> 编写HelloWorld.ts文件，然后进行保存，代码如下
```
var test: string = "Hello Word !!"
console.log(test)
```

> 在Vscode的任务菜单下，打开运行生成任务，然后选择tsc：构建-tsconfig.json，这时候就会生成一个helloWorld.js文件

## TS变量类型
**typescript中的数据类型：**
- Undefined；
- Number:数值类型；
- string : 字符串类型;
- Boolean: 布尔类型；
- **enum：枚举类型**；
- **any : 任意类型，一个牛X的类型；**
- **void：空类型；**
- Array : 数组类型;
- **Tuple : 元祖类型；**
- Null ：空类型。

> **Undefined类型：**
声明一个变量，不给其赋值，使用该变量时，值为undefined，eg:
```
var age: number
console.log(age) //undefined
```

> **Number类型：**
在TypeScript中，所有的数字都是Number类型，这不分是整数还是小数。比如下面我们声明一个年龄是18岁，身高是178.5厘米
```
var age:number = 18
var stature:number = 178.5
console.log(age)
console.log(stature)
```
**在TypeScrip中有几种特殊的Number类型:**
- NaN：它是Not a Number 的简写，意思就是不是一个数值。如果一个计算结果或者函数的返回值本应该是数值，但是由于种种原因，他不是数字。出现这种状况不会报错，而是把它的结果看成了NaN。
- Infinity :正无穷大
- -Infinity：负无穷大。

> **enum枚举类型:**
这个世界有很多值是多个并且是固定的，比如：
- 世界上人的类型：男人、女人、中性
- 一年的季节：春、夏、秋、冬 ，有四个结果。
这时就是我们使用枚举的时机，eg:
```
enum REN{nan, nv, mindle}
console.log(REN.mindle) //返回了2，这是索引index，跟数组很像
```
如果我们想给这些枚举赋值，可以直接使用=,来进行赋值

```
enum REN {
    nan = '男',
    nv = '女',
    mindle = '中性' 
}
console.log(REN.mindle) //返回了 中性
```

> **any类型：**
有时候我们分不清是什么类型时，ts提供了any类型，这样变化类型，也不会报错, eg:
`var test: any = 10`


## typescript的函数：
typescript函数需要注意的是：
- 声明（定义）函数必须加 function 关键字；
- 函数名与变量名一样，命名规则按照标识符规则；
- 函数参数可有可无，多个参数之间用逗号隔开；
- 每个参数参数由名字与类型组成，之间用分号隔开；
- 函数的返回值可有可无，没有时，返回类型为 void；
- 大括号中是函数体

> **TypeScript语言中的函数参数**

TypeScript的函数参数是比较灵活的，它不像那些早起出现的传统语言那么死板。在TypeScript语言中，函数的形参分为：可选形参、默认形参、剩余参数形参等

> **1. 有可选参数的函数**

可选参数，就是我们定义形参的时候，可以定义一个可传可不传的参数。这种参数，在定义函数的时候通过?标注。eg:
```
function TypeTest(age: number, state: string): string { //此时state就是可选参数
    let yy:string = ""
    yy = "找到了"+age+'岁'
    if(state != undefined) {
        yy = yy + state
    }
    return yy + "的小街基"
}
```

> **2. 有默认参数的函数**

有默认参数就更好理解了，就是我们不传递的时候，他会给我们一个默认值，而不是undefined了

`function TypeTest(age: number=18, state: string): string {//age就是默认参数`

> **3.有剩余参数的函数**

传递给函数的参数个数不确定。剩余参数就是形参是一个数组，传递几个实参过来都可以直接存在形参的数组中
```
function TypeTest(...xuqiu:string[]): string {
//调用
var result:string  =  TypeTest('22岁','222','瓜子脸','梵蒂冈')
```

## 三种函数的定义方式
> **1. 函数声明法**

函数声明法创建函数是最常用的函数定义法。使用function关键字和函数名去定义一个函数。
```
function add(n1:number,n2:number):number{
    return n1+n2
}
```

> **2. 函数表达式法**

函数表达式法是将一个函数赋值给一个变量，这个变量名就是函数名。通过变量名就可以调用函数了。这种方式定义的函数，必须在定义之后，调用函数。下面例子中等号右边的函数没有函数名，称为匿名函数

```
var add = function(n1:number,n2:number):number{
    return n1+n2
}

console.log(add(1,4))
```
> **3. 箭头函数**

箭头函数是 ES6 中新增的函数定义的新方式，我们的 TypeScript 语言是完全支持 ES6 语法的。箭头函数定义的函数一般都用于回调函数中

```
var add = (n1:number,n2:number):number=>{
    return n1+n2
}

console.log(add(1,4))
```

## 引用类型-数组
TypeScript中的数据分为值类型和引用类型，前面用的都是值类型，现在虚拟一个人物的对象，代码如下
```
let webMan = {
    name:'菜前端',
    website:'jspang.com',
    age:18,
    saySometing:function(){
        console.log('为了前端技术')
    }
}
console.log(webMan.name)
webMan.saySometing()
```
通过上面的案例，我们看到引用类型是一种复合的数据类型，引用类型中封装了很多对属性，每一对属性都有属性名和属性值

在TypeScript中也给我们提供了一些引用类型，例如：Array（数组）、String（字符串）、Date（日期对象）、RegExp（正则表达式）等

> **初始化数组的两种方式**

声明数组跟声明一个普通变量是一样的，都是通过 var let 关键字实现的，只不过数组的类型说明符比较复杂而已
```
let arr1:number[] //声明一个数值类型的数组
let arr2:Array<string> // 声明一个字符串类型的数组
```

**给数组赋值**

数组是存储大量数据的集合，声明数组之后，需要给数组存储数据。这时候有两种方法：
- 字面量赋值法：直接使用“[ ]”对数组进行赋值，例如
```
//定义一个空数组，数组容量为0
let arr1:number[] = [] 
//定义一个数组时，直接给数组赋值
let arr2:number[] = [1,2,3,4,5]
//定义数组 的同事给数组赋值
let arr4:Array<boolean> = [ true,false,false]
```
> 需要注意的是，在TypeScript中指定数据类型的数组只能存储同一类型的数组元素
let arr5:number[] = [1,2,true] //报错
- 构造函数赋值法，例如
```
let arr1:number[] = new Array()
let ara2:number[] = new Array(1,2,3,4,5)
let arr4:Array<boolean> = new Array(true,false,false)
```
**元数组，一种特特的数组，(开发中很少用)**

元祖是一种特殊的数组，元祖类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。比如，你可以定义一对值分别为string和number类型的元祖

```
//声明一个元祖类型
let x : [string,number]
//正确的初始化
x = ['hello',10] //顺序不能颠倒
```

## 引用类型-字符串
> **字符串的两种类型**
- 基本类型字符串：由单引号或者双引号括起来的一串字符串。
- 引用类型字符串：用new 实例化的 String类型。
```
let web1:string = '菜前端'
let web2:String = new String("maozongtai.com")
```
> 常用方法
- str.indexOf(subStr) 返回索引，没有找到返回-1
- lastIndexOf() 从字符串尾部开始查找字符串的位置
- str.substring(startIndex,[endIndex]) 截取字符串
- str.replace(subStr,newstr) 替换字符串

## 引用类型-日期对象
> let d:Date = new Date()//参数可以不传，可以是整数(1000)，可以是字符串('2018/09/06 05:30:00')

## 引用类型-正则表达式
- 构造函数法

`let reg2:RegExp = new RegExp("test",'gi')`

- 字面量法

`let reg2:RegExp = /test/gi`

> RegExp中常用的方法
- **test(string)**: 在字符串中查找是否存在指定的正则表达式并返回布尔值，如果存在则返回 true，不存在则返回 false。

- **exec(string)**: 用于在字符串中查找指定正则表达式，如果 exec() 方法执行成功，则返回包含该查找字符串的相关信息数组。如果执行失败，则返回 null

## 面向对象编程-类的声明跟使用

TypeScript提供了强大的类的支持，只有会了类才可以new出对象来，下面创建一个Personer的类：
```
class Personer {
    name: string;
    age: string; //跟js不同，ts这里是;
    constructor(name:string, age:string) {
        this.name = name;
        this.age = age;
    };
    say() {
        console.log("hello !");
    }
}

let hanmeimei:Personer = new Personer("韩梅梅", "18");
console.log(hanmeimei);
hanmeimei.say();
```
我们先用class关键字声明了一个类，并在里边声明了name和age属性。constructor为构造函数。构造函数的主要作用是给类中封装的属性进行赋值

## 面向对象编程-修饰符

**访问修饰符**

访问修饰符分为：public、protected、private
- public:公有修饰符，可以在类内或者类外使用public修饰的属性或者行为，默认修饰符
- protected:受保护的修饰符，可以本类和子类中使用protected修饰的属性和行为
- private : 私有修饰符，只可以在类内使用private修饰的属性和行为。
```
class Personer {
    public sex:string;
    protected name:string;
    private age:number;
    constructor(sex:string, name:string, age:number) {
        this.sex = sex;
        this.name = name;
        this.age = age;
    }
    public sayHello() {
        console.log("hello");
    }
    protected sayLove() {
        console.log("I love you");
    }
}
let hanmeimei:Personer = new Personer("女", "韩梅梅", 18)
console.log(hanmeimei.sex);
console.log(hanmeimei.name); //报错
console.log(hanmeimei.age); //报错
hanmeimei.sayHello();
hanmeimei.sayLove() //报错
```
**只读修饰符**

使用readonly修饰符将属性设置为只读，只读属性必须在生命时或者构造函数里被初始化（注意）
```
class Man {
    public readonly sex:string = '男';
}
let man:Man = new Man()
man.sex = "女" //报错
```

## 面向对象编程-继承和重写

**类的继承**

继承：允许我们创建一个类（子类），从已有的类（父类）上继承所有的属性和方法，子类可以新建父类中没有的属性和方法，我们先创建一个父类:
```
class Parent{
    public name:string;
    public age:string;
    public skill:string;
    constructor(name:string, age:string, skill:string) {
        this.name = name;
        this.age = age;
        this.skill = skill;
    }
    public aihao() {
        console.log("打球");
    }
}
let myParent:Parent = new Parent("JackeyLove", "18", "web")
myParent.aihao();
```
再创建子类，要求包含父类的信息并有属于自己的技能，用到关键词 extends 如：
```
class Son extends Parent{
    public mySkill:string = "读书";
    public getMenoy() {
        console.log("一天赚一亿");
    }
}
let mySon:Son = new Son("son", "3", "吃"); //继承父类的属性
mySon.aihao();
mySon.getMenoy()
```

**类的重写**: 重写就是在子类中重写父类的方法,如：
```
class Son extends Parent{
    public mySkill:string = "读书";
    public aihao() { //重写aihao方法
        super.aihao() //用super调用父类的方法
        console.log("看定影");
    }
    public getMenoy() {
        console.log("一天赚一亿");
    }
}
```

## 面向对象编程-接口
定义接口的关键字是**interface**

```
interface Personer {
    sex:string
    interest:string
    maiBaoBao?:Boolean
}
let jackson:Personer ={ sex:'男',interest:'看书、作家务',maiBaoBao:true}
```
## 面向对象-命名空间

命名空间，又称内部模块，被用于组织有些具有内在联系的特性和对象。我们来看一个例子：

```
namespace shuaiDe{
    export class Dehua{
        public name:string = '刘德华';
        say() {
            console.log("我是刘德华");
        }
    }
}

namespace bajie{
    export class Dehua{
        public name:string = '马德华';
        say() {
            console.log("我是二师兄");
        }
    }
}
let dehua1:shuaiDe.Dehua = new shuaiDe.Dehua();
dehua1.say()
```
这样就能避免多人开发时造成的命名污染