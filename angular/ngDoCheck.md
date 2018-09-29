# ngDoCheck

> 官方文档上是这样解释的：
>
> Detect and act upon changes that Angular can't or won't detect on its own.
>
> Called during every change detection run, immediately after `ngOnChanges()` and `ngOnInit()`.
>
> 中文官网：
>
> 检测，并在发生 Angular 无法或不愿意自己检测的变化时作出反应。
>
> 在每个 Angular 变更检测周期中调用，`ngOnChanges()` 和 `ngOnInit()` 之后。
>
> 
>
> 中文文档确实翻译的不咋地，但是好歹我们能看出一些端倪。重点有两个：
>
> 1. Angular 自己无法检测到变化时可以做出反应。
> 2. 在每个 Angular 变更检测周期中调用。
>
> 首先看第一点：
>
> Angular 自己无法检测到变化时。Angular 还有检测不到变化的时候吗？
>
> 有的。下面是个例子：



```typescript
// 这个父组件包含两个子组件
@Component({
  selector: 'app-optimization',
  template: `
    <app-name-input (add)="add($event)"></app-name-input>
    <app-employee-list [data]="list"></app-employee-list>
  `,
  styles: []
})
export class OptimizationComponent {
	list: EmployeeData[] = [{name: 'john', num: 1}];
 
    add(name: string) {
        const employee: EmployeeData = {
          name,
          num: Math.random(),
        };
        this.list.unshift(employee);
  }
}
```



> 第一个子组件：NameInputComponent

```typescript
@Component({
  selector: 'app-name-input',
  template: `
    <div>
      <input [(ngModel)]="name"
             (keydown.enter)="addEmployee()">
    </div>
  `,
  styles: []
})
export class NameInputComponent {
  name: string;
  @Input() data: EmployeeData[];
  @Output() add = new EventEmitter();

  addEmployee() {
    this.add.emit(this.name);
    this.name = '';
  }

}
```

> 第二个子组件 EmployeeListComponent

```typescript
@Component({
  selector: 'app-employee-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <ul>
      <li *ngFor="let item of data">
        <span>{{item.name}}</span>
        <span class="score">{{item.num}}</span>
      </li>
    </ul>
  `,
})
export class EmployeeListComponent implements {
  @Input() data: EmployeeData[] = [];
}
```

> 用到的模型：EmployeeData

```typescript
export interface EmployeeData {
  name: string;
  num: number;
}
```

> 当你在input组件中输入字母并敲下回车时，发现新的数据并没有添加到列表中。Angular 没有检测到变化的情况出现了。看一下出现的条件：
>
> * 组件使用了 OnPush 的 检测机制。
> * Componet的 Input 没有发生变化。
> * 组件内部没有任何事件被触发。
>
> 看下源码中的关于 ChangeDetectorRef 的 markForCheck 方法的注释：
>
> ```
> This can be used to ensure an {@link ChangeDetectionStrategy#OnPush OnPush}  component is checked when it needs to be re-rendered but the two normal triggers haven't marked it dirty (i.e. inputs haven't changed and events haven't fired in the view).
> ```
>
> 里面提到了所有的三个条件。
>
> 我们的 EmployeeListComponent 组件恰好满足了所有条件：
>
> 1. 使用了 OnPush 变更检测机制。
> 2. Input 属性没有变化。（Angular 使用===检测变化，对象的引用没有改变）
> 3. 组件内部没有发生事件。（事件是在 NameInputComponent 组件中发生的）
>
> 我们要想办法让列表更新才行。这时候 ngDoCheck 这个钩子函数就派上用场了。

```typescript
export class EmployeeListComponent  implements OnInit,DoCheck{
  @Input() data: EmployeeData[] = [];
  oldLength:number;
  // 依赖注入 ChangeDetectorRef
  constructor(private cd: ChangeDetectorRef) {
  }

  ngOnInit() {
    // 初始化时保存 data 的 length
    this.oldLength = this.data.length;
  }

  ngDoCheck() {
    // 即使Input属性没有发生变化，最顶层的OnPush Component的 ngDoCheck
    // 依然会被调用
    console.log('list checked');
    // 通过对比 data 现在的 length 和从前的 length，
    // 来手动触发变更检测
    if (this.data.length !== this.oldLength) {
      // 这个方法以会 将本组件以及所有的父组件都标记为 dirty ，
      // angular就会对这一整个树的组件进行变更检测。
      this.cd.markForCheck();
      this.oldLength = this.data.length;
    }
  }
}
```

> 现在，输入一个名子，回车后，它被添加到列表中了
>
> 关于 Angular 的变更检测，知识点真的太多。需要耐心，一点一点去挖。欢迎入群交流。