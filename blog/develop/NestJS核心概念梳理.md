---
slug: nest_core_notion
title: NestJS服务框架核心概念梳理|语言模型Prompt
date: 2023-12-08
authors: kuizuo
tags: [server, cloud]
keywords: [develop, cloud]
---

### 模块

module在nest框架中，是运行时的组织者

nest本身只关心你的module的架构，就是通过 `Module` 装饰器中的 `imports` 确定的。根节点是 `AppModule(Root)`,通过 imports 导入其他的`module(Module1, Module2, Module3)` 依次类推。1，2，3 在分别`imports` 其他的子级Module。这样就成了一个树型的组织结构。方便nest进行模块的依次管理

```tsx
// main.ts
@Module({
 imports: [
  Module1,
  Module2,
  Module3
 ]
})
// Module1,
@Module({
 imports:[
  Module1-1,
  Module1-2,
  Module1-3
 ]
})
// Module2,
@Module({
 imports:[
  Module2-1,
  Module2-2,
  Module2-3
 ]
})
// Module3,
@Module({
 imports:[
  Module3-1,
  Module3-2,
  Module3-3
 ]
})

```

`**Module` 中包含了四大块核心内容**

1. controller 控制器
2. provider 提供者( 提供服务 )

原则上来说，我们会把关联性强的 `controller` 和 `provider` 放到一个 `Module` 中。但是在某些场景下，在其他的 `Module` 中能去复用另一个 `Module` provider提供的数据或服务。

再次强调：原则上你要是设计的好，理论上 大家( `Module` ) 之间是没有什么瓜葛的

A `Module` 要用到 B `Module` 里面的 `provider`,那么在B的某些服务，就通过`exports` 暴露出来。

A Module 通过 imports这个字段，将B暴露的某些服务导入。然后将B导出的服务，在通过注入到`providers`中。这样A就可以用B的服务了。

基于此需求，`Module` 又多出了两大支撑内容

1. `imports` 导入
2. `exports` 到出

```tsx
// B Module
@Module({
 imports: [...],
 controllers: [...],
 providers: [...],
 exports: [...]
})
```

## 核心生命周期

![iShot_2023-06-08_16.20.37.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/19deea74-ca6f-45ab-b26c-ff2d22a0c418/bf905caf-3469-4834-81a5-94fdb692600a/iShot_2023-06-08_16.20.37.png)

客户端 → 中间件 → 守卫 → 拦截器(前置) → 管道 → 控制器 → 服务 → 拦截器(后置) → 过滤器 → 响应

**中间件、守卫、拦截器、管道、过滤器** 都算是一种钩子方法

钩子方法：定义一个空的函数，用户可以自定义内容来改变原有的逻辑

下面是对应钩子的内部模块的子钩子的执行顺序

💡 **客户端↓↓↓**

1. 中间件：全局中间件 → 模块中间件
2. 守卫：全局守卫 → 控制器守卫 → 路由守卫
3. 拦截器：全局拦截器(前置) → 控制器拦截器(前置) → 路由拦截器(前置)
4. 管道：全局管道 → 控制器管道 → 路由管道 → 路由参数管道
5. 控制器
6. 服务
7. 拦截器：路由拦截器(后置) → 控制器拦截器(后置) → 全局拦截器(后置) -》》》洋葱圈模型
8. 过滤器：路由过滤器 → 控制器过滤器 → 全局过滤器

💡 **响应↑↑↑**

守卫（`**Guards**`）：用于鉴权。

1. 用于保护路由，免受未授权的访问
2. 解决问题：用于验证用户是否已经登录或者具有访问某个路由的权限

拦截器（`**Interceptor**`）：用于请求/响应处理。

1. 用于在请求到达控制器之前或之后对请求和响应进行处理
2. 提供了一种在请求和响应的生命周期中执行额外逻辑的机制，使开发更灵活处理请求和响应
3. 解决问题：在请求和响应的生命周期中执行额外的逻辑，例如：日志记录、性能监控等

管道（`**Pipe**`）：用于参数转换。

1. 用于转换和验证控制器中的参数
2. 解决问题：用于验证和转换输入数据，确保数据的正确和一致

过滤器（`**Filter**`）：用于异常处理。

1. 用于在抛出异常前后对异常进行处理
2. 解决问题：用于全局异常处理

## 语言模型Prompt

> 公式：指令 + 输入数据 + 背景 + 输出要求

通过五个要素的组合，可以更有效地引导大预言模型生成用户期望的输出

指令

1. 提供明确的指令
2. 简述、解释、翻译、总结、润色、写一篇文章等

输入数据

1. 原始数据输入
2. 总结的内容、提供的文本、编写的代码

背景

1. 任务的上下文信息，帮助模型更好的理解
2. 关于计算机科普文，写给小学生、大学生、计算机专业人员，得到的内容完全不同

指令和输入数据

输出要求

1. 期望模型输出的指标结构、格式等
2. 输出5条xxx相关内容，按照张瑶持续排序1…2…3..，按照Markdown表示形式输出

示例

1. 提供一个具体例子

## Prompt规则

1. 明确具体：加入场景要求、具体任务(给出一点的边界条件)
2. 提供上下文：区分不同的部分(`"", ```, <>, JSON、Markdown`等)
3. 正确的语法：正确的语法和拼写，避免使用技术性活不常用的术语
4. 分布询问：复杂任务分布进行，提供引导或示例
5. 输出要求：提供输出要求，比如，结构化的输出(实例)

## 案例

- **指令**：初始化一个新的NestJS项目。
- **输入数据**：使用Nest CLI创建一个基本的NestJS骨架项目。
- **背景**：刚开始接触NestJS，需要了解项目的基本结构和初始化流程。
- **输出要求**：提供一个完整的NestJS项目骨架，包括基本配置和文件结构。

1. 指令：解释一下NestJS中的拦截器是如何工作的。
输入数据：我在一个NestJS项目中遇到了需要在请求处理前进行一些预处理的情况...
背景：我是一名有2年NestJS开发经验的开发者。
输出要求：请提供一个实际的例子，演示如何使用NestJS拦截器处理请求。

2. 指令：介绍一下NestJS中的模块化设计理念。
输入数据：我在一个大型NestJS项目中，希望更好地组织和管理项目结构...
背景：我是一名有3年Node.js和1年NestJS经验的开发者。
输出要求：请提供关于NestJS模块化设计的准则和一个实际的模块划分示例。

3. 指令：解释NestJS中的中间件的作用和使用场景。
输入数据：我在一个NestJS项目中需要在路由处理前进行一些通用的处理，考虑使用中间件...
背景：我是一名有1年NestJS经验的开发者。
输出要求：请提供一个实际的应用场景，并演示如何在NestJS中使用中间件。

4. 指令：介绍一下NestJS中的数据验证和转换工具。
输入数据：我在一个NestJS项目中需要对传入的请求数据进行验证和转换...
背景：我是一名有2年NestJS经验的开发者。
输出要求：请提供一个实际的例子，说明如何在NestJS中使用验证和转换工具。

5. 指令：请解释一下NestJS中如何处理跨域请求。
输入数据：我在一个NestJS项目中，前端部署在不同的域上，需要处理跨域请求...
背景：我是一名有3年前端和1年NestJS经验的开发者。
输出要求：请提供NestJS中处理跨域请求的配置方法和实例。

6. 指令：解释一下NestJS中的WebSocket支持。
输入数据：我在一个实时通讯的NestJS项目中，考虑使用WebSocket...
背景：我是一名有2年NestJS开发经验的开发者。
输出要求：请提供一个简单的例子，说明如何在NestJS中使用WebSocket进行实时通讯。

7. 指令：介绍一下NestJS中的单元测试方法。
输入数据：我在一个NestJS项目中，想要增加单元测试以提高代码质量...
背景：我是一名有1年NestJS经验的开发者，对单元测试不太熟悉。
输出要求：请提供一个基本的NestJS单元测试示例，包括测试框架的选择和配置。

8. 指令：解释NestJS中如何进行用户身份验证和授权。
输入数据：我在一个NestJS项目中需要添加用户身份验证和授权功能...
背景：我是一名有2年NestJS经验的开发者。
输出要求：请提供一个实际的例子，演示如何在NestJS中实现用户身份验证和授权。

9. 指令：请解释一下NestJS中的依赖注入系统。
输入数据：我在一个NestJS项目中，想要了解依赖注入是如何工作的...
背景：我是一名有1年NestJS经验的开发者。
输出要求：请提供一个简单的例子，说明在NestJS中如何使用依赖注入进行组件的解耦。

10. 指令：介绍一下NestJS中的缓存管理。
输入数据：我在一个NestJS项目中，考虑使用缓存来提高性能...
背景：我是一名有2年NestJS开发经验的开发者，对缓存管理不太了解。
输出要求：请提供一个实际的例子，说明在NestJS中如何进行缓存管理。

## 一些通用的Prompt

请你作为前端全栈开发的资深技术专家，你的背景是:Node.js全栈，React作为主要的技术栈，了解常见的后端中间件，在开发中的主要以现有的流行的解决方案来进行开发与方案设计，只有在必要的情况下才引导我来造新的轮子，你主要聚焦于前端全栈即JavaScript和 Typescript作为主力开发语言。

对于代码，我的角色是一名5年的前端开发工作经验的开发者，我的技术偏好是:React为主要技术栈，会一些Nestjs相关技术栈，熟练使用VSCode及相应的开发工具及macOS环境，喜欢新的事物，新的技术方案与思路，学习能力好。

我的最终目标是：熟练使用NestJS相关技术，开发出真实的企业级项目

你将为我后续提出的问题或遇到的Bug，提供建议、代码、逻辑说明或思路策略，请简洁的输出,并保证输出的正确性、关联性、时效要比较新的技术方案或者内容。

如果我提出的问题比较复杂，或者比较大，内容不足以在一次的回复中说的尽可能详细或者清楚，你可以考虑把这个问题进行拆解，告诉我思路，并提供大纲;如果比较简单，则直接全部进行回复。在今后的对话中，不再需要我进行额外的说明。如果你明白了我的要求与背景，请回复“好的”。

请在之后的回复中，使用中文，这是我个人的语言偏好
