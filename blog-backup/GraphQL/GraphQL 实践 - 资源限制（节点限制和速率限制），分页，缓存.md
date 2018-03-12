[参考资料](http://graphql.cn/learn/pagination/)
[GitHub 官方的资源限制相关（节点限制和速率限制）的文档](https://developer.github.com/v4/guides/resource-limitations/)
#1. 资源限制
资源限制，需要在服务端实现。
##1.1 节点限制
每次调用必须加节点数的限制，否则当数据量过大时会出问题。
###1.1.1 GitHub 的 GraphQL  接口调用规则：
- 客户必须为每个 connection 提供 first 或 last 参数。
- first 或 last 参数的取值范围在 1-100 之间。
- 每一个独立调用最多可以请求50万个节点。

###1.1.2 GitHub 的 GraphQL  接口 node 节点数量计算规则：
####简单查询：
```
query {
  viewer {
    repositories(first: 50) {
      edges {
        repository:node {
          name

          issues(first: 10) {
            totalCount
            edges {
              node {
                title
                bodyHTML
              }
            }
          }
        }
      }
    }
  }
}
```
计算：
```
50         = 50 repositories
+
50 x 10  = 500 repository issues

            = 550 total nodes
```
#### 复杂查询：
```
query {
  viewer {
    repositories(first: 50) {
      edges {
        repository:node {
          name

          pullRequests(first: 20) {
            edges {
              pullRequest:node {
                title

                comments(first: 10) {
                  edges {
                    comment:node {
                      bodyHTML
                    }
                  }
                }
              }
            }
          }

          issues(first: 20) {
            totalCount
            edges {
              issue:node {
                title
                bodyHTML

                comments(first: 10) {
                  edges {
                    comment:node {
                      bodyHTML
                    }
                  }
                }
              }
            }
          }
        }
      }
    }

    followers(first: 10) {
      edges {
        follower:node {
          login
        }
      }
    }
  }
}
```
计算：
```
50              = 50 repositories
+
50 x 20       = 1,000 pullRequests
+
50 x 20 x 10 = 10,000 pullRequest comments
+
50 x 20       = 1,000 issues
+
50 x 20 x 10 = 10,000 issue comments
+
10              = 10 followers

                 = 22,060 total nodes
```

##1.2 速率限制
###1.2.1 GraphQL API 的速率限制与 REST API 不同
对于 REST 请求，简单的限制请求次数即可。而对于 GraphQL，一个复杂的 GraphQL 调用可能等同于数千个 REST 请求，而一个简单的 GraphQL 调用可能只等同于一两个 REST 请求。
###1.2.2 GitHub 的 GraphQL 接口如何限制速率
为了准确地计算一个查询带来的服务器负载，GitHub 的 GraphQL API v4根据标准化点数（normalized scale of points）来计算速率限制分数（rate limit score）。每个 GraphQL 请求的分数，由父连接及其子节点上 的first 和 last 参数决定。

- 公式使用父连接及其子级上的 `first` 和 `last` 参数来预先计算GitHub系统（如MySQL，ElasticSearch和Git）上的潜在负载。
- 每个新连接都有自己的点数。这个点数与请求中的其他点数合并为一个总体的速率限制分数（rate limit score）。
- GraphQL API v4 速率限制是每小时5000点（与每小时5000个请求不同）。

###1.2.3 GitHub 如何获取接口的速率限制状态
使用 REST API v3 时，返回的 HTTP 头部带有速率限制状态。
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 56
X-RateLimit-Reset: 1372700873
```
|HTTP 头部名称| 解释|
|-|-|
|X-RateLimit-Limit|每小时最大调用次数|
|X-RateLimit-Remaining|当前周期剩余的调用次数|
|X-RateLimit-Reset|下个周期的开始时间|

使用 GraphQL API v4 时，可以通过查询 `rateLimit` 对象上的字段来检查速率限制状态：
```
// 请求
query {
  viewer {
    login
  }
  rateLimit {
    limit
    cost
    remaining
    resetAt
  }
}
// 响应
{
  "data": {
    "viewer": {
      "login": "kikajack"
    },
    "rateLimit": {
      "limit": 5000,
      "cost": 1,
      "remaining": 4999,
      "resetAt": "2018-01-21T04:42:30Z"
    }
  }
}
```
|rateLimit 中的属性|  解释|
|-|-|
|limit|每小时最大的可用点数|
|cost|本次接口调用的点数，该点数将与费率限制相比较|
|remaining|当前周期中剩余的点数|
|resetAt|下个周期的开始时间|
###1.2.4 GitHub 如何计算速率限制分数（rate limit score）
每次通过 `rateLimit` 查询速率限制分数时，也会算作一次查询。可以在运行之前计算速率限制分数：

- 累加所需要的请求数，使其可以覆盖请求中的所有唯一 connection。假设每个请求都会达到 first 或 last 参数的限制。
- 将数字除以100并四舍五入，得出最终的总成本。

下面例子的查询需要5,101个请求来完成：

- 虽然我们需要获取当前用户的前100个仓库，但 GraphQL API 只需要一次调用就可以获取到这100个仓库的列表。所以，请求个数是1。
- 虽然我们需要获取50个问题，但 GraphQL API 必须连接到100个仓库中的每一个以获取每个仓库的问题列表。所以，请求个数是100。
- 虽然我们需要获取60个标签，但 GraphQL API 必须连接到5000个潜在总问题中的每一个，以获取每个问题的标签列表。所以，对标签的请求个数是5000。
- 总计请求个数是 5101。除以100，四舍五入后查询的最后得分：51。
```
// 请求
query {
  viewer {
    login
    repositories(first: 100) {
      edges {
        node {
          id

          issues(first: 50) {
            edges {
              node {
                id

                labels(first: 60) {
                  edges {
                    node {
                      id
                      name
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
  rateLimit {
    limit
    cost
    remaining
    resetAt
  }
}
// 响应
{
  "data": {
    "viewer": {
      "login": "kikajack",
      "repositories": {
        "edges": [
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk1NjUxMjkyMw==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk2MDE4MTIxOQ==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk2MDE4MTY4OA==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk2MDUzNTc4Mw==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk2MDUzNjMzOQ==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk2MDUzNjU1OQ==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk2MDUzNjc4OA==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk2MDUzNzAyMQ==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk2MjU2NDUwNQ==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk2MjcxOTE0MQ==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk5NzU0NzM2MA==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnkxMDc0MTk5MzM=",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnkxMDc4NzcxNjg=",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnkxMDc5MTkyMDQ=",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnkxMDg4NTg3ODE=",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnkxMDg4NjE3ODA=",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnkxMTQ1MDY5NzA=",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnkxMTUxNjEwNzA=",
              "issues": {
                "edges": []
              }
            }
          }
        ]
      }
    },
    "rateLimit": {
      "limit": 5000,
      "cost": 51,
      "remaining": 4947,
      "resetAt": "2018-01-21T04:42:30Z"
    }
  }
}
```
#2. 分页
分页规则由 GraphQL 服务端实现，不同的系统会有所差异。
GraphQL 中经常会遍历对象集合的连接（connection）。比如上面的例子中遍历了仓库对象集合。
##2.1 分页相关的参数
分页参数是由服务端的具体的对象定义的，只有提供了对应的参数，才可以使用。常用的参数解释如下：
|参数|类型|解释|
|-|-|-|
|first| Int |返回列表 List 中的前 n 个元素|
|last|Int|返回列表 List 中的最后 n 个元素|
|after| String|返回列表 List 中的在给定的 global ID 之后的所有元素|
|before|String  |返回列表 List 中的在给定的 global ID 之前的所有元素|
|offset|Int |列表 List 中的游标偏移 n 个元素|
|orderBy|Input Object|列表 List 中的元素安装哪个字段排序，升序还是降序|
##2.2 基于游标的分页
通过为游标设置偏移或 ID 来实现的基于偏移或基于 ID 的分页是最强大的分页。
例如获取指定元素后面的头两个元素：`repositories(first: 2,after: "Y3Vyc29yOnYyOpHOA5ZK4w==") {...}`

游标是连接的属性，而不是对象的属性，所以不能放置在对象类型上。可以引入一个新的中间层 `edges`。`edges` 表示父对象的所有边，是一个列表 List。`edges` 列表中的每一个对象同时具有游标  `cursor` 和下层节点 `node`。`node` 就是真正的对象。例如 GitHub 中对 repositories 对象的查询：
```
// 请求
query {
  viewer {
    login
    repositories(first: 2) {
      edges {
        cursor
        node {
          id
          issues(first:2) {
            edges {
              node {
                id
              }
            }
          }
        }
      }
    }
  }
}
// 返回
{
  "data": {
    "viewer": {
      "login": "kikajack",
      "repositories": {
        "edges": [
          {
            "cursor": "Y3Vyc29yOnYyOpHOA15Rmw==",
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk1NjUxMjkyMw==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "cursor": "Y3Vyc29yOnYyOpHOA5ZK4w==",
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk2MDE4MTIxOQ==",
              "issues": {
                "edges": []
              }
            }
          }
        ]
      }
    }
  }
}
```
##2.3 列表的结尾、计数以及连接
GitHub 中的连接对象 `pageInfo` 具有边其中的字段以及其他信息（如总数量 `totalCount` 和下一页是否存在等信息）。通过这两个字段，可以只查询分页信息，或者通过中间层查询数据时同时判断下一页是否存在。

`totalCount` 表示当前查询对象的总数量。
`pageInfo` 中的四个字段的解释：
|参数|类型|解释|
|-|-|-|
|hasNextPage| Boolean |下一页是否存在|
|hasPreviousPage| Boolean |上一页是否存在|
|startCursor| String|当前页起始元素的 Cursor|
|endCursor| String|当前页结束元素的 Cursor|
```
query {
  viewer {
    login
    repositories(first: 2,after: "Y3Vyc29yOnYyOpHOA5ZK4w==") {
      totalCount
      pageInfo {
        hasNextPage
        endCursor
        startCursor
        hasPreviousPage
      }
      edges {        
        cursor
        node {
          id
          issues(first:2) {
            edges {
              node {
                id
              }
            }
          }
        }
      }
    }
  }
}
// 响应
{
  "data": {
    "viewer": {
      "login": "kikajack",
      "repositories": {
        "totalCount": 18,
        "pageInfo": {
          "hasNextPage": true,
          "endCursor": "Y3Vyc29yOnYyOpHOA5uz5w==",
          "startCursor": "Y3Vyc29yOnYyOpHOA5ZMuA==",
          "hasPreviousPage": true
        },
        "edges": [
          {
            "cursor": "Y3Vyc29yOnYyOpHOA5ZMuA==",
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk2MDE4MTY4OA==",
              "issues": {
                "edges": []
              }
            }
          },
          {
            "cursor": "Y3Vyc29yOnYyOpHOA5uz5w==",
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnk2MDUzNTc4Mw==",
              "issues": {
                "edges": []
              }
            }
          }
        ]
      }
    }
  }
}
```
##2.4 总结
GraphQL 的分页模型可以提供：

- 列表分页功能（通过参数 `first`,`after` 等）。
- 获取有关连接本身的信息的功能，（通过属性 `totalCount`，`pageInfo` 等）。
- 获取有关边本身的信息的功能，（通过 `edges` 的 `cursor` 等）。

#3. 缓存（基于 Global ID）
在 REST 等基于 URL 的 API 中，客户端可以根据 URL 使用 HTTP 缓存。这些 API 中的 URL 就是全局唯一标识符。而 GraphQL 是**单入口**的 API，无法用 URL 实现缓存。
GraphQL 中提供 Global ID 字段作为全局唯一标识符。可以将一个字段（如 id）保留为全局唯一标识符，例如`"id": "MDEwOlJlcG9zaXRvcnk2MDE4MTY4OA=="`。
如果后端使用类似 UUID 的标识符，那么直接暴露这个全局唯一 ID 即可。如果后端并没有给每个对象分配全局唯一 ID，则 GraphQL 层需要构造此 ID。
使用的时候需要注意：**GraphQL 构造出的 Global ID，不再是对象原先的那个 ID。**