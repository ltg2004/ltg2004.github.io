---
# permalink: "Posts"
layout: single

title: 'Vue - Global Components'
excerpt: 'Vue-handbook'

author_profile: false

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5 # 투명도

data: 2021-10-16 22:12:00 +0900
last_modified_at: 2021-10-16 22:12:00 +0900

categories:
  - post
tags:
  - Vue
  - Components
  - global
---

# Global Components

---

### 전역 컴포넌트 넣기 코드

```
 - src/globals.js

 import { upperFirst, camelCase } from 'lodash';
 const requireComponent = require.context(
   './components',
   false,
   /Shop[A-Z]\w+\.(vue|js)$/,
 );
 requireComponent.keys().forEach((fileName) => {
   const componentConfig = requireComponent(fileName);
   const componentName = upperFirst(
     camelCase(fileName.replace(/^\.\/(.*)\.\w+$/, '$1')),
   );
   Vue.component(componentName, componentConfig.default);
 });
```
