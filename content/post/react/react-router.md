---
title: "React Router"
date: 2022-04-23T14:59:34+08:00
lastmod: 2022-04-23T14:59:34+08:00
draft: true
keywords: []
description: ""
tags: []
categories: []
author: "Kyle Zhao"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

React Router

<!--more-->

`useParams`

```
<Route path="/user/:id" element={<User />} />

// 获取 id 参数
let {id} = useParams()
```

`useSearchParams`

```
/path?user=dr-kyle

const MyComponent = ()=>{
   const [searchParams, setSearchParams] = useSearchParams();
   let user = searchParams.get("user");
   const onChange=(event)=>{
     const {name, value} = event?.target;
     setSearchParams({[name]: value})       
   }
   return <input name="search" defaultValue={user ?? undefined} onChange={onChange} />
}
```



