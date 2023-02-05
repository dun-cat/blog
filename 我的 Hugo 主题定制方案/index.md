## 我的 Hugo 主题定制方案 
### Markdown 

#### 使用 shortcodes 创建 notice 提示

通常在文章需要特殊说明的地方可以使用 `notice 提示`，同时它支持 `dark` 和 `light` 两种主题。如下面的效果：

**Note**

{{% notice note %}}
A notice disclaimer
{{% /notice %}}

**Info**

{{% notice info %}}
An information disclaimer
{{% /notice %}}

**Tip**

{{% notice tip %}}
A tip disclaimer
{{% /notice %}}

**Warning**

{{% notice warning %}}
A warning disclaimer
{{% /notice %}}

你可以在[这里](https://github.com/martignoni/hugo-notice/tree/master/layouts/shortcodes)看到它的 shortcode 源码，并根据[官方文档](https://gohugo.io/templates/shortcode-templates/)来创建自己的 shortcode 或调整你想要的样式。
