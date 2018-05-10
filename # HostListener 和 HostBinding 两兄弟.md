# HostListener 和 HostBinding 两兄弟

> 老实说，我早就想写他们了，可是觉得他们太简单，没什么好说的。
>
> 这兄弟俩都是在 Directive 内部使用的。用处不同，但都一个意思。
>
> HostBinding 监听指令的宿主元素身上的属性。
>
> HostListener 监听指令的宿主元素身上的事件。
>
> 但是它们都很懒，只监听你命令他们去监听的东西。



```typescript
// spy.directive.ts
import {Directive, ElementRef, HostBinding, HostListener} from '@angular/core';

// 这个很明显，是一个指令，而且取了个和合适的名字。
// 指令就像一个间谍，它埋伏在宿主元素身上，监视着其一举一动
@Directive({
  selector: '[appSpy]'
})
export class SpyDirective {
  
  hover = false;
  // 你看，指令一旦被创建出来，它就拥有了对宿主元素的引用！
  // 这意味着，宿主注定要任其摆布！
  constructor(private ele: ElementRef) {
  }
  // 老大出场了，想操控宿主的 style 属性中的 font-size
  // 它得逞了，前端界至高之神 Angular 赋予了它这样的能力。
  // 每当 hover 属性变成 true 时，它就将宿主 style 属性中的 font-size 变成特大。
  // 宿主很无奈，只能任其摆布。
  @HostBinding('style.fontSize')
  get fontSize() {
    return this.hover ? 'xx-large' : 'small';
  }
  // 老大觉得不够过瘾，又来玩 style.color 属性。
  // 宿主继续任其摆布
  @HostBinding('style.color')
  get color() {
    return this.hover ? 'inherit' : '#fff';
  }
  // 老二不甘寂寞，也来了。它想监听宿主身上的 mouseenter 事件，
  // 每当鼠标指针移动到宿主身上，它就把宿主的字体颜色变成红色。
  // 玩的不亦乐乎。
  @HostListener('mouseenter', ['"red"'])
  mouseenter(color: string) {
    this.ele.nativeElement.style.backgroundColor = color;
    this.hover = true;
  }
 // 看到宿主幽怨的眼神，老二决定当鼠标从宿主身上离开时，还宿主一个清白。
 // 可是，what？搞错了颜色。。宿主的脸色变了。。。
  @HostListener('mouseleave', ['"blue"'])
  mouseleave(color: string) {
    this.ele.nativeElement.style.backgroundColor = color;
    this.hover = false;
  }
}

```

> 可怜可怜这个纯真的你好组件吧！

```typescript
// hello.component.ts
import {Component} from '@angular/core';

// 因为太纯真，已经被间谍指令附身，接下来的日子只能听天由命了。。。
@Component({
  selector: 'app-hello',
  template: `
    <p appSpy>
      hello works!
    </p>
  `,
})
export class HelloComponent {
}

```

> 注意参数，注意参数，虽然参数是字符串数组，可你注意到它们有几个引号了么？`['"red"'] or ['red'] `?

