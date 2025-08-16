---
title: Obsidian 进阶：用 Dataview 和 Templater 打造自动化知识库
date: 2025-08-16 16:04:00
categories: 实用教程
tags:
  - Obsidian
  - Markdown
  - 教程
cover: https://s2.loli.net/2025/08/16/FkYaEtGQmqC4vhR.jpg
---
## 1. 为什么需要自动化？摆脱重复性工作

当您的 Obsidian 笔记数量增多后，可能会遇到以下瓶颈：

- **创建笔记结构重复**：每次写周报、会议纪要或读书笔记，都需要手动输入相同的标题、日期和属性。
    
- **索引维护困难**：想创建一个“所有项目A的会议纪要”列表，需要手动复制粘贴链接，一旦新增或修改就得手动更新，极易遗漏。
    
- **信息关联孤立**：笔记写完后就成了孤岛，难以从宏观视角发现特定类型（如“所有待办任务”、“今年读过的所有五星书籍”）的笔记。
    

**Dataview** 和 **Templater** 这两个社区插件的组合，正是解决上述问题的终极方案。

- **Templater**：负责**输入自动化**。它是一个强大的模板引擎，能帮您创建结构一致、包含动态信息（如日期、标题）的笔记。
    
- **Dataview**：负责**输出自动化**。它是一个查询引擎，能将您的整个知识库视为一个数据库，根据您设定的条件，动态地聚合、展示和索引您的笔记。
    

本教程将引导您从入门到精通，最终将两者结合，构建一个真正“活”起来的知识库。

## 2. Dataview：将你的知识库变成动态数据库

Dataview 的核心是让您通过查询代码，动态地从您的笔记中提取信息并以列表、表格等形式展示。

### 2.1 核心理念：为笔记赋予“属性”

要让 Dataview 能够查询，首先要让您的笔记“可被查询”。这需要为笔记添加结构化的元数据（Metadata），主要有两种方式：

1. **YAML Frontmatter**：在文件最顶部的 `---` 区域内，以 `键: 值` 的格式定义。
    
    ```YAML
    ---
    type: "Book"
    author: "James Clear"
    rating: 5
    status: "read"
    tags: [productivity, habits]
    ---
    ```
    
2. **行内字段 (Inline Fields)**：在笔记正文中，使用 `键:: 值` 的格式定义。这种方式更灵活，适合在段落中自然地标记信息。
    
    ```Markdown
    - **状态**:: #in-progress
    - **负责人**:: [[张三]]
    - **截止日期**:: 2025-08-30
    ```
    

### 2.2 Dataview 查询语言（DQL）入门

Dataview 的查询代码块以 ` ```dataview ` 开始。其语法结构类似于 SQL，主要由几个关键字组成：

- `LIST` | `TABLE` | `TASK`：决定输出的格式。
    
- `FROM`：指定查询范围（例如某个标签 `#book`、某个文件夹 `"Meetings"`、或它们的组合）。如果省略，则查询整个仓库。
    
- `WHERE`：设定筛选条件，过滤出符合要求的笔记。
    
- `SORT`：定义排序规则（例如按文件创建日期 `file.ctime` 升序 `asc` 或降序 `desc`）。
    

### 2.3 三种核心查询类型详解

**1. LIST (列表查询)**

这是最简单的查询，用于列出符合条件的笔记文件名。

**案例**：列出所有在 `"Projects"` 文件夹下，但还未添加 `#done` 标签的笔记。

```
LIST
FROM "Projects"
WHERE !contains(file.tags, "#done")
```

**2. TABLE (表格查询)**

最常用也最强大的查询类型。您可以自定义表格的列，显示笔记的各种属性。

**案例**：以表格形式展示所有评分为5星的读书笔记，并显示作者和阅读状态。

- 假设笔记元数据为：`type: "Book"`, `author: "作者"`, `rating: 5`, `status: "read"`

```
TABLE author, rating, status
FROM #book
WHERE rating = 5
SORT file.mtime desc
```

_这里的 `author`, `rating`, `status` 会自动从每篇笔记的元数据中抓取。_

**3. TASK (任务查询)**

专门用于聚合您在各个笔记中创建的待办事项。

**案例**：找出所有笔记中，包含 `#project-A` 标签且未完成的待办事项。

- 您需要在笔记中这样写任务：`- [ ] 完成报告的初稿 #project-A`

```
TASK
FROM #project-A
WHERE !completed
```

_Dataview 会自动渲染出任务复选框，并链接回原笔记。_

## 3. Templater：你的智能笔记模板引擎

Templater 远比 Obsidian 自带的核心模板插件强大，它允许您在模板中嵌入动态命令甚至是 JavaScript 脚本。

### 3.1 基础配置与使用

1. 在社区插件市场安装 Templater。
    
2. 在 Templater 的设置中，指定一个“模板文件夹路径”，例如 `Templates`。
    
3. 将您所有的模板文件都存放在这个文件夹中。
    
4. 通过命令面板或快捷键执行 `Templater: Open Insert Template modal` 来使用模板。
    

### 3.2 常用函数与动态命令详解

Templater 的动态命令通常包裹在 `<%* ... %>` 或 `<% ... %>` 中。

- **获取文件名**：`<% tp.file.title %>`
    
    - 当您从一个名为“2025-08-16-周报”的空文件创建笔记时，它会自动将标题填充为 `2025-08-16-周报`。
        
- **动态日期和时间**：`<% tp.date.now("YYYY-MM-DD HH:mm") %>`
    
    - 可以自定义任何日期格式，`now` 可以替换为 `yesterday`, `tomorrow` 等。
        
- **光标定位**：`<% tp.file.cursor() %>`
    
    - 模板插入后，光标会自动移动到这个命令所在的位置，非常适合用于内容主体部分的填写。
        
- **系统交互命令（提示框）**：`<%* const author = await tp.system.prompt("请输入作者：") %>`
    
    - 这是 Templater 的强大之处。它会弹出一个输入框让您输入信息，然后可以将返回的结果用于笔记内容。
        

### 3.3 实战案例：创建一个交互式会议纪要模板

在您的模板文件夹（如 `Templates`）下，创建 `Meeting-Template.md` 文件，内容如下：

```Markdown
---
tags: meeting
date: <% tp.date.now("YYYY-MM-DD") %>
participants: 
---

# Meeting Notes: <% tp.file.title %>

- **会议时间**: <% tp.date.now("YYYY-MM-DD HH:mm") %>
- **与会人员**: <%* await tp.system.prompt("与会人员 (用逗号分隔)") %>
- **会议主题**: <%* await tp.system.prompt("会议主题") %>

---

## 议程

- <% tp.file.cursor() %>

## 行动项
- [ ] 

```

**使用效果**：当您基于此模板创建一个新笔记时，它会自动填入当前日期和笔记标题，然后连续弹出两个输入框，让您输入“与会人员”和“会议主题”，最后将光标定位在“议程”部分，等待您开始记录。

## 4. 终极合体：Dataview + Templater 自动化工作流

现在，我们将两者结合，真正实现“输入 → 处理 → 输出”的全流程自动化。

### 4.1 实战项目：打造一个全自动的“电影追踪系统”

**目标**：

1. 通过 Templater 模板，快速记录想看或看过的电影，并自动添加元数据。
    
2. 通过一个 Dataview 页面，自动生成“待看列表”、“已看列表”和“五星推荐”等动态索引。
    

**步骤一：创建 Templater 电影模板**

在模板文件夹下创建 `Movie-Template.md`：

```Markdown
---
type: "Movie"
tags: [movie]
director: <%* await tp.system.prompt("导演") %>
rating: <%* await tp.system.prompt("评分 (1-5)") %>
status: <%* await tp.system.suggester(["待看", "已看"], ["#tosee", "#watched"]) %>
date_watched: <%* if (await tp.system.suggester(["是", "否"], [true, false])) { tR += tp.date.now("YYYY-MM-DD") } %>
---

# 观影记录：<% tp.file.title %>

> [!INFO] 电影信息
> - **导演**: <% tp.frontmatter.director %>
> - **我的评分**: <% tp.frontmatter.rating %> / 5
> - **观看状态**: <% tp.frontmatter.status %>

## 简评

<% tp.file.cursor() %>

```

_这个模板使用了 `suggester` 函数，它会弹出选择框，让您选择“待看”或“已看”，并自动写入对应的标签，非常高效。_

**步骤二：创建 Dataview 聚合页面**

创建一个新的普通笔记，命名为 `My Movie Dashboard.md`，并写入以下内容：

````Markdown
## 🎬 我的电影仪表盘

## 🍿 待看列表 (Want to See)
```dataview
TABLE director as "导演", rating as "期待分"
FROM #movie
WHERE status = "#tosee"
SORT rating desc
````

## ✅ 已看列表 (Watched)

```
TABLE director as "导演", rating as "我的评分", date_watched as "观看日期"
FROM #movie
WHERE status = "#watched"
SORT date_watched desc
```

## ⭐ 五星推荐

```
LIST
FROM #movie
WHERE rating = 5 AND status = "#watched"
```

**工作流演示**：

1. 您想记录一部新电影《沙丘》，通过 Templater 新建笔记。
    
2. 模板自动运行，依次弹出输入框让您输入导演“丹尼斯·维伦纽瓦”，评分“5”，状态选择“已看”，并自动记录观看日期。
    
3. 笔记创建完成，元数据完美无缺。
    
4. 您打开 `My Movie Dashboard.md`，无需任何操作，Dataview 已自动将《沙丘》添加到了“已看列表”和“五星推荐”中。
    

## 5. 总结与注意事项

通过“Templater 定义结构化输入”和“Dataview 进行动态化输出”的组合，您可以将这套工作流应用到项目管理、文献追踪、习惯养成等任何领域。

**注意事项**：

- **性能**：在拥有数万篇笔记的超大仓库中，复杂的 Dataview 查询可能会有轻微延迟。
    
- **一致性**：这套系统的核心在于元数据的一致性。务必通过 Templater 来创建笔记，以保证 `type`, `status` 等字段的拼写和格式完全统一。
    
- **由简入繁**：先从简单的 `LIST` 和 `TABLE` 查询开始，在实际使用中逐步迭代您的模板和查询需求。