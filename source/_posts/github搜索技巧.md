---
title: "Github搜索技巧"
date: 2024-11-18 15:41:52
---

相信一般人搜索项目时，都是直接搜索技术栈相关的项目。

高级一点的搜索，会根据 最匹配、最多 Star 来进行排序、选择相应的语言、选择仓库或者代码来进行筛选

<a href="https://camo.githubusercontent.com/f3929ff08853b6f8ff35117877a85116e14b7b1359ae5945e96da04b953b724d/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d333538626533343634326433356335662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><img src="https://camo.githubusercontent.com/f3929ff08853b6f8ff35117877a85116e14b7b1359ae5945e96da04b953b724d/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d333538626533343634326433356335662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></a>

但是 GitHub 的搜索功能只支持以上这些而已吗 ？

No!

如果你只会用以上的功能，那你知道的仅仅是 GitHub 搜索的冰山一角！

GitHub 的搜索是非常强大的！下面介绍更高级的搜索技巧。

<a href="https://camo.githubusercontent.com/8d5f4f7a2706addb28d1eed32d70b2b3e38094aa2b5c702e4bb3e2f6bf1e5fdd/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d336536383336323832666234626335392e6769663f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/8d5f4f7a2706addb28d1eed32d70b2b3e38094aa2b5c702e4bb3e2f6bf1e5fdd/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d336536383336323832666234626335392e6769663f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970" style="display: inline-block;height:100.0%" width="298" /></u></a>

## 搜索语法

搜索 GitHub 时，你可以构建匹配特定数字和单词的查询。

### 查询大于或小于另一个值的值

您可以使用 `>`、`>=`、`<` 和 `<=` 搜索大于、大于等于、小于以及小于等于另一个值的值。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 查询 | 示例 |
| `>*n*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+stars%3A%3E1000&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>cats vue:&gt;1000</u></strong></a> 匹配含有 "vue" 字样、星标超过 1000 个的仓库。 |
| `>=*n*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+topics%3A%3E%3D5&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue topics:&gt;=5</u></strong></a> 匹配含有 "vue" 字样、有 5 个或更多主题的仓库。 |
| `<*n*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+size%3A%3C10000&amp;type=Code" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue size:&lt;10000</u></strong></a> 匹配小于 10 KB 的文件中含有 "vue" 字样的代码。 |
| `<=*n*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+stars%3A%3C%3D50&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue stars:&lt;=50</u></strong></a> 匹配含有 "vue" 字样、星标不超过 50 个的仓库。 |

</div>

您还可以使用 范围查询 搜索大于等于或小于等于另一个值的值。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 查询 | 示例 |
| `*n*..*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+stars%3A10..*&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue stars:10..*</u></strong></a> 等同于 `stars:>=10` 并匹配含有 "vue" 字样、有 10 个或更多星号的仓库。 |
| `*..*n*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+stars%3A%22*..10%22&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue stars:*..10</u></strong></a> 等同于 `stars:<=10` 并匹配含有 "vue" 字样、有不超过 10 个星号的仓库。 |

</div>

### 查询范围之间的值

您可以使用范围语法 `*n*..*n*` 搜索范围内的值，其中第一个数字 *n* 是最低值，而第二个是最高值。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 查询 | 示例 |
| `*n*..*n*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=cats+stars%3A10..50&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue stars:10..50</u></strong></a> 匹配含有 "vue" 字样、有 10 到 50 个星号的仓库。 |

</div>

### 查询日期

您可以通过使用 `>`、`>=`、`<`、`<=` 和 范围查询 搜索早于或晚于另一个日期，或者位于日期范围内的日期。

日期格式必须遵循 <a href="http://en.wikipedia.org/wiki/ISO_8601" rel="nofollow" target="_blank"><u>ISO8601</u></a> 标准，即 `YYYY-MM-DD`（年-月-日）。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 查询 | 示例 |
| `>*YYYY*-*MM*-*DD*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+created%3A%3E2016-04-29&amp;type=Issues" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue created:&gt;2016-04-29</u></strong></a> 匹配含有 "vue" 字样、在 2016 年 4 月 29 日之后创建的议题。 |
| `>=*YYYY*-*MM*-*DD*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+created%3A%3E%3D2017-04-01&amp;type=Issues" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue created:&gt;=2017-04-01</u></strong></a> 匹配含有 "vue" 字样、在 2017 年 4 月 1 日或之后创建的议题。 |
| `<*YYYY*-*MM*-*DD*` | <a href="https://github.com/search?q=vue+pushed%3A%3C2012-07-05&amp;type=Code&amp;utf8=%E2%9C%93" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue pushed:&lt;2012-07-05</u></strong></a> 匹配在 2012 年 7 月 5 日之前推送的仓库中含有 "vue" 字样的代码。 |
| `<=*YYYY*-*MM*-*DD*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+created%3A%3C%3D2012-07-04&amp;type=Issues" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue created:&lt;=2012-07-04</u></strong></a> 匹配含有 "vue" 字样、在 2012 年 7 月 4 日或之前创建的议题。 |
| `*YYYY*-*MM*-*DD*..*YYYY*-*MM*-*DD*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+pushed%3A2016-04-30..2016-07-04&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue pushed:2016-04-30..2016-07-04</u></strong></a> 匹配含有 "vue" 字样、在 2016 年 4 月末到 7 月之间推送的仓库。 |
| `*YYYY*-*MM*-*DD*..*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+created%3A2012-04-30..*&amp;type=Issues" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue created:2012-04-30..*</u></strong></a> 匹配在 2012 年 4 月 30 日之后创建、含有 "vue" 字样的议题。 |
| `*..*YYYY*-*MM*-*DD*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+created%3A*..2012-07-04&amp;type=Issues" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue created:*..2012-04-30</u></strong></a> 匹配在 2012 年 7 月 4 日之前创建、含有 "vue" 字样的议题。 |

</div>

您也可以在日期后添加可选的时间信息 `THH:MM:SS+00:00`，以便按小时、分钟和秒进行搜索。 这是 `T`，随后是 `HH:MM:SS`（时-分-秒）和 UTC 偏移 (`+00:00`)。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 查询 | 示例 |
| `*YYYY*-*MM*-*DD*T*HH*:*MM*:*SS*+*00*:*00*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+created%3A2017-01-01T01%3A00%3A00%2B07%3A00..2017-03-01T15%3A30%3A15%2B07%3A00&amp;type=Issues" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue created:2017-01-01T01:00:00+07:00..2017-03-01T15:30:15+07:00</u></strong></a> 匹配在 2017 年 1 月 1 日凌晨 1 点（UTC 偏移为 `07:00`）与 2017 年 3 月 1 日下午 3 点（UTC 偏移为 `07:00`）之间创建的议题。 UTC 偏移量 `07:00`，2017 年 3 月 1 日下午 3 点。 UTC 偏移量 `07:00`。 |
| `*YYYY*-*MM*-*DD*T*HH*:*MM*:*SS*Z` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=vue+created%3A2016-03-21T14%3A11%3A00Z..2016-04-07T20%3A45%3A00Z&amp;type=Issues" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue created:2016-03-21T14:11:00Z..2016-04-07T20:45:00Z</u></strong></a> 匹配在 2016 年 3 月 21 日下午 2:11 与 2016 年 4 月 7 日晚上 8:45 之间创建的议题。 |

</div>

### 排除特定结果

您可以使用 `NOT` 语法排除包含特定字词的结果。 `NOT` 运算符只能用于字符串关键词， 不适用于数字或日期。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 查询 | 示例 |
| `NOT` | <a href="https://github.com/search?q=hello+NOT+world&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>hello NOT world</u></strong></a> 匹配含有 "hello" 字样但不含有 "world" 字样的仓库。 |

</div>

缩小搜索结果范围的另一种途径是排除特定的子集。 您可以为任何搜索限定符添加 `-` 前缀，以排除该限定符匹配的所有结果。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 查询 | 示例 |
| `-*QUALIFIER*` | <a href="https://github.com/search?q=vue+stars%3A%3E10+-language%3Ajavascript&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue stars:&gt;10 -language:javascript</u></strong></a> 匹配含有 "vue" 字样、有超过 10 个星号但并非以 JavaScript 编写的仓库。 |
|  | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=mentions%3Adefunkt+-org%3Agithub&amp;type=Issues" rel="noopener noreferrer nofollow" target="_blank"><strong><u>mentions:biaochenxuying -org:github</u></strong></a> 匹配提及 <a href="https://github.com/biaochenxuying" class="user-mention notranslate" rel="noopener noreferrer nofollow" target="_blank"><u>@biaochenxuying</u></a> 且不在 GitHub 组织仓库中的议题 |

</div>

### 对带有空格的查询使用引号

如果搜索含有空格的查询，您需要用引号将其括起来。 例如：

- <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=cats+NOT+%22hello+world%22&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><u>cats NOT "hello world"</u></a> 匹配含有 "vue" 字样但不含有 "hello world" 字样的仓库。

- <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=build+label%3A%22bug+fix%22&amp;type=Issues" rel="noopener noreferrer nofollow" target="_blank"><u>build label:"bug fix"</u></a> 匹配具有标签 "bug fix"、含有 "build" 字样的议题。

某些非字母数字符号（例如空格）会从引号内的代码搜索查询中删除，因此结果可能出乎意料。

### 使用用户名的查询

如果搜索查询包含需要用户名的限定符，例如 `user`、`actor` 或 `assignee`，您可以使用任何 GitHub 用户名指定特定人员，或使用 `@me` 指定当前用户。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 查询 | 示例 |
| `QUALIFIER:USERNAME` | `author:biaochenxuying` 匹配 <a href="https://github.com/biaochenxuying" class="user-mention notranslate" rel="noopener noreferrer nofollow" target="_blank"><u>@biaochenxuying</u></a> 创作的提交。 |
| `QUALIFIER:@me` | `is:issue assignee:@me` 匹配已分配给结果查看者的议题 |

</div>

`@me` 只能与限定符一起使用，而不能用作搜索词，例如 `@me main.workflow`。

## 高级的搜索

### 按仓库名称、说明或自述文件内容搜索

通过 `in` 限定符，您可以将搜索限制为仓库名称、仓库说明、自述文件内容或这些的任意组合。

如果省略此限定符，则只搜索仓库名称和说明。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `in:name` | <a href="https://github.com/search?q=vue+in%3Aname&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue in:name</u></strong></a> 匹配其名称中含有 "jquery" 的仓库。 |
| `in:description` | <a href="https://github.com/search?q=vue+in%3Aname%2Cdescription&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue in:name,description</u></strong></a> 匹配其名称或说明中含有 "vue" 的仓库。 |
| `in:readme` | <a href="https://github.com/search?q=vue+in%3Areadme&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue in:readme</u></strong></a> 匹配其自述文件中提及 "vue" 的仓库。 |
| `repo:owner/name` | <a href="https://github.com/search?q=repo%3Abiaochenxuying%2Fblog" rel="noopener noreferrer nofollow" target="_blank"><strong><u>repo:biaochenxuying/blog</u></strong></a> 匹配特定仓库名称，比如：用户为 biaochenxuying 的 blog 项目。 |

</div>

<a href="https://camo.githubusercontent.com/2738e403641590b0b09cbac7994221e5f6ea23df64ae8018af3b36e070cd0d4b/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d623230643133616164346164383161662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/2738e403641590b0b09cbac7994221e5f6ea23df64ae8018af3b36e070cd0d4b/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d623230643133616164346164383161662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

### 在用户或组织的仓库内搜索

要在 `特定用户或组织` 拥有的所有仓库中搜索，您可以使用 `user` 或 `org` 限定符。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `user:*USERNAME*` | <a href="https://github.com/search?q=user%3Abiaochenxuying+forks%3A%3E%3D100&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>user:biaochenxuying forks:&gt;=100</u></strong></a> 匹配来自 <a href="https://github.com/biaochenxuying" class="user-mention notranslate" rel="noopener noreferrer nofollow" target="_blank"><u>@biaochenxuying</u></a>、拥有超过 100 复刻的仓库。 |
| `org:*ORGNAME*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=org%3Agithub&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>org:github</u></strong></a> 匹配来自 GitHub 的仓库。 |

</div>

### 按仓库大小搜索

`size` 限定符使用 <a href="https://docs.github.com/cn/free-pro-team@latest/articles/understanding-the-search-syntax" rel="noopener noreferrer nofollow" target="_blank"><u>大于、小于和范围限定符</u></a> 查找匹配特定大小（以千字节为单位）的仓库。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `size:*n*` | <a href="https://github.com/search?q=size%3A1000&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>size:1000</u></strong></a> 匹配恰好为 1 MB 的仓库。 |
|  | <a href="https://github.com/search?q=size%3A%3E%3D30000&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>size:&gt;=30000</u></strong></a> 匹配至少为 30 MB 的仓库。 |
|  | <a href="https://github.com/search?q=size%3A%3C50&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>size:&lt;50</u></strong></a> 匹配小于 50 KB 的仓库。 |
|  | <a href="https://github.com/search?q=size%3A50..120&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>size:50..120</u></strong></a> 匹配介于 50 KB 与 120 KB 之间的仓库。 |

</div>

<a href="https://camo.githubusercontent.com/d2b961855cbe69cd408d764c8f1d9982f458b9569d63b14d32a84621ed8f2a1b/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d353738353232623036316436663966662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/d2b961855cbe69cd408d764c8f1d9982f458b9569d63b14d32a84621ed8f2a1b/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d353738353232623036316436663966662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

### 按关注者数量搜索

您可以使用 `followers` 限定符以及<a href="https://docs.github.com/cn/free-pro-team@latest/articles/understanding-the-search-syntax" rel="noopener noreferrer nofollow" target="_blank"><u>大于、小于和范围限定符</u></a>基于仓库拥有的关注者数量过滤仓库。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `followers:*n*` | <a href="https://github.com/search?q=node+followers%3A%3E%3D10000" rel="noopener noreferrer nofollow" target="_blank"><strong><u>node followers:&gt;=10000</u></strong></a> 匹配有 10,000 或更多关注者提及文字 "node" 的仓库。 |
|  | <a href="https://github.com/search?q=styleguide+linter+followers%3A1..10&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>styleguide linter followers:1..10</u></strong></a> 匹配拥有 1 到 10 个关注者并且提及 "styleguide linter" 一词的的仓库。 |

</div>

<a href="https://camo.githubusercontent.com/7955e26ddd5ebd5b9be9f1f5a8f43e0cbbda8a2cf2043dbbaf9ea1504accec2f/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d376261306465636439393164383836312e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/7955e26ddd5ebd5b9be9f1f5a8f43e0cbbda8a2cf2043dbbaf9ea1504accec2f/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d376261306465636439393164383836312e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

<a href="https://camo.githubusercontent.com/47feb29edfcbc2a836ea5b0f77c8d5444bacf2320a7a7562e340ceb83d7084f4/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d336463333439363630626236343766622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/47feb29edfcbc2a836ea5b0f77c8d5444bacf2320a7a7562e340ceb83d7084f4/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d336463333439363630626236343766622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

### 按复刻数量搜索

`forks` 限定符使用<a href="https://docs.github.com/cn/free-pro-team@latest/articles/understanding-the-search-syntax" rel="noopener noreferrer nofollow" target="_blank"><u>大于、小于和范围限定符</u></a>指定仓库应具有的复刻数量。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `forks:*n*` | <a href="https://github.com/search?q=forks%3A5&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>forks:5</u></strong></a> 匹配只有 5 个复刻的仓库。 |
|  | <a href="https://github.com/search?q=forks%3A%3E%3D205&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>forks:&gt;=205</u></strong></a> 匹配具有至少 205 个复刻的仓库。 |
|  | <a href="https://github.com/search?q=forks%3A%3C90&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>forks:&lt;90</u></strong></a> 匹配具有少于 90 个复刻的仓库。 |
|  | <a href="https://github.com/search?q=forks%3A10..20&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>forks:10..20</u></strong></a> 匹配具有 10 到 20 个复刻的仓库。 |

</div>

<a href="https://camo.githubusercontent.com/86cb1066c5d3bdc1e076d43d579af4ff5db2b748c9d198395fb3cc553030fe1b/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d663531373334623433323735313639632e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/86cb1066c5d3bdc1e076d43d579af4ff5db2b748c9d198395fb3cc553030fe1b/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d663531373334623433323735313639632e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

### 按星号数量搜索

您可以使用 <a href="https://docs.github.com/cn/free-pro-team@latest/articles/understanding-the-search-syntax" rel="noopener noreferrer nofollow" target="_blank"><u>大于、小于和范围限定符</u></a> 基于仓库具有的 <a href="https://docs.github.com/cn/free-pro-team@latest/articles/saving-repositories-with-stars" rel="noopener noreferrer nofollow" target="_blank"><u>星标</u></a> 数量搜索仓库

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `stars:*n*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=stars%3A500&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>stars:500</u></strong></a> 匹配恰好具有 500 个星号的仓库。 |
|  | <a href="https://github.com/search?q=stars%3A10..20+size%3A%3C1000&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>stars:10..20</u></strong></a> 匹配具有 10 到 20 个星号、小于 1000 KB 的仓库。 |
|  | <a href="https://github.com/search?q=stars%3A%3E%3D500+fork%3Atrue+language%3Avue&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>stars:&gt;=500 fork:true language:vue</u></strong></a> 匹配具有至少 500 个星号，包括复刻的星号（以 vue 编写）的仓库。 |

</div>

<a href="https://camo.githubusercontent.com/9ae74b5f981aa8ceb29b26652278d2ac3f8fb8c3d26ef636638c256f576830ea/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d663232633263663061623664646538352e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/9ae74b5f981aa8ceb29b26652278d2ac3f8fb8c3d26ef636638c256f576830ea/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d663232633263663061623664646538352e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

### 按仓库创建或上次更新时间搜索

你可以基于创建时间或上次更新时间过滤仓库。

- 对于仓库创建，您可以使用 `created` 限定符；

- 要了解仓库上次更新的时间，您要使用 `pushed` 限定符。 `pushed` 限定符将返回仓库列表，按仓库中任意分支上最近进行的提交排序。

两者均采用日期作为参数。 日期格式必须遵循 ISO8601 标准，即 `YYYY-MM-DD`（年-月-日）。

也可以在日期后添加可选的时间信息 `THH:MM:SS+00:00`，以便按小时、分钟和秒进行搜索。 这是 `T`，随后是 `HH:MM:SS`（时-分-秒）和 UTC 偏移 (`+00:00`)。

日期支持 `大于、小于和范围限定符`。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `created:*YYYY-MM-DD*` | <a href="https://github.com/search?q=vue+created%3A%3C2020-01-01&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue created:&lt;2020-01-01</u></strong></a> 匹配具有 "vue" 字样、在 2020 年之前创建的仓库。 |
| `pushed:*YYYY-MM-DD*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=css+pushed%3A%3E2020-02-01&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>css pushed:&gt;2020-02-01</u></strong></a> 匹配具有 "css" 字样、在 2020 年 1 月之后收到推送的仓库。 |
|  | <a href="https://github.com/search?q=vue+pushed%3A%3E%3D2020-03-06+fork%3Aonly&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue pushed:&gt;=2020-03-06 fork:only</u></strong></a> 匹配具有 "vue" 字样、在 2020 年 3 月 6 日或之后收到推送并且作为复刻的仓库。 |

</div>

<a href="https://camo.githubusercontent.com/355c1975f5dca940251af61ba33bdf3fddde0dda6164a6229419af4108cece30/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d383163623363623731346565363931332e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/355c1975f5dca940251af61ba33bdf3fddde0dda6164a6229419af4108cece30/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d383163623363623731346565363931332e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

### 按语言搜索

您可以基于其编写采用的主要语言搜索仓库。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `language:*LANGUAGE*` | <a href="https://github.com/search?q=vue+language%3Ajavascript&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>vue language:javascript</u></strong></a> 匹配具有 "vue" 字样、以 JavaScript 编写的仓库。 |

</div>

<a href="https://camo.githubusercontent.com/4ca07f091280e9556d17cdee571b167586005c2bcfe5ebe52a0c88e5f023fe44/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d346238663765313365303761383539302e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/4ca07f091280e9556d17cdee571b167586005c2bcfe5ebe52a0c88e5f023fe44/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d346238663765313365303761383539302e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

### 按主题搜索

您可以查找归类为特定 <a href="https://docs.github.com/cn/free-pro-team@latest/articles/classifying-your-repository-with-topics" rel="noopener noreferrer nofollow" target="_blank"><u>主题</u></a> 的所有仓库。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `topic:*TOPIC*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=topic%3Aalgorithm&amp;type=Repositories&amp;ref=searchresults" rel="noopener noreferrer nofollow" target="_blank"><strong><u>topic:algorithm</u></strong></a> 匹配已归类为 "algorithm" 主题的仓库。 |

</div>

估计又有很多人不知道 GitHub 上有话题一说的吧。

<a href="https://camo.githubusercontent.com/b7add5e1a05d52926723b06e67cdf91de3539406f7c25eb63549d3ee188e575e/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d333033653434346436626361336239642e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/b7add5e1a05d52926723b06e67cdf91de3539406f7c25eb63549d3ee188e575e/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d333033653434346436626361336239642e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

<a href="https://camo.githubusercontent.com/c8d91da7ad1a36b6d5cc650f2c55b3e1230ca867e8be6ff6492f17c56196fc84/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d623065643837313531613461303662622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/c8d91da7ad1a36b6d5cc650f2c55b3e1230ca867e8be6ff6492f17c56196fc84/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d623065643837313531613461303662622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

### 按主题数量搜索

您可以使用 `topics` 限定符以及 <a href="https://docs.github.com/cn/free-pro-team@latest/articles/understanding-the-search-syntax" rel="noopener noreferrer nofollow" target="_blank"><u>大于、小于和范围限定符</u></a> 按应用于仓库的 <a href="https://docs.github.com/cn/free-pro-team@latest/articles/classifying-your-repository-with-topics" rel="noopener noreferrer nofollow" target="_blank"><u>主题</u></a> 数量搜索仓库。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `topics:*n*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=topics%3A5&amp;type=Repositories&amp;ref=searchresults" rel="noopener noreferrer nofollow" target="_blank"><strong><u>topics:5</u></strong></a> 匹配具有五个主题的仓库。 |
|  | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=topics%3A%3E3&amp;type=Repositories&amp;ref=searchresults" rel="noopener noreferrer nofollow" target="_blank"><strong><u>topics:&gt;3</u></strong></a> 匹配超过三个主题的仓库。 |

</div>

<a href="https://camo.githubusercontent.com/f7fe38d2c8c3e3026380c6b1d0dd8080624712af8daa0ebe9d300575c6c14d37/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d386165323861656363633866383833642e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/f7fe38d2c8c3e3026380c6b1d0dd8080624712af8daa0ebe9d300575c6c14d37/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d386165323861656363633866383833642e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

### 使用可视界面搜索

还可以使用 <a href="https://github.com/search" rel="noopener noreferrer nofollow" target="_blank"><u>search</u></a> page 或 <a href="https://github.com/search/advanced" rel="noopener noreferrer nofollow" target="_blank"><u>advanced search</u></a> page 搜索 GitHub 哦。

这种搜索方式，估计就更少人知道了吧。

<a href="https://github.com/search/advanced" rel="noopener noreferrer nofollow" target="_blank"><u>advanced search</u></a> page 提供用于构建搜索查询的可视界面。

您可以按各种因素过滤搜索，例如仓库具有的星标数或复刻数。 在填写高级搜索字段时，您的查询将在顶部搜索栏中自动构建。

<a href="https://camo.githubusercontent.com/402e05f41eff28b9f1920f397440983227e2bc9b523b5dd83fbbb1f0c58ddc48/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d376439323064633838333131363039622e6769663f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/402e05f41eff28b9f1920f397440983227e2bc9b523b5dd83fbbb1f0c58ddc48/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d376439323064633838333131363039622e6769663f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970" style="display: inline-block;height:100.0%" width="806" alt="高级搜索" /></u></a>

### 按许可搜索

您可以按其<a href="https://docs.github.com/cn/free-pro-team@latest/articles/licensing-a-repository" rel="noopener noreferrer nofollow" target="_blank"><u>许可</u></a>搜索仓库。 您必须使用<a href="https://docs.github.com/cn/free-pro-team@latest/articles/licensing-a-repository/#searching-github-by-license-type" rel="noopener noreferrer nofollow" target="_blank"><u>许可关键词</u></a>按特定许可或许可系列过滤仓库。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `license:*LICENSE_KEYWORD*` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=license%3Aapache-2.0&amp;type=Repositories&amp;ref=searchresults" rel="noopener noreferrer nofollow" target="_blank"><strong><u>license:apache-2.0</u></strong></a> 匹配根据 Apache License 2.0 授权的仓库。 |

</div>

### 按公共或私有仓库搜索

您可以基于仓库是公共还是私有来过滤搜索。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `is:public` | <a href="https://github.com/search?q=is%3Apublic+org%3Agithub&amp;type=Repositories&amp;utf8=%E2%9C%93" rel="noopener noreferrer nofollow" target="_blank"><strong><u>is:public org:github</u></strong></a> 匹配 GitHub 拥有的公共仓库。 |
| `is:private` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=pages+is%3Aprivate&amp;type=Repositories" rel="noopener noreferrer nofollow" target="_blank"><strong><u>is:private pages</u></strong></a> 匹配您有访问权限且包含 "pages" 字样的私有仓库。 |

</div>

### 按公共或私有仓库搜索

您可以根据仓库是否为镜像以及托管于其他位置托管来搜索它们。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `mirror:true` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=mirror%3Atrue+GNOME&amp;type=" rel="noopener noreferrer nofollow" target="_blank"><strong><u>mirror:true GNOME</u></strong></a> 匹配是镜像且包含 "GNOME" 字样的仓库。 |
| `mirror:false` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=mirror%3Afalse+GNOME&amp;type=" rel="noopener noreferrer nofollow" target="_blank"><strong><u>mirror:false GNOME</u></strong></a> 匹配并非镜像且包含 "GNOME" 字样的仓库。 |

</div>

### 基于仓库是否已存档搜索

你可以基于仓库是否<a href="https://docs.github.com/cn/free-pro-team@latest/articles/about-archiving-repositories" rel="noopener noreferrer nofollow" target="_blank"><u>已存档</u></a>来搜索仓库。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `archived:true` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=archived%3Atrue+GNOME&amp;type=" rel="noopener noreferrer nofollow" target="_blank"><strong><u>archived:true GNOME</u></strong></a> 匹配已存档且包含 "GNOME" 字样的仓库。 |
| `archived:false` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=archived%3Afalse+GNOME&amp;type=" rel="noopener noreferrer nofollow" target="_blank"><strong><u>archived:false GNOME</u></strong></a> 匹配未存档且包含 "GNOME" 字样的仓库。 |

</div>

### 基于具有 `good first issue` 或 `help wanted` 标签的议题数量搜索

您可以使用限定符 `help-wanted-issues:>n` 和 `good-first-issues:>n` 搜索具有最少数量标签为 `help-wanted` 或 `good-first-issue` 议题的仓库。

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| 限定符 | 示例 |
| `good-first-issues:>n` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=javascript+good-first-issues%3A%3E2&amp;type=" rel="noopener noreferrer nofollow" target="_blank"><strong><u>good-first-issues:&gt;2 javascript</u></strong></a> 匹配具有超过两个标签为 `good-first-issue` 的议题且包含 "javascript" 字样的仓库。 |
| `help-wanted-issues:>n` | <a href="https://github.com/search?utf8=%E2%9C%93&amp;q=react+help-wanted-issues%3A%3E4&amp;type=" rel="noopener noreferrer nofollow" target="_blank"><strong><u>help-wanted-issues:&gt;4 react</u></strong></a> 匹配具有超过四个标签为 `help-wanted` 的议题且包含 "React" 字样的仓库。 |

</div>

## 学习

其实，以上很多内容的都是来自于 GitHub 的官方文档，如果你还想学习更多技巧，请看

> <a href="https://docs.github.com/cn" rel="noopener noreferrer nofollow" target="_blank"><u>GitHub 官方文档</u></a> : <a href="https://docs.github.com/cn" rel="noopener noreferrer nofollow" target="_blank"><u>https://docs.github.com/cn</u></a>

<a href="https://camo.githubusercontent.com/4e7e11e27f4205783b6f3ef7bb6a1814c2949aba27beb4af9011e2565d8cd149/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d653535313733323934366635626136322e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/4e7e11e27f4205783b6f3ef7bb6a1814c2949aba27beb4af9011e2565d8cd149/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d653535313733323934366635626136322e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

<a href="https://camo.githubusercontent.com/197bf0b4027c75c264570e217359b03e42060b58af467ed882b15b59c8c18484/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d346262363233346434646464306435352e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/197bf0b4027c75c264570e217359b03e42060b58af467ed882b15b59c8c18484/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d346262363233346434646464306435352e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

如果你还不了解或者不会使用 GitHub ，可以看看这一章节：

> <a href="https://docs.github.com/cn/free-pro-team@latest/github/getting-started-with-github/git-and-github-learning-resources" rel="noopener noreferrer nofollow" target="_blank"><u>Git 和 GitHub 学习资源</u></a> ：<a href="https://docs.github.com/cn/free-pro-team@latest/github/getting-started-with-github/git-and-github-learning-resources" rel="noopener noreferrer nofollow" target="_blank"><u>https://docs.github.com/cn/free-pro-team@latest/github/getting-started-with-github/git-and-github-learning-resources</u></a>

<a href="https://camo.githubusercontent.com/1b1be229678f65f610ae37d9114ad464f4eb548bbd6356e1d86b908855ae00f6/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d376130613065636663386661356231622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/1b1be229678f65f610ae37d9114ad464f4eb548bbd6356e1d86b908855ae00f6/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f31323839303831392d376130613065636663386661356231622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

## 最后

平时如何发现好的开源项目，可以看看这篇文章：<a href="https://mp.weixin.qq.com/s/98P-ARrYlkmog8i79zX6Og" rel="nofollow" target="_blank"><u>GitHub 上能挖矿的神仙技巧 - 如何发现优秀开源项目</u></a>。

推荐文章

- <a href="https://mp.weixin.qq.com/s/qmO-xivboW8LgfGiaiOeeg" rel="nofollow" target="_blank"><u>全球最火的WEB开发学习路线！没有之一！3 天就在GitHub收获了接近 1w 点赞</u></a>

- <a href="https://mp.weixin.qq.com/s/VgGiNbQBDh2ldK5apAHz3g" rel="nofollow" target="_blank"><u>Github标星1.6W+，程序员不得不知的“潜规则”又火了，早知道就不会秃头了</u></a>
