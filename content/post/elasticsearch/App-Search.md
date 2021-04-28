---
title: "App Search"
date: 2021-04-23T16:55:41+08:00
lastmod: 2021-04-23T16:55:41+08:00
draft: true
keywords: ["app search", "Elasticsearch"]
description: "Elasticsearch app search"
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
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

Elasticsearch App Search

<!--more-->

1. 创建 App Search Engine
2. 写入数据
   - Paste JSON
   - Import a .json file
   - Use POST API
   - Use the web crawler
3. 设置 schema

## Documents API

- 指定 id 将会覆盖或者新建
- 没有 id 将会自动生成
- 文档最多可包含 64 个键值对
- 每个请求最大为100个文档，每个文档必须小于 100kb
- 索引请求不能超过 10mb
- 字段名称不能是 external_id, engine_id, highlight, or, and, not, any, all, none

1. Create or Update Documents

   ```shell
   curl -X POST 'https://localhost:3002/api/as/v1/engines/{ENGINE_NAME}/documents' \
     -H 'Content-Type: application/json' \
     -H 'Authorization: Bearer private-xxxxxxxxxxxxxxxxxxxx' \
     -d '[
           {
             "id": "kyle",
             "title": "Kyle"
           }
         ]'
   
   Reponse:
   [
     {
       "id": "kyle",
       "errors": []
     }
   ]
   
   ```

2. Partial Update (PATCH)

   ```shell
   curl -X PATCH 'https://localhost:3002/api/as/v1/engines/{ENGINE_NAME}/documents' \
     -H 'Content-Type: application/json' \
     -H 'Authorization: Bearer private-xxxxxxxxxxxxxxxxxxxx' \
     -d '[
     		{ "id": "park_yosemite" },
     		{ "title": "Everglades" },
     		{ "id": "park_wind-cave", "date_established": "1903-01-09T06:00:00Z" }
         ]'
   
   Reponse:
   [
     {
       "id": "park_yosemite",
       "errors": []
     },
     {
       "id": "",
       "errors": [
         "Missing required key 'id'"
       ]
     },
     {
       "id": "park_wind-cave",
       "errors": []
     }
   ]
   
   ```

3. Delete Documents

   delete documents by `id`

   ```shell
   curl -X DELETE 'https://localhost:3002/api/as/v1/engines/{ENGINE_NAME}/documents' \
   -H 'Content-Type: application/json' \
   -H 'Authorization: Bearer private-xxxxxxxxxxxxxxxxxxxx' \
   -d '["park_zion", "park_yosemite", "does_not_exist"]'
   
   Response:
   [
     {
       "id": "park_zion",
       "deleted": true
     },
     {
       "id": "park_yosmite",
       "deleted": true
     },
     {
       "id": "does_not_exist",
       "deleted": false
     }
   ]
   ```

4. Get a Document

   ```shell
   curl -X GET 'https://localhost:3002/api/as/v1/engines/{ENGINE_NAME}/documents' \
   -H 'Content-Type: application/json' \
   -H 'Authorization: Bearer private-xxxxxxxxxxxxxxxxxxxx' \
   -d '["park_zion", "does_not_exist"]'
   OR
   # ids[]=xxx
   curl -X GET 'https://localhost:3002/api/as/v1/engines/{ENGINE_NAME}/documents?ids%5B%5D=park_zion&ids%5B%5D=does_not_exist' \
   -H 'Content-Type: application/json' \
   -H 'Authorization: Bearer private-xxxxxxxxxxxxxxxxxxxx'
   
   
   Response:
   [
     {
       "description": "Located at the junction of the Colorado Plateau, Great Basin, and Mojave Desert, this park contains sandstone features such as mesas, rock towers, and canyons, including the Virgin River Narrows. The various sandstone formations and the forks of the Virgin River create a wilderness divided into four ecosystems: desert, riparian, woodland, and coniferous forest.",
       "nps_link": "https://www.nps.gov/zion/index.htm",
       "states": [
         "Utah"
       ],
       "title": "Zion",
       "visitors": "4295127",
       "world_heritage_site": "false",
       "location": "37.3,-113.05",
       "acres": "147237.02",
       "square_km": "595.8",
       "date_established": "1919-11-19T06:00:00Z",
       "id": "park_zion"
     },
     null
   ]
   ```

5. List Document

   最多列出 10,000 个文档

   `size` 最大值 100

   ```shell
   curl -X GET 'https://localhost:3002/api/as/v1/engines/{ENGINE_NAME}/documents/list' \
   -H 'Content-Type: application/json' \
   -H 'Authorization: Bearer private-xxxxxxxxxxxxxxxxxxxx' \
   -d '{
     "page": {
       "current": 2,
       "size": 15
     }
   }'
   OR
   curl -X GET 'https://localhost:3002/api/as/v1/engines/{ENGINE_NAME}/documents/list?page[size]=15&page[current]=2' \
   -H 'Authorization: Bearer private-xxxxxxxxxxxxxxxxxxxx'
   
   Response:
   {
     "meta": {
       "page": {
         "current": 2,
         "total_pages": 4,
         "total_results": 59,
         "size": 15
       }
     },
     "results": [
       {
         "nps_link": "https://www.nps.gov/deva/index.htm",
         "title": "Death Valley",
         "visitors": "1296283"
       },
       {
         "title": "Denali",
         "visitors": "587412",
         "id": "park_denali"
       },
       # ... Truncated!
     ]
   }
   ```

6

