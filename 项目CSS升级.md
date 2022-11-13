## CSS的7种方式，你知道几种?

CSS作为前端三剑客之一，几乎是所有前端同学最先学习的样式表语言。在生产环境的项目工程中，很少见到直接原生使用CSS的。

目前业界还没有通用的CSS工程化方案。这篇文章简单介绍下7种在React/Next.js中较为流行使用CSS的方式，并说说他们的优缺点。

### 原生 CSS
这是一种用选择器来划分css作用域的方式。
- 缺点：
1. 作用域问题
CSS样式之间会层叠覆盖，需要用大量的classname来指定选择器，来限定CSS的作用域范围。频繁的命名给开发人员增加不少心智负担，而且容易搞错搞重复。
```
// pure css example
.card {
	/* styles */
}
.card__header {
	/* styles */
}
.card--focus {
	/* styles */
}
```
采用BEM规则来进行命名或许会简单些。
但在需要维护特别多样式的时候，BEM还是不够用。尤其是当代码中开始大量出现```!important```这种破坏优先级的东西的时候。
```
// css with !important
.card {
	color: blue !important;
}
.card {
	color: red;
}
```
2. 打包体积大
使用大量冗长的原生CSS，可能会导致
打出来的包变大。包越大，项目自然跑的就越慢。

### CSS MODULES
这是一种在原生CSS的基础上，通过modules（也可以理解为文件）来划分CSS的作用域。

首先先建一些以.module.css结尾的文件，这些文件里的样式可以只针对某个组件（某个module）生效。这种做法在Next.js尤为常见，因为CSS modules在Next.js是可以开箱即用的。

下面是一个例子，在```Home.module.css```和```other.module.css```对**同样的类名**书写样式，也不会产生冲突。
```
@file Home.module.css
.page {
	color: bule;
}
```
```
@file other.module.css
.page {
	color: yellow;
}
```
```
// 只会生效这里import的样式
import styles from '../styles/Home.module.css';
export default function Home(){
	return (
		// 蓝色
		<div className={styles.page}>
			<h1> Home Page </h1>
		<div>
	)
}
```
- 优势：

1. 当需要复用样式的时候，不同的组件可以import同一份样式文件，减少很多重复样式代码，减轻打包体积～
2. 说到样式复用，CSS modules还有个特殊的composes属性，能引入别的module的css样式，也能重写（override）。
```
.page {
	composes: className from "./shared.css"
}
```
- 缺点：
1. 不够“程序化”
CSS modules在原生CSS的基础上增加了以modules（文件）划分的作用域，解决了作用域问题，但仍逃不过在单个module内以原生的方式书写CSS。原生的CSS只能纯纯的枚举出每一条样式，如果能在书写CSS的时候也支持一些程序特性岂不是更好？比如最常用的循环、遍历、函数、继承...

### CSS PREPROCESSOR 预处理器
Sass、Less、Stylus... 这些预处理器就是为了解决CSS不够“程序化”而诞生的。他们允许你用一种不一样的语法来写CSS，之后再经过编译转化成原生CSS。

这里是一个例子：
```
// 只需一键安装sass
$ npm install sass
```
```
// 然后把原本的css后缀文件改成scss
// 就可以直接使用sass的方式来编写css啦，比如变量名、循环、...
@ file Home.module.scss

$ primary-color: red;
$ font-stack：Helvetica

body {
	font: 100% $font-stack;
	color: $primary-color;
}
```
- 优势：
1. 可以用变量、继承、循环、函数、...等程序特性
- 缺点：
1. 学习成本
每种预处理器都有各自特定的语法，虽然是用一种类CSS的语言来编写，但总有有些差异。这意味着开发人员必须配合工具掌握新的语法。
2. 样式和项目代码微微割裂
在解决完作用域、程序化问题后，样式在前端项目中完完全全的独立出来了，似乎少了一些联动能力。既然我们有JSX这样整合JS和HTML的合体语言，为什么不能把CSS也合体进来呢？

### CSS IN JS
这是一种把CSS写进JS的解决方案，就像把HTML写进JS后就有了JSX。这一类的库有styled components、emotion、jss、style tron、...

举个使用styled jsx的例子：
```
import styles from '../styles/Home.module.css';

export default function Home(){
	const [color, serColor] = useState('orange');
	return (
		<div className={styles.page}>
			<style jsx>{`
				h1 {
					// 取的是组件state，可以随state变化！
					color: ${color};
				}
			`}</style>
			<h1> Home Page </h1>
		<div>
	)
}
```
- 优势：
1. 轻松能实现的程序化能力
在sass里的程序化能力，CSS in JS都能做到，甚至更强，这种方式可以直接使用JS书写这种程序化语言，也不需要额外学习成本。
2. 创建动态样式
在sass里，程序代码或许和样式文件是完全独立开来的。而使用CSS in JS，样式和JS强绑定，我们的样式能够跟着代码、跟着组件的state等特性实现动态样式，特别灵活！
3. 不会有作用域问题
类似module，CSS in JS的样式只会绑定在样式定义的组件内，不会污染全局样式～
- 缺点：
1. CSS和JS混写，代码管理困难。

### UTILITY CLASSES 原子类
时下最火的新概念就是tailwindcss、windi css这些原子类CSS库，能够提供大量的原子类样式，帮助我们快速构建样式。
```
// 配置好tailwind之后
export default function Home(){
	return (
		// 在这里写上tailwind的原子类classname，而不需要写样式
		<div className="text-5xl font-bold">
		<h1> Home Page </h1>
		<div>
	)
}
```
- 缺点
1. 需要比较麻烦的额外配置
2. 打包后，生成的HTML文件可读性非常非常低 
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77ff846122eb448ab0d3129739c252e1~tplv-k3u1fbpfcp-watermark.image?)
3. 没有任何的内置组件
- 优势
1. 打包时，能自动优化，去除没有使用的css样式，减轻打包产物体积。

### CSS FRAMEWORK
bootstrap、bulma、这一类库既能提供特定的样式主题，又有内置的组件，比如bottom、cards、...等等。我个人在自己倒腾东西的时候非常喜欢用这一类框架，因为实在是太方便啦！这种方式在生产上几乎很少采用，因为开发人员往往需要根据产品原型来绘制前端界面，而不是这些框架固定的样式。另外采用这种方式，也容易对线上性能造成比较大影响。
```
// 想使用这一类框架，只用一键安装上
$ npm install bootstrap
```
```
// 引入框架的样式文件
import 'bootstrap/dist/css/bootstrap.css'

export default function Home(){
	return (
		// 想要使用的样式都在bootstrap中用各种classname封装好啦，直接调用boostrap的预留classname，搞定
		<div className="alert alert-primary">
		<h1> Home Page </h1>
		<div>
	)
}
```
- 缺点：
1. 在只使用bootstrap来搭建组件和修改样式的话，会不太方便
由于这类框架已经自带了许多预留组件，而bootstrap的样式又是用classname来获取的。假设我需要频繁使用```<Bottom />```组件，却又不想在每次使用的时候，都重复的写相同的classname，那么就会将他们封装成```<CustomButtom />```。这么做的话，项目代码中就可能会有大量的仅仅是为了封装classname而存在的组件。
2. 打包文件过大
整个bootstrap文件是直接import进来的。因此在打包时，会把大量没使用到的classname也打包进来，会造成打包产物较大～

### 组件库
这是大家最熟悉的方式啦，ant design、material design、t design、rebase、....

### 我用的
不同的CSS处理方式各有优劣，在实际开发中，可以自行选择和搭配合适的CSS处理手段。

在我目前工作中，是将项目的原生CSS，升级成css module + less 的组合，这样既能解决当前项目的核心矛盾：作用域和样式污染问题，又能让CSS的编写过程变得更“程序”，比如使用变量、继承等特性。

没有使用css in js 是因为当前项目没有主题切换和动态样式这样场景，此外css in js 会让一个组件文件变得非常冗长，尤其是目前我的工作特别多复杂图表的封装，仅jsx部分代码行数已经非常长，再引入CSS代码容易变得更混乱。我个人也更加偏向能用独立文件区分出CSS代码的方式，这样展示出更好的项目分层。

没有使用原子类的理由就更简单了，配置麻烦，可读性低，而且对团队内每个人都有较高的学习成本，不方便团队管理，直接pass了。

在前端工程开发的过程中，面对多人协作的场景，建立标准和团队内的规范是非常重要的一个环节。尤其当前业界的前端，就是没有通用标准的情况下，影响项目工程稳健性的往往是缺乏规范和标准，而不是开发人员的水平。在我工作的项目中，最初就是因为大量人员流动，大家在项目中各按各自的方式写CSS，导致在一个项目中存在3种以上CSS写法，非常难维护，也出现了样式互相污染、互相冲突的情况，所以才有了这次对CSS的调研，以及对项目进行升级和改造的工作。


附上一些参考资料：

https://webflow.com/blog/class-naming-101-bem

https://www.nextjs.cn/

https://www.tailwindcss.cn/

https://www.youtube.com/watch?v=ouncVBiye_M

https://www.youtube.com/watch?v=hdGsFpZ0J2E
