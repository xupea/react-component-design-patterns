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



### 2. 期望

> 子贡问曰：有一言而可以终身行之者乎？子曰：其恕乎！己所不欲，勿施于人。

#### 1. 假设别人写个组件给你使用，你期望是什么？

好用：简洁的 api 设计和使用文档， 功能强大，无需知道其实现细节，无bug，方便，快捷

#### 2. 假设别人写个组件给你维护，你期望是什么？

易维护：良好的设计，完备的单元测试和文档

#### 3. 如果这个“别人” 和 “你”互换一下，为了能让组件好用和可维护，你该怎么办？

到底有没有这样的规则可以遵循？



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



### 8. 规则4：X 行以内原则

不管类组件还是函数组件通常都会在一个模块文件中？那个一个模块文件多少行合适呢？

就像是一个函数多少行才合适一样，存在争议，但是我做了一点优化，具体操作如下：

#### 1. 设置X初始值

假设我们设定 一个组件文件的的行数设置为，X1和X2，最大 100 行（类组件） 和 50 行（函数组件，毕竟函数组件自称可以写更少的代码）



#### 2. 原则1和原则2

编写组件发现超过初始值，优先使用 原则1 和 原则2，得到Y1，X2



#### 3.优化初始值

如果第2步无法解决，即Y1和Y2大于初始值X1和X2，则Y1和Y2作为新的初始值



#### 4.持续优化初始值

可以通过 ESLint 输出最大的 组件 行数，持续更新 最大的阈值



这个原则是最简单的，也是最有效的，它核心在于：

1.X值并不是固定，而是根据组件的添加或删除不断更新的，它是那一时刻最合理的最大行数

2.它会强制我们review和使用原则1和原则2进行优化

3.持续优化，最大行数不会越来越大



具体可以设置 ESLint和SonarQube



### 9. 最佳实践



#### 1. 组件UML图：

组件UML图是UML在组件设计中的一种应用，由于组件间或内部通常是嵌套的结构，通过组件UML图，我们很清晰地看到一个复杂组件的全局。它可以显示 State，Props，Methods，以及其它组件的关系，特别是各个组件的职责，以表格为例子：

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/27/16f455c0555eae88~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

组件UML同时也是很好的文档，为组件的维护和重构提供基础。



#### 2. 多读源码和其组件文档

多读优秀的代码和其使用文档，我们能在文档上封装和职责，代码上看到封装和职责

多读一下antd和element-ui的组件库源码，我们可以找到很多可以参照的代码，



#### 3. 为所有 interface 或 type 添加注释

特别是组件 的 Props 和 State 接口添加注释，能快速了解各个字段的含义

```tsx
interface State {
  /**
   * 数量
   */
  count: number;
}
interface Props {
  /**
   * 数量
   */
  count: number;
}
```



#### 4. 扁平化的 State 和 Props

封装组件时，暴露的 props 以及内部维护的state，应该保持扁平化，而不是嵌套的对象。



尽量避免传递给组件不需要的属性

不要去修改数据



业务不需要了，是否方便删除

别人用，怎么想



透传props的合理性



#### 5. 持续改进

1.第一次把组件的结构设计好很难的

原因可能是：

时间紧，任务重

最开始选择方案是错的

发现更好的开源解决方案

越复杂的组件，就越需要经常验证和重构

开发就是不断检查之前的决策，然后提升

保证组件的质量需要精力和定期的review，

随着项目的体积增大，你很有可能忘记计划矫正结构，降低耦合



#### 6. 语义化标签

语义化的标签最大的好处就是让页面或组件具有良好的语义和结构，举例如下：

![HTML 5的革新——语义化标签(一)](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/1/5/160c3bf5485ff9ce~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

不要使用无语义化的 `div + css + span` 。

常用的HTML5语义化标签：

|         |                       |             |
| ------- | --------------------- | ----------- |
| header  | 网页或section的页眉   |             |
| footer  | 网页或section的页脚   |             |
| hgroup  | 网页或section的标题组 | 多个h1-h6时 |
| nav     | 页面的导航链接区域    |             |
| aside   | 独立的内容区域        |             |
| section | 文档中的“节”或“段”    |             |
| article | 网页中自成一体的内容  |             |
| address | 地址                  |             |
| h1-h6   | 标题                  |             |

详细使用场景可以参考：https://html.spec.whatwg.org/multipage/sections.html#the-header-element



#### 7. 不允许注释掉代码

注释掉的代码真的是百害而无一利，它会：

- 分散我们注意力

- 增加我们的认知负荷

- 随着时间的推移，容易过期

不用的代码统统删掉，相信 `git diff`

```
// ❌
function foo(bar) {
  const baz = bar(false)
  // we no longer do this for some good reason
  // if (baz === 'foobar') {
  // return baz
  // } else {
  // return bar.foobar()
  // }
  return baz
}

// ✅
function foo(bar) {
  return bar(false)
}
```



### 10. 参考：

1.React哲学[https://zh-hans.reactjs.org/docs/thinking-in-react.html]

2.Applying SOLID principles in React[https://konstantinlebedev.com/solid-in-react/]

3.Applying SOLID To React[https://medium.com/docler-engineering/applying-solid-to-react-ca6d1ff926a4]

4.面向对象编程的 SOLID 原则[https://chinese.freecodecamp.org/news/solid-principles/]

5.7 Architectural Attributes of a Reliable React Component[https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component/]

6.组件原子设计[https://atomicdesign.bradfrost.com/]

7.前端组件设计之一——设计原则[https://juejin.cn/post/6844904032700481550]

8.请不要注释掉代码[https://kentcdodds.com/blog/please-dont-commit-commented-out-code]
