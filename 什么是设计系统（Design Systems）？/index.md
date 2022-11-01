## 什么是设计系统（Design Systems）？ 
### 什么是设计系统？

设计系统并不是一个新观念，可以回溯于早期 2013 年 Brad Forst 提出的[Atomic Design](https://bradfrost.com/blog/post/atomic-web-design/)。Google[Material Design](https://m3.material.io/)在 2014 年大放异彩。2016 年[Airbnb](https://karrisaarinen.com/posts/building-airbnb-design-system/)开始进行创建设计系统。

陆陆续续，许多知名公司包含[Salesforce](https://www.lightningdesignsystem.com/),[Atlasssian](https://atlassian.design/),[Shopify](https://polaris.shopify.com/)对外公布产品的设计系统，让大众认识设计系统的概要及推广`设计规模化`（modularity）。

### 为什么需要设计系统？

较具规模的公司往往有多数个产品，有时候你用户使用公司的不同产品时，会发现视觉、使用方式和整个体验感觉像是来自不同的公司。

#### 市场/平台逐趋成熟

无论是桌面端产品还是移动端产品，经过这么多年的发展，都较为成熟。大众对产品的品质要求更高。

#### 创建产品一致性

不但是在单一产品之内，包含在不同平台及设备间的转换（iOS, Andriod, Saas, Mobile Web, iPad, or TV, etc），都希望能创建产品的连贯性。

#### 加速开发过程

由于有统一的设计系统，团队成员可以随时领取元素、同步更新，减少设计与开发反覆确认的过程。

#### 扩张产品团队

设计系统是由有清楚规范、一系列可重复利用的元素所组成，当产品`模块化`，可延展性就增加了。

### 设计系统团队组成

* 用户体验设计师 UX Designer
* 视觉设计师 Visual Designer
* 动效设计师 Motion Designer
* 内容写手[Content Copy](https://zhuanlan.zhihu.com/p/33919842)
* 前端工程师 Front End Engineer
* 产品经理 Product Manger

### 设计令牌

设计令牌（Design tokens）的概念在不同的设计系统有很多不同的描述：

在[Material](https://m3.material.io/foundations/design-tokens/overview)中，令牌描述为存储样式值，比如颜色和字体。这些值可以被用于设计、代码、工具以及平台。

在[Lightning](https://www.lightningdesignsystem.com/design-tokens/)中，令牌描述为设计系统的可视化设计原子。具体来说，它们是存储可视化设计属性的命名实体。我们使用他们替代硬编码值，并用来维护一个可伸缩和一致视觉系统。

在[designbetter 的设计系统手册](https://www.designbetter.co/design-systems-handbook)中，令牌描述为设计系统的实现基础，由名称和值组成的存储数据，用于抽象你需要管理的设计属性。

令牌可以存储：`颜色`、`字体`、`间距`、`透明度`、`行高`、`阴影`、`圆角`、`网格`等值，我们可以通过表格描述他们：

**颜色**：

|令牌|值|描述|
|--|--|--|
|$color-gray-1|rgb(255, 255, 255)|灰色颜色 1|
|$color-gray-2|rgb(250, 250, 249)|灰色颜色 2|

**背景颜色**：

|令牌|值|描述|
|--|--|--|
|$color-background|rgb(243, 243, 243)|整个 app 默认背景色|
|$color-background-alt|rgb(255, 255, 255)|整个 app 第二默认背景色|


我们可以通过一个简单的 JSON 文件存储他们：

``` json
{
  "color": {
    "base": {
      "red": { "value": "#ff0000" }
    },
    "background": {
      "primary": { "value": "#eee" },
      "secondary": { "value": "#ccc" },
      "tertiary": { "value": "#999" }
    }
  }
}
```

#### 如何设计令牌？

文献参考：

\> [https://designtongue.me/design_system_how_to_begin/](https://designtongue.me/design_system_how_to_begin/)
\> [https://24ways.org/2012/design-systems/](https://24ways.org/2012/design-systems/)
