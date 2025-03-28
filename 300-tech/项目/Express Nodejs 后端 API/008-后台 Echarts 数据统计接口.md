---
title: 008-后台 Echarts 数据统计接口
date: 2025-03-17T14:00:16+08:00
updated: 2025-03-17T15:24:39+08:00
dg-publish: false
---

<https://echarts.apache.org/zh/index.html>

可以看它示例和文档，根据形式返回对应的数据

例如用户的数量统计、用户性别统计、订单统计、销售额统计等，可以使用图标进行展示，这样管理员可以对项目的运营情况有一个直观的掌控

## 用户性别统计

适合使用饼图，先挑选一个示例，查看数据格式

```js
data: [
	{ value: 1048, name: 'Search Engine' },
	{ value: 735, name: 'Direct' },
	{ value: 580, name: 'Email' },
	{ value: 484, name: 'Union Ads' },
	{ value: 300, name: 'Video Ads' }
]
```

那我们返回的数据格式就应该是

```js
data: [
	{ value: 2, name: '男性' },
	{ value: 1, name: '女性' },
	{ value: 1, name: '未选择性别' }
]
```

### 新建路由

`routes/admin/charts.js`

```js
const express = require('express')
const router = express.Router()
const { User } = require('../../models')
const {
  success,
  failure
} = require('../../utils/response')

router.get('/sex', async function(req, res, next) {
  try {
    const male = await User.count({ where: { sex: '0' } })
    const female = await User.count({ where: { sex: '1' } })
    const unknown = await User.count({ where: { sex: '2' } })
    const data = [
      { name: '男', value: male },
      { name: '女', value: female },
      { name: '未知', value: unknown }
    ]
    success(res, '查询用户性别成功', { data })
  } catch (error) {
    failure(res, error)
  }
})

module.exports = router
```

`app.js`

```js
const adminChartsRouter = require('./routes/admin/charts')

app.use('/admin/charts', adminChartsRouter)
```

## 用户数量统计

每月用户注册数量，适合使用折线图

<https://echarts.apache.org/examples/zh/editor.html?c=line-simple>

```js
option = {
  xAxis: {
    type: 'category',
    data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']  // m
  },
  yAxis: {
    type: 'value'
  },
  series: [
    {
      data: [150, 230, 224, 218, 135, 147, 260], // v
      type: 'line'
    }
  ]
};
```

提供的数据

```js
{
	months: ['2024-04', '2024-05'],
	values: [1, 3]
}
```

### 分组查询

使用到分组查询

```sql
SELECT DATE_FORMAT(`createdAt`, '%Y-%m') AS 'month',
COUNT(*) AS `value`
FROM `Users`
GROUP BY `month`
ORDER BY `month` ASC
```

对于这种复杂的在 ORM 没有现成的指令的情况，在代码中可以使用 `sequelize.query` 执行 SQL 语句

### 新建路由

```js
const { sequelize, User } = require('../../models')

/**
 * @api {get} /admin/charts/user 查询每月用户注册数
 */
router.get('/user', async function(req, res, next) {
  try {
    const [results] = await sequelize.query("SELECT DATE_FORMAT(`createdAt`, '%Y-%m') AS 'month', COUNT(*) AS `value` FROM `Users` GROUP BY `month` ORDER BY `month` ASC")
    const data = {
      months: [],
      values: []
    }
    results.forEach(item => {
      data.months.push(item.month)
      data.values.push(item.value)
    })
    success(res, '查询每月用户数量成功', { data })
  } catch (error) {
    failure(res, error)
  }
})
```

需要更多路由可以根据上述步骤慢慢实现
