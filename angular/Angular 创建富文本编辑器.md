# Angular 创建富文本编辑器

在创建富文本编辑器之前，首先了解一下如何获取 dom 的引用。

> ViewChild：
>
> 在模板中使用模板变量标记元素，然后在 Component 类中使用 @ViewChild 来获取元素的引用。

```ts
import {Component, ElementRef, OnInit, ViewChild} from '@angular/core';

@Component({
  selector: 'app-find-element',
  template: `
              <input type="text"
                     value="123"
                     #input>
            `,
  styles:   [],
})
export class FindElementComponent implements OnInit {

  @ViewChild('input') input: ElementRef<HTMLInputElement>;

  constructor() {
  }

  ngOnInit() {
    console.log('onInit', this.input.nativeElement.value); // 123
    console.log('onInit', this.input.nativeElement.offsetWidth); // 132
  }

}
```

> 如果想获取 host 元素，需要在 component 的 constructor 中注入 ElementRef。

```ts
import {Component, ElementRef, OnInit, ViewChild} from '@angular/core';

@Component({
  selector: 'app-find-element',
  template: `
              <input type="text"
                     value="123"
                     #input>
            `,
  styles:   [],
})
export class FindElementComponent implements OnInit {

  @ViewChild('input') input: ElementRef<HTMLInputElement>;

  constructor(private host: ElementRef) {

  }

  ngOnInit() {
    console.log(this.host.nativeElement);
    // <app-find-element><input type="text" value="123"></app-find-element>
  }

}
```

接下来，就可以创建富文本编辑器了。

```
npm i --save @ckeditor/ckeditor5-build-classic
```

创建一个组件

```ts
import {
  Component,
  OnInit,
  ViewChild,
  ElementRef,
  AfterViewInit
} from '@angular/core';

import ClassicEditor from '@ckeditor/ckeditor5-build-classic';

@Component({
  selector: 'app-editor',
  template: `
  <textarea #container>
    <p>
      editor works!
    </p>
  </textarea>
  `,
  styles: []
})
export class EditorComponent implements AfterViewInit {
  @ViewChild('container') container: ElementRef;
  constructor() {}

  ngAfterViewInit(): void {
    const c = this.container.nativeElement;
    ClassicEditor.create(c
      .then(editor => {
        console.log(editor);
      })
      .catch(error => {
        console.error(error);
      });
  }
}

```

看一下成果：

![屏幕快照 2018-07-24 下午10.16.21](/Users/yang/blog/屏幕快照 2018-07-24 下午10.16.21.png)