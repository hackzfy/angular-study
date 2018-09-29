# Angular Router 跳转到指定 Tab 页

在开发中遇到一个需求：A页面包含一个 tab 组件，不同的 tab标签点击后显示不同的内容。B页面中包含一组 链接，要求点击链接时跳到A页面对应的 tab 标签页下。看了下官方文档，实现起来很简单。需要注意三个点：

- Router
- ActivatedRoute
- ParamMap

B页面具体实现如下：

```type
@Component({
  selector: 'app-link-page',
  template: `
   <span (click)="toTab(1)">to tab1</span>
   <span (click)="toTab(2)">to tab2</span>
   <span (click)="toTab(3)">to tab3</span>
  `,
  styles: []
})
export class LinkPageComponent implements OnInit {
  constructor(private router: Router) {}

  ngOnInit() {}

  toTab(index) {
  	// 注意这里参数的传递方式
    this.router.navigate(['tab-page', { tab: index }]);
  }
}
```

A页面实现如下：

```ts
@Component({
  selector: 'app-tab-page',
  template: `
   <ul>
    <li [class.active]="tab==1">tab1</li>
    <li [class.active]="tab==2">tab2</li>
    <li [class.active]="tab==3">tab3</li>
   </ul>
  `,
  styles: [
    `
      li {
        display: inline-block;
        list-style: none;
      }
      .active {
        background: blue;
        color: white;
      }
    `
  ]
})
export class TabPageComponent implements OnInit {
  tab: any = 1;
  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    this.route.paramMap
      .pipe(
        switchMap(params => {
          this.tab = params.get('tab');
          return null;
        })
      )
      .subscribe();
  }
}
```

也可以用其它的方式，Angular 非常灵活。