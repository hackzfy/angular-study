# 更快的Angular应用程序

应用程序的运行时性能完全掌握在开发者自己手中。接下来，我们以一个简单的应用来演示如何进行性能优化。

## 一个简单的应用

我们将会在这个应用程序中，一步一步的引入所有可行的性能优化方案。

![应用概览](./image/app.gif)

在图片中我们可以看到应用的概览，包含以下功能：

1. 两个部门的职员列表（销售部和研发部）
2. 每个职员对应一个名字和一个数字。数字是通过计算生成的，不需要关注太多细节，记住是通过计算生成的就好。
3. 数字右边有一个删除按钮，可以删除对应的职员。
4. 每个部门列表有一个输入框，我们可以输入员工姓名，将其添加到列表中，同时会计算出一个数字与该员工对应。

就这么多，应用的组件结构如下：
![组件结构](image/structure.png)

`AppComponent` 包含两个用来显示部门职员列表的 `EmployeeListComponent`。

看一下 `EmployeeListComponent` 的模板代码：

```html
<h1 title="Department">{{ department }}</h1>

<mat-form-field>
  <input placeholder="Enter name here" matInput type="text" [(ngModel)]="label" (keydown)="handleKey($event)">
</mat-form-field>

<mat-list>
  <mat-list-item *ngFor="let item of data">
    <h3 matLine title="Name">
      {{ item.label }}
    </h3>
    <mat-chip-list>
      <mat-chip title="Score" class="mat-chip mat-primary mat-chip-selected" color="primary" selected="true">
        {{ calculate(item.num) }}
      </mat-chip>
    </mat-chip-list>
    <i title="Delete" class="fa fa-trash-o" aria-hidden="true" (click)="remove.emit(item)"></i>
  </mat-list-item>
</mat-list>
```

首先，我们显示出部门名称。然后，我们创建了一个文本域用来输入职员姓名。最后，我们使用一个列表来显示所有该部门下的职员。 `calculate` 方法计算出员工对应的数字。代码如下：

```ts
const fibonacci = (num: number): number => {
  if (num === 1 || num === 2) {
    return 1;
  }
  return fibonacci(num - 1) + fibonacci(num - 2);
};

@Component(...)
export class EmployeeListComponent {
  @Input() data: EmployeeData[];
  @Input() department: string;

  @Output() remove = new EventEmitter<EmployeeData>();
  @Output() add = new EventEmitter<string>();

  label: string;

  handleKey(event: any) {
    if (event.keyCode === 13) {
      this.add.emit(this.label);
      this.label = '';
    }
  }

  calculate(num: number) {
    return fibonacci(num);
  }
}
```

`EmployeeListComponent` 的定义很简单：

- 两个Input：
  - data 部门职员列表
  - department 部门名称
- 两个Output：
  - remove 删除职员时触发
  - add 新增职员时触发

Input 属性将由 `AppComponent` 传递给 `EmployeeListCompoment`。

## 业务的计算

我们使用 `calculate` 方法来模拟业务的计算逻辑，它只是调用了 fibonacci 函数而已。真实的业务逻辑过于复杂，而且对我们的演示来说没有必要。

### 应用结构回顾

1. 一个根元素 `AppComponent`.
2. 两个 `EmployeeListComponent`.
3. `EmployeeListComponent` 会通过计算分配给职员一个数字。

## 输入速度

你可以从下面的图片中看到输入体验，这是当两个列表共有140条记录时的状态。注意，我们的打字速度是极快的，而界面出现了明显的卡顿。

![slow-typing](image/slow-typing.gif)

很明显有问题，我们打开 Chrome DevTools > Performance 进行 debug。

![script-time](image/script-time.png)

从截图中可以看到，占用时间最长的是脚本运算。如果我们转到 Bottom-Up 标签页，可以看到如下情况：

![fibonacci-slow](image/fibonacci-slow.png)

可以看到函数 `fibonacci` 占用了几乎所有的脚本运算时间。通过在 `EmployeeListComponent` 的  `calculate` 方法中添加 log 信息，我们可以看一下它的执行频率：

![why-slow](image/why-slow.gif)

从图中可以看到，当我们输入一个字符时， `fibonacci` 函数至少对列表中的每一项调用了两次 （ 在非 production 模式下为四次）。而且，在输入框获取焦点时，和失去焦点时，也会调用相同的次数。即使列表中的每一项其实已经经过计算并且显示在界面上，应用还是为他们重新进行计算。之所以出现这种情况，是因为 Angular 的变更检测在上述事件发生时会被触发。当变更检测被触发后，所有模板中的表达式将被重新计算，并和原先的值进行比较，如果发生改变，则更新 DOM。这意味着，每一次变更检测，都会导致模板中的表达式被重新计算。所以，要避免在模板中插入运算量大的表达式。

现在，我们很清楚列表中的数据并没有发生该变（仅仅是输入了职员姓名，还未添加到列表之前），Angular 不需要在这时更新 DOM。问题是，如何告诉 Angular 此时不需要进行变更检测？

## On Push Change Detection Strategy

答案是使用另一种变更检测机制。实际上，`OnPush` 变更检测机制正是我们寻找的方式。通过设置 `changeDetection: ChangeDetectionStrategy.OnPush`,可以让 Angular 明白只有在组件的 Input 属性发生变化时，才运行变更检测。**注意 Angular 使用 `===` 来比较值是否发生了改变**。

## Components as Functions

为了解释 OnPush 的运行方式，我们将 EmployeeListComponent 以函数的方式进行模拟。假设函数的参数就是组件的 Input 属性，函数的返回值就是界面渲染的 DOM 结构。

```ts
function runChangeDetection() {
  console.log('Detecting for changes');
}

function EmployeeListComponent(args) {
  const shouldRun = Object.keys(args).reduce((a, i) => {
    return a || args[i] !== EmployeeListComponent[i];
  }, false);

  Object.keys(args).forEach(i => {
    EmployeeListComponent[i] = args[i];
  });

  if (shouldRun) {
    runChangeDetection();
  }
}

const f = EmployeeListComponent;
const data = [e1];
```

上面的代码中，我们把 EmployeeListComponent 作为一个函数，有三个要素：

```ts
const shouldRun = Object.keys(args).reduce((a, i) => {
  return a || args[i] !== EmployeeListComponent[i];
}, false);
```

当 args 的每一项都和函数对应的属性值相等时，返回false，只要有一项不相等，则返回 true。

在这之后将函数的属性值更新：

```ts
Object.keys(args).forEach(i => {
  EmployeeListComponent[i] = args[i];
});
```

最后,如果值发生了改变，就运行变更检测。

```ts
if (shouldRun) {
  runChangeDetection();
}
```

假如传给 EmployeeListComponent 的参数中包含一个属性名 data.

```ts
let arr=[];
EmployeeListComponent({data: arr});
```

那么，当运行以下代码时：

```ts
arr.push(1);
```

变更检测不会发生，因为 data 属性的值还是指向同一个数组 arr. 使用 === 比较，是相等的。

可以通过下面的方式，来改变 arr 的引用。

```ts
arr.push(1);
arr = arr.slice()
```

这时， Angular 才会认为 Input 属性的值发生了变化，运行变更检测。

经过梳理后，我们将应用的组件更新如下：

```ts
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  ...
})
export class EmployeeListComponent {
  @Input() data: EmployeeData[];
  @Input() department: string;

  handleKey(event: any) {
    // 当按下回车时，触发 add 事件
    if (event.keyCode === 13) {
      this.add.emit(this.label);
      this.label = '';
    }
  }
  ...
}

```

AppComponent

```ts
@Component({
  template: `
    <sd-employee-list [data]="salesList" department="Sales" (add)="addToSales($event)">
  `
})
export class AppComponent implements OnInit {
  salesList: EmployeeData[];

  addToSales(name: string) {
    this.salesList = this.salesList
      .unshift({ label: name, num: this.generator.generateNumber(NumRange) })
      // 注意这里
      .slice();
  }
  ...
}
```

这样，每当用户按Enter键时，我们将使用add事件发出标签的值。AppComponent将使用其addToSales方法处理输出，该方法将新职员推送到salesList，复制整个数组，并将返回的新引用设置为salesList的值。
然而，我们引入了两个问题:它将是缓慢的：

- 每次添加员工时，我们都需要复制整个数组。垃圾收集器还应该运行以清理未使用的内存（原来的arr)。
- 它需要很多内存。我们为新数组分配的内存可能很大，这取决于数组的大小。为了处理这两个问题，我们可以使用高效实现的不可变数据结构，如immutable .js。

## 引入 Immutable.js

Immutable.js提供了一组不可变的数据结构。它们都有两个基本属性:

1. 它们是不可变的(显然)，因此当我们打算应用一个操作来改变这些数据结构中的任何一个实例时，我们将得到一个新实例(分别具有一个新引用)。
2. 基于immutable.js的修改操作生成新的数据结构，不会复制整个数据结构，它将尽可能多地重用原数据结构。

这是第一点的一个例子：

```ts
import { List } from 'immutable';

const data = List([a, b, c]);

console.log(data.toJS()); // [a, b, c]

const appended = data.push(d);

console.log(data.toJS()); // [a, b, c]
console.log(appended.toJS()); // [a, b, c, d]
```

从上面的代码片段可以看出，当我们调用List实例的push方法时，我们会得到一个新的List。最重要的是，原来的数据保持不变。
基于第二个属性，我们可以得出这样的结论:来自immutable.js的不可变数据结构将比我们在上面的示例中复制整个数据结构更有效率。

## OnPush And Immutable.js

下面优化前后的性能对比图：

![onpush-vs-non-optimized](image/onpush-vs-non-optimized.png)

看起来我们得到了一个很好的优化。我们快了三倍!根据基准测试，在销售部输入字符串AngularConnect，对于非优化版本，输入13124.81ms，对于优化版本，输入5804.12ms。
现在我们看一下应用的体验：

![slow-on-push](image/slow-on-push.gif)

。。。依然很慢。正如我们所看到的，calculate方法执行的频率较低，但调用的次数仍然很多。如果我们仔细观察，可以注意到，这一次我们只重新计算销售部所有员工的数值。OnPush优化几乎成功了。

在输入时（按 Enter 之前），我们不会得到新的salesList实例，因为我们没有进行任何改变它的值的操作。为什么当前组件依然触发了变更检测？我们不是使用了 ChangeStrategyDetection.OnPush 吗？

答案在于OnPush的工作方式。使用OnPush变更检测策略，当我们将一个新值传递给组件的 Input 属性**或当组件内部发生事件时**，将触发该组件的变更检测。第二部分（加粗部分）在文档中不是很明显，但是在Angular core 仓库中的e2e测试中可以看到。

## 组件分割

为了解决这个问题，我们需要进行一些重构。这不仅可以帮助我们消除不必要的变更检测调用，还可以在应用程序中实现更好的逻辑解耦。

为此，我们将EmployeeListComponent分解为:

- NameInputComponent——负责保存要添加到列表中的职员的名称。
- ListComponent——将列出各个职员并计算他们的数值

![enforce-separation-of-concerns.png](image/enforce-separation-of-concerns.png)

现在看一下优化后的性能对比：

![fast-typing.png](image/fast-typing.png)

看起来图表上有个小故障。不，没有，我们只是快了几百倍——13124.81ms vs. 10.45ms。输入字符串为 “AngularConnect”。

这是用户在优化版本输入文本时的体验：

![fast-typing.gif](image/fast-typing.gif)

在“更快的Angular应用程序”系列的这一部分中，我们讨论了如何通过使用不可变的数据结构和自定义的变更检测机制来优化Angular应用程序的运行时性能。

我们首先介绍了一个示例业务应用程序，它列出了职员的两个部门，并使用大量计算为每个员工计算一个数值。

由于我们使用双向数据绑定，应用程序的相应出现了一些明显的迟滞——输入新职工的速度很慢。为了减少计算量，我们使用了OnPush变化检测策略和不可变数据结构。

尽管OnPush带来了一个显著的改进，但我们忽略了一个非常重要的细节——当给定组件中的一个事件被触发时，它会导致Angular运行变更检测。我们通过分解EmployeeListComponent解决了这个问题。

本文摘译自： https://blog.mgechev.com/2017/11/11/faster-angular-applications-onpush-change-detection-immutable-part-1/

github仓库地址： https://github.com/mgechev/optimizing-an-angular-application.git

ng-conf 专题视频地址：https://www.youtube.com/watch?v=ybNj-id0kjY
