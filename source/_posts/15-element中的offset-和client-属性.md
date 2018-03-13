---
title: 15.element中的offset*和client*属性
date: 2018-03-13 14:38:34
tags: [DOM]
---

## HTMLElement

### offsetWidth

offsetWidth为只读，会返回元素布局宽度。它包括元素的边距、水平内边距、垂直滚动条（如果有）以及css宽度。如果元素是隐藏的（例如，该元素或者父元素的```style.display```为```none```），则会返回0。

![image](../images/15/Dimensions-offset.png)

**offsetWidth = width + padding + scrollbar + border**（注：标准盒子模型下）

[MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetWidth)

## Element

### clientWidth

当该元素没有设置css或者元素为内联 (inline) 时，clientWidth为 0，否则表示内部宽度像素值。它包括内边距，但不包括垂直滚动条（如果有）、边框、外边距。

![image](../images/15/Dimensions-client.png)

**clientWidth = width + padding**（注：标准盒子模型下）

[MDN](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientWidth)

更多[Determining the dimensions of elements](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Object_Model/Determining_the_dimensions_of_elements)