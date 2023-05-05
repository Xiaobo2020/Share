# Outline

<!-- markdown-toc GFM -->

- [目的](#目的)
- [痛点](#痛点)
- [建议](#建议)
- [DangerJS](#dangerjs)
- [Q&A](#qa)

<!-- markdown-toc -->

## 目的

~2min

- 发现潜在问题。
- 提高产品性能。
- 统一团队规范。
- 促进团队成长。

## 痛点

~3min

- 时间紧，任务重
  - Author: 开发时间本来就紧张，还要等待 CR
  - Reviewer: 自己也有任务，还要抽时间给别人 CR
- 难以审核
  - MR 太大
  - 不熟悉业务逻辑
  - 代码本身难以理解
  - 干扰内容太多
  - 无从下手

## 建议

~15min

> 共识：CR 的顺利执行需要作者和审核人的共同努力

- 作者
  - 完善前期准备工作
    - Automated Tests
    - Linting
    - Formatting
  - 自我审核
    - 减少低级错误
    - 增加解释性描述
  - 补充 MR 描述
    - 需求背景
    - 设计思路简要介绍
    - 主要改动内容
    - 前后对比截图
  - 控制 MR 大小
  - 合适的审核人
    - 对于代码逻辑熟悉的人
    - 对于项目逻辑精通的人
    - 能找到的人
  - 积极推动
    - 组织线下沟通，但同步结论到线上
    - 当 Reviewer 长时间没有响应的时候主动沟通
- 审核人
  - 友好的态度
  - 不要吝啬赞美
  - 提供更进一步的解释
  - 及时反馈，避免阻塞
    - 当暂时无法马上审核时，需要及时通知 Author 这一情况
    - 当已经提供了一些评论时，需要及时通知 Author ，方便其知晓并做出下一步操作
    - 当认可当前 MR 时，需要及时 Approve 并标注 LGTM(look good to me)并提醒 Author
  - 遵守原则，尊重偏好
    - Checklist

|        | 审核内容                                                                                             |
| :----- | :--------------------------------------------------------------------------------------------------- |
| 功能   | 符合描述？符合需求？功能演示？                                                                       |
| 安全   | 边缘情况？死锁？竞争？溢出？有风险的编码方式？                                                       |
| 复杂度 | 单行是否太复杂？函数是否太复杂？类是否太复杂？无法快速理解？其他人在开发或调试此代码是否会引入错误？ |
| 冗余   | 过度设计？重复造轮子？代码冗余？                                                                     |
| 测试   | MR 中应包含覆盖改动内容的有效测试，除非是处理紧急情况。                                              |
| 文档   | 重要的设计逻辑改动,应该在对应的文档中修改对应的说明,如修改启动命令,则需要在 `README.md` 中同步更新。 |
| 备注   | 解释性备注；文档性备注；提醒性备注。                                                                 |
| 命名   | 风格（常量、变量、函数、文件、文件夹）？意义？                                                       |
| 风格   | 以团队规范或项目配置为准。                                                                           |

---

命名：

```javascript
// 常量
const MAX_IMAGE_SIZE = 1024 * 1024;

// 变量
let isVisited = false;

// 函数
function queryProductList() {
  // ...
}
```

---

备注：

```javascript
// TODO: JIRA-0001 Clean up after some feature is done
/**
 * @description Query product list by id
 * @param {number} id
 * @returns {Array<{name: string; price: number;}>}
 */
function queryProductList(id) {
  // ...
  // 解释性备注，描述为什么，而不是干了什么
}
```

---

简单代码：

```javascript
// not recommended
const key = `${date}/${filename}${jobId && '_' + jobId}.zip`,

// fixed
const key = `${date}/${filename}${(jobId && '_' + jobId) || ''}.zip`,

// recommended
const key = `${date}/${filename}${jobId ? '_' + jobId : ''}.zip`,
```

## DangerJS

~15min

- 介绍
  - 是什么
  - 能干什么
- 配置
  - 评论账号
  - dangerfile
  - pipeline
- 最佳实践

## Q&A

~5min
