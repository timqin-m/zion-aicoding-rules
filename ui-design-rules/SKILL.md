---
description: "React UI design rules based on Organic/Natural style with wabi-sabi aesthetics. Use when: (1) Building React components with Tailwind CSS, (2) Creating organic blob shapes and natural textures, (3) Using earth-tone color palette, (4) Implementing colored shadows (moss green/clay orange), (5) Designing cards, buttons, inputs with organic style, (6) Following responsive design patterns"
alwaysApply: true
---

# UI设计规则 - 有机/自然风格

## Overview

本文档为AI代码生成工具提供UI设计规则，所有规则必须严格遵循。**本规则专为React项目设计**，设计风格采用**有机/自然风格（Organic/Natural）**，强调wabi-sabi美学——接受短暂性和不完美，追求**温暖、柔软和自然连接**。

> **设计哲学**: 
> - **wabi-sabi美学**：接受不完美，追求真实和自然
> - **有机形状**：软质blob形状，拒绝90度直角
> - **自然纹理**：全局颗粒纹理叠加（3-4%不透明度，multiply混合模式）
> - **大地色调**：来自森林、粘土、未漂白纸张的调色板
> - **彩色阴影**：柔和阴影带自然色彩色调（苔藓绿、粘土橙），禁止纯黑色

## React技术栈 [MUST]

- **框架**: React（函数组件 + Hooks）
- **样式方案**: Tailwind CSS（使用3.4.0）或 styled-components
- **组件库**: shadcn/ui（复杂组件，需覆盖样式以符合有机风格）
- **规则**: 优先使用Tailwind CSS工具类，有机形状使用内联样式

## 规则执行优先级

- **MUST**: 必须遵循，无例外
- **SHOULD**: 应该遵循，除非有特殊原因
- **MAY**: 可以选择遵循

---

## 1. 颜色系统 [MUST]

### 调色板

```css
--background: #FDFCF8;        /* 米白色，米纸 */
--foreground: #2C2C24;        /* 深壤土/木炭 */
--primary: #5D7052;           /* 苔藓绿 */
--primary-foreground: #F3F4F1; /* 淡雾色 */
--secondary: #C18C5D;         /* 赤陶土/粘土 */
--secondary-foreground: #FFFFFF;
--accent: #E6DCCD;            /* 沙色/米色 */
--accent-foreground: #4A4A40;  /* 树皮色 */
--muted: #F0EBE5;             /* 石头色 */
--muted-foreground: #78786C;   /* 干草色 */
--border: #DED8CF;            /* 原木色 */
--destructive: #A85448;       /* 烧焦的赭石色 */
```

**规则**:
- 页面背景必须使用米白色（#FDFCF8），卡片背景使用极浅米色（#FEFEFA）
- 必须添加全局纹理叠加（3-4%不透明度，multiply混合模式）
- 所有颜色来自自然材料，保持温暖和真实感

### 对比度要求

- 主要文字（#2C2C24）在背景上：14.5:1（AAA）
- 苔藓绿（#5D7052）在背景上：6.2:1（AA）
- 静音文字（#78786C）在背景上：4.8:1（AA）

---

## 2. 排版系统 [MUST]

### 字体

- **标题**: 'Fraunces' (Google Font)，字重600-800
- **正文**: 'Nunito' 或 'Quicksand' (Google Font)

### 字体大小

```css
h1: text-5xl md:text-7xl (移动端48px，桌面端72px)
h2: text-4xl md:text-5xl (移动端36px，桌面端48px)
h3: text-3xl (30px)
h4: text-2xl (24px)
body: text-base (16px)
small: text-sm (14px)
```

**规则**: 响应式字体大小，移动端较小，桌面端较大

---

## 3. 圆角与形状 [MUST]

### 标准圆角

- 按钮: `rounded-full` (完全圆形)
- 标准元素: `rounded-2xl` (16px) 或 `rounded-3xl` (24px)
- 卡片: `rounded-[2rem]` (32px) 作为基础

### 有机形状 [MUST]

**核心特征**: 使用复杂的百分比border-radius创建blob形状。

```css
/* 有机blob形状示例 */
border-radius: 60% 40% 30% 70% / 60% 30% 70% 40%;
border-radius: 30% 60% 70% 40% / 50% 60% 30% 60%;
border-radius: 40% 60% 50% 50% / 40% 50% 60% 50%;

/* 不对称卡片圆角 */
rounded-tl-[4rem] rounded-tr-[2rem] rounded-bl-[2rem] rounded-br-[5rem]
```

**规则**:
- 重要装饰元素（blob背景、图片遮罩）必须使用有机形状
- 卡片可以循环使用不同的border-radius模式
- **禁止**90度直角

---

## 4. 阴影系统 [MUST]

**核心原则**: 使用柔和的、扩散的阴影，带有自然色彩色调（苔藓绿、粘土橙），**禁止纯黑色**。

```css
/* 柔和阴影 - 苔藓色调 */
shadow-[0_4px_20px_-2px_rgba(93,112,82,0.15)]

/* 浮动阴影 - 粘土色调 */
shadow-[0_10px_40px_-10px_rgba(193,140,93,0.2)]

/* 悬停阴影加深 */
hover:shadow-[0_6px_24px_-4px_rgba(93,112,82,0.25)]
hover:shadow-[0_20px_40px_-10px_rgba(93,112,82,0.15)]
```

**规则**: 必须使用彩色阴影，禁止纯黑色阴影

---

## 5. 组件设计规则

### 5.1 卡片 [MUST]

```tsx
<div className="bg-[#FEFEFA] border border-[#DED8CF]/50 rounded-[2rem] shadow-[0_4px_20px_-2px_rgba(93,112,82,0.15)] p-8 transition-all duration-300 hover:-translate-y-1 hover:shadow-[0_20px_40px_-10px_rgba(93,112,82,0.15)]">
  {children}
</div>
```

**规则**:
- 背景: 极浅米色（#FEFEFA）
- 边框: 原木色，50%不透明度
- 圆角: rounded-[2rem] (32px)
- 阴影: 苔藓色调柔和阴影
- 悬停: 轻微提升（translateY -4px）并加深阴影
- 卡片可以循环使用不同的border-radius模式创造变化

### 5.2 按钮 [MUST]

```tsx
/* 主要按钮 */
<button className="bg-[#5D7052] text-[#F3F4F1] rounded-full px-8 py-3 font-bold text-base shadow-[0_4px_20px_-2px_rgba(93,112,82,0.15)] transition-all duration-300 hover:scale-105 hover:shadow-[0_6px_24px_-4px_rgba(93,112,82,0.25)] active:scale-95">
  按钮文字
</button>

/* 轮廓按钮 */
<button className="bg-transparent text-[#C18C5D] border-2 border-[#C18C5D] rounded-full px-8 py-3 font-bold transition-all duration-300 hover:bg-[#C18C5D]/10">
  按钮文字
</button>
```

**规则**:
- 背景: 苔藓绿（#5D7052）或赤陶土（#C18C5D）
- 圆角: rounded-full（完全圆形）
- 高度: 默认h-12 (48px)，sm h-10，lg h-14
- 水平内边距: px-8到px-10
- 悬停: scale(1.05)并加深阴影
- 激活: scale(0.95)提供触觉反馈

### 5.3 输入框 [MUST]

```tsx
<input className="bg-white/50 border border-[#DED8CF] rounded-full px-6 py-3 text-sm text-[#2C2C24] font-nunito h-12 transition-all duration-300 focus-visible:ring-2 focus-visible:ring-[#5D7052]/30 focus-visible:ring-offset-2 focus-visible:outline-none" />
```

**规则**:
- 背景: 半透明白色（rgba(255, 255, 255, 0.5)），显示下方纹理
- 边框: 原木色（#DED8CF）
- 圆角: rounded-full（完全圆形）
- 高度: h-12 (48px)
- 焦点: 苔藓绿色调，柔和光晕（ring-2，30%不透明度）

### 5.4 导航 [SHOULD]

```tsx
<nav className="sticky top-4 z-100 bg-white/70 backdrop-blur-md border border-[#DED8CF]/50 rounded-full px-8 py-3 shadow-[0_4px_20px_-2px_rgba(93,112,82,0.15)]">
  {/* 导航内容 */}
</nav>
```

**规则**: sticky定位，top: 1rem，70%不透明度白色背景，backdrop-blur，rounded-full

---

## 6. 布局与间距 [MUST]

### 容器宽度

- Hero/Features/Blog/Pricing: `max-w-7xl` (1280px)
- How It Works/FAQ: `max-w-6xl` (1152px)
- Final CTA: `max-w-5xl` (1024px)
- 文本密集: `max-w-4xl` (896px) 或 `max-w-2xl` (512px)

### 部分内边距

- 垂直: `py-32` (128px)
- 水平: `px-4` (移动端) → `sm:px-6` → `lg:px-8`

### 网格模式

- 统计: `grid-cols-2 md:grid-cols-4`
- Features/Blog/Testimonials: `md:grid-cols-2 lg:grid-cols-3`
- 两列布局: `lg:grid-cols-2`
- 网格间距: `gap-8` (32px)，可选 `md:gap-12` (48px)

**规则**: 不同部分使用不同的最大宽度，创造视觉节奏

---

## 7. 动画规则 [MUST]

### 过渡

```css
transition: all 0.3s ease;  /* duration-300 */
transition: all 0.5s ease;  /* duration-500 */
```

**规则**: 使用transition-all duration-300或duration-500，所有过渡使用ease缓动，持续时间300-700ms

### 悬停动画

- 按钮: `hover:scale-105` 并加深阴影
- 卡片: `hover:-translate-y-1` (提升) 或 `hover:rotate-1` (轻微倾斜)
- 统计数字: `group-hover:scale-110`
- 图片: `hover:scale-105`，700ms持续时间
- 图标容器: 背景色填充过渡

### 激活状态

- 按钮: `active:scale-95` 提供触觉反馈

**规则**: 禁止突然变化，所有过渡使用ease缓动

---

## 8. 非通用元素 [SHOULD]

### Blob背景

```tsx
<div className="absolute inset-0 blur-3xl opacity-30 -z-10" style={{ borderRadius: '60% 40% 30% 70% / 60% 30% 70% 40%', background: '#5D7052' }} />
```

**使用场景**: Hero（2个blob）、产品详情、Features、Final CTA

### 旋转图片框架

```css
transform: rotate(-2deg);
border: 4px solid white;
```

**使用场景**: 产品详情图片

### 有机图片遮罩

```css
border-radius: 30% 70% 70% 30% / 30% 30% 70% 70%;
```

**使用场景**: 收益部分图片

### 曲线SVG连接器

**使用场景**: How It Works部分，使用手绘风格的曲线虚线SVG路径

### 悬停微旋转

```css
.testimonial-card:hover { transform: rotate(1deg); }
```

**使用场景**: 推荐卡片

### 变化的部分背景

在米白色、石头色调（#F0EBE5/30）、沙色（#E6DCCD/30）、苔藓绿（#5D7052）、赤陶土（#C18C5D）之间交替

---

## 9. 图标系统 [MUST]

**使用**: Lucide React图标库

**规则**:
- 默认stroke-width: 2px
- 颜色: 苔藓绿（#5D7052）作为默认，深色背景上使用白色
- 容器: `h-14 w-14` (56px)，`rounded-2xl`，`bg-[#5D7052]/10`
- 悬停: 容器完全填充为实心苔藓绿，图标切换为白色
- 尺寸: 功能图标28px，收益检查标记24px

---

## 10. 响应式策略 [MUST]

### 移动优先

- 所有样式从移动端开始
- 使用Tailwind的移动优先断点系统

### 断点

- `sm:` (640px): 水平内边距增加，一些flex-row布局
- `md:` (768px): 主要网格转换（2-3列），导航显示桌面版本
- `lg:` (1024px): 3列网格，2列hero/收益布局

### 排版缩放

- Hero标题: 移动端text-5xl，桌面端text-7xl
- 部分标题: 移动端text-4xl，桌面端text-5xl

### 堆叠行为

- 所有网格在移动端折叠为单列
- Flex布局切换为flex-col
- 导航: 移动端使用汉堡菜单，桌面端内联导航

---

## 11. 可访问性 [MUST]

### 对比度

- 主要文字（#2C2C24）在背景上：14.5:1（AAA）
- 苔藓绿（#5D7052）在背景上：6.2:1（AA）
- 静音文字（#78786C）在背景上：4.8:1（AA）

### 焦点状态

```css
focus-visible:ring-2 focus-visible:ring-[#5D7052]/30 focus-visible:ring-offset-2
```

### 触摸目标

- 所有交互元素必须满足44px最小尺寸（按钮h-12 = 48px）

### 语义HTML

- 使用适当的标题层次、导航地标、图片alt文本、需要时使用aria-labels

### 键盘导航

- 所有交互元素键盘可访问

---

## 12. 代码生成检查清单 [MUST]

### 颜色系统
- [ ] 使用定义的调色板（苔藓绿、赤陶土、米白色等）
- [ ] 背景使用米白色（#FDFCF8），卡片使用极浅米色（#FEFEFA）
- [ ] 必须添加全局纹理叠加（3-4%不透明度，multiply混合模式）
- [ ] 文字颜色符合层次规则，对比度符合WCAG标准

### 排版系统
- [ ] 标题使用Fraunces字体（字重600-800）
- [ ] 正文使用Nunito或Quicksand字体
- [ ] 响应式字体大小（移动端较小，桌面端较大）

### 圆角与形状
- [ ] 按钮使用rounded-full（完全圆形）
- [ ] 标准元素使用rounded-2xl或rounded-3xl
- [ ] 卡片使用rounded-[2rem]作为基础
- [ ] 重要装饰元素使用有机blob形状
- [ ] **禁止**90度直角

### 阴影系统
- [ ] **禁止**使用纯黑色阴影
- [ ] 必须使用彩色阴影（苔藓绿或粘土橙色调）

### 组件设计
- [ ] 卡片：极浅米色背景，原木色边框，苔藓色调阴影
- [ ] 按钮：苔藓绿或赤陶土背景，rounded-full，悬停scale(1.05)，激活scale(0.95)
- [ ] 输入框：半透明白色背景，rounded-full，h-12，焦点苔藓绿色调

### 动画
- [ ] 使用transition-all duration-300或duration-500
- [ ] 所有过渡使用ease缓动，持续时间300-700ms
- [ ] 禁止突然变化

### 响应式
- [ ] 移动端全宽布局
- [ ] 使用正确的间距值（gap-8, gap-12）
- [ ] 容器max-width限制（根据部分变化）

### 可访问性
- [ ] 装饰元素添加aria-hidden
- [ ] 表单元素有aria-label
- [ ] 颜色对比度符合WCAG标准
- [ ] 所有交互元素有清晰的焦点状态
- [ ] 触摸目标满足44px最小尺寸

---

## 13. 禁止事项

以下做法**严格禁止**：

1. ❌ 使用纯白色 `#ffffff` 作为页面背景（应使用米白色 #FDFCF8）
2. ❌ 使用纯黑色阴影（应使用彩色阴影，苔藓绿或粘土橙色调）
3. ❌ 使用90度直角（应使用有机圆角或blob形状）
4. ❌ 缺少全局纹理叠加（必须添加3-4%不透明度，multiply混合模式）
5. ❌ 使用非有机字体（标题必须使用Fraunces，正文必须使用Nunito或Quicksand）
6. ❌ 缺少悬停和焦点状态
7. ❌ 使用linear或突然的snap效果（应使用ease缓动，300-700ms）
8. ❌ 按钮不使用rounded-full（完全圆形）

---

## 14. 快速参考

### 调色板

```css
--background: #FDFCF8;        /* 米白色 */
--foreground: #2C2C24;        /* 深壤土 */
--primary: #5D7052;           /* 苔藓绿 */
--primary-foreground: #F3F4F1;
--secondary: #C18C5D;         /* 赤陶土 */
--border: #DED8CF;            /* 原木色 */
```

### 字体

- 标题: 'Fraunces', serif (字重600-800)
- 正文: 'Nunito' 或 'Quicksand', sans-serif

### 常用圆角

- `rounded-full` - 按钮
- `rounded-2xl` (16px) - 标准元素
- `rounded-3xl` (24px) - 大元素
- `rounded-[2rem]` (32px) - 卡片基础
- 有机blob形状 - 装饰元素

### 常用阴影

```css
shadow-[0_4px_20px_-2px_rgba(93,112,82,0.15)]  /* 柔和阴影 */
shadow-[0_10px_40px_-10px_rgba(193,140,93,0.2)]  /* 浮动阴影 */
hover:shadow-[0_6px_24px_-4px_rgba(93,112,82,0.25)]  /* 悬停加深 */
```

### 常用间距

- `gap-8` (32px) - 标准网格间距
- `gap-12` (48px) - 大网格间距
- `py-32` (128px) - 部分垂直间距
- `px-4` (16px) - 移动端水平间距
- `px-8` (32px) - lg水平间距

### 容器宽度

- `max-w-7xl` (1280px) - Hero, Features, Blog, Pricing
- `max-w-6xl` (1152px) - How It Works, FAQ
- `max-w-5xl` (1024px) - Final CTA
- `max-w-4xl` (896px) - 文本密集部分
- `max-w-2xl` (512px) - 产品详情文本

### 有机Blob形状

```css
border-radius: 60% 40% 30% 70% / 60% 30% 70% 40%;
border-radius: 30% 60% 70% 40% / 50% 60% 30% 60%;
border-radius: 40% 60% 50% 50% / 40% 50% 60% 50%;
```

---

**执行规则**: 生成任何UI代码时，必须严格遵循本文档中标记为[MUST]的规则，优先遵循[SHOULD]规则。所有设计决策应该表达设计系统的个性，而非产生通用或样板UI。
