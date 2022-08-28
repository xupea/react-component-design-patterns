## 可靠组件的设计原则

<img src="https://ae01.alicdn.com/kf/H6081faeb783440fe944f4fb3ddfedddch/Original-New-Mavic-Mini-Upper-Cover-Module-Bottom-Cover-Middile-Shell-Motor-Arm-Flat-Cable-Repair.jpg_Q90.jpg_.webp" alt="img" style="zoom: 50%;" />

### 1. 组件和组件化思维？

> 组件，从概念上类似于 JavaScript 函数。它接受任意的入参（即 “props”），并返回用于描述页面展示内容的 React 元素。
>
> 组件允许你将 UI 拆分为独立可复用的代码片段，并对每个片段进行独立构思。
>
> —— React官方文档[https://zh-hans.reactjs.org/docs/components-and-props.html]

> 组件允许我们将 UI 划分为独立的、可重用的部分，并且可以对每个部分进行单独的思考。
>
> ——Vue官方文档[https://cn.vuejs.org/guide/essentials/component-basics.html]

组件就是一段代码片段，用来实现指定的功能或渲染特定的UI。





当我们拿到 figma，sketch的设计稿，通过自顶向下进行拆分，形成组件树



### 2. 期望

> 子贡问曰：有一言而可以终身行之者乎？子曰：其恕乎！己所不欲，勿施于人。

#### 1. 假设别人写个组件给你使用，你期望是什么？

好用：简洁的 api 设计和使用文档， 功能强大，无需知道其实现细节，无bug，方便，快捷

#### 2. 假设别人写个组件给你维护，你期望是什么？

易维护：良好的设计，完备的单元测试和文档

#### 3. 如果这个“别人” 和 “你”互换一下，为了能让组件好用和可维护，你该怎么办？

到底有没有这样的规则可以遵循？答案是有的。



### 3. 组件的分类

#### 1. 基础组件（原子组件）

第三方组件库如 antd，element-ui，提供的组件，以及 原生 dom 标签

```tsx
<Button></Button>

<el-button type="primary">主要按钮</el-button>

<div></div>
```



#### 2. 展示型组件（标准组件）

只描述UI，通过props来决定

通过组合基础组件



#### 3. 业务型组件

包含业务组件

处理一个业务



#### 4. 容器组件

对应一个页面，按照区域（上下左右，左中右，上中下，左右）



多个业务组件



业务组件之间联动



专门负责和store通信



副作用 和 组合业务组件



中间组件传递 props



一个页面通常就是一个容器组件，它需要准备页面所需的数据，以及组合出对应的UI



### 4. 原则0：把大象分三步进冰箱

> 把大象装进冰箱需要几步？答：三步，把冰箱门打开，大象塞进去，关上冰箱门。

当我们拿到设计师提供 figma或sketch的设计稿，通常是以页面为单位，我们需要先对大的页面进行第一次拆分，拆分的原则也很简单：

#### 1. 渐进式

不要尝试一次把所有部分一次性拆分完，如果你不确定，我们可以自顶向下，由外到内的方式拆分

#### 2. 拆分依据

分割线，背景颜色变化，元素之间间距

#### 3. 重复部分

重复的部分通常都是列表类，统一拆分为一个部分

#### 4. 不超过三个

每次拆分尽量不超过三个部分

#### 5. 直到原子组件（基础组件）



举个例子，以 Medium 网站主页分析：

<img src="https://github.com/xupea/react-component-design-patterns/blob/main/images/medium-original.png?raw=true" style="zoom: 25%;" />

根据设计稿中的分割线，我们可以按照以下拆分（我们可以打印出来，用方框圈出每一个部分）：

<img src="https://github.com/xupea/react-component-design-patterns/blob/main/images/medium-1st.png?raw=true" alt="image-20220828175400339" style="zoom:25%;" />

此时整个划分为三个部分：

```tsx
import React from "react";

const Medium = () => {
  return (
    <div>
      <LeftSideBar />
      <Main />
      <RightSideBar />
    </div>
  );
};

export default Medium;
```

接着我们可以继续按照上述规则继续拆分，例如按照间距和重复划分：

<img src="https://github.com/xupea/react-component-design-patterns/blob/main/images/medium-2nd.png?raw=true" alt="image-20220828175400339" style="zoom:25%;" />

此时的页面在上面基础下，继续拆分为：

```tsx
import React from "react";

const LeftSideBar = () => {
  return (
    <div>
      <Logo />
      <Nav />
      <Account />
    </div>
  );
};

const Main = () => {
  return (
    <div>
      <Filter />
      <Articles />
    </div>
  );
};

const RightSideBar = () => {
  return (
    <div>
      <GetUnlimitedAccessLink />
      <SearchBar />
      <ReadingList />
    </div>
  );
};

const Medium = () => {
  return (
    <div>
      <LeftSideBar />
      <Main />
      <RightSideBar />
    </div>
  );
};

export default Medium;
```

如果可以我们继续拆分，直到拆分到原子组件（基础组件）为止。这样我们会得到一个静态版本，这时还不是最终组件树。



### 5. 原则1：单一职责原则

每一个组件都应该只实现一个**职责**，并且只有一个改变状态的理由。

可以按照一下规则拆分组件：

#### 1. 按数据逻辑和UI渲染拆分

#### 2. 数据逻辑再拆分：数据请求和数据处理

#### 3. UI渲染再拆分：按逻辑或业务功能



举个例子：

```tsx
import React, { useState, useEffect } from "react";

type User = {
  /** 是否被禁 */
  isBanned: boolean;
  /** 上一次活跃时间 */
  lastActivityAt: Date;
  /** id */
  id: string;
  /** 头像地址 */
  avatarUrl: string;
  /** 全名 */
  fullName: string;
  /** 角色 */
  role: string;
};

const ActiveUsersList = () => {
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    const loadUsers = async () => {
      const response = await fetch("/some-api");
      const data = await response.json();
      setUsers(data);
    };
    loadUsers();
  }, []);

  const weekAgo = new Date();
  weekAgo.setDate(weekAgo.getDate() - 7);
  return (
    <ul>
      {users
        .filter((user) => !user.isBanned && user.lastActivityAt >= weekAgo)
        .map((user) => (
          <li key={user.id}>
            <img src={user.avatarUrl} alt="" />
            <p>{user.fullName}</p>
            <small>{user.role}</small>
          </li>
        ))}
    </ul>
  );
};

export default ActiveUsersList;
```

可以看出组件做了很多事情：获取数据、过滤数据、根据数据渲染UI。



useEffect 总是一个可能的拆分点，因为它总是在处理副作用

如果useEffect 和 useState 成对出现，通常是一个Custom Hook



```tsx
const useUsers = () => {
  const [users, setUsers] = useState([])
  
  useEffect(() => {
    const loadUsers = async () => {  
      const response = await fetch('/some-api')
      const data = await response.json()
      setUsers(data)
    }

    loadUsers()
  }, [])
  
  return { users }
}
```

useUsers Hook 只关心一件事——从API中获取用户数据



```tsx
const UserItem = ({ user }) => {
  return (
    <li>
      <img src={user.avatarUrl} />
      <p>{user.fullName}</p>
      <small>{user.role}</small>
    </li>
  )
}
```

列表的子项通常也可以抽离出来



```tsx
const getOnlyActive = (users) => {
  const weekAgo = new Date()
  weekAgo.setDate(weekAgo.getDate() - 7)
  
  return users.filter(user => !user.isBanned && user.lastActivityAt >= weekAgo)
}
```

从 API 获取到的用户列表中过滤出所有非活跃用户的逻辑是相对独立的，可以在其他部分重用，所以可以将其提取到一个公共函数中



### 6. 原则2：开放封闭原则（封装和组合）

组件应该对扩展开放，对修改关闭。允许在不更改源代码的情况下扩展组件的方式来构造组件。

#### 1. 封装

良好的组件应该封装内部结构和实现细节，仅提供一些必要的props来控制其行为和输出，将其容易变化的部分交由外部

#### 2. 组合

组合是组件间依赖最基本的方式，（使用children，render props，或者 HOC）



例子1：

```tsx
const Header = () => {
  const { pathname } = useRouter()
  
  return (
    <header>
      <Logo />
      <Actions>
        {pathname === '/dashboard' && <Link to="/events/new">Create event</Link>}
        {pathname === '/' && <Link to="/dashboard">Go to dashboard</Link>}
      </Actions>
    </header>
  )
}

const HomePage = () => (
  <>
    <Header />
    <OtherHomeStuff />
  </>
)

const DashboardPage = () => (
  <>
    <Header />
    <OtherDashboardStuff />
  </>
)
```

根据所在页面的不同，呈现指向不同页面组件的链接。那现在考虑一下，如果需要将这个`Header`组件添加到更多的页面中会发生什么呢？每次创建新页面时，都需要引用 `Header` 组件，并修改其内部实现。这种方式使得 `Header` 组件与使用它的上下文紧密耦合。

为了解决这个问题，我们可以使用**组件组合**。`Header` 组件不需要关心它将在内部渲染什么，相反，它可以将此责任委托给将使用 `children` 属性的组件：

```tsx
const Header = ({ children }) => (
  <header>
    <Logo />
    <Actions>
      {children}
    </Actions>
  </header>
)

const HomePage = () => (
  <>
    <Header>
      <Link to="/dashboard">Go to dashboard</Link>
    </Header>
    <OtherHomeStuff />
  </>
)

const DashboardPage = () => (
  <>
    <Header>
      <Link to="/events/new">Create event</Link>
    </Header>
    <OtherDashboardStuff />
  </>
)
```

使用这种方法，我们完全删除了 `Header` 组件内部的变量逻辑。现在可以使用组合将想要的任何内容放在`Header`中，而无需修改组件本身。

遵循开放封闭原则，可以减少组件之间的耦合，使它们更具可扩展性和可重用性。



例子2：

```tsx
import api from '~/common/api'

const LoginForm = () => {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')

  const handleSubmit = async (evt) => {
    evt.preventDefault()
    await api.login(email, password)
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" value={email} onChange={e => setEmail(e.target.value)} />
      <input type="password" value={password} onChange={e => setPassword(e.target.value)} />
      <button type="submit">Log in</button>
    </form>
  )
}
```

`LoginForm` 组件直接引用了 `api` 模块，因此它们之间存在紧密耦合。这种依赖关系就会导致一个组件的更改会影响其他组件。依赖倒置原则就提倡打破这种耦合



```tsx
import api from '~/common/api'

const ConnectedLoginForm = () => {
  const handleSubmit = async (email, password) => {
    await api.login(email, password)
  }

  return (
    <LoginForm onSubmit={handleSubmit} />
  )
}
```



`ConnectedLoginForm` 组件充当 `api` 和 `LoginForm` 之间的粘合剂，而它们本身保持完全独立。这样就可以对这两个组件进行单独的修改和维护，而不必担心修改会影响其他组件。



规则0拆分最小粒度，规则1最小合理粒度，规则2最小化依赖



### 7. 原则3：完备的测试

> 自动化测试就像健身一样，大家都知道有用，但就是不去做

自动化测试是最简单最有效的保证质量的手段，也是最好的设计文档



测试的作用：

1.保证质量 

因为人工回归根本就不靠谱，又低效

2.保证设计

几乎组件都不会写文档，除了作者没人能知道为什么这样设计

自动化的测试代码可以很好的表达意图，活文档



如果一个组件测试不易于测试，很大可能是你的组件设计存在问题。



例子：

```tsx
import React from "react";
import { render, screen } from "@testing-library/react";
import ActiveUserList from "../ActiveUserList";

describe("<ActiveUserList />", () => {
  it("fetch users from some-api", () => {
    //
  });

  it("only renders active users for the last 7 days", () => {
    //
  });

  it("only renders active users who are not banned", () => {
    //
  });

  it("renders active user list", () => {
    //
  });
});
```



一个组件应该在多少行之内才是合理的？



100 行 200行 300行？我们尝试让一个组件变小



怎么让一个组件变小呢？



小的组件 通过 组合 HOC复用



单一职责原则



一个会“偷懒”的组件



### 8. 规则4：X行以内

一个类组件或一个函数组件到底多行内是合理的？

复杂度，缺少合适的设计模式？



### 9. 最佳实践



#### 使用函数组件



#### 组件UML图：

State，Props，Methods，其它组件的关系

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/27/16f455c0555eae88~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 多读源码

antd组件库，以及其api 设计文档

#### 类型注释

Bad：

![image-20220808001903504](/Users/xiaomingxu/Library/Application Support/typora-user-images/image-20220808001903504.png)



#### 为所有 interface 或 type 添加注释

react 的 Props 和 State 接口，能快速了解各个字段的含义

```tsx
interface State {}
interface Props {}

```



#### 扁平化的 state 和 props

封装组件时，暴露的 props 以及内部维护的state，应该保持扁平化，而不是嵌套的对象。



尽量避免传递给组件不需要的属性

不要去修改数据



业务不需要了，是否方便删除

别人用，怎么想



透传props的合理性



#### 可靠性

保证组件的质量需要精力和定期的review，

随着项目的体积增大，你很有可能忘记计划矫正结构，降低耦合



#### 持续改进

1.第一次把组件的结构设计好很难的

原因可能是：

时间紧，任务重

最开始选择方案是错的

发现更好的开源解决方案



越复杂的组件，就越需要经常验证和重构



开发就是不断检查之前的决策，然后提升



#### 富有意义

开发人员大部分时间都在阅读和理解代码，而不是实际编写代码。 有意义的函数、变量命名，可以让代码具有良好的可读性。

函数 动宾

类 名词



html tags 



按照 结构 和 业务 来划分



结合 html tags 和 antd（dui）



#### 纯组件和非纯组件

非纯组件有显式的副作用，我们要隔离副作用代码

##### 1. 全局变量

通过props传递给组件，而非直接注入组件的作用域中

##### 2. 网络请求

将网络请求和组件渲染分离



#### 纯组件/非纯组件

非纯组件有显示的副作用，我们要尽量隔离非纯代码。

将全局变量作为props传递给组件，而非将其注入到组件的作用域中。

将网络请求和组件渲染分离，只将数据传递给组件，保证组件职责的单一性，也能将非纯代码从组件中隔离。



不允许注释代码



参考：

1.React哲学[https://zh-hans.reactjs.org/docs/thinking-in-react.html]

2.Applying SOLID principles in React[https://konstantinlebedev.com/solid-in-react/]

3.Applying SOLID To React[https://medium.com/docler-engineering/applying-solid-to-react-ca6d1ff926a4]

4.面向对象编程的 SOLID 原则[https://chinese.freecodecamp.org/news/solid-principles/]

5.7 Architectural Attributes of a Reliable React Component[https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component/]

6.组件原子设计[https://atomicdesign.bradfrost.com/]

7.前端组件设计之一——设计原则[https://juejin.cn/post/6844904032700481550]





### 如何构建一个应用?

组件化

组件允许你将 UI 拆分为独立可复用的代码片段，并对每个片段进行独立构思。

组件，从概念上类似于 JavaScript 函数。它接受任意的入参（即 “props”），并返回用于描述页面展示内容的 React 元素。（组件 = props + state + render）

React哲学

antd组件：良好的设计，才能保证 组合 和 复用 才能受益

很多功能加载在一起的大组件，组件间耦合，没有单元测试 = 技术债，很难修改和新增功能



问一下自己：

1.怎么正确地构造组件的结构

2.什么时候拆分大组件为小组件

3.组件间通信解耦

组件即封装（高内聚，低耦合）

组件的层次结构

自顶向下设计：





错误1：

prop drilling

Pass-through props

Redux



错误2：

传递 className prop



错误3：

margin

