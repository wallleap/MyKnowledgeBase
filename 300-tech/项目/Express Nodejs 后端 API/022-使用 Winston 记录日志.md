---
title: 022-使用 Winston 记录日志
date: 2025-03-21T10:57:51+08:00
updated: 2025-03-26T10:21:31+08:00
dg-publish: false
---

在本地开发可以随时观察命令行中的错误，可以通过 log 调试代码

部署上线如何得知是否有错误，程序崩溃了如何定位

## 使用 winston

```sh
npm install winston
```

文件 `utils/logger.js`

```js
const winston = require('winston')

const logger = winston.createLogger({
  level: 'info',  // info > warn > error，低于该级别的日志不会输出
  format: winston.format.combine(
    winston.format.errors({ stack: true }),  // 错误信息包含堆栈信息
    winston.format.timestamp(),  // 时间戳
    winston.format.json()  // 输出的格式
  ),
  defaultMeta: { service: 'express-api' },  // 预设的元数据
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
})

// 如果不是生产环境，则将日志输出到控制台
if (process.env.NODE_ENV !== 'production' || process.env.NODE_ENV !== 'docker') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple()
    )
  }))
}

module.exports = logger
```

## 存储到数据库

创建迁移文件

```sh
sequelize model:generate --name Log --attributes level:string,message:string,meta:string,timestamp:date
```

修改文件

```js
'use strict';
/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('Logs', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER.UNSIGNED
      },
      level: {
        allowNull: false,
        type: Sequelize.STRING(16)
      },
      message: {
        allowNull: false,
        type: Sequelize.STRING(2048)
      },
      meta: {
        allowNull: false,
        type: Sequelize.STRING(2048)
      },
      timestamp: {
        type: Sequelize.DATE
      }
    });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('Logs');
  }
};
```

运行迁移

```sh
sequelize db:migrate
```

修改模型文件

```js
'use strict';
const {
  Model
} = require('sequelize');
module.exports = (sequelize, DataTypes) => {
  class Log extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate(models) {
      // define association here
    }
  }
  Log.init({
    level: DataTypes.STRING,
    message: DataTypes.STRING,
    meta: {
      type: DataTypes.JSON,
      get() {
        return JSON.parse(this.getDataValue('meta'));
      }
    },
    timestamp: DataTypes.DATE
  }, {
    sequelize,
    modelName: 'Log',
    timestamps: false  // 禁用默认的创建时间戳和修改时间戳
  });
  return Log;
};
```

`utils/logger.js` 中连接数据库并自动新增记录

```js
const winston = require('winston')
const Transport = require('winston-transport')
const mysql = require('mysql2/promise')

class MySQLTransport extends Transport {
  constructor(opts) {
    super(opts)

    if (typeof opts !== 'object') {
      throw new Error('MySQLTransport expects options')
    }
    if (!opts.host || !opts.user || !opts.database) {
      throw new Error('MySQLTransport expects options.host, options.user, and options.database')
    }

    this.pool = mysql.createPool({
      host: opts.host,
      user: opts.user,
      password: opts.password || '',
      database: opts.database,
      port: opts.port || 3306
    })
    this.table = opts.table || 'logs'
  }

  async log(info, callback) {
    setImmediate(() => {
      this.emit('logged', info)
    })

    let { level, message, timestamp, ...meta } = info
    level = level ?? 'info'
    message = message ?? ''
    timestamp = timestamp ?? new Date()
    meta = meta ?? {}
    meta = JSON.stringify(meta)

    if (!this.pool) {
      console.error('MySQL pool is not initialized')
      return callback()
    }

    const sql = `INSERT INTO ${this.table} (level, message, timestamp, meta) VALUES (?, ?, ?, ?)`
    const values = [level, message, timestamp, meta]

    try {
      await this.pool.execute(sql, values)
    } catch (err) {
      console.error('Error inserting log into MySQL:', err)
    }

    callback()
  }
}

const env = process.env.NODE_ENV || 'development'
const config = require(__dirname + '/../config/config.json')[env]
const options = {
  host: config.host,
  user: config.username,
  password: config.password,
  port: config.port,
  database: config.database,
  table: 'logs'
}

const logger = winston.createLogger({
  level: 'info',  // info > warn > error，低于该级别的日志不会输出
  format: winston.format.combine(
    winston.format.errors({ stack: true }),  // 错误信息包含堆栈信息
    winston.format.json()  // 输出的格式
  ),
  defaultMeta: { service: 'express-api' },  // 预设的元数据
  transports: [
    new MySQLTransport(options)
  ]
})

// 如果不是生产环境，则将日志输出到控制台
if (process.env.NODE_ENV !== 'production' || process.env.NODE_ENV !== 'docker') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple()
    )
  }))
}

logger.info('Logger initialized')
logger.warn('Logger initialized')
logger.error('Logger initialized')

module.exports = logger
```

其他地方使用，例如 `utils/responses.js`

```js
const createError = require('http-errors')
const multer = require('multer')
const logger = require('./logger')

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
  let statusCode
  let errors

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
  } else if (error instanceof multer.MulterError) {
    if (error.code === 'LIMIT_FILE_SIZE') {
      statusCode = 413
      errors = '文件大小超出限制'
    } else {
      statusCode = 400
      errors = error.message
    }
  } else {
    statusCode = 500
    errors = '服务器内部错误'
    logger.error('服务器内部错误', error)
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

`utils/redis.js`、`utils/rabbit-mq.js`、`utils/mail.js` 中 log 使用 `logger.error`

`app.js` 中去掉 `console.log`

## 管理员管理日志接口

`routes/admin/logs.js`

```js
const express = require('express')
const router = express.Router()
const { Log } = require('../../models')
const { NotFound } = require('http-errors')
const { success, failure } = require('../../utils/responses')

/**
 * @api {get} /admin/logs 查询日志列表
 * @apiDescription 查询日志列表
 * @apiName GetLogs
 * @apiGroup 后台接口-日志
 * 
 * @apiHeader {String} Authorization 用户授权 token
 * 
 * @apiSuccessExample {json} 请求成功
 *    HTTP/1.1 200 OK
 *    {
        "success": true,
        "message": "查询日志列表成功",
        "data": {
            "logs": [
                {
                    "meta": {
                        "service": "express-api",
                        "stack": "Error: 测试错误\n    at /Users/luwang/Workspace/code/Server/express-api/routes/index.js:187:11\n    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)"
                    },
                    "id": 2,
                    "level": "error",
                    "message": "服务器内部错误 测试错误",
                    "timestamp": "2025-03-26T02:06:08.000Z"
                },
                {
                    "meta": {
                        "service": "express-api",
                        "stack": "ReferenceError: Course1 is not defined\n    at /Users/luwang/Workspace/code/Server/express-api/routes/index.js:175:7\n    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)"
                    },
                    "id": 1,
                    "level": "error",
                    "message": "服务器内部错误 Course1 is not defined",
                    "timestamp": "2025-03-26T02:05:19.000Z"
                }
            ]
        }
    }
 */
router.get('/', async (req, res) => {
  try {
    const logs = await Log.findAll({
      order: [['id', 'DESC']]
    })
    
    success(res, '查询日志列表成功', { logs })
  } catch (err) {
    failure(res, err)
  }
})

/**
 * @api {get} /admin/logs/:id 查询日志
 * @apiDescription 查询日志
 * @apiName GetLog
 * @apiGroup 后台接口-日志
 * 
 * @apiHeader {String} Authorization 用户授权 token
 * 
 * @apiParam {Number} id 日志 ID
 * 
 * @apiSuccessExample {json} 请求成功
 *    HTTP/1.1 200 OK
 *    {
        "success": true,
        "message": "查询日志成功",
        "data": {
            "log": {
                "meta": {
                    "service": "express-api",
                    "stack": "ReferenceError: Course1 is not defined\n    at /Users/luwang/Workspace/code/Server/express-api/routes/index.js:175:7\n    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)"
                },
                "id": 1,
                "level": "error",
                "message": "服务器内部错误 Course1 is not defined",
                "timestamp": "2025-03-26T02:05:19.000Z"
            }
        }
    }
 */
router.get('/:id', async (req, res) => {
  try {
    const log = await Log.findByPk(req.params.id)
    
    success(res, '查询日志成功', { log })
  } catch (err) {
    failure(res, err)
  }
})

/**
 * @api {delete} /admin/logs/:id 删除日志
 * @apiDescription 删除日志
 * @apiName DeleteLog
 * @apiGroup 后台接口-日志
 * 
 * @apiHeader {String} Authorization 用户授权 token
 * 
 * @apiParam {Number} id 日志 ID
 * 
 * @apiSuccessExample 请求成功
 *    HTTP/1.1 200 OK
 *    {
        "success": true,
        "message": "删除日志成功",
        "data": null
    }
 */
router.delete('/:id', async (req, res) => {
  try {
    const log = await Log.findByPk(req.params.id)
    if (!log) {
      throw new NotFound('日志不存在')
    }
    await log.destroy()
    
    success(res, '删除日志成功', null)
  } catch (err) {
    failure(res, err)
  }
})

/**
 * @api {delete} /admin/logs 删除所有日志
 * @apiDescription 删除所有日志
 * @apiName DeleteLogs
 * @apiGroup 后台接口-日志
 * 
 * @apiSuccessExample 请求成功
 *    HTTP/1.1 200 OK
 *    {
        "success": true,
        "message": "删除所有日志成功",
        "data": null
    }
 */
router.delete('/', async (req, res) => {
  try {
    await Log.destroy({
      where: {},
      truncate: true
    })
    
    success(res, '删除所有日志成功', null)
  } catch (err) {
    failure(res, err)
  }
})

module.exports = router
```

`app.js`

```js
const adminLogsRouter = require('./routes/admin/logs')

app.use('/admin/logs', adminAuth, adminLogsRouter)
```
