# TODO.html 项目记忆

## 项目概述
单页面待办事项管理应用，使用纯 HTML/CSS/JavaScript 实现，数据持久化到 IndexedDB + localStorage。

## 主要功能

### 1. 折叠功能
- 点击分类/子类标题可折叠/展开
- 折叠状态保存到 localStorage
- 使用 CSS 类 `.collapsed` 控制显示/隐藏

### 2. 删除功能
- 可删除任意分类和子类（包括内置的）
- 删除按钮在悬停时显示（垃圾桶图标）
- 支持撤销删除（Toast 提示 5 秒）
- 删除状态持久化到 `deletedSections` 和 `deletedSubsections`

### 3. 数据存储
- **IndexedDB** 作为主要存储（file:// 协议下更稳定）
- **localStorage** 作为备份
- 自动数据迁移：首次加载时从 localStorage 迁移到 IndexedDB
- 导出/导入功能（JSON 文件）

### 4. 暗夜模式
- 右上角切换按钮（🌙/☀️）
- 主题偏好保存到 localStorage
- 完整的暗色主题样式

### 5. 拖拽排序
- 使用 Sortable.js 库实现
- 支持桌面端和移动端触摸拖拽
- 拖拽完成后自动保存新顺序
- CSS 类：`.sortable-ghost`（占位符）、`.sortable-drag`（拖拽中）、`.sortable-chosen`（选中）

### 6. 搜索功能
- 折叠式搜索按钮（点击展开）
- 实时搜索（300ms 防抖）
- 快捷键：`Ctrl+F` 或 `/` 聚焦搜索框，`Esc` 清除
- 搜索框自动收起（失焦且无内容时）
- 显示"没有找到匹配的待办事项"提示

### 7. 备注功能
- 每个自定义待办可添加备注
- 点击备注图标展开/收起备注区域
- 失焦自动保存
- 有备注时图标高亮显示

### 8. 任务优先级
- 三级优先级：高（红）、中（橙）、低（绿）
- 添加待办时选择优先级，默认"中"
- 待办左侧彩色边条指示优先级
- 待办内容旁显示优先级标签
- 过滤栏可按优先级筛选

## 数据结构

```javascript
{
  customTodos: [],           // 自定义待办事项（含 priority 字段）
  states: {},                // 待办完成状态
  customSections: {},        // 自定义分类
  customSubsections: {},     // 自定义子类
  deletedSections: [],       // 已删除的内置分类
  deletedSubsections: {},    // 已删除的内置子类
  collapsedSections: {},     // 分类折叠状态
  collapsedSubsections: {},  // 子类折叠状态
}

// customTodos 单项结构
{
  id: 'custom_xxx',
  section: 'learning',
  subsection: '1.1',
  text: '待办内容',
  date: '2026-03-25',
  priority: 'high',          // high | medium | low
  completed: false,
  notes: '备注内容'
}
```

## 关键函数

| 函数 | 用途 |
|------|------|
| `initDB()` | 初始化 IndexedDB |
| `loadFromStorageSync()` | 同步加载数据 |
| `saveToStorageSync(data)` | 同步保存数据 |
| `toggleSection(sectionId)` | 切换分类折叠状态 |
| `toggleSubsection(subsectionId)` | 切换子类折叠状态 |
| `deleteSection(sectionId)` | 删除分类（含导航链接） |
| `deleteSubsection(sectionId, subsectionId)` | 删除子类 |
| `undoDelete()` | 撤销删除操作 |
| `loadSavedState()` | 页面加载时恢复状态 |
| `initTheme()` | 初始化主题 |
| `toggleTheme()` | 切换暗夜模式 |
| `initSortable()` | 初始化拖拽排序 |
| `saveTodoOrder()` | 保存拖拽后的新顺序 |
| `initSearch()` | 初始化搜索功能 |
| `performSearch(query)` | 执行搜索 |
| `clearSearch()` | 清除搜索 |
| `saveNote(todoId, notes)` | 保存备注 |
| `toggleNotes(element)` | 切换备注展开/收起 |
| `selectPriority(priority)` | 选择优先级 |
| `filterByPriority(priority)` | 按优先级过滤 |

## 分类结构

| ID | 名称 | 子类 |
|----|------|------|
| learning | 学习计划 | 1.1-1.5 |
| work | 工作项目 | 无序号子类 |
| problems | 问题分析记录 | 3.1-3.2 |
| daily | 日常记录 | - |
| commands | 常用命令 | - |
| notes | 技术笔记 | av_s |

## 已删除的分类
- links（参考链接）- 已从 HTML 和 SECTIONS 中移除

## 注意事项
1. 子类存储键名直接使用 sectionKey（如 `learning` 或 `custom_123`）
2. 删除内置内容时需要同时删除导航链接
3. undoDelete 需要处理三种类型：section、subsection、todo
4. renderNewSubsection 使用 `afterend` 插入到最后一个子类后面
5. 优先级值：`high`、`medium`、`low`，默认为 `medium`
6. 关闭模态框时需重置优先级选择为默认值
