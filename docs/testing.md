---
sidebar_position: 99
---

# ...test page...

## Testing importing code

import CodeBlock from '@theme/CodeBlock';
import MyComponentSource from '!!raw-loader!./how-to-guides/panorama-config-commit-push.md';

<CodeBlock language="jsx">{MyComponentSource}</CodeBlock>
