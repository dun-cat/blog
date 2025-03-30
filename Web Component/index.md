## Web Component 
### 简介

Web Components 是一套 Web 平台 API，旨在通过创建可重用的自定义元素，增强代码的模块化和封装性。这些技术包括：

1. **自定义元素（Custom Elements）**：允许定义新的 HTML 标签及其行为。
2. **影子 DOM（Shadow DOM）**：提供独立的 DOM 树和样式封装，避免与页面其他部分冲突。
3. **HTML 模板（HTML Templates）**：使用 `<template>` 和 `<slot>` 元素定义可重用的 HTML 结构。

**特性**：Shadow DOM 提供`样式`和`行为`封装。

### 示例代码

**1.定义 HTML 模板**

```html
<template id="my-template">
    <style>
        div {
            color: blue;
        }
    </style>
    <div>Template Content</div>
</template>
```

**2.定义自定义元素**

```javascript
class MyCustomElement extends HTMLElement {
  static observedAttributes = ["color", "size"];

  constructor() {
    // 必须首先调用 super 方法
    super();

      const shadow = this.attachShadow({ mode: 'open' });
      const template = document.getElementById('my-template');
      const instance = template.content.cloneNode(true);
      shadow.appendChild(instance);
  }

  connectedCallback() { console.log("自定义元素添加至页面。"); }

  disconnectedCallback() { console.log("自定义元素从页面中移除。");}

  adoptedCallback() { console.log("自定义元素移动至新页面。");}

  attributeChangedCallback(name, oldValue, newValue) { console.log(`属性 ${name} 已变更。`);}
}

customElements.define("my-custom-element", MyCustomElement);

```

**3.使用自定义元素**

在 HTML 文件中使用自定义元素：

```html
<my-template-element></my-template-element>
```

参考资料：

\> [https://developer.mozilla.org/zh-CN/docs/Web/API/Web_components](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_components)