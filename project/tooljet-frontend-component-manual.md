## ToolJet Frontend 组件使用手册

本文档基于 ToolJet CE 源码分析生成，面向二次开发/在 ToolJet 前端代码里复用既有组件的场景。

- **上游源码**：ToolJet CE（仓库内克隆到 `project/tooljet-ce/`）
- **分析范围**：`frontend/`（重点：`frontend/src/components/ui` 与 `frontend/src/ToolJetUI`）
- **上游版本**：`b81ed81`
- **文档产出日期**：2026-01-28

---

## 1. 快速上手（如何在项目里引用）

### 1.1 路径别名与工具函数

ToolJet 前端使用 `@/` 作为 `frontend/src/` 的别名（例如 `@/lib/utils`）。

- **类名拼接**：统一使用 `cn()`（基于 `classnames + tailwind-merge`，Tailwind 前缀为 `tw-`）

```js
import { cn } from '@/lib/utils';
```

### 1.2 组件分层（你应该从哪里找组件）

ToolJet 前端实际存在多套 UI 组件来源，优先级建议如下：

- **优先使用**：`frontend/src/components/ui/`
  - 这是目前“设计系统化”更明显的一套（大量使用 `@radix-ui/*` + `class-variance-authority` + `tw-` 前缀 Tailwind）
  - 对外接口多数通过 `PropTypes` 明确了 props
  - 部分组件配了 Storybook（可直接对照用法）
- **其次使用**：`frontend/src/ToolJetUI/`
  - 偏业务工具组件（Tabs、Timepicker、DateRangePicker 等）
  - 有些组件并未在 `ToolJetUI/index.js` 统一导出，需要按路径直接引入
- **谨慎使用**：`frontend/src/_ui/`、`frontend/src/_components/`
  - 历史包袱更重、命名/样式体系不完全统一（很多地方依赖 bootstrap / 老的变量体系）
  - 适合在既有页面里“就地复用”，不推荐作为新功能的首选组件库

---

## 2. 开发约定（和组件使用强相关）

### 2.1 样式与主题

- **Tailwind**：大量使用 `tw-` 前缀（避免与全局 class 冲突），并通过 `cn()` 合并。
- **CSS 变量**：颜色/语义值大量来自 CSS Variables（例如 `var(--icon-default)`、`var(--interactive-focus-outline)`）。
- **SCSS**：部分组件仍使用 `.scss`（例如 `Button.scss`、`ToolJetUI/*/*.scss`）。

### 2.2 表单校验（validation 约定）

`Input`、`Dropdown`、`Textarea` 等组件常见约定：

- **validation**：传入一个函数 `validation(eOrValue)`，返回：
  - `{ valid: boolean, message: string }`
- **onChange**：组件会把校验结果作为第二参数回传：
  - `onChange(eOrValue, validateObj)`

注意：不同组件的第一个参数可能是 **DOM event** 或 **Radix 的 value（string）**，见各组件说明。

---

## 3. `components/ui` 组件清单与用法

路径：`frontend/src/components/ui/`

> 说明：该目录里同时出现了 `TextArea/` 与 `Text Area/` 两份相似实现（大小写/空格差异）。二次开发时建议统一选择其中一份（通常用 `TextArea/`）。

### 3.1 Button（按钮）

- **入口**：`@/components/ui/Button/Button`
- **导出**：`{ Button, buttonVariants }`

核心 props（来自 `PropTypes`）：

- **variant**：`primary | secondary | outline | ghost | ghostBrand | dangerPrimary | dangerSecondary`
- **size**：`large | default | medium | small`
- **leadingIcon / trailingIcon**：图标名（字符串）
  - 默认走 `SolidIcon`（ToolJet 内置图标）
  - `isLucid={true}` 时走 Lucide `DynamicIcon`（`leadingIcon`/`trailingIcon` 为 Lucide icon name）
- **iconOnly**：纯图标按钮（会影响 padding 与 icon 颜色策略）
- **isLoading / loaderText**：加载态（可选显示 loaderText）
- **fill**：覆盖 icon 颜色（仅在 `fill` 非默认值且传入时生效）
- **asChild / component**：可渲染为 `Slot` 或指定标签（例如 `button`、`a`）

示例：

```jsx
import { Button } from '@/components/ui/Button/Button';

export function Example() {
  return (
    <>
      <Button variant="primary" size="default">保存</Button>
      <Button variant="secondary" leadingIcon="smilerectangle">带前置图标</Button>
      <Button variant="outline" isLucid leadingIcon="rocket">Lucide 图标</Button>
      <Button variant="ghost" iconOnly trailingIcon="smilerectangle" aria-label="更多" />
      <Button variant="dangerPrimary" isLoading loaderText="删除中">删除</Button>
    </>
  );
}
```

### 3.2 Input（输入框）

- **入口**：`@/components/ui/Input/Index`
- **类型分发**：
  - `type === 'editable title'`：走 `EditableTitleInput`
  - 其它：走 `CommonInput`（内部再区分 `TextInput`/`NumberInput`）

核心 props（来自 `PropTypes` + 代码行为）：

- **type**：`text | number | editable title | password | email`
- **size**：`small | medium | large`
- **leadingIcon**：字符串（ToolJet `SolidIcon` 名称）
- **trailingAction**：`clear | loading`
  - `clear` 会读取 `onClear`
  - `trailingActionDisabled` 可单独禁用 trailing action
- **validation / isValidatedMessages**：
  - `validation(e)` 返回 `{valid,message}`，并会在 `onChange(e, validateObj)` 回传
  - `isValidatedMessages`（对象）可外部注入展示状态（常用于复杂表单场景）
- **password/encrypted 场景**：
  - `type === 'password'` 时会出现 “Edit/Cancel + Encrypted” 的 UI（依赖 `propertyKey`/`isEditing`/`handleEncryptedFieldsToggle` 等业务 props）
  - `isWorkspaceConstant` 会在占位符包含 `{{constants` 或 `{{secrets` 时触发（用于显示明文/特殊逻辑）

示例（最常见用法）：

```jsx
import Input from '@/components/ui/Input/Index';

export function Example() {
  return (
    <Input
      type="text"
      size="medium"
      label="名称"
      placeholder="请输入"
      leadingIcon="search"
      trailingAction="clear"
      onClear={() => {}}
      validation={(e) => {
        const v = e.target.value ?? '';
        return v.length > 0 ? { valid: true, message: 'OK' } : { valid: false, message: '必填' };
      }}
      onChange={(e, validateObj) => {
        // e 是 DOM event
        // validateObj 可能为 undefined（未传 validation 时）
      }}
    />
  );
}
```

### 3.3 Checkbox（复选 / 单选 / Checkmark）

- **入口**：`@/components/ui/Checkbox/Index`
- **实现**：Radix `@radix-ui/react-checkbox`

核心 props：

- **type**：`checkbox | radio | checkmark`
- **size**：`default | large`
- **intermediate**：中间态（优先级：`intermediate && !disabled` 时渲染中间态 icon）
- **align**：`left | right`（右对齐时布局会反转）
- **label / helper**：可选文案

示例：

```jsx
import Checkbox from '@/components/ui/Checkbox/Index';

export function Example() {
  return (
    <>
      <Checkbox />
      <Checkbox type="radio" label="选项 A" />
      <Checkbox type="checkbox" intermediate label="半选" helper="用于层级选择" />
    </>
  );
}
```

### 3.4 Switch（开关）

- **入口**：`@/components/ui/Switch/Index`
- **实现**：Radix `@radix-ui/react-switch`

核心 props：

- **checked / onCheckedChange**：受控（Radix 约定）
- **size**：`default | large`
- **align**：`left | right`
- **label / helper**

示例：

```jsx
import Switch from '@/components/ui/Switch/Index';

export function Example() {
  return <Switch checked={true} onCheckedChange={() => {}} label="启用" helper="立即生效" />;
}
```

### 3.5 Dropdown（下拉选择）

- **入口**：`@/components/ui/Dropdown/Index`
- **实现**：Radix `@radix-ui/react-select`

核心 props：

- **options**：对象字典（会 `Object.keys(options)` 遍历）
  - 每项结构常见为：`{ value, label?, leadingIcon?, avatarSrc?, avatarAlt?, avatarFall? }`
- **onChange(value, validateObj)**：
  - 第一个参数是 **选中的 value（string）**，不是 DOM event
- **validation(value)**：返回 `{valid,message}`，并透传给 `onChange`
- **container**：可传 DOM 或函数，控制 Portal 挂载点（解决 modal/popover 内 z-index/滚动问题）
- **theme**：`light | dark`（影响 portal 侧的 class）

关于“只允许一个 dropdown 打开”：

- `Dropdown` 内部通过 `useDropdownContext()` 实现“打开一个时关闭其它”
- 如果页面最外层包了 `DropdownProvider`，效果更完整；没包也会回退成 no-op，不会报错

示例：

```jsx
import Dropdown from '@/components/ui/Dropdown/Index';
import { DropdownProvider } from '@/components/ui/Dropdown/DropdownProvider';

export function Example() {
  return (
    <DropdownProvider>
      <Dropdown
        label="成员"
        placeholder="请选择"
        options={{
          a: { value: 'a', label: 'Alice', avatarSrc: 'https://github.com/shadcn.png', avatarAlt: '@shadcn' },
          b: { value: 'b', label: 'Bob' },
        }}
        onChange={(value, validateObj) => {
          // value 为 string
        }}
      />
    </DropdownProvider>
  );
}
```

### 3.6 Textarea（文本域）

- **入口**：`@/components/ui/TextArea/Index`
- **特性**：自动高度（最小 34px，最大 88px）

核心 props：

- **width / initialHeight**
- **onChange(e, validateObj)**：第一个参数是 DOM event
- **validation(e)**：返回 `{valid,message}`
- **label / helperText / required**

示例：

```jsx
import Textarea from '@/components/ui/TextArea/Index';

export function Example() {
  return (
    <Textarea
      label="描述"
      placeholder="请输入"
      helperText="最多建议 200 字"
      onChange={(e) => {}}
    />
  );
}
```

### 3.7 Tooltip（静态气泡容器）

- **入口**：`@/components/ui/Tooltip/Tooltip`
- **注意**：这是一个“内容容器 + 箭头”的实现，不是 Radix Tooltip 触发器；通常配合 hover/定位 class 使用

核心 props：

- **tooltipLabel**：无 children 时显示
- **supportingText**：可选二行文案
- **theme**：`light | dark`
- **arrow**：`Bottom Center | Bottom Left | Bottom Right | Top Center | Left | Right`
- **width**

示例（在 `EditableTitleInput` 内部的典型用法是：`peer-hover:tw-flex` 控制显示）：

```jsx
import Tooltip from '@/components/ui/Tooltip/Tooltip';

export function Example() {
  return (
    <div className="tw-relative">
      <Tooltip arrow="Top Center" theme="dark" className="tw-absolute">
        <div className="tw-text-text-on-solid">提示内容</div>
      </Tooltip>
    </div>
  );
}
```

### 3.8 FileUploader（文件上传）

- **入口**：
  - 上传区：`@/components/ui/FileUploader/FileUpload/Index`
  - 文件列表：`@/components/ui/FileUploader/FileList/Index`
  - Storybook 里通常组合使用（上传 + 列表）

核心 props（上传区 `FileUpload`）：

- **type**：`single | multiple`
- **files**：`File[]`
- **onFilesChange**：更新文件数组（`setFiles`）
- **acceptedFormats / maxSize**：仅用于展示“约束提示”（不做强校验/拦截）
- **disabled / label / helperText / required / width**

示例（参考 story 组合）：

```jsx
import React from 'react';
import FileUpload from '@/components/ui/FileUploader/FileUpload/Index';
import FileList from '@/components/ui/FileUploader/FileList/Index';

export function Example() {
  const [files, setFiles] = React.useState([]);
  return (
    <>
      <FileUpload
        type="multiple"
        files={files}
        onFilesChange={setFiles}
        width="300px"
        label="上传文件"
        helperText="支持 PNG/JPG/PDF"
        acceptedFormats="PNG, JPG, PDF"
        maxSize={10}
      />
      <FileList type="multiple" files={files} onRemove={(f) => setFiles(files.filter((x) => x !== f))} width="300px" />
    </>
  );
}
```

### 3.9 ListItems（列表行 / 可编辑条目）

- **入口**：`@/components/ui/ListItems/Index`
- **特点**：
  - 支持 leadingIcon / addon / error / supportingText
  - 支持 trailing actions（edit/delete/menu/duplicate）
  - 内置 “edit 模式”（会在行内渲染一个简易输入框并提供 save/cancel）

核心 props：

- **background**：是否默认给一层背景
- **indexed**：是否显示缩进（无 leadingIcon 时）
- **trailingActionEdit/Delete/Menu/Duplicate**：是否显示对应 action
- **onSaveEdit / onDelete / onMenu / onDuplicate**

示例：

```jsx
import ListItems from '@/components/ui/ListItems/Index';

export function Example() {
  return (
    <ListItems
      width="260px"
      label="List Item"
      trailingActionEdit
      trailingActionDelete
      onSaveEdit={(v) => {}}
      onDelete={() => {}}
    />
  );
}
```

### 3.10 Avatar / Label / Card

- **Avatar**：`@/components/ui/Avatar/avatar`
  - `Avatar` / `AvatarImage` / `AvatarFallback`（Radix Avatar）
- **Label**：`@/components/ui/Label/Label`
  - `type`：`label | helper`，`size`：`default | large`
- **Card**：`@/components/ui/Card/Index`
  - 典型的 shadcn 风格：`Card`/`CardHeader`/`CardContent`/`CardFooter` 等

---

## 4. `ToolJetUI` 组件清单与用法

路径：`frontend/src/ToolJetUI/`

### 4.1 统一导出的组件

`ToolJetUI/index.js` 目前只导出了：

- **DateRangePicker**：`@/ToolJetUI`（内部目录名是 `DateRangepicker`）
- **Tag**：`@/ToolJetUI`

示例：

```jsx
import { DateRangePicker, Tag } from '@/ToolJetUI';
```

#### DateRangePicker（日期/日期时间范围选择）

- **入口**：`@/ToolJetUI/DateRangepicker`（或从 `@/ToolJetUI` 解构）
- **类型**：
  - `type="date"`：日期范围（默认）
  - `type="datetime"`：日期时间范围（组件内部使用 `DateTimeRangePicker`）
- **静态方法**：
  - `DateRangePicker.setMaxDate(date, range, timeUnit)`
  - `DateRangePicker.setMinDate(date, range, timeUnit)`

#### Tag（标签）

- **入口**：`@/ToolJetUI/Tag`（或从 `@/ToolJetUI` 解构）
- **type**：`base | default | filter`（内部映射到不同 class）
- **回调**：
  - `callBack`：点击整个 tag
  - `handleIconCallBack`：点击“清除 icon”（会 `stopPropagation`）
  - `renderIcon`：自定义 icon 渲染函数

### 4.2 未统一导出，但可直接引入的组件（常见）

> 这些组件在代码里常见被直接 `import ... from '@/ToolJetUI/xxx'` 使用。

- **ToggleGroup / ToggleGroupItem**：`@/ToolJetUI/SwitchGroup/*`
  - 基于 Radix ToggleGroup（single）
- **Tabs / Tab**：`@/ToolJetUI/Tabs/*`
  - 基于 `react-bootstrap/Tabs` 与 `react-bootstrap/Tab`
- **Timepicker**：`@/ToolJetUI/Timepicker/Timepicker`
  - 基于 `react-datepicker` 的 time-only picker
- **Loader**：`@/ToolJetUI/Loader/Loader`
  - SVG spinner（支持 `reverse`、`absolute`、自定义颜色）
- **List**：`@/ToolJetUI/List/List`
  - 基于 `react-bootstrap/ListGroup`，并集成 actions menu、拖拽等（偏业务）
- **AddNewButton**：`@/ToolJetUI/Buttons/AddNewButton/AddNewButton`
  - 内部用 `ButtonSolid`（旧 UI 体系的按钮）

---

## 5. 新增/维护组件的建议（用于二次开发）

### 5.1 优先补齐 Storybook

`frontend` 自带 Storybook：

- 目录：`frontend/src/stories/` 与 `frontend/src/components/ui/*/*.stories.jsx`
- 常用命令（在 ToolJet 仓库 `frontend/` 内执行）：

```bash
npm install
npm run storybook
```

### 5.2 组件实现风格建议（与现有体系一致）

- **优先使用 Radix primitives**（Checkbox/Select/Switch 等）作为可访问性与交互基础
- **使用 `cva` + `cn()`** 管理 variants 与 class 合并
- **使用语义化 CSS 变量**（例如 icon/text/background 语义变量）避免硬编码颜色
- **对外 props 明确化**：保持 `PropTypes`（当前项目仍以 JS/JSX 为主）

