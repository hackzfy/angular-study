# Angular 开发最佳实践 — 不应该做的事

## 重复代码

```js
@Component({
  selector: 'app-some-component-with-form',
  template: `
        <div [formGroup]="form">
          <div class="form-control">
            <label>First Name</label>
            <input type="text" formControlName="firstName" />
          </div>
          
          <div class="form-control">
            <label>Last Name</label>
            <input type="text" formControlName="lastName" />
          </div>
          
          <div class="form-control">
            <label>Age</label>
            <input type="text" formControlName="age" />
          </div>
        </div>
    `
})
export class SomeComponentWithForm {
  
  public form: FormGroup;
  
  constructor(private formBuilder: FormBuilder){
    this.form = formBuilder.group({
      firstName: ['', Validators.required],
      lastName: ['', Validators.required],
      age: ['', Validators.max(120)],      
    })
  }
  
}
```

> 这种情况你应该将其封装为子组件。

```js
// 子组件
@Component({
  selector: 'app-single-control-component',
  template: `
        <div class="form-control">
          <label>{{ label }}</label>
          <input type="text" [formControl]="control" />
        </div>
      `
})
export class SingleControlComponent{
  @Input() control: AbstractControl 
  @Input() label: string;
}

// 父组件
@Component({
  selector: 'app-some-component-with-form',
  template: `
        <div>
  			<app-single-control-component 
				[control]="form.controls['firstName']"
				[label]="'First Name'" />
              <app-single-control-component 
                [control]="form.controls['lastName']" 
                [label]="'Last Name'" />

              <app-single-control-component 
                [control]="form.controls['age']" 
                [label]="'Age'" />
         </div>
    `
})
export class SomeComponentWithForm {
  
  public form: FormGroup;
  
  constructor(private formBuilder: FormBuilder){
    this.form = formBuilder.group({
      firstName: ['', Validators.required],
      lastName: ['', Validators.required],
      age: ['', Validators.max(120)],      
    })
  }
  
}
```

> 如果情况就是这么简单，可以通过 ngFor 把它循环出来。



## 使用 toPromise（）

使用 toPromise 的劣势：

1. 添加非必要的操作符。你没有必要将一个 Observable 转换为一个 Promise，你可以直接使用它。
2. 失去了很多 RxJS 提供的强大功能。RxJS 大量的操作符让你可以自如的操作数据流，而转换为 Promise 之后，大部分功能都丢失了，你只能使用 Promise 提供的少得可怜的功能。



## 不经常使用RxJS

RxJS是一个优秀的工具，你应该使用它来操作数据，事件，以及应用中的各种状态。尽快掌握它。



## 没有给数据定义interface

比如说从服务器端请求回来的数据，一定要定义 interface。以后操作的时候，typescript 会给你精准的提示，而当你写错时，也会及时提醒你。

```js
// server response
{
    success: true,
    code: 200,
    users:[
            {name:'aaa',age:20,favo:['singing','writing']}
    ]
}

// interface

export interface UsersResponse{
    success: boolean;
    code: number;
    users: User[]
}
export interface User{
    name:string;
    age: number;
    favo:Favo[]
}
export type Favo = 'singing'|'writing';

// 使用时如果写错的话
userResponse.username // error! 没有 username 这个属性
userResponse.favo = 'singing' // error! favo 是一个数组类型，不能赋予 string 类型的值
userResponse.favo[0] = 'swimming' // error! favo 只可能是 'singing' 或者 'writing'
```

## 在Component 中修改数据

下面是一个例子

```js
interface Movie {
  id: number;
  title: string;
}

@Component({
  selector: 'app-user-form',
  template: `
	<form [formGroup]="form">
		<input type="text" formControlName="firstName" >
		<input type="text" formControlName="lastName" >
		<input type="number" formControlName="age" >
		<select formControlName="favoriteMovies" multiple>
			<option *ngFor="let movie of movies" [value]="movie">
				{{movie.title}}
			</option>
		</select>



	</form>
  ` 
})
export class SomeComponentWithForm {

  public form: FormGroup;
  public movies: Array<Movie>

  constructor(private formBuilder: FormBuilder){
    this.form = formBuilder.group({
      firstName: ['', Validators.required],
      lastName: ['', Validators.required],
      age: ['', Validators.max(120)],
      favoriteMovies: [[]],  /*喜爱的电影，多选*/
    });
  }
  
  public onSubmit(values){ 
    /*
     'values' 是表单要提交的值，包含 form 内的所有字段。但是假设服务器端要求喜欢的电影是
      一个包含电影 id 的数组，而不是一个电影的数组，那我们需要对数据进行一些预处理。
    */
    values.favouriteMovies = values.favouriteMovies.map((movie: Movie) => movie.id);
    /* 处理过后，可以通过 service 将数据提交给服务器了。*/ 
  }
}
```

目前，这样的方式并没有带来什么问题。仅仅在提交数据之前做了一点点修改数据的工作。但是想象一下，如果待提交的数据和很多其他因素有关联，或者字段在数据中所处层次较深，而数据的结构又较复杂时，这段代码会立刻变得臃肿。一个好的实践是下面这样做：

```js

interface Movie {
  id: number;
  title: string;
}

interface User {
  firstName: string;
  lastName: string;
  age: number;
  favoriteMovies: Array<Movie | number>; 
  /*
  这里喜爱的电影是一个数组，
  可以是 movie类型，
  也可以是number类型（id)。
  但是，不能既有 movie 又有 number。
  */
}

class UserModel implements User {
  firstName: string;
  lastName: string;
  age: number;
  favoriteMovies: Array<Movie | number>;

  constructor(source: User){
    this.firstName = source.firstName;
    this.lastName = source.lastName;
    this.age = source.age;
    const type = typeof source.favoriteMovies[0];
      if( type!=null &&  type!=='number'){
           this.favoriteMovies =
               source.favoriteMovies
               .map((movie: Movie) => movie.id);
      }
    /*
    我们将修改数据的逻辑分离到一个单独的 class 里，
    这个 class 同时也代表了 User 的数据模型。
    Componet 里面会变得简洁清晰
    */
  }
}
```

修改后的 component

```js
public onSubmit(values: User){
    let user: UserModel = new UserModel(values);
}
```

> 原来我们只有一个 interface User，现在增加了一个 interface UserModel。但是代码更加清晰。



## 不用或者滥用 Pipes

下面是一个例子

```js
// 不好的做法
@Component({
  selector: 'some-component',
  template: `
    <div>
      <dropdown-component [options]="weightUnits" />
      <dropdown-component [options]="slashedWeightUnits"/>

    </div>
`
})
export class SomeComponent {
  public weightUnits = [{value: 1, label: 'kg'}, {value: 2, label: 'oz'}];
  public slashedWeightUnits = [{value: 1, label: '/kg'}, {value: 2, label: '/oz'}];
}
```

较好的做法

```js

@Pipe({
  name: 'slashed'
})
export class Slashed implements PipeTransform {
  transform(value){
    return value.map(item => {
      return {
        label: '/' + item.label,
        value: item.value
      };
    })
  }
}


@Component({
  selector: 'some-component',
  template: `
    <div>
      <dropdown-component [options]="weightUnits" />
      <dropdown-component [options]="(weightUnits | slashed)" />
    </div>
`
})
export class SomeComponent {
  public weightUnits = [{value: 1, label: 'kg'}, {value: 2, label: 'oz'}];
}
```

但是，想象一下，如果你的应用中有很多需要转换的东西，但是这些东西的转换通常很少用到，可能只用到一次。那么需要为每一个转换逻辑编写 Pipe 吗？那样会出现一大堆的 Pipe。更灵活的解决办法：



```js

@Pipe({
  name: 'map'
})
export class Mapping implements PipeTransform {
  /* 
  this will be a universal pipe for array mappings. You may add more 
  type checkings and runtime checkings to make sure it works correctly everywhere
  */
   /*
   	这个通用pipe，接收一个转换函数作为参数。
   	因为通用，所以 value 的类型也可能是多种多样的，我们只能使用 any
   	在实际使用的时候，要注意检查参数类型是否正确
   */
  transform(value, mappingFunction: Function){
    return mappingFunction(value)
  }
}


@Component({
  selector: 'some-component',
  template: `
    <div>
      <dropdown-component [options]="weightUnits" />
      <dropdown-component [options]="(weightUnits | map : slashed)" />
    </div>
`
})
export class SomeComponent {
  public weightUnits = [{value: 1, label: 'kg'}, {value: 2, label: 'oz'}];
  
  /* 
  我们把转换逻辑定义在组件内部
  需要注意的是，转换函数必须是 pure function
  不能有任何副作用
  不能使用 this
  */
  public slashed(units){
    return units.map(unit => {
      return {
        label: '/' + unit.label,
        value: unit.value
      };
    });
  }
}
```



> 原文地址：https://codeburst.io/angular-best-practices-4bed7ae1d0b7