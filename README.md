## nextjs 官方文档（current branch 对应如下文档）

- [Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)

---

路由处理程序允许您使用 `Web` 请求为给定路由创建自定义请求处理程序和响应

```md
需要了解：路由处理程序仅在 `app` 目录中可用。它们相当于 `pages` 目录中的 `API` 路由，这意味着您不需要同时使用 `API` 路由和路由处理程序。
```

## 一、约定

路由处理程序在 `app` 目录内的 `route.js|ts` 文件中定义：[app/api/posts/route.ts](app/api/posts/route.ts)

路由处理程序可以嵌套在 `app` 目录中的任何位置，类似于 `page.js` 和 `layout.js` 。但不能在与 `page.js` 相同的路由段级别上存在 `route.js` 文件。

<img src="assets/04.png" style="width:70%">

每个 route.js 或 page.js 文件接管该路由的所有 HTTP 动词。

## 二、Request Method

我们从 GET 请求开始，现在写一个获取文章列表的接口

```ts
import { NextResponse } from "next/server";

export const GET = async () => {
  const res = await fetch("https://jsonplaceholder.typicode.com/posts");
  const data = await res.json();

  return NextResponse.json(data);
};
```

浏览器访问 `http://localhost:3000/api/posts` 查看接口返回的数据：

<img src="assets/02.png" style="width:70%">

以下 HTTP 方法支持以下方法： `GET` 、 `POST` 、 `PUT` 、 `PATCH` 、 `DELETE` 、 `HEAD` 和 `OPTIONS` 。如果调用了不受支持的方法，`Next.js` 将返回 `405 Method Not Allowed` 响应。

```ts
export const POST = async (request: Request) => {};

export const HEAD = async (request: Request) => {};

export const PUT = async (request: Request) => {};

export const PATCH = async (request: Request) => {};

export const DELETE = async (request: Request) => {};

// 如果 `OPTIONS` 没有定义, Next.js 会自动实现 `OPTIONS`
export const OPTIONS = async (request: Request) => {};
```

修改[app/api/posts/route.ts](app/api/posts/route.ts) 的 `POST` 方法为

```ts
export const POST = async (request: Request) => {
  console.log("request", request);
  const article = await request.json();

  return NextResponse.json(
    {
      id: Math.random().toString(36).slice(-8),
      data: article,
    },
    { status: 201 }
  );
};
```

<img src="assets/03.png" style="width:70%">

## 三、获取请求参数

每个请求方法的处理函数会被传入两个参数，一个 request，一个 context 。两个参数都是可选的：

```ts
export async function GET(request, context) {}
```

`request` 对象是一个 `NextRequest` 对象，它是基于 `Web Request API` 的扩展。使用 `request` ，你可以快捷读取 cookies 和处理 `URL`。

### 1、request query 参数获取

见：[app/api/test/route.ts](app/api/test/route.ts)
打开浏览器输入 http://localhost:3000/api/test?name=myName

```ts
import { NextResponse, type NextRequest } from "next/server";

export const GET = async (request: NextRequest) => {
  // request /api/test
  console.log("request", request.nextUrl.pathname);

  // request URLSearchParams { 'name' => 'myName' }
  console.log("searchParams", request.nextUrl.searchParams);

  return NextResponse.json({
    message: "Hello, world!",
  });
};
```

### 2、request params 参数获取
见：[app/api/blog/[id]/route.ts](app/api/blog/[id]/route.ts)
打开浏览器输入 http://localhost:3000/api/blog/123

```ts
import { NextResponse, type NextRequest } from "next/server";

export const GET = async (
  { params }: { params: { id: string } }
) => {
  // request params { id: '123' }
  console.log("params", await params);

  return NextResponse.json({
    message: "Hello, world!",
  });
};
```

### 3、获取多个参数

见：[app/api/detail/[name]/[age]/route.ts](app/api/detail/[name]/[age]/route.ts)
打开浏览器输入 http://localhost:3000/api/detail/shmily/404
```ts
import { NextResponse, type NextRequest } from "next/server";

export const GET = async (
  { params }: { params: { name: string; age: string } }
) => {
  // request params { name: 'shmily', age: '20' }
  console.log("params", await params);

  return NextResponse.json({
    message: "Hello, world!",
  });
};
```

见：[app/api/home/[...req]/route.ts](app/api/home/[...req]/route.ts)
打开浏览器输入 http://localhost:3000/api/home/123/456/789
```ts
import { NextResponse, type NextRequest } from "next/server";

export const GET = async (
  { params }: { params: { req: string } }
) => {
  // request params { req: ['123', '456','789'] }
  console.log("params", await params);

  return NextResponse.json({
    message: "Hello, world!",
  });
};
```

## 四、Caching 缓存

默认情况下，路由处理程序不会缓存。但是，您可以选择缓存 `GET` 方法。其他受支持的 `HTTP` 方法不会被缓存。要缓存 `GET` 方法，请在路由处理程序文件中使用路由配置选项（例如 `export const dynamic = 'force-static'` ）。
