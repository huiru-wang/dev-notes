
## RestAPI



# Important features of API Design
## 1. Idempotent

A well-designed API should be idempotent which means calling it multiple times doesn't change the result, and it should always return the same result.

>一个良好的API需要满足幂等性，这意味着多次调用同一个方法，都应该返回相同的结果

## 2. Pagination

Common queries also include limit and offset for pagination which allows users or the client to retrieve specific sets of data without overwhelming the system.

>通常查询方法需要包含limit和offset进行分页查询，以防止压垮系统
## 3. Backward Compatibility

When modifying endpoints, it's important to maintain backward compatibility.
A common practice is to introduce a new version of API.

> 修改API时，保持向后兼容性非常重要
> 一种常见的做法是引入新版本的 API
## 4. Rate Limitation

Rate Limitation is used to control the number of requests a user can make in a certain time. It prevents a single user from sending too many requests to a single API.
For endpoints with expected high traffic, set rate limits in advance to protect the system.

> 速率限制用于控制用户在一定时间内可以发出的请求数。它可以防止单个用户向单个 API 发送过多的请求
> 对于可预期的访问量较高的接口，提前做好速率限制，以保护系统
## 5. CORS
Cross-origin policies can control which domains can access your API, preventing unwanted cross-site interactions.

> 跨域策略可以控制哪些域可以访问您的 API，从而防止不必要的跨站点交互
