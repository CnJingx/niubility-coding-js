## 受控和非受控组件真的那么难理解吗？(React实际案例详解)

## 前言

你盼世界，我盼望你无`bug`。Hello 大家好！我是霖呆呆。

今天咱还是先来聊点技术相关的东西吧，也就是本篇的标题——受控和非受控组件。

写这篇文章的原因是呆呆在写`HOC`时有涉及到受控和非受控组件的内容，然后发现能说的内容还是挺多的，但是搜索了一下网络上的教材大多说的都比较混乱，对新手来说不太好理解。

所以呆呆也是希望能发挥自身所长将这部分内容说的短而精，方便大家理解。

(没错，这里的长就是你们想的长，而短不是你们想的短...)

表情包去你妈的

好的👌，不皮了😁，来看看通过阅读本篇文章你可以学习到：

- 受控组件基本概念
- select受控组件
- 动态表单受控组件案例
- 非受控组件
- 特殊的文件file标签



## 正文

### 受控组件基本概念

通过名称，我们可以猜测一下这两个词是什么意思：

- 受控组件：受我们控制的组件
- 非控组件：不受我们控制的组件

(这提莫的不是废话吗...)

咳咳，好吧，这里的受控和非控是什么意思呢？其实也就是我们**对某个组件状态的掌控，它的值是否只能由用户设置，而不能通过代码控制**。

我们知道，在`React`中定义了一个`input`输入框的话，它并没有类似于`Vue`里`v-model`的这种双向绑定功能。也就是说，我们并没有一个指令能够将数据和输入框结合起来，用户在输入框中输入内容，然后数据同步更新。

就像下面这个案例：

```jsx
class TestComponent extends React.Component {
  render () {
    return <input name="username" />
  }
}
```

用户在界面上的输入框输入内容时，它是自己维护了一个`"state"`，这样的话就能根据用户的输入自己进行`UI`上的更新。(这个`state`并不是我们平常看见的`this.state`，而是每个表单元素上抽象的`state`)

想想此时如果我们想要控制输入框的内容可以怎样做呢？唔...输入框的内容取决的是`input`中的`value`属性，那么我们可以在`this.state`中定义一个名为`username`的属性，并将`input`上的`value`指定为这个属性：

```jsx
class TestComponent extends React.Component {
  constructor (props) {
    super(props);
    this.state = { username: 'lindaidai' };
  }
  render () {
    return <input name="username" value={this.state.username} />
  }
}
```

但是这时候你会发现`input`的内容是只读的，因为`value`会被我们的`this.state.username`所控制，当用户输入新的内容时，`this.state.username`并不会自动更新，这样的话`input`内的内容也就不会变了。

哈哈，你可能已经想到了，我们可以用一个`onChange`事件来监听输入内容的改变并使用`setState`更新`this.state.username`：

```jsx
class TestComponent extends React.Component {
  constructor (props) {
    super(props);
    this.state = {
      username: "lindaidai"
    }
  }
  onChange (e) {
    console.log(e.target.value);
    this.setState({
      username: e.target.value
    })
  }
  render () {
    return <input name="username" value={this.state.username} onChange={(e) => this.onChange(e)} />
  }
}
```

现在不论用户输入什么内容`state`与`UI`都会跟着更新了，并且我们可以在组件中的其它地方使用`this.state.username`来获取到`input`里的内容，也可以通过`this.setState()`来修改`input`里的内容。

OK👌，现在让我们来看看**受控组件**的定义：

在HTML的表单元素中，它们通常自己维护一套`state`，并随着用户的输入自己进行`UI`上的更新，这种行为是不被我们程序所管控的。而如果将`React`里的`state`属性和表单元素的值建立依赖关系，再通过`onChange`事件与`setState()`结合更新`state`属性，就能达到控制用户输入过程中表单发生的操作。被`React`以这种方式控制取值的表单输入元素就叫做**受控组件**。

(额，呆呆认为上面👆这个总结就可以用在面试当中了)



### select受控组件

在上面呆呆用`input`向大家演示了一个最基本的受控组件，那么其实对于其它的表单元素使用起来也差不多，可能就是属性名和事件不同而已。

例如`input`类型为`text`的表单元素中使用的是：

- `value`
- `onChange`

对于`textarea`标签也和它一样是使用`value`和`onChange`：

```html
<textarea value={this.state.value} onChange={this.handleChange} />
```

#### 单选select

对于`select`表单元素来说，`React`中将其转化为受控组件可能和原生`HTML`中有一些区别。

在原生中，我们默认一个`select`选项选中使用的是`selected`，比如下面这样：

```html
<select>
  <option value="sunshine">阳光</option>
  <option value="handsome">帅气</option>
  <option selected value="cute">可爱</option>
  <option value="reserved">高冷</option>
</select>
```

给`"可爱"`的选项设置了`selected`，默认选中的就是它了。

但是如果是使用`React`受控组件来写的话就不用那么麻烦了，因为它允许在根`select`标签上使用`value`属性，去控制选中了哪个。这样的话，对于我们也更加便捷，在用户每次重选之后我们只需要在根标签中更新它，就像是这个案例🌰：

```jsx
class SelectComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'cute' };
  }
  handleChange(event) {
    this.setState({value: event.target.value});
  }
  handleSubmit(event) {
    alert('你今日相亲对象的类型是: ' + this.state.value);
    event.preventDefault();
  }
  render() {
    return (
      <form onSubmit={(e) => this.handleSubmit(e)}>
        <label>
          你今日相亲对象的类型是:
          <select value={this.state.value} onChange={(e) => this.handleChange(e)}>
            <option value="sunshine">阳光</option>
            <option value="handsome">帅气</option>
            <option value="cute">可爱</option>
            <option value="reserved">高冷</option>
          </select>
        </label>
        <input type="submit" value="提交" />
      </form>
    );
  }
}
export default SelectComponent;
```

可以看到不论是`input`类型为`text`的控件还是`textarea、select`在实现为**受控组件**上都差不多。



#### 多选select

多选`select`的话，对比单选来说，只有这两处改动：

- 给`select`标签设置`multiple`属性为`true`
- `select`标签`value`绑定的值为一个数组

呆呆这里也来小小的写一个案例吧：

```jsx
class SelectComponent extends React.Component {
  constructor(props) {
    super(props);
    // this.state = { value: 'cute' };
    this.state = { value: ['cute'] };
  }

  handleChange(event) {
    console.log(event.target.value)
    const val = event.target.value;
    const oldValue = this.state.value;
    const i = this.state.value.indexOf(val);
    const newValue = i > -1 ? [...oldValue].splice(i, 1) : [...oldValue, val];
    this.setState({value: newValue});
  }

  handleSubmit(event) {
    alert('你今日相亲对象的类型是: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={(e) => this.handleSubmit(e)}>
        <label>
          你今日相亲对象的类型是:
          <select multiple={true} value={this.state.value} onChange={(e) => this.handleChange(e)}>
            <option value="sunshine">阳光</option>
            <option value="handsome">帅气</option>
            <option value="cute">可爱</option>
            <option value="reserved">高冷</option>
          </select>
        </label>
        <input type="submit" value="提交" />
      </form>
    );
  }
}
export default SelectComponent;
```

(但是呆呆在Mac，Chrome 测试这个多选好像是有问题的)



### 动态表单受控组件案例

上面👆咱们实现了一些简单的受控组件案例，接着来玩个稍微难点的。

先看一下我们的需求：

实现一个组件，传入以下数组，自动渲染出表单：

(`CInput`代表一个输入框，`CSelect`代表一个选择框)

```javascript
// 决定表单的结构
const formConfig = [
  {
    Component: CInput,
    label: '姓名',
    field: 'name',
  },
  {
    Component: CSelect,
    label: '性别',
    field: 'sex',
    options: [{ label: '男', value: 'man' }, { label: '女', value: 'woman' }]
  }
]
// 决定表单的内容
this.state = {
  name: '霖呆呆',
  sex: 'man'
}
```

效果：

图片sk1

也就是来实现一个简单的动态表单，看看受控组件在其中的应用。

- `formConfig`决定了表单的结构，也就是定义表单中会有哪些项

- `this.state`中定义了表单中各项的值是什么，它与`formConfig`是靠`formConfig`中各项的`field`字段来建立链接的。

知道了上面👆这些东西，我们就能很快写出这个动态表单组件的大概样子了：

(我们就把这个组件命名为`formComponent`吧，重点看`render`部分)

```jsx
import React, { Component } from 'react';
import { CInput, CSelect } from './components'
export default class FormComponent extends Component {
	constructor (props) {
    super(props);
    this.state = {
      name: '霖呆呆',
      sex: 'man'
    }
  }
  formConfig = [
    {
      Component: CInput,
      label: '姓名',
      field: 'name',
    },
    {
      Component: CSelect,
      label: '性别',
      field: 'sex',
      options: [{ label: '男', value: 'man' }, { label: '女', value: 'woman' }]
    }
  ]
  render () { // 重点在这
    return (
      <form style={{marginTop: '50px'}}>
        {
          this.formConfig.map((item) => { // 枚举formConfig
            const { Type, field, name } = item;
            const ControlComponent = item.Component; // 提取Component
            return (
              <ControlComponent key={field} />
            )
          })
        }
      </form>
    )
  }
}
```

可以看到，`render`部分我们做了这么几件事：

- 枚举`formConfig`数组
- 提取出每一项里的`Component`，赋值给`ControlComponent`
- 渲染出每一项`Component`

`ControlComponent`变量的意义在于告诉`React`，需要渲染出哪一个组件，如果`item.Component`是`CInput`，那么最终渲染出的就是`<CInput />`。

这样就保证了能把`formConfig`数组中的每一个表单项都渲染出来，但是这些表单项现在还是不受我们控制的，我们需要用前面学到的`value`和`onChange`和每个`ControlComponent`建立联系，就像是这样：

```jsx
<ControlComponent
  key={field}
  name={field}
  value={this.state[field]}
  onChange={this.onChange}
  {...item}
/>
```

我们把`this.state[field]`设置到`value`上，把`this.onChange`设置到`onChange`属性上。(可想而之，`CInput`和`CSelect`组件中就能用`this.props`来接收传入的属性了，例如`this.props.value`)

那么这时候`value`已经确定了，它就是由`this.state[field]`决定的，如果`field`是`"sex"`的话，`value`的值就是`"man"`。

所以来看看`onChange`方法该怎样写：

```jsx
onChange = (event, field) => {
  const target = event.target;
  this.setState({
    [field]: target.value
  })
}
```

这个方法其实也很简单，它接受一个`event`和`field`，在`event`中就可以获取到用户输入/选择的值了。

好的👌，接下来让我们快速的看一下`CInput`和`CSelect`是如何实现的吧：

*components/CInput.jsx*:

```jsx
import React, { Component } from 'react';

export default class CInput extends Component {
  constructor (props) {
    super(props);
  }
  render () {
    const { name, field, value, onChange } = this.props;
    return (
      <>
        <label>
          {name}
        </label>
        <input name={field} value={value} onChange={(e) => onChange(e, field)} />
      </>
    )
  }
}
```

*components/CSelect.jsx*:

```jsx
import React, { Component } from 'react';

export default class CSelect extends Component {
  constructor (props) {
    super(props);
  }
  render () {
    const { name, field, options, value, onChange } = this.props;
    return (
      <>
        <label>
          {name}
        </label>
        <select name={field} value={value} onChange={(e) => onChange(e, field)}>
          {options.length>0 && options.map(option => {
            return <option key={option.value} value={option.value}>{option.label}</option>
          })}
        </select>
      </>
    )
  }
}
```

当然，这里仅仅演示的是一个简单的动态表单的实现，如果你想要在项目中实现的话要远比这个复杂多了。



### 非受控组件

上面👆向大家展示的是受控组件的一些基本概念还有相关操作，对于受控组件，我们需要为每个`状态更新`(例如`this.state.username`)编写一个`事件处理程序`(例如`this.setState({ username: e.target.value })`)。

那么还有一种场景是：我们仅仅是想要获取某个表单元素的值，而不关心它是如何改变的。对于这种场景，我们有什么应对的方法吗🤔️？

唔...`input`标签它实际也是一个`DOM`元素，那么我们是不是可以用获取`DOM`元素信息的方式来获取表单元素的值呢？也就是使用`ref`。

就像下面👇这个案例一样：

```jsx
import React, { Component } from 'react';

export class UnControll extends Component {
  constructor (props) {
    super(props);
    this.inputRef = React.createRef();
  }
  handleSubmit = (e) => {
    console.log('我们可以获得input内的值为', this.inputRef.current.value);
    e.preventDefault();
  }
  render () {
    return (
      <form onSubmit={e => this.handleSubmit(e)}>
        <input defaultValue="lindaidai" ref={this.inputRef} />
        <input type="submit" value="提交" />
      </form>
    )
  }
}
```

在输入框输入内容后，点击提交按钮，我们可以通过`this.inputRef`成功拿到`input`的`DOM`属性信息，包括用户输入的值，这样我们就不需要像受控组件一样，单独的为每个表单元素维护一个状态。

同时我们也可以用`defaultValue`属性来指定表单元素的默认值。



### 特殊的文件file标签

另外在`input`中还有一个比较特殊的情况，那就是`file`类型的表单控件。

**对于file类型的表单控件它始终是一个不受控制的组件，因为它的值只能由用户设置，而不是以编程方式设置。**

例如我现在想要通过状态更新来控制它：

```jsx
import React, { Component } from 'react';

export default class UnControll extends Component {
  constructor (props) {
    super(props);
    this.state = {
      files: []
    }
  }
  handleSubmit = (e) => {
    e.preventDefault();
  }
  handleFile = (e) => {
    console.log(e.target.files);
    const files = [...e.target.files];
    console.log(files);
    this.setState({
      files
    })
  }
  render () {
    return (
      <form onSubmit={e => this.handleSubmit(e)}>
        <input type="file" value={this.state.files} onChange={(e) => this.handleFile(e)} />
        <input type="submit" value="提交" />
      </form>
    )
  }
}
```

在选择了文件之后，我试图用`setState`来更新，结果却报错了：

图片sk2

所以我们应当使用非受控组件的方式来获取它的值，可以这样写：

```jsx
import React, { Component } from 'react';

export default class FileComponent extends Component {
  constructor (props) {
    super(props);
    this.fileRef = React.createRef();
  }
  handleSubmit = (e) => {
    console.log('我们可以获得file的值为', this.fileRef.current.files);
    e.preventDefault();
  }
  render () {
    return (
      <form onSubmit={e => this.handleSubmit(e)}>
        <input type="file" ref={this.fileRef} />
        <input type="submit" value="提交" />
      </form>
    )
  }
}
```

这里获取到的`files`是一个数组哈，当然，如果你没有开启多选的话，这个数组的长度始终是`1`，开启多选也非常简单，只需要添加`multiple`属性即可：

```jsx
<input type="file" multiple ref={this.fileRef} />
```

