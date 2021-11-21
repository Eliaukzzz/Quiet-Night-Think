---
title: 'Nest.js 学习笔记'
date: '2021-11-21'
---

# Nest.js学习笔记

# 文件路径路由

在Nest中，页面的路由直接由文件在pages文件夹中的路径映射。

举个例子：

首页文件路径为 pages/index.js ，首页路由为'/'

posts页面文件路径为 pages/posts/index.js，posts页面路由为'/posts'

first-post页面文件路径为 pages/posts/first-post.js ，first-post页面路由为 '/posts/first-post'

## 跳转路由

在next中，跳转路由可以使用Link组件。

```javascript
// 引入Link组件
import Link from "next/link"
```

Link组件的使用和 html 中的 a 标签类似，可以使用href属性绑定需要跳转的连接。

```javascript
return (
  <h1 className={styles.title}>
    Learn{" "}
    <Link href="/posts/first-post">
    	<a>Next.js!</a>
    </Link>
  </h1>
)
```

## 客户端导航（Client-Side Navigation）

- 概念：在next中，路由的切换、页面的跳转是通过客户端导航的方式完成的，这意味着你不需要进行任何网络请求，就可以进行页面的切换，这个过程是通过JavaScript完成的，你可以点击开发者工具设置首页的样式，将首页 body 背景颜色改为蓝色后进行路由切换，样式没有随着路由切换而丢失，这证明了这两个路由其实是同一个页面。（在页面切换的过程，next帮你完成了 location.history 的操作，可以进行页面前进后退验证）
- 优点：由 Javascript 操作页面跳转可比发送请求快多了。

## 代码拆分和代码预加载（Code splitting and prefetching）

- 代码拆分：next会自动拆分代码，当**加载一个页面时，只会加载和这个页面必须的所有组件**。这保证了当你拥有数以百计的页面时，当前页面的加载速度不会收到影响。
- 代码预加载：当一个**页面中出现了 Link 组件时，next 会在后台进行对应路由页面代码的预加载**。当你点击并切换到对应的路由时，该页面已经在后台加载完毕了，代码预加载使页面的切换丝滑如德芙。

## 关于 next 路由的总结

- next 通过客户端导航、代码拆分和预加载，自动优化web应用。
- next 不需要使用任何路由库，就可以实现路由的切换，只需要在 pages 文件夹里面创建对应的页面文件，就可以通过 Link 组件导航到对应的路由中。

- Link 组件中需要包含一个 a 标签，因为 Link 组件不会自动生成 a 标签包裹内容，同样的，如果需要添加样式和其他的 attributes 属性，可以添加在 a 标签中。

# next 使用静态资源、页面head属性和CSS

## 使用静态资源

nest 的所有静态资源（图片、icon等）都放在项目根目录的 public 目录中，使用静态资源时，可以直接用 "/" 代表 public 目录

```javascript
// 使用图片时，需要使用Image组件
import Image from "next/image";

// 图片连接/vercel.svg 代表 public 目录下的 vercel.svg 文件
<Image src="/vercel.svg" alt="Vercel Logo" width={72} height={16} />
```

## 修改页面 head 标签中的内容

- 从上面的文件系统路由中我们可以知道，当路由切换时，其实页面文件并没有进行切换，但是当页面的 head 标签中的内容需要随着页面切换而改变时，应该如何操作呢？
- next 封装了 Head 组件，专门用来处理这种需求，通过 Head 组件可以实现页面标题和页面favicon.ico。

```javascript
import Head from "next/head"; // 引入 Head 组件

<Head>
  <title>{title}</title>
	<link rel="icon" href="/favicon.ico" />
</Head>
```

## 模块化引入样式

- 模块化方式在 react 组件中引入 css 文件。
- 模块化方式要求命名 css 文件是以 `.module.css` 作为文件后缀。

- 在 React 组件中可以使用类似 `import styles from "../../styles/xxx.module.css";`的方式引入。

```css
/* layout.module.css */
.container {
  min-width: 36rem;
  padding: 0 1rem;
  margin: 3rem auto 6rem;
  color: skyblue;
}
import styles from "../../styles/layout.module.css"; // 模块化引入
export default function Post() {
  // 在h1中使用styles模块中container选择器的样式
  return <h1 className={styles.container}>Posts</h1>;
}
```

- 模块化引入样式的优点：

1. 1. 不需要自定义 class 的名称，next会自动为标签生成不会重复的 class 属性名。点开开发者工具时可以看到 next 为标签添加了唯一的 class 属性名称。不必担心命名冲突问题。
   2. next 代码拆分和预加载同样支持 css 模块化引入，可以确保打开页面时仅仅加载页面所需要的最少的 css 样式。

## 全局引入样式

- 因为 next 中切换路由并不加载新的 html 文件，所以全局的样式会影响所有页面的样式。
- 在 next 中引入全局样式仅仅可以在 `"pages/_app.js"` 组件下引入，这个组件是 next 应用最顶层的组件，它还可以进行全部页面的状态保存。

```css
// "pages/_app.js"
import '../styles/globals.css' // 引入全局样式

function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}

export default MyApp
```

# Nest预渲染

## 预渲染的优点

- 预渲染会在返回响应时为页面预先生成html，而普通的客户端渲染则是交由客户端的Javascript生成页面。

## 两种预渲染方式：

- 静态生成：在应用构建时，生成HTML结构，在不同的请求中重用预渲染的HTML。
- 服务端渲染：在服务器收到不同的请求时，为不同的请求渲染指定的页面。

- 如何选择：

- - 如果页面不随着请求参数改变改变内容，使用静态生成方式。
  - 如果页面内容需要频繁更新，而且随着用户的请求改变内容，使用服务端渲染方式。

## 预渲染时获取依赖的外部数据

- 可以使用 next 提供的两种开箱即用的方式，在 react 组件中需要使用服务器或者数据库或者外部 api 的资源时，获取数据。
- `getStaticProps`：

- - 在导出一个页面组件时，同时导出一个异步函数`getStaticProps`. 在项目打包构建时，会调用该函数，在函数中可以获取依赖的静态数据，同时可以将函数返回的数据作为该页面组件的props。
  - 该函数只会在服务端调用，不会被打包到客户端运行。

- - 开发环境中，每次请求都会调用`getStaticProps`，生产环境中，只会在项目打包的时候执行。

- `getServerSideProps`：

- - 服务端渲染方式，需要在发送请求时获取数据，可以在导出页面组件时，导出一个异步函数`getServerSideProps`，它的参数可以包含请求所需的信息。

# 动态路由

- 当页面路径依赖于外部静态数据、数据库、网页 API 等等时，可以使用动态路由。

## 创建动态路由步骤

1. 创建一个形如`[id].js`的页面组件。组件文件名由`[`开头，`].js`结尾。这种形式的页面组件，就是动态路由组件。
2. 在动态路由组件中导出一个异步函数`getStaticPaths`，用于获取所有可能的路由。

3. 最后在导出的异步函数`getStaticProps`参数中可以接收到`[]`中的参数。

```javascript
// [id].js
// 获取指定路由数据
export async function getStaticProps({ params }) {
  // params中可以获取id属性 params.id
  return {data: getData(params.id)}
}


// 获取动态路由列表
export async function getStaticPaths() {
  // 这个函数可能是查外部文件、数据库等，获取可能的路由
  const paths = getAllPathIds(); 
  return {
    paths,
    fallback: false,
  };
}
```

## 使用动态路由的注意点

- `**getStaticPaths**`

1. 1. 返回的对象中，第一个属性 paths 必须是一个由对象组成的数组，每个对象中都需要有一个params属性，里面用来存放路由所需的信息。如果不是这种结构，`getStaticPaths`会失败。

```javascript
// paths类似如下结构

paths = [
    {
      params: {
        id: 'ssg-ssr'
      }
    },
    {
      params: {
        id: 'pre-rendering'
      }
    }
]
```

1. 2. 返回对象的第二个属性（todo）

2. 3. 在开发环境中，`getStaticPaths`会在每次请求时调用，而生产环境中，仅会在构建项目时调用。



- **近似匹配动态路由**

- - 当命名动态路由页面组件时，使用形如`[...id].js`这种方式命名，会近似匹配路由
  - 也就是说假设动态路由组件路径为`pages/posts/[...id].js`，会命中"/posts/a"，也会命中"/post/a/b/c"路由。

- - 如果使用这种方式，`getStaticPaths`，需要返回的 paths 应该用如下数据结构

```javascript
[
  {
    params: {
      // Statically Generates /posts/a/b/c
      id: ['a', 'b', 'c']
    }
  }
  //...
]
```



- **定制404页面：**

- - 创建页面组件，路径为`pages/404.js`。

# API路由

## 创建方式：在`pages/api`目录下创建一个函数即可

```javascript
export default function handler(req, res) {
  res.status(200).json({ text: 'Hello' })
}
```

## 使用API路由的注意点

1. 不要在页面组件的 `getStaticProps` 或 `getStaticPaths`中向API路由中发出请求。需要请求参数和资源，可以直接在 `getStaticProps` 或 `getStaticPaths`中编写服务端代码。

- - 原因：`getStaticProps` 或 `getStaticPaths`只会在服务端调用，也不会被打包到JS文件中，发送给客户端。所以应该直接在`getStaticProps` 或 `getStaticPaths`中进行服务端资源的获取、数据库的查询操作。

2. 应用场景：处理表单输入。

- - 可以在页面上创建一个表单，并让它向您的 API 路由发送 POST 请求。然后在API路由中编写代码直接将表单内容存到数据库中。

3. 利用API路由实现预览模式。