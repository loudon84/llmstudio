# ToolJet Frontend 组件使用手册

## 目录

1. [项目概述](#1-项目概述)
2. [目录结构](#2-目录结构)
3. [状态管理 (Zustand)](#3-状态管理-zustand)
4. [基础 UI 组件库 (_ui)](#4-基础-ui-组件库-_ui)
5. [App Builder 可视化组件 (Widgets)](#5-app-builder-可视化组件-widgets)
6. [服务层 (Services)](#6-服务层-services)
7. [自定义 Hooks](#7-自定义-hooks)
8. [组件配置系统](#8-组件配置系统)
9. [开发最佳实践](#9-开发最佳实践)
10. [二次开发指南](#10-二次开发指南)

---

## 1. 项目概述

ToolJet Frontend 是一个基于 React 的低代码应用构建平台前端项目，主要包含以下核心模块：

- **App Builder**: 可视化应用编辑器
- **Query Manager**: 数据查询管理界面
- **ToolJet Database UI**: 内置数据库界面
- **Editor**: 主编辑界面

### 技术栈

| 技术 | 用途 |
|------|------|
| React | UI 框架 |
| Zustand | 状态管理 |
| Immer | 不可变状态更新 |
| SCSS | 样式系统 |
| Webpack | 构建工具 |
| Tabler Icons | 图标库 |

---

## 2. 目录结构

```
frontend/src/
├── _components/          # 公共业务组件
├── _helpers/             # 工具函数
├── _hoc/                 # 高阶组件
├── _hooks/               # 自定义 Hooks
├── _lib/                 # 第三方库封装
├── _services/            # API 服务层
├── _stores/              # Zustand 状态管理
├── _styles/              # 全局样式
├── _ui/                  # 基础 UI 组件库
├── _utils/               # 工具函数
├── App/                  # 应用入口
├── AppBuilder/           # App Builder 核心模块
│   ├── _contexts/        # React Context
│   ├── _helpers/         # 辅助函数
│   ├── _hooks/           # Builder 专用 Hooks
│   ├── _stores/          # Builder 状态管理
│   ├── _utils/           # 工具函数
│   ├── AppCanvas/        # 画布组件
│   ├── CodeBuilder/      # 代码构建器
│   ├── CodeEditor/       # 代码编辑器
│   ├── Header/           # 顶部导航
│   ├── LeftSidebar/      # 左侧边栏
│   ├── QueryManager/     # 查询管理器
│   ├── QueryPanel/       # 查询面板
│   ├── RightSideBar/     # 右侧属性面板
│   ├── Viewer/           # 应用预览器
│   ├── WidgetManager/    # 组件管理器配置
│   └── Widgets/          # 可视化组件
├── Editor/               # 旧版编辑器
├── HomePage/             # 首页
├── modules/              # 功能模块
├── TooljetDatabase/      # 数据库模块
└── ToolJetUI/            # 新版 UI 组件
```

---

## 3. 状态管理 (Zustand)

ToolJet 使用 Zustand 进行状态管理，采用模块化的 Slice 模式。

### 3.1 核心 Store 列表

| Store | 文件路径 | 用途 |
|-------|----------|------|
| `editorStore` | `_stores/editorStore.js` | 编辑器核心状态 |
| `currentStateStore` | `_stores/currentStateStore.js` | 应用运行时状态 |
| `dataQueriesStore` | `_stores/dataQueriesStore.js` | 数据查询状态 |
| `dataSourcesStore` | `_stores/dataSourcesStore.js` | 数据源状态 |
| `appDataStore` | `_stores/appDataStore.js` | 应用数据状态 |
| `queryPanelStore` | `_stores/queryPanelStore.js` | 查询面板状态 |
| `resolverStore` | `_stores/resolverStore.js` | 引用解析状态 |
| `appVersionStore` | `_stores/appVersionStore.js` | 应用版本状态 |

### 3.2 editorStore 使用示例

```javascript
import { useEditorStore, useEditorActions } from '@/_stores/editorStore';

// 获取状态
const currentLayout = useEditorStore((state) => state.currentLayout);
const selectedComponents = useEditorStore((state) => state.selectedComponents);

// 获取 actions
const { 
  toggleCurrentLayout, 
  setSelectedComponents,
  setHoveredComponent 
} = useEditorActions();

// 切换布局
toggleCurrentLayout('mobile');

// 设置选中组件
setSelectedComponents([componentId]);
```

#### editorStore 状态结构

```javascript
const initialState = {
  currentLayout: 'desktop',        // 当前布局模式
  showComments: false,             // 是否显示评论
  hoveredComponent: '',            // 悬停组件 ID
  selectionInProgress: false,      // 是否正在选择
  selectedComponents: [],          // 选中的组件列表
  isEditorActive: false,           // 编辑器是否激活
  canUndo: false,                  // 是否可撤销
  canRedo: false,                  // 是否可重做
  currentVersion: {},              // 当前版本
  appDefinition: {},               // 应用定义
  isLoading: true,                 // 加载状态
  showLeftSidebar: true,           // 是否显示左侧栏
  currentPageId: null,             // 当前页面 ID
  editorCanvasWidth: 1092,         // 画布宽度
  canvasBackground: {},            // 画布背景
};
```

### 3.3 currentStateStore 使用示例

```javascript
import { useCurrentStateStore, getCurrentState } from '@/_stores/currentStateStore';

// 在组件中使用
const queries = useCurrentStateStore((state) => state.queries);
const components = useCurrentStateStore((state) => state.components);
const globals = useCurrentStateStore((state) => state.globals);

// 获取完整状态（不在组件中）
const fullState = getCurrentState();

// 设置状态
const { setCurrentState, setErrors } = useCurrentStateStore((state) => state.actions);

setCurrentState({
  queries: { ...updatedQueries },
  components: { ...updatedComponents }
});
```

#### currentStateStore 状态结构

```javascript
const initialState = {
  queries: {},                    // 查询数据
  components: {},                 // 组件状态
  globals: {
    theme: { name: 'light' },     // 主题
    urlparams: null,              // URL 参数
    environment: { id: null, name: null },
    currentUser: {},              // 当前用户
  },
  errors: {},                     // 错误信息
  variables: {},                  // 变量
  client: {},                     // 客户端变量
  server: {},                     // 服务端变量
  page: {
    handle: '',
    variables: {},
  },
  constants: {},                  // 常量
};
```

### 3.4 dataQueriesStore 使用示例

```javascript
import { 
  useDataQueriesStore, 
  useDataQueries, 
  useDataQueriesActions 
} from '@/_stores/dataQueriesStore';

// 获取所有查询
const dataQueries = useDataQueries();

// 获取 actions
const { 
  fetchDataQueries,
  createDataQuery,
  deleteDataQueries,
  renameQuery,
  saveData 
} = useDataQueriesActions();

// 创建查询
createDataQuery(selectedDataSource, shouldRunQuery, customOptions);

// 删除查询
deleteDataQueries(queryId);

// 重命名查询
renameQuery(queryId, newName);
```

### 3.5 Store 开发规范

**约束：使用 Immer 中间件确保不可变更新**

```javascript
import { create, zustandDevTools } from './utils';

export const useMyStore = create(
  zustandDevTools(
    (set, get) => ({
      // 状态
      myData: [],
      
      // Actions
      actions: {
        updateData: (newData) => {
          // ✅ 正确：使用 set 函数
          set({ myData: newData });
        },
        
        addItem: (item) => {
          // ✅ 正确：基于当前状态更新
          set((state) => ({
            myData: [...state.myData, item]
          }));
        },
      },
    }),
    { name: 'My Store' }
  )
);
```

---

## 4. 基础 UI 组件库 (_ui)

`_ui` 目录包含可复用的基础 UI 组件。

### 4.1 Button 按钮

```jsx
import Button from '@/_ui/Button';

<Button
  variant="primary"    // 'primary' | 'outline'
  loading={false}
  disabled={false}
  className="custom-class"
  onClick={() => {}}
>
  Click Me
</Button>
```

### 4.2 Input 输入框

```jsx
import Input from '@/_ui/Input';

<Input
  type="text"
  placeholder="Enter text..."
  value={value}
  onChange={(e) => setValue(e.target.value)}
  disabled={false}
/>
```

### 4.3 Select 选择器

```jsx
import Select from '@/_ui/Select';

<Select
  options={[
    { label: 'Option 1', value: '1' },
    { label: 'Option 2', value: '2' },
  ]}
  value={selectedValue}
  onChange={(value) => setSelectedValue(value)}
  placeholder="Select an option"
/>
```

### 4.4 Modal 模态框

```jsx
import Modal from '@/_ui/Modal';

<Modal
  show={isOpen}
  onHide={() => setIsOpen(false)}
  title="Modal Title"
  size="md"          // 'sm' | 'md' | 'lg' | 'xl'
>
  <Modal.Body>
    Modal content here
  </Modal.Body>
  <Modal.Footer>
    <Button onClick={() => setIsOpen(false)}>Close</Button>
  </Modal.Footer>
</Modal>
```

### 4.5 Drawer 抽屉

```jsx
import Drawer from '@/_ui/Drawer';

<Drawer
  isOpen={isDrawerOpen}
  onClose={() => setIsDrawerOpen(false)}
  position="right"    // 'left' | 'right'
  width="400px"
>
  <DrawerContent />
</Drawer>
```

### 4.6 Toast 提示

```jsx
import { Toast } from '@/_ui/Toast';
import { toast } from 'react-hot-toast';

// 成功提示
toast.success('Operation successful!');

// 错误提示
toast.error('Something went wrong!');

// 自定义提示
toast('Custom message', {
  duration: 4000,
  icon: '👏',
});
```

### 4.7 Alert 警告

```jsx
import Alert from '@/_ui/Alert';

<Alert
  type="info"        // 'info' | 'success' | 'warning' | 'error'
  message="This is an alert message"
  closable={true}
  onClose={() => {}}
/>
```

### 4.8 Checkbox 复选框

```jsx
import Checkbox from '@/_ui/CheckBox';

<Checkbox
  checked={isChecked}
  onChange={(e) => setIsChecked(e.target.checked)}
  label="Check me"
  disabled={false}
/>
```

### 4.9 Toggle 开关

```jsx
import Toggle from '@/_ui/Toggle';

<Toggle
  checked={isToggled}
  onChange={(checked) => setIsToggled(checked)}
  disabled={false}
/>
```

### 4.10 Popover 弹出框

```jsx
import Popover from '@/_ui/Popover';

<Popover
  content={<div>Popover content</div>}
  trigger="click"     // 'click' | 'hover'
  placement="bottom"  // 'top' | 'bottom' | 'left' | 'right'
>
  <Button>Click me</Button>
</Popover>
```

### 4.11 Icon 图标

```jsx
import * as Icons from '@tabler/icons-react';
// 或使用封装组件
import { Icon } from '@/_ui/Icon';

// 直接使用 Tabler Icons
<Icons.IconHome size={24} stroke={1.5} />

// 使用 Icon 组件目录下的自定义图标
import { HomeIcon } from '@/_ui/Icon';
```

### 4.12 JSONTreeViewer JSON 查看器

```jsx
import JSONTreeViewer from '@/_ui/JSONTreeViewer';

<JSONTreeViewer
  data={jsonData}
  expandLevel={2}
  theme="dark"
/>
```

### 4.13 Pagination 分页

```jsx
import Pagination from '@/_ui/Pagination';

<Pagination
  currentPage={currentPage}
  totalPages={totalPages}
  onPageChange={(page) => setCurrentPage(page)}
  showFirstLast={true}
/>
```

### 4.14 Search 搜索框

```jsx
import Search from '@/_ui/Search';

<Search
  placeholder="Search..."
  value={searchValue}
  onChange={(value) => setSearchValue(value)}
  onSearch={(value) => handleSearch(value)}
/>
```

### 4.15 Label 标签

```jsx
import Label from '@/_ui/Label';

<Label
  label="Field Label"
  width={100}
  color="#333"
  direction="left"
  isMandatory={true}
/>
```

---

## 5. App Builder 可视化组件 (Widgets)

位于 `AppBuilder/Widgets/` 目录下，这些是可在 App Builder 中拖拽使用的可视化组件。

### 5.1 组件通用 Props

所有 Widget 组件都接收以下通用 props：

```typescript
interface WidgetProps {
  id: string;                    // 组件唯一 ID
  height: number;                // 组件高度
  width: number;                 // 组件宽度
  properties: object;            // 组件属性配置
  styles: object;                // 组件样式配置
  fireEvent: (eventName: string) => void;  // 触发事件
  setExposedVariable: (name: string, value: any) => void;  // 设置暴露变量
  setExposedVariables: (variables: object) => void;        // 批量设置暴露变量
  dataCy: string;                // 测试属性
  darkMode: boolean;             // 暗色模式
}
```

### 5.2 Button 按钮组件

**文件位置**: `AppBuilder/Widgets/Button.jsx`

**属性 (properties)**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `text` | string | 按钮文本 |
| `loadingState` | boolean | 加载状态 |
| `visibility` | boolean | 是否可见 |
| `disabledState` | boolean | 是否禁用 |
| `tooltip` | string | 提示文本 |

**样式 (styles)**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `type` | 'primary' \| 'outline' | 按钮类型 |
| `backgroundColor` | string | 背景颜色 |
| `textColor` | string | 文字颜色 |
| `borderColor` | string | 边框颜色 |
| `borderRadius` | number | 边框圆角 |
| `icon` | string | 图标名称 |
| `iconColor` | string | 图标颜色 |
| `direction` | 'left' \| 'right' | 图标方向 |

**事件**:
- `onClick`: 点击时触发
- `onHover`: 悬停时触发

**暴露变量**:
- `buttonText`: 按钮文本
- `isVisible`: 是否可见
- `isDisabled`: 是否禁用
- `isLoading`: 是否加载中

**Actions**:
- `click()`: 触发点击
- `setText(text)`: 设置文本
- `setVisibility(value)`: 设置可见性
- `setDisable(value)`: 设置禁用状态
- `setLoading(value)`: 设置加载状态

### 5.3 TextInput 文本输入框

**文件位置**: `AppBuilder/Widgets/TextInput.jsx`

**属性**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `label` | string | 标签文本 |
| `placeholder` | string | 占位符 |
| `value` | string | 默认值 |
| `visibility` | boolean | 是否可见 |
| `disabledState` | boolean | 是否禁用 |

**样式**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `backgroundColor` | string | 背景颜色 |
| `textColor` | string | 文字颜色 |
| `borderColor` | string | 边框颜色 |
| `borderRadius` | number | 边框圆角 |
| `alignment` | 'side' \| 'top' | 标签对齐方式 |

**事件**:
- `onChange`: 值变化时触发
- `onBlur`: 失焦时触发
- `onFocus`: 聚焦时触发
- `onEnterPressed`: 按回车时触发

### 5.4 Table 表格组件

**文件位置**: `AppBuilder/Widgets/Table/` 和 `AppBuilder/Widgets/NewTable/`

**属性**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `title` | string | 表格标题 |
| `data` | array | 表格数据 |
| `columns` | array | 列配置 |
| `loadingState` | boolean | 加载状态 |
| `rowsPerPage` | number | 每页行数 |
| `enablePagination` | boolean | 启用分页 |
| `serverSidePagination` | boolean | 服务端分页 |
| `displaySearchBox` | boolean | 显示搜索框 |
| `showFilterButton` | boolean | 显示筛选按钮 |
| `allowSelection` | boolean | 允许选择行 |
| `showBulkSelector` | boolean | 批量选择 |

**列配置**:
```javascript
columns: [
  {
    name: 'id',
    key: 'id',
    columnType: 'string',    // 'string' | 'number' | 'image' | 'datepicker' | 'boolean' 等
    columnSize: 100,
    isEditable: false,
  },
  // ...
]
```

**事件**:
- `onRowClicked`: 行点击
- `onRowHovered`: 行悬停
- `onPageChanged`: 页码变化
- `onSearch`: 搜索
- `onSort`: 排序
- `onFilterChanged`: 筛选变化
- `onCellValueChanged`: 单元格值变化
- `onBulkUpdate`: 批量更新

**暴露变量**:
- `selectedRow`: 选中的行
- `selectedRows`: 选中的多行
- `changeSet`: 变更集
- `pageIndex`: 当前页码
- `searchText`: 搜索文本
- `filters`: 筛选条件

**Actions**:
- `setPage(page)`: 设置页码
- `selectRow(key, value)`: 选择行
- `deselectRow()`: 取消选择
- `selectAllRows()`: 全选
- `deselectAllRows()`: 取消全选
- `downloadTableData(type)`: 下载数据 (xlsx/csv/pdf)

### 5.5 Form 表单组件

**文件位置**: `AppBuilder/Widgets/Form/`

**属性**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `showHeader` | boolean | 显示头部 |
| `showFooter` | boolean | 显示底部 |
| `buttonToSubmit` | string | 提交按钮 ID |
| `generateFormFrom` | 'rawJson' \| 'jsonSchema' | 生成方式 |
| `JSONData` | object | JSON 数据 |
| `validateOnSubmit` | boolean | 提交时验证 |
| `resetOnSubmit` | boolean | 提交后重置 |
| `loadingState` | boolean | 加载状态 |
| `dynamicHeight` | boolean | 动态高度 |

**事件**:
- `onSubmit`: 提交时触发
- `onInvalid`: 验证失败时触发

**暴露变量**:
- `data`: 表单数据
- `isValid`: 是否有效
- `isVisible`: 是否可见
- `isDisabled`: 是否禁用
- `isLoading`: 是否加载中

**Actions**:
- `submitForm()`: 提交表单
- `resetForm()`: 重置表单
- `setVisibility(value)`: 设置可见性
- `setDisable(value)`: 设置禁用状态
- `setLoading(value)`: 设置加载状态

### 5.6 Modal 模态框组件

**文件位置**: `AppBuilder/Widgets/Modal.jsx` 和 `AppBuilder/Widgets/ModalV2/`

**属性**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `title` | string | 模态框标题 |
| `size` | 'sm' \| 'md' \| 'lg' \| 'xl' | 尺寸 |
| `hideOnEsc` | boolean | ESC 键关闭 |
| `hideCloseButton` | boolean | 隐藏关闭按钮 |
| `useDefaultButton` | boolean | 使用默认按钮 |
| `triggerButtonLabel` | string | 触发按钮文本 |

**事件**:
- `onOpen`: 打开时触发
- `onClose`: 关闭时触发

**Actions**:
- `open()`: 打开模态框
- `close()`: 关闭模态框

### 5.7 Container 容器组件

**文件位置**: `AppBuilder/Widgets/Container/`

用于包含其他组件的容器。

**属性**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `visibility` | boolean | 是否可见 |
| `disabledState` | boolean | 是否禁用 |

**样式**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `backgroundColor` | string | 背景颜色 |
| `borderColor` | string | 边框颜色 |
| `borderRadius` | number | 边框圆角 |

### 5.8 Listview 列表视图

**文件位置**: `AppBuilder/Widgets/Listview/`

**属性**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `data` | array | 列表数据 |
| `mode` | 'list' \| 'grid' | 显示模式 |
| `columns` | number | 列数 (grid 模式) |
| `rowHeight` | number | 行高 |
| `showBorder` | boolean | 显示边框 |

**事件**:
- `onRowClicked`: 行点击

### 5.9 Dropdown 下拉选择

**文件位置**: `AppBuilder/Widgets/DropdownV2/`

**属性**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `label` | string | 标签 |
| `placeholder` | string | 占位符 |
| `options` | array | 选项列表 |
| `value` | any | 选中值 |
| `visibility` | boolean | 是否可见 |
| `disabledState` | boolean | 是否禁用 |

**选项格式**:
```javascript
options: [
  { label: 'Option 1', value: '1' },
  { label: 'Option 2', value: '2' },
]
```

**事件**:
- `onSelect`: 选择时触发
- `onSearchTextChanged`: 搜索文本变化

### 5.10 DatePicker 日期选择器

**文件位置**: `AppBuilder/Widgets/Date/`

**属性**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `label` | string | 标签 |
| `format` | string | 日期格式 |
| `enableTime` | boolean | 启用时间选择 |
| `enableDate` | boolean | 启用日期选择 |
| `defaultValue` | string | 默认值 |

**事件**:
- `onSelect`: 选择时触发

### 5.11 Chart 图表组件

**文件位置**: `AppBuilder/Widgets/Chart.jsx`

**属性**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `title` | string | 图表标题 |
| `chartType` | string | 图表类型 (line/bar/pie 等) |
| `data` | array | 图表数据 |
| `jsonDescription` | object | JSON 配置 |
| `plotFromJson` | boolean | 从 JSON 绘制 |
| `showGridLines` | boolean | 显示网格线 |

### 5.12 Image 图片组件

**文件位置**: `AppBuilder/Widgets/Image/`

**属性**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `source` | string | 图片 URL |
| `loadingState` | boolean | 加载状态 |
| `alternativeText` | string | 替代文本 |
| `zoomButtons` | boolean | 显示缩放按钮 |
| `rotateButton` | boolean | 显示旋转按钮 |

**事件**:
- `onClick`: 点击时触发

### 5.13 Tabs 标签页组件

**文件位置**: `AppBuilder/Widgets/Tabs.jsx`

**属性**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `tabs` | array | 标签配置 |
| `defaultTab` | string | 默认标签 |
| `hideTabs` | boolean | 隐藏标签栏 |

**标签配置**:
```javascript
tabs: [
  { id: 'tab1', title: 'Tab 1' },
  { id: 'tab2', title: 'Tab 2' },
]
```

**事件**:
- `onTabSwitch`: 切换标签时触发

### 5.14 FilePicker 文件选择器

**文件位置**: `AppBuilder/Widgets/FilePicker/`

**属性**:
| 属性 | 类型 | 描述 |
|------|------|------|
| `label` | string | 标签 |
| `acceptedFileTypes` | string | 接受的文件类型 |
| `maxFileCount` | number | 最大文件数 |
| `maxFileSize` | number | 最大文件大小 (MB) |
| `minFileSize` | number | 最小文件大小 (MB) |
| `enableMultiple` | boolean | 允许多选 |
| `enableDropzone` | boolean | 启用拖放区 |

**事件**:
- `onFileSelected`: 选择文件时触发
- `onFileDeselected`: 取消选择时触发

**暴露变量**:
- `file`: 选择的文件
- `files`: 选择的文件列表

---

## 6. 服务层 (Services)

位于 `_services/` 目录，封装所有 API 调用。

### 6.1 appService - 应用服务

```javascript
import { appService } from '@/_services';

// 获取所有应用
const apps = await appService.getAll(page, folder, searchKey);

// 创建应用
const newApp = await appService.createApp({ name: 'My App' });

// 获取应用详情
const app = await appService.getApp(appId);

// 获取指定版本
const appVersion = await appService.fetchAppByVersion(appId, versionId);

// 保存应用
await appService.saveApp(appId, attributes);

// 克隆应用
const clonedApp = await appService.cloneApp(appId);

// 导出应用
const exportData = await appService.exportApp(appId, versionId);

// 导入应用
const importedApp = await appService.importApp(body);

// 删除应用
await appService.deleteApp(appId);
```

### 6.2 datasourceService - 数据源服务

```javascript
import { datasourceService } from '@/_services';

// 获取所有数据源
const dataSources = await datasourceService.getAll(
  appVersionId, 
  environmentId, 
  includeStaticSources
);

// 创建数据源
const newDs = await datasourceService.create({
  plugin_id,
  name,
  kind,
  options,
  app_id,
  app_version_id,
  environment_id
});

// 保存数据源
await datasourceService.save({
  id,
  name,
  options,
  app_id,
  environment_id
});

// 测试连接
const testResult = await datasourceService.test(body);

// 删除数据源
await datasourceService.deleteDataSource(id);

// 设置 OAuth2 Token
await datasourceService.setOauth2Token(dataSourceId, body, currentOrgId);
```

### 6.3 dataqueryService - 数据查询服务

```javascript
import { dataqueryService } from '@/_services';

// 获取所有查询
const queries = await dataqueryService.getAll(appVersionId);

// 创建查询
const newQuery = await dataqueryService.create(
  appId,
  appVersionId,
  name,
  kind,
  options,
  dataSourceId,
  pluginId
);

// 更新查询
await dataqueryService.update(id, name, options);

// 运行查询
const result = await dataqueryService.run(queryId, options);

// 预览查询
const preview = await dataqueryService.preview(query, options, versionId);

// 删除查询
await dataqueryService.del(queryId);

// 批量更新查询选项
await dataqueryService.bulkUpdateQueryOptions(queryOptions, appVersionId);
```

### 6.4 authenticationService - 认证服务

```javascript
import { authenticationService } from '@/_services';

// 登录
const result = await authenticationService.login(email, password);

// 登出
await authenticationService.logout();

// 获取当前会话
const currentSession = authenticationService.currentSessionValue;

// 刷新 Token
await authenticationService.refreshToken();

// 验证 Token
const isValid = await authenticationService.validateToken();
```

### 6.5 userService - 用户服务

```javascript
import { userService } from '@/_services';

// 获取用户
const user = await userService.get(userId);

// 更新用户
await userService.update(userId, userData);

// 更新密码
await userService.updatePassword(currentPassword, newPassword);

// 获取当前用户
const currentUser = await userService.getCurrentUser();
```

### 6.6 organizationService - 组织服务

```javascript
import { organizationService } from '@/_services';

// 获取所有组织
const orgs = await organizationService.getAll();

// 获取组织
const org = await organizationService.get(orgId);

// 创建组织
const newOrg = await organizationService.create(name);

// 切换组织
await organizationService.switchOrganization(orgId);

// 获取组织用户
const users = await organizationService.getUsers();

// 邀请用户
await organizationService.inviteUser(email, role);
```

---

## 7. 自定义 Hooks

位于 `_hooks/` 目录。

### 7.1 useDebounce - 防抖 Hook

```javascript
import useDebounce from '@/_hooks/useDebounce';

const MyComponent = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    // 在防抖后执行搜索
    if (debouncedSearchTerm) {
      performSearch(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <input 
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
    />
  );
};
```

### 7.2 useHover - 悬停检测 Hook

```javascript
import useHover from '@/_hooks/useHover';

const MyComponent = () => {
  const [hoverRef, isHovered] = useHover();
  
  return (
    <div ref={hoverRef}>
      {isHovered ? 'Hovered!' : 'Hover me'}
    </div>
  );
};
```

### 7.3 useLocalStorage - 本地存储 Hook

```javascript
import useLocalStorage from '@/_hooks/use-local-storage';

const MyComponent = () => {
  const [value, setValue] = useLocalStorage('myKey', 'defaultValue');
  
  return (
    <input 
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
};
```

### 7.4 useEventListener - 事件监听 Hook

```javascript
import useEventListener from '@/_hooks/use-event-listener';

const MyComponent = () => {
  useEventListener('keydown', (event) => {
    if (event.key === 'Escape') {
      // 处理 ESC 键
    }
  });
  
  return <div>Press ESC</div>;
};
```

### 7.5 useWindowResize - 窗口大小监听 Hook

```javascript
import useWindowResize from '@/_hooks/useWindowResize';

const MyComponent = () => {
  const { width, height } = useWindowResize();
  
  return (
    <div>
      Window size: {width} x {height}
    </div>
  );
};
```

### 7.6 usePortal - Portal Hook

```javascript
import usePortal from '@/_hooks/use-portal';

const MyComponent = () => {
  const { openPortal, closePortal, isOpen, Portal } = usePortal();
  
  return (
    <>
      <button onClick={openPortal}>Open Modal</button>
      {isOpen && (
        <Portal>
          <div className="modal">
            <button onClick={closePortal}>Close</button>
          </div>
        </Portal>
      )}
    </>
  );
};
```

### 7.7 useAppDarkMode - 暗色模式 Hook

```javascript
import useAppDarkMode from '@/_hooks/useAppDarkMode';

const MyComponent = () => {
  const darkMode = useAppDarkMode();
  
  return (
    <div className={darkMode ? 'dark' : 'light'}>
      Content
    </div>
  );
};
```

### 7.8 useKeyHooks - 键盘快捷键 Hook

```javascript
import useKeyHooks from '@/_hooks/useKeyHooks';

const MyComponent = () => {
  useKeyHooks('ctrl+s', (e) => {
    e.preventDefault();
    handleSave();
  });
  
  return <div>Press Ctrl+S to save</div>;
};
```

---

## 8. 组件配置系统

### 8.1 组件配置结构

每个 Widget 组件都有对应的配置文件，位于 `AppBuilder/WidgetManager/widgets/` 目录。

```javascript
// widgets/button.js
export const buttonConfig = {
  // 基本信息
  name: 'Button',
  displayName: 'Button',
  description: 'Trigger actions: queries, alerts, set variables etc.',
  component: 'Button',
  
  // 默认尺寸
  defaultSize: {
    width: 4,
    height: 40,
  },
  
  // 显示控制
  others: {
    showOnDesktop: { type: 'toggle', displayName: 'Show on desktop' },
    showOnMobile: { type: 'toggle', displayName: 'Show on mobile' },
  },
  
  // 属性定义
  properties: {
    text: {
      type: 'code',
      displayName: 'Label',
      validation: { schema: { type: 'string' } },
    },
    loadingState: {
      type: 'toggle',
      displayName: 'Loading state',
      validation: { schema: { type: 'boolean' } },
      section: 'additionalActions',
    },
    // ...
  },
  
  // 事件定义
  events: {
    onClick: { displayName: 'On click' },
    onHover: { displayName: 'On hover' },
  },
  
  // 样式定义
  styles: {
    backgroundColor: {
      type: 'colorSwatches',
      displayName: 'Background',
      validation: { schema: { type: 'string' } },
      accordian: 'button',
    },
    borderRadius: {
      type: 'numberInput',
      displayName: 'Border radius',
      accordian: 'button',
    },
    // ...
  },
  
  // 暴露变量
  exposedVariables: {
    buttonText: 'Button',
    isVisible: true,
    isDisabled: false,
    isLoading: false,
  },
  
  // 组件方法
  actions: [
    {
      handle: 'click',
      displayName: 'Click',
    },
    {
      handle: 'setText',
      displayName: 'Set text',
      params: [
        { handle: 'text', displayName: 'Text', defaultValue: 'New Text' }
      ],
    },
    // ...
  ],
  
  // 默认值定义
  definition: {
    others: {
      showOnDesktop: { value: '{{true}}' },
      showOnMobile: { value: '{{false}}' },
    },
    properties: {
      text: { value: 'Button' },
      visibility: { value: '{{true}}' },
      // ...
    },
    events: [],
    styles: {
      backgroundColor: { value: 'var(--cc-primary-brand)' },
      // ...
    },
  },
};
```

### 8.2 属性类型

| 类型 | 描述 | 示例 |
|------|------|------|
| `code` | 代码编辑器 | 支持 {{}} 表达式 |
| `toggle` | 开关 | true/false |
| `select` | 下拉选择 | 预定义选项 |
| `switch` | 切换按钮 | 两个选项切换 |
| `colorSwatches` | 颜色选择器 | 颜色值 |
| `numberInput` | 数字输入 | 数值 |
| `icon` | 图标选择器 | 图标名称 |
| `boxShadow` | 阴影配置 | CSS 阴影 |
| `array` | 数组配置 | 列配置等 |

### 8.3 添加新组件

1. **创建组件文件** (`AppBuilder/Widgets/MyComponent.jsx`):

```jsx
import React, { useEffect, useState } from 'react';

export const MyComponent = (props) => {
  const { 
    height, 
    properties, 
    styles, 
    fireEvent, 
    setExposedVariable, 
    setExposedVariables,
    dataCy 
  } = props;
  
  const { text, visibility } = properties;
  const { backgroundColor, textColor } = styles;
  
  const [value, setValue] = useState(text);
  
  useEffect(() => {
    setExposedVariables({
      value: value,
      setValue: async (newValue) => {
        setValue(newValue);
        setExposedVariable('value', newValue);
      }
    });
  }, []);
  
  const handleClick = () => {
    fireEvent('onClick');
  };
  
  if (!visibility) return null;
  
  return (
    <div 
      data-cy={dataCy}
      style={{ 
        height, 
        backgroundColor, 
        color: textColor 
      }}
      onClick={handleClick}
    >
      {value}
    </div>
  );
};
```

2. **创建配置文件** (`AppBuilder/WidgetManager/widgets/myComponent.js`):

```javascript
export const myComponentConfig = {
  name: 'MyComponent',
  displayName: 'My Component',
  description: 'A custom component',
  component: 'MyComponent',
  defaultSize: { width: 6, height: 50 },
  
  properties: {
    text: {
      type: 'code',
      displayName: 'Text',
      validation: { schema: { type: 'string' } },
    },
    visibility: {
      type: 'toggle',
      displayName: 'Visibility',
      section: 'additionalActions',
    },
  },
  
  events: {
    onClick: { displayName: 'On click' },
  },
  
  styles: {
    backgroundColor: {
      type: 'colorSwatches',
      displayName: 'Background',
      accordian: 'container',
    },
    textColor: {
      type: 'colorSwatches',
      displayName: 'Text color',
      accordian: 'container',
    },
  },
  
  exposedVariables: {
    value: '',
  },
  
  actions: [
    {
      handle: 'setValue',
      displayName: 'Set Value',
      params: [{ handle: 'value', displayName: 'Value', defaultValue: '' }],
    },
  ],
  
  definition: {
    properties: {
      text: { value: 'Hello World' },
      visibility: { value: '{{true}}' },
    },
    styles: {
      backgroundColor: { value: '#ffffff' },
      textColor: { value: '#000000' },
    },
    events: [],
  },
};
```

3. **注册组件** (`AppBuilder/WidgetManager/widgets/index.js`):

```javascript
export { myComponentConfig } from './myComponent';
```

4. **添加到组件类型** (`AppBuilder/WidgetManager/componentTypes.js`):

```javascript
import { MyComponent } from '../Widgets/MyComponent';
import { myComponentConfig } from './widgets/myComponent';

export const componentTypes = {
  // ... 其他组件
  MyComponent: {
    component: MyComponent,
    config: myComponentConfig,
  },
};
```

---

## 9. 开发最佳实践

### 9.1 状态管理最佳实践

```javascript
// ✅ 正确：使用选择器获取特定状态
const selectedComponents = useEditorStore((state) => state.selectedComponents);

// ❌ 错误：获取整个 store
const store = useEditorStore();

// ✅ 正确：使用浅比较优化
import { shallow } from 'zustand/shallow';
const { queries, components } = useCurrentStateStore(
  (state) => ({ queries: state.queries, components: state.components }),
  shallow
);

// ✅ 正确：通过 actions 修改状态
const { setSelectedComponents } = useEditorActions();
setSelectedComponents([componentId]);

// ❌ 错误：直接修改状态
store.selectedComponents = [componentId];
```

### 9.2 组件开发最佳实践

```jsx
// ✅ 正确：使用 memo 优化性能
import React, { memo, useCallback } from 'react';

export const MyWidget = memo((props) => {
  const { fireEvent, setExposedVariable } = props;
  
  // ✅ 正确：使用 useCallback 缓存回调
  const handleClick = useCallback(() => {
    fireEvent('onClick');
  }, [fireEvent]);
  
  // ✅ 正确：在 useEffect 中设置暴露变量
  useEffect(() => {
    setExposedVariable('value', initialValue);
  }, [initialValue]);
  
  return <div onClick={handleClick}>Content</div>;
});
```

### 9.3 样式开发最佳实践

```scss
// ✅ 正确：使用 CSS 变量
.my-component {
  background-color: var(--cc-surface1-surface);
  color: var(--cc-primary-text);
  border: 1px solid var(--cc-default-border);
}

// ✅ 正确：支持暗色模式
.my-component {
  background-color: var(--cc-surface1-surface);
  
  .theme-dark & {
    background-color: var(--cc-surface1-surface-dark);
  }
}

// ✅ 正确：使用 BEM 命名
.widget-button {
  &__icon { }
  &__label { }
  &--disabled { }
  &--loading { }
}
```

### 9.4 CSS 变量参考

```scss
// 颜色变量
--cc-primary-brand        // 主品牌色
--cc-primary-text         // 主要文字
--cc-placeholder-text     // 占位符文字
--cc-surface1-surface     // 表面颜色 1
--cc-surface2-surface     // 表面颜色 2
--cc-default-border       // 默认边框
--cc-weak-border          // 弱边框

// 状态颜色
--status-error-strong     // 错误状态
--status-success-strong   // 成功状态
--status-warning-strong   // 警告状态

// 图标颜色
--icons-strong            // 强调图标
--icons-weak-disabled     // 禁用图标
--icons-on-solid          // 实心背景图标
```

### 9.5 事件处理最佳实践

```javascript
// ✅ 正确：使用 fireEvent 触发自定义事件
const handleClick = () => {
  fireEvent('onClick');
};

// ✅ 正确：传递事件数据
const handleRowClick = (row) => {
  setExposedVariable('selectedRow', row);
  fireEvent('onRowClicked');
};

// ✅ 正确：异步事件处理
const handleSubmit = async () => {
  setExposedVariable('isLoading', true);
  try {
    await submitData();
    fireEvent('onSubmit');
  } catch (error) {
    setExposedVariable('error', error.message);
    fireEvent('onError');
  } finally {
    setExposedVariable('isLoading', false);
  }
};
```

---

## 10. 二次开发指南

### 10.1 遵循的约束

根据 ToolJet 架构，二次开发时必须遵循以下约束：

#### 10.1.1 多租户数据隔离

```javascript
// ✅ 正确：API 请求携带工作区 ID
import { authHeader } from '@/_helpers';

function myApiCall() {
  const requestOptions = { 
    method: 'GET', 
    headers: authHeader(),  // 自动添加 tj-workspace-id
    credentials: 'include' 
  };
  return fetch(url, requestOptions);
}
```

#### 10.1.2 状态管理约束

```javascript
// ✅ 正确：使用 Immer 确保不可变更新
import { create, zustandDevTools } from '@/_stores/utils';

export const useMyStore = create(
  zustandDevTools(
    (set, get) => ({
      items: [],
      actions: {
        addItem: (item) => {
          set((state) => ({
            items: [...state.items, item]  // 不可变更新
          }));
        },
      },
    }),
    { name: 'My Store' }
  )
);

// ❌ 错误：直接修改状态
actions: {
  addItem: (item) => {
    get().items.push(item);  // 直接修改会破坏撤销/重做功能
  },
}
```

#### 10.1.3 认证约束

```javascript
// ✅ 正确：使用 RxJS BehaviorSubject 管理会话
import { authenticationService } from '@/_services';

// 订阅会话变化
authenticationService.currentSession.subscribe(session => {
  // 响应会话变化
});

// 获取当前会话值
const session = authenticationService.currentSessionValue;
```

### 10.2 添加新页面

```jsx
// 1. 创建页面组件
// src/MyPage/MyPage.jsx
import React from 'react';

const MyPage = () => {
  return (
    <div className="my-page">
      <h1>My Page</h1>
    </div>
  );
};

export default MyPage;

// 2. 添加路由
// src/Routes/Routes.jsx
import MyPage from '../MyPage/MyPage';

const routes = [
  // ...existing routes
  {
    path: '/my-page',
    component: MyPage,
    isPrivate: true,  // 需要登录
  },
];
```

### 10.3 添加新服务

```javascript
// src/_services/myService.js
import config from 'config';
import { authHeader, handleResponse } from '@/_helpers';

export const myService = {
  getAll,
  getById,
  create,
  update,
  delete: _delete,
};

function getAll() {
  const requestOptions = { 
    method: 'GET', 
    headers: authHeader(), 
    credentials: 'include' 
  };
  return fetch(`${config.apiUrl}/my-resource`, requestOptions)
    .then(handleResponse);
}

function getById(id) {
  const requestOptions = { 
    method: 'GET', 
    headers: authHeader(), 
    credentials: 'include' 
  };
  return fetch(`${config.apiUrl}/my-resource/${id}`, requestOptions)
    .then(handleResponse);
}

function create(data) {
  const requestOptions = {
    method: 'POST',
    headers: authHeader(),
    credentials: 'include',
    body: JSON.stringify(data),
  };
  return fetch(`${config.apiUrl}/my-resource`, requestOptions)
    .then(handleResponse);
}

function update(id, data) {
  const requestOptions = {
    method: 'PUT',
    headers: authHeader(),
    credentials: 'include',
    body: JSON.stringify(data),
  };
  return fetch(`${config.apiUrl}/my-resource/${id}`, requestOptions)
    .then(handleResponse);
}

function _delete(id) {
  const requestOptions = { 
    method: 'DELETE', 
    headers: authHeader(), 
    credentials: 'include' 
  };
  return fetch(`${config.apiUrl}/my-resource/${id}`, requestOptions)
    .then(handleResponse);
}

// 导出
// src/_services/index.js
export * from './myService';
```

### 10.4 添加新 Store

```javascript
// src/_stores/myStore.js
import { create, zustandDevTools } from './utils';
import { myService } from '@/_services';

const initialState = {
  items: [],
  loading: false,
  error: null,
};

export const useMyStore = create(
  zustandDevTools(
    (set, get) => ({
      ...initialState,
      
      actions: {
        fetchItems: async () => {
          set({ loading: true, error: null });
          try {
            const items = await myService.getAll();
            set({ items, loading: false });
          } catch (error) {
            set({ error: error.message, loading: false });
          }
        },
        
        addItem: async (data) => {
          set({ loading: true });
          try {
            const newItem = await myService.create(data);
            set((state) => ({
              items: [...state.items, newItem],
              loading: false,
            }));
          } catch (error) {
            set({ error: error.message, loading: false });
          }
        },
        
        updateItem: async (id, data) => {
          set({ loading: true });
          try {
            const updatedItem = await myService.update(id, data);
            set((state) => ({
              items: state.items.map(item => 
                item.id === id ? updatedItem : item
              ),
              loading: false,
            }));
          } catch (error) {
            set({ error: error.message, loading: false });
          }
        },
        
        deleteItem: async (id) => {
          set({ loading: true });
          try {
            await myService.delete(id);
            set((state) => ({
              items: state.items.filter(item => item.id !== id),
              loading: false,
            }));
          } catch (error) {
            set({ error: error.message, loading: false });
          }
        },
        
        reset: () => set(initialState),
      },
    }),
    { name: 'My Store' }
  )
);

// 便捷 hooks
export const useMyItems = () => useMyStore((state) => state.items);
export const useMyLoading = () => useMyStore((state) => state.loading);
export const useMyActions = () => useMyStore((state) => state.actions);
```

### 10.5 目录结构建议

新功能建议按以下结构组织：

```
src/modules/myFeature/
├── components/
│   ├── MyComponent/
│   │   ├── MyComponent.jsx
│   │   ├── myComponent.scss
│   │   └── index.js
│   └── index.js
├── pages/
│   ├── MyPage/
│   │   ├── MyPage.jsx
│   │   └── index.js
│   └── index.js
├── stores/
│   └── myFeature.store.js
├── services/
│   └── myFeature.service.js
├── hooks/
│   └── useMyFeature.js
└── index.js
```

---

## 附录

### A. 常用导入路径别名

| 别名 | 实际路径 |
|------|----------|
| `@/_components` | `src/_components` |
| `@/_helpers` | `src/_helpers` |
| `@/_hooks` | `src/_hooks` |
| `@/_services` | `src/_services` |
| `@/_stores` | `src/_stores` |
| `@/_ui` | `src/_ui` |
| `@/AppBuilder` | `src/AppBuilder` |
| `@/Editor` | `src/Editor` |
| `@/ToolJetUI` | `src/ToolJetUI` |

### B. 常用命令

```bash
# 开发模式
npm run start

# 构建
npm run build

# 测试
npm run test

# Lint 检查
npm run lint

# Storybook
npm run storybook
```

### C. 环境变量

```bash
# API 地址
TOOLJET_SERVER_URL=http://localhost:3000

# 服务器端口
TOOLJET_SERVER_PORT=3000

# 环境
NODE_ENV=development|production
```

---

**文档版本**: 1.0  
**最后更新**: 2026-01-28  
**基于 ToolJet 版本**: CE (社区版)
