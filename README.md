
  - [1.1 Css/Scss](#11-cssscss)
    - [1.1.1 层叠性](#111-层叠性)
    - [1.1.2 继承性](#112-继承性)
<!-- /TOC -->


### 1.1 Css/Scss
#### 1.1.1 层叠性
```
浏览器的渲染机制是从上至下, 当有多个样式同时应用到同一个dom元素时, 默认使用最后一个样式。(不考虑手动设置权重的case)
```
#### 1.1.2 继承性
```
  1. 前提: 标签之间是嵌套关系
  2. 文字颜色、字体大小、字体、行高、文字有关的属性都可被继承
  3. 特殊点:
    a标签不能实现字体颜色的集成, 字体大小可被继承
    h1标签不可以继承字体大小, 继承过来会做一些 `计算`.