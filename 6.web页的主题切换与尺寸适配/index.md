### 基础介绍
为自己的网站添加主题和做移动端适配算是前端中非常常见的需求,实现的方法也是非常多  
本质上基本都是通过切换属性或者是类名,移动端适配则是使用媒体查询  
虽然都是些非常成熟的技术了,但是我还是想记录一下我的实现方法(唉,说好的不反复造轮子)

### 环境介绍
我就以我自己的博客网站为例,他是一个单页应用,前端框架是vue,状态管理使用的是比较轻量的pinia,样式则是使用了scss

### 实现流程

#### 主题构建

##### 构建基础样式

想要实现不同的主题,自然也必须为不同的主题配置不同的基础变量,**这里的变量必须使用css变量**,scss的变量只存在于编译时,编译之后就没了,而css变量在运行时生效.

这里没什么好说的,不管是自己写好,还是ai生成好,你想要多少种主题,就得写多少套对应的变量

我自己就实现了两种,再多我觉得也没什么必要

```
theme.scss

/* 默认浅色主题 */
:root {
  --color-primary: #2563eb; //链接、按钮、强调
  --color-primary-hover: #1d4ed8; //主色hover
  --color-accent: #94a3b8; //次级文字、边框
  --color-bg: #f5f3ef; //主背景
  --color-surface: #ffffff; //卡片背景、列表项背景
  --color-surface-alt: #f1f5f9; //辅助色浅背景，比如表格header、引用背景
  --color-text-primary: #1e293b; //标题、正文主文字
  --color-text-secondary: #475569; //次级文字、辅助说明文字
  --color-border: #e2e8f0; //分隔线、输入框边框
  --color-muted: #f1f5f9; //弱化背景

  --color-surface-rgb: 248, 250, 252;
  --color-surface-alt-rgb: 241, 245, 249;
  --color-border-rgb: 226, 232, 240;
  --color-primary-rgb: 37, 99, 235;

  /* Markdown 专属 */
  --color-heading: #0f172a;
  --color-blockquote-bg: #f8fafc;
  --color-blockquote-border: #2563eb33; /* 主色 + 透明度 */
  --color-code-bg: #f1f5f9;
  --color-pre-bg: #0f172a08;
  --color-pre-text: #1e293b;
  --color-highlight-bg: #fff3b0; /* mark 背景 */
  --color-table-header: #f1f5f9;
  --color-scrollbar-track: #f1f5f9;
  --color-scrollbar-thumb: #cbd5e1;
  --color-scrollbar-thumb-hover: #94a3b8;

  --color-success: #38b000;
  --color-warning: #f59e0b;
  --color-error: #dc2626;
  --color-info: #3b82f6;

  /* 视觉辅助 */
  --radius-small: 4px;
  --radius-medium: 8px;
  --radius-large: 12px;
  --radius-xl: 16px;

  /* 阴影 */
  --shadow-light: 0 2px 6px rgba(0, 0, 0, 0.05);
  --shadow-image: 0 4px 12px rgba(0, 0, 0, 0.08);
  --shadow-card: 0 4px 16px rgba(0, 0, 0, 0.08);

  /*滑块*/
  --scrollbar-thumb: rgba(0, 0, 0, 0.2);
  --scrollbar-thumb-hover: rgba(0, 0, 0, 0.4);
  --scrollbar-track: transparent;

  //昼夜切换动画
  --transition: all 0.3s ease-in-out;
}

/* 夜间主题 */
[data-theme='dark'] {
  --color-primary: #3b82f6;
  --color-primary-hover: #60a5fa;
  --color-accent: #64748b;
  --color-bg: #11161a;
  --color-surface: #191d21;
  --color-surface-alt: #334155;
  --color-text-primary: #f1f5f9;
  --color-text-secondary: #cbd5e1;
  --color-border: #334155;
  --color-muted: #1e293b;

  --color-surface-rgb: 30, 41, 59;
  --color-surface-alt-rgb: 51, 65, 85;
  --color-border-rgb: 51, 65, 85;
  --color-primary-rgb: 59, 130, 246;

  /* Markdown 专属 */
  --color-heading: #f8fafc;
  --color-blockquote-bg: #1e293b;
  --color-blockquote-border: #3b82f666;
  --color-code-bg: #1e293b;
  --color-pre-bg: #1e293b;
  --color-pre-text: #e2e8f0;
  --color-highlight-bg: #facc1533;
  --color-table-header: #334155;
  --color-scrollbar-track: #0f172a;
  --color-scrollbar-thumb: #475569;
  --color-scrollbar-thumb-hover: #64748b;

  --color-success: #4ade80;
  --color-warning: #fbbf24;
  --color-error: #f87171;
  --color-info: #60a5fa;

  /* 阴影 */
  --shadow-light: 0 2px 6px rgba(255, 255, 255, 0.05);
  --shadow-image: 0 4px 12px rgba(0, 0, 0, 0.3);

  /* 滑块 */
  --scrollbar-thumb: rgba(255, 255, 255, 0.2);
  --scrollbar-thumb-hover: rgba(255, 255, 255, 0.4);
}
```

在main.js/ts中导入你的样式文件即可

我是有一个集中导入的入口文件,所以就导入入口文件了
```
index.scss

@use "theme";
......你其他的scss文件

```

```
main.ts

<!-- 填上自己的文件的位置即可 -->
import '@/assets/styles/index.scss'
```

##### 引用变量

上一步把变量都导入了项目,接下来就在vue文件中使用对应的属性就行了,记得使用前导入,除非你在vite的配置文件中配置了自动导入,我发现我配置了就一直受到ts的警告,我就移除了,看的很不舒服.

```
<!-- 任意你需要切换主题的vue文件中 -->
@use "../assets/styles/variables" as vars;

 .tag {
    height: 100px;
    margin: 0 10px;
    padding: 10px;
    border-radius: var(--radius-large);
    background-color: var(--color-bg);
}
```

做到这一步应该是能够让默认主题生效了

##### 主题切换

这里我使用pinia做全局状态管理,构建了一个store用来存储记录选用的主题,而存储方式就直接使用localstorage就行,如果有更多需求,比如需要远程操控主题切换,那最好还是使用接口请求的方式去获取.

```
useThemeStore.ts

export const useThemeStore = defineStore("theme", () => {
    // 0. 获取系统的主题作为初始主题
    const getSystemTheme = (): ThemeType => {
        return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
    }

    // 1. 初始化从本地缓存读取，增加类型安全检查
    const savedTheme = localStorage.getItem("theme") as ThemeType | null
    const theme = ref<ThemeType>(savedTheme || getSystemTheme())

    // 2. 封装一个统一的同步方法
    const applyTheme = (val: ThemeType) => {
        document.documentElement.setAttribute("data-theme", val)
        document.documentElement.className = val //这里是为了elementUI的主题切换,可以忽视
        localStorage.setItem("theme", val)
    }

    // 3. 初始执行一次
    applyTheme(theme.value)

    const toggleTheme = () => {
        theme.value = theme.value === "light" ? "dark" : "light"
        applyTheme(theme.value)
    }
    return {theme, toggleTheme}
})
```

上面的store中暴露了一个变量theme和切换函数toggleTheme,用来手动控制主题切换,它为body元素挂载了一个属性**data-theme**,当然这个可以自己随意命名,和第一步构建的属性属性选择器中保持一致即可.

还需要注意一点的是store的挂载是滞后的,所以为了初次进入主题正确,可以在index.html中手动初始化一下,放在meta标签之后执行即可,当然,也可以设置一个默认值之类的,直接在html标签中加入属性.

```
<!-- 手动初始化 -->
<script>
    (function () {
        const saved = localStorage.getItem('theme');
        const systemDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
        const theme = saved || (systemDark ? 'dark' : 'light');
        document.documentElement.setAttribute('data-theme', theme);
        document.documentElement.classList.add(theme);
    })();
</script>
```

```
<!-- 默认值 -->
<html lang="zh-CN" data-theme='light'>
</html>
```

如果操作完发现首次进入会闪屏,有可能是因为你为主题切换配置了transition,可以在刚渲染时候关闭,之后在开启.

在此之后,你就可以顺利使用了,在代码里构建一个button,将toggleTheme作为触发函数试试吧

#### 移动端适配

其实这个也没什么特别好说的点,主要就是媒体查询,我只想提一点,因为尺寸变化往往伴随的不仅仅是一些属性的变化,还会有组件的切换,比如pc的时候菜单展开,而切换成pe就折叠起来,这两个其实并不是一个组件.

但是在模板中又怎么方便的观测到变化呢,可以使用api:**window.matchMedia**来获取是否满足某个媒体查询的条件

那么我就以此构建了一个store

```
useAppStore.ts
//预设的手机尺寸
const MOBILE_WIDTH = 768;

export const useAppStore = defineStore("app", () => {
    const isMobile = ref<boolean>(false)

    const mediaQuery = window.matchMedia(`(max-width: ${MOBILE_WIDTH}px)`)

    const updateSize = (e: MediaQueryListEvent | MediaQueryList) => {
        isMobile.value = e.matches
    }
    // 初始状态
    updateSize(mediaQuery)

    // 尺寸发生变化就会去更新isMobile的值
    mediaQuery.addEventListener('change', updateSize);

    return {isMobile}
})
```

那么你在vue文件中通过监听isMobile的变化就能知道当前的尺寸范围了

```
//注意使用的时候别忘了转成响应式变量
const {isMobile} = storeToRefs(useAppStore())
```

到此便完成了简单的主题切换以及尺寸适配的方案
