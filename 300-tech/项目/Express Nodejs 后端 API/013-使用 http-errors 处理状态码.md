---
title: 013-使用 http-errors 处理状态码
date: 2025-03-20T00:31:15+08:00
updated: 2025-03-24T09:47:45+08:00
dg-publish: false
---

之前已经写了一个 `errors.js` 工具类，但是错误状态码还有很多，一个个加有点麻烦，可以使用现有的库

<https://www.npmjs.com/package/http-errors>

| Status Code | Constructor Name              |
| ----------- | ----------------------------- |
| 400         | BadRequest                    |
| 401         | Unauthorized                  |
| 402         | PaymentRequired               |
| 403         | Forbidden                     |
| 404         | NotFound                      |
| 405         | MethodNotAllowed              |
| 406         | NotAcceptable                 |
| 407         | ProxyAuthenticationRequired   |
| 408         | RequestTimeout                |
| 409         | Conflict                      |
| 410         | Gone                          |
| 411         | LengthRequired                |
| 412         | PreconditionFailed            |
| 413         | PayloadTooLarge               |
| 414         | URITooLong                    |
| 415         | UnsupportedMediaType          |
| 416         | RangeNotSatisfiable           |
| 417         | ExpectationFailed             |
| 418         | ImATeapot                     |
| 421         | MisdirectedRequest            |
| 422         | UnprocessableEntity           |
| 423         | Locked                        |
| 424         | FailedDependency              |
| 425         | TooEarly                      |
| 426         | UpgradeRequired               |
| 428         | PreconditionRequired          |
| 429         | TooManyRequests               |
| 431         | RequestHeaderFieldsTooLarge   |
| 451         | UnavailableForLegalReasons    |
| 500         | InternalServerError           |
| 501         | NotImplemented                |
| 502         | BadGateway                    |
| 503         | ServiceUnavailable            |
| 504         | GatewayTimeout                |
| 505         | HTTPVersionNotSupported       |
| 506         | VariantAlsoNegotiates         |
| 507         | InsufficientStorage           |
| 508         | LoopDetected                  |
| 509         | BandwidthLimitExceeded        |
| 510         | NotExtended                   |
| 511         | NetworkAuthenticationRequired |

安装

```sh
npm install http-errors
```

之前的 `utils/errors.js` 就不需要了

直接修改 `utils/responses.js`

```js
const createError = require('http-errors')

/**
 * 请求成功
 * @param {*} res
 * @param {*} message
 * @param {*} data
 * @param {*} statusCode
 */
function success(res, message, data = {}, statusCode = 200) {
  res.status(statusCode).json({
    success: true,
    message,
    data
  })
}

/**
 * 请求失败
 * @param {*} res
 * @param {*} error
 */
function failure(res, error) {
  let statusCode = 500
  let errors = 'Server Error'

  if (error.name === 'SequelizeValidationError') {
    statusCode = 400
    errors = error.errors.map(err => err.message)
  } else if (error.name === 'JsonWebTokenError') {
    statusCode = 401
    errors = ['Token 无效']
  } else if (error.name === 'TokenExpiredError') {
    statusCode = 401
    errors = ['Token 已过期']
  } else if (error instanceof createError.HttpError) {
    statusCode = error.status
    message = error.name
    errors = error.message
  }

  res.status(statusCode).json({
    success: false,
    message: `请求失败: ${error.name}`,
    errors: Array.isArray(errors) ? errors : [errors]
  })
}

module.exports = {
  success,
  failure,
}
```

使用的时候，例如

```js
const { NotFound } = require('http-errors')

router.get('/:id', async function(req, res, next) {
  try {
    const { id } = req.params
    const article = await Article.findByPk(id)
    if (!article) {
      throw new NotFound(`ID 为 ${id} 的文章未找到`)
    }
    success(res, '查询文章成功', { article })
  } catch (error) {
    failure(res, error)
  }
})
```

其他的 name 也可以直接这样使用

了解：常用 HTTP 相应状态码

成功

- 200 请求成功
- 201 创建成功

重定向

- 301 永久重定向
- 307 临时重定向

客户端错误

- 400 错误请求 BadRequest
- 401 未认证 Unauthorized
- 403 没权限 Forbidden
- 404 找不到 NotFound
- 405 不允许的请求方法 MethodNotAllowed

服务端错误

- 500 服务器内部错误 InternalServerError
- 502 网关错误 BadGateway
