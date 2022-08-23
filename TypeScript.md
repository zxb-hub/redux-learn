## 接口与类型别名

> 几乎所有的 interface 接口特性都可以在 type 类型别名中使用。

### 区别

- 不能重新在类型中添加新属性，而接口总是可拓展的。
- 类型别名可以作用于原始值（基本类型），联合类型，元组以及其它任何你需要手写的类型。起别名不会新建一个类型 - 它创建了一个新 名字来引用那个类型

### 基础类型定义

- `boolean`:

  ```js
  let isDone: boolean = false;
  ```

- `number`:
  ```js
  let decLiteral: number = 6;
  let hexLiteral: number = 0xf00d;
  let binaryLiteral: number = 0b1010;
  let octalLiteral: number = 0o744;
  ```
- `string`：

  ```js
  let name: string = "bob";
  name = "smith";
  ```

  可以使用模版字符串，它可以定义多行文本和内嵌表达式。 这种字符串是被反引号包围（ `），并且以${ expr }这种形式嵌入表达式

  ```js
  let name: string = `Gene`;
  let age: number = 37;
  let sentence: string = `Hello, my name is ${name}.
  
  I'll be ${age + 1} years old next month.`;
  // 与下面的方式效果相同
  let sentence: string =
    "Hello, my name is " +
    name +
    ".\n\n" +
    "I'll be " +
    (age + 1) +
    " years old next month.";
  ```

- `数组`：
  ```js
  let list: number[] = [1, 2, 3];
  let list: Array<number> = [1, 2, 3];
  ```
- `元组（Tuple）`：
  元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。 比如，你可以定义一对值分别为 string 和 number 类型的元组。
  ```js
  // Declare a tuple type
  let x: [string, number];
  // Initialize it
  x = ["hello", 10]; // OK
  // Initialize it incorrectly
  x = [10, "hello"]; // Error
  ```
  当设置越界的元素时，会使用联合类型替代
- `枚举（menu）`：
  为一组数据赋予一个名字

  ```js
  enum Color {Red, Green, Blue}
  let c: Color = Color.Green;
  ```

  默认状态下，从 0 开始编号，类似于数组的下标，但是可以指定开始编号，也可给每个元素指定编号

  ```js
  enum Color {Red = 1, Green, Blue = 4}
  let colorName: string = Color[2];
  let colorName1: string = Color[4]

  console.log(colorName);  // 显示'Green'因为上面代码里它的值是2
  console.log(colorName1); // 显示‘Blue’
  ```

- `Any`:

  当不清楚类型时，可以使用 any 进行定义，跳过类型检查器，直接通过类型检查

- `Void`:

  表示没有任何类型，常用于函数没有返回值时，返回为 void；
  同时申明一个 void 类型，只能为其赋值为 null 或者 undefined

- `Null和Undefined`：

  默认情况下 null 和 undefined 是所有类型的子类型，可以把 null 和 undefined 赋值给其他类型，但是指定标记之后，null 和 undefined 只能赋值给 void 和他们自身，如果要使用可以使用联合属性

- `Never`:

  never 类型表示的是那些永不存在的值的类型。never 类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型； 当变量被永不为真的类型保护所约束时，也可能是 never 类型。

  never 类型是任何类型的子类型，也可以赋值给任何类型；然而，没有类型是 never 的子类型或可以赋值给 never 类型（除了 never 本身之外）。 即使 any 也不可以赋值给 never 。

- `Object`:

  object 表示非原始类型，也就是除 number，string，boolean，symbol，null 或 undefined 之外的类型。

  ```ts
  // 使用object类型，就可以更好的表示像Object.create这样的API
  declare function create(o: object | null): void;

  create({ prop: 0 }); // OK
  create(null); // OK

  create(42); // Error
  create("string"); // Error
  create(false); // Error
  create(undefined); // Error
  ```

- 类型断言：

  发生在你能更准确知道的类型。

  ```ts
  // 类型断言有两种形式。 其一是“尖括号”语法
  let someValue: any = "this is a string";
  let strLength: number = (<string>someValue).length;

  // 另一个为as语法：
  let someValue: any = "this is a string";
  let strLength: number = (someValue as string).length;
  ```

- 接口：

  使用 interface 定义对象，可使用 readonly ，？等修饰符进行定义对象属性。

  ```ts
  interface SquareConfig {
    color?: string;
    width?: number;
  }

  function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
  }

  let mySquare = createSquare({ color: "red", width: 100 });
  ```

- 函数：

  定义函数传递参数，可结合 `interface` 进行定义，同时也可进行参数可选和参数设定默认值

  ```ts
  function buildName(firstName?: string, lastName = "Smith") {
    return firstName + " " + lastName;
  }

  // 剩余参数
  function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
  }
  ```

- 类：

- 泛型：

  规范返回值类型和入参类型相同

  ```ts
  // 我们给identity添加了类型变量T。 T帮助我们捕获用户传入的类型（比如：number），之后我们就可以使用这个类型
  function identity<T>(arg: T): T {
    return arg;
  }
  ```
