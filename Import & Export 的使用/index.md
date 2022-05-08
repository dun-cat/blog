## Import & Export 的使用 
 Export 和 Import

定义模块和引用模块

with "default"
``` javascript
export default function foo() {} // file.js
import xxx from 'path/file'
```

 no "default"
``` javascript

export function foo() {} // file.js
import { foo } from 'path/file'
```

 1.每个模块只能有一个 default export