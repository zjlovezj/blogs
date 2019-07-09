# 背景
在找图标的过程中，看见很多图标是SVG格式的，虽然知道svg的基本知识，但一直没在意SVG是怎么做到的。  
直到今天，有点时间看了看SVG的相关知识，以后碰见SVG的相关代码，起码也知道是怎么回事，也许未来在某些场景下，也能恰好应用到SVG。

## SVG的历史
SVG很早前就有规范了(1999)，但是直到IE9才支持，这延缓了SVG的流行。  
SVG是矢量图形，可编程，代码可以相对少，缩放不会造成模糊。

## SVG的基本概念
svg代码可以写在html里面，并且JS可以访问并操作。  
```html
<svg width="500" height="500">
  <circle
    cx="200"
    cy="200"
    r="100"
    style="fill:red; stroke: black; stroke-width: 5pt;"
  ></circle>
</svg>
```
SVG有一些基本的图形，circle/rect/cllipse/line/polyline，都是声明式的画图，SVG就像是声明式的canvas。  
其中最强大的就属 **path** 了，它能够控制每一条曲线的运动规则，最后造出合适的形状。  
不过path中的d属性特别是A/a很难懂。。。

## SVG与其他工具的结合
* image标签可以使用SVG。  
* svg有webpack的loader
* svg可以尽量傻瓜化的做成组件吗？
