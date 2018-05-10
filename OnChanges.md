#  你了解ngOnChanges吗？

> 作为 Angular 组件生命周期中的一个重要钩子函数，我们需要着重了解三个核心要素：
>
> 1. 什么情况下被调用。
> 2. 什么时间被调用。
> 3. 实际项目中如何使用。



```JS
import {Component, Input, OnChanges, OnInit, SimpleChanges} from '@angular/core';

@Component({
  selector: 'app-on-change',
  template: `
    <div>
      <label>parent:</label><input type="text" [(ngModel)]="value" >
      <app-on-change-child [value]="value"></app-on-change-child>
    </div>`,
  styleUrls: ['./on-change.component.css']
})
export class OnChangeComponent implements OnInit {
  // 父组件的 value 属性不是外部传入的，不是 [input] 属性
  value = 'a';

  constructor() {
  }

  ngOnInit() {
  }

  ngOnChanges(changes: SimpleChanges): void {
    console.group('parent ngOnChanges called.');
    console.log(changes);
    console.groupEnd();
  }

}

@Component({
  selector:'app-on-change-child',
  template:`<label>child:</label><input type="text" [value]="value">`
})
export class OnChangeChildComponent implements OnChanges {
  // 子组件的 value 属性是外部传入的，是一个 【input】 属性
  @Input() value = '';

  ngOnChanges(changes: SimpleChanges): void {
    console.group('child ngOnChanges called.');
    console.log(changes);
    console.groupEnd();
  }

}
```

> 在parent input 中输入 'aaa',输出如下,其中 changes 是一个对象，它的 key 就是组件中发生改变的 input property，value 是一个 SimpleChange 类型的对象。SimpleChange 对象包含3个字段：
>
> 1. previousValue: 发生改变之前的值。
> 2. currentValue: 发生改变之后的值。
> 3. firstChange:  值是否是第一次改变。

![](/Users/yang/Desktop/docs/屏幕快照 2018-05-10 下午7.15.00.png)

> 什么是 [input] 属性？官网对 ngOnChanges 的说明：
>
> Respond when Angular (re)sets data-bound **input properties**. The method receives a `SimpleChanges` object of current and previous property values.
>
> Called before `ngOnInit()` and whenever one or more data-bound input properties change.
>
> 也就是说，只有当设置数据绑定的输入属性时，以及输入属性绑定的数据发生改变时，才会调用此函数。
>
> 这就解释了为什么当在父组件的输入框中输入数据时，并不会调用父组件的 ngOnChanges 方法，因为父组件中的 value 属性并不是一个 **input property**。而子组件因为将父组件的 value 作为输入数据，绑定在自身上， 这个 value 是个 **input property** ，所以会做出响应，调用自己的 ngOnChanges 方法。
>
> 所以，看起来很简单的东西，也可能并不是想象中的那样。

