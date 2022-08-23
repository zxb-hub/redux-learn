## git 使用

### commit 规范

> Git Commit 格式要求：<类型: feat|fix|build|doc|log|chore>(<模块名>): to#<AoneId1> <该提交的描述>

```
示例1：完成一个需求：feat(testModuleName): to#123456 完成testModuleName的需求开发

示例2：修复一个Bug：fix: to#123456 修复关于文本的展示问题
```

完整代码：

`<type>(<修改内容范围>): <一句话说明改了什么> <空行>; <内容详情，可以写多行>`

简单一些, 可以一行：

`<type>(<修改内容范围>): <一句话说明改了什么>`

```
feature(Button): 添加自动转账按钮

我是一个神奇的按钮，点一下中一百万。
点二下二百万。
```

- type 可选值：

  - feat: 新功能

  - fix: 修复问题

  - docs: 文档修改

  - format: 修改代码格式，不影响代码逻辑

  - clean: 清理代码，移除过期代码

  - refactor: 重构代码，非添加新功能，非修复漏洞

  - perf: 性能提升

  - test: 添加测试

  - chore: 修改工具相关（不限于文档、代码生成，Makefile 修改）

  - deps: 依赖更新

## 杂项

1. react 封装组件时，传递 props 可以是任何格式数据，包括 React.ReactNode。
