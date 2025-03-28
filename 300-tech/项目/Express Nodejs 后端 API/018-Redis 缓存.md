---
title: 018-Redis 缓存
date: 2025-03-21T10:56:39+08:00
updated: 2025-03-24T11:30:30+08:00
dg-publish: false
---

在高并发下，一个项目最先出问题的不是程序本身，而是数据库最先承受不住，在数据库层面可以做一些优化，例如优化 SQL 语句、优化索引、数据量大分库/分表等

项目中大部分是查询操作，可以在第一次查询之后将数据存储在某个地方，例如内存中，下次查询直接读取之前保存的，数据发生改变就将保存的数据删掉，重新保存最新的，这样可以大幅度减少数据库的查询，缓存就是这个原理

Redis 是一个开源的基于内存的数据存储系统，可以用作数据库、缓存和消息中间件，Redis 的数据中存储在内存中，这样它的读写速度就非常快，而且还会将内存中的数据写入本地硬盘，可以在服务器重启后自动将数据恢复到内存中

现在在项目中使用 Redis 的缓存能力

## 启动 Redis 服务

`docker-compose.yml`

```yml
services:
  mysql:
    image: hub.vonne.me/mysql:8.3.0
    # 下面这行是为了解决mysql8.0.23版本之后，默认的编码为utf8mb4，而express-admin-ui中默认使用的是utf8编码，所以需要修改为utf8mb4编码
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} # 设置 root 用户的密码
      - MYSQL_LOWER_CASE_TABLE_NAMES=0  # 默认表名不区分大小写，设置为0，则区分大小写
    ports:
      - "12306:3306"  # 映射端口，前面端口是宿主机端口，后面端口是容器端口
    volumes:
        - ./data/mysql:/var/lib/mysql  # 数据持久化
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 3
    profiles:
      - local
      - server

  redis:
    image: hub.vonne.me/redis:7.4
    ports:
      - "26379:6379"
    volumes:
      - ./data/redis:/data
    profiles:
      - local
      - server
      
  node-app:
    # 构建镜像，使用当前目录下的 Dockerfile
    build:
      context: .
      args:
        NODE_ENV: docker
    # 容器名称
    container_name: node-app-container
    # 端口映射，将容器的 3001 端口映射到主机的 3300 端口
    ports:
      - "3300:3001"
    # 重启策略，始终重启
    restart: always
    # 依赖于 mysql 服务
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_started
    profiles:
      - server
```

使用 profiles 当运行 `docker compose --profile local up -d` 就会运行含有 local 的服务，同理改成 server 就会运行带有 server 的服务

修改一下 `deploy.sh`

```sh
#!/bin/bash

# Set the path to the directory where the script is located
cd "$(dirname "$0")"
script_dir=$(pwd)
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color
echo -e "${GREEN}Deploying the application${NC}"
echo "==============================================="
echo ""

# build the docker image
echo -e "${GREEN}Building the docker image${NC}"
echo "------------------------------------------------"
docker-compose down
docker-compose build
echo ""

# 选择是否在本地运行
read -p "Do you want to run the docker container locally? (y/N): " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]
then
    echo -e "${GREEN}Running the docker container locally${NC}"
    echo "------------------------------------------------"
    docker compose --profile local up -d
else
    echo -e "${GREEN}Running the docker container on server${NC}"
    echo "------------------------------------------------"
    docker compose --profile server up -d
    
    # install dependencies
    echo -e "${GREEN}Installing dependencies${NC}"
    echo "------------------------------------------------"
    docker exec -it node-app-container bash -c "npm i dotenv sequelize-cli apidoc -g"

    # Check if database needs to be created
    read -p "Do you want to create the database? (y/N): " -n 1 -r
    echo    # move to a new line
    if [[ $REPLY =~ ^[Yy]$ ]]
    then
        echo -e "${GREEN}Creating the database${NC}"
        echo "------------------------------------------------"
        docker exec -it node-app-container bash -c "npx sequelize-cli db:create --charset utf8mb4 --collate utf8mb4_general_ci --env docker"  # 之前已经指定了环境变量，这里也可以不加 --env docker
    fi

    # Check if migrations need to be run
    read -p "Do you want to run migrations? (y/N): " -n 1 -r
    echo    # move to a new line
    if [[ $REPLY =~ ^[Yy]$ ]]
    then
        echo -e "${GREEN}Running migrations${NC}"
        echo "------------------------------------------------"
        docker exec -it node-app-container bash -c "npm run migrate --env docker"
    fi

    # Check if data needs to be seeded
    read -p "Do you want to seed data? (y/N): " -n 1 -r
    echo    # move to a new line
    if [[ $REPLY =~ ^[Yy]$ ]]
    then
        echo -e "${GREEN}Seeding data${NC}"
        echo "------------------------------------------------"
        docker exec -it node-app-container bash -c "npm run seed --env docker"
    fi

    echo -e "${GREEN}Generating API documentation${NC}"
    echo "------------------------------------------------"
    docker exec -it node-app-container bash -c "npm run docs"
    echo ""
fi
echo ""
```

本地运行 `./deploy.sh` 直接输入 `y` 即可启动 MySQL 和 Redis

## 安装 Redis 客户端

可以使用 Redis Insight

## 封装操作

<https://www.npmjs.com/package/redis>

安装

```sh
npm i redis
```

创建一个工具文件 `utils/redis.js`

```js
const { createClient } = require('redis')

let client

/**
 * 初始化 Redis 客户端
 */
const redisClient = async () => {
  if (client) return

  client = await createClient({
    url: process.env.REDIS_URL,
  })
    .on('error', (err) => { console.log(`Redis 连接失败: ${err}`) })
    .connect()
}

/**
 * 存入数组或对象，可选过期时间
 * @param {*} key 键名
 * @param {*} value 要存储的值
 * @param {*} ttl [可选] 过期时间，单位为秒
 */
const setKey = async (key, value, ttl = null) => {
  if (!client) await redisClient()

  value = JSON.stringify(value)
  await client.set(key, value)

  if (ttl !== null) {
    await client.expire(key, ttl)
  }
}

/**
 * 获取键值
 * @param {*} key 键名
 * @returns 键值
 */
const getKey = async (key) => {
  if (!client) await redisClient()

  const value = await client.get(key)
  return value ? JSON.parse(value) : null
}

/**
 * 删除键
 * @param {*} key 键名
 */
const delKey = async (key) => {
  if (!client) await redisClient()

  await client.del(key)
}

module.exports = {
  redisClient,
  setKey,
  getKey,
  delKey,
}
```

`.env` 中添加 `REDIS_URL="redis://127.0.0.1:26379"`

## 缓存策略和示例

哪些地方需要缓存：缓存一切

### 整体缓存

`routes/admin/settings.js` 和前台 `routes/settings.js`

查询的时候读取，没有值就查询数据库并设置到缓存中，有值就直接使用缓存中的值

```js
const { getKey, setKey } = require('../../utils/redis')

router.get('/', async function(req, res, next) {
  try {
    const cacheKey = 'setting'
    let setting = await getKey(cacheKey)

    if (!setting) {
      setting = await getSetting()
      await setKey(cacheKey, setting)
    }
    success(res, '查询系统设置详情成功', { setting })
  } catch (error) {
    failure(res, error)
  }
})
```

对 setting 进行修改的时候清除缓存

```js
await delKey(cacheKey)
```

`routes/categories.js` 前台查询分类列表可以直接缓存

```js
let categories = await getKey('categories')
if (!categories) {
	categories = await Category.findAll({
		order: [['rank', 'ASC'], ['id', 'ASC']]
	})
	await setKey('categories', categories)
}
```

`routes/admin/categories.js` 中涉及到修改的（创建、删除、更新）都需要清除缓存

```js
async function clearCache() {
  await delKey(cacheKey)
}

// 需要的地方调用
await clearCache()
```

### 利用过期时间设置缓存

不需要很好的时效性，修改很频繁的表

`routes/index.js` 设置过期时间 30s

```js
router.get('/', async function(req, res, next) {
  try {
    let data = await getKey('index')
    if (data) {
      return success(res, '查询首页数据成功', data)
    }

    const reCondition = {
      attributes: { exclude: ['CategoryId', 'UserId', 'content'] },
      include: [
        {
          model: Category,
          as: 'category',
          attributes: ['id', 'name']
        },
        {
          model: User,
          as: 'user',
          attributes: ['id', 'username', 'nickname', 'avatar', 'company']
        },
      ],
      where: {
        recommended: true
      },
      order: [['id', 'DESC']],
      limit: 10
    }
    const likesCondition = {
      attributes: { exclude: ['CategoryId', 'UserId', 'content'] },
      order: [['likesCount', 'DESC'], ['id', 'DESC']],
      limit: 10
    }
    const introductoryCondition = {
      attributes: { exclude: ['CategoryId', 'UserId', 'content'] },
      where: { introductory: true },
      order: [['id', 'DESC']],
      limit: 10
    }
    
    const [recommendCourses, likesCourses, introductoryCourses] = await Promise.all([
      Course.findAll(reCondition),
      Course.findAll(likesCondition),
      Course.findAll(introductoryCondition)
    ])

    data = {
      recommendCourses,
      likesCourses,
      introductoryCourses,
    }
    await setKey('index', data, 30 * 60)

    success(res, '查询首页数据成功', data)
  } catch (error) {
    failure(res, error)
  }
})
```

### 分页查询及查询详情 通配符

可以设置 key 为带有当前页和每页条数的例如 `artiles:${page}:${pageSize}`，详情可以直接加 id `article:${id}`

`routes/articles.js`

```js
router.get('/', async function(req, res, next) {
  try {
    const query = req.query
    const page = Math.abs(Number(query.page)) || 1
    const pageSize = Math.abs(Number(query.pageSize)) || 10
    const offset = (page - 1) * pageSize

    const cacheKey = `articles:${page}:${pageSize}`
    let data = await getKey(cacheKey)
    if (data) {
      return success(res, '查询文章列表成功', data)
    }

    const condition = {
      attributes: { exclude: ['content'] },
      order: [['id', 'DESC']],
      limit: pageSize,
      offset
    }
    const { rows: articles, count: total } = await Article.findAndCountAll(condition)
    data = {
      pagination: { page, pageSize, total },
      articles
    }
    await setKey(cacheKey, data)
    success(res, '查询文章列表成功', data)
  } catch (error) {
    failure(res, error)
  }
})
```

如果需要删除，得先把 `articles:*` 的都找出来，然后再删除，可以先创建函数

```js
/**
 * 获取匹配模式的所有键名
 * @param {*} pattern 匹配模式
 */
const getKeyByPattern = async (pattern) => {
  if (!client) await redisClient()

  return await client.keys(pattern)
}
```

导出后直接使用

```js
/**
 * 清除缓存
 */
async function clearCache() {
  const keys = await getKeyByPattern('articles:*')
  
  if (keys.length > 0) {
    await delKey(keys)
  }
}

// 需要用到的地方
await clearCache()
```

查询单条文章 `article:${id}`

```js
router.get('/:id', async function(req, res, next) {
  try {
    const { id } = req.params
    let article = await getKey(`article:${id}`)
    if (!article) {
      article = await Article.findByPk(id)
      if (!article) {
        throw new NotFound(`ID 为 ${id} 的文章未找到`)
      }
      await setKey(`article:${id}`, article)
    }
    success(res, '查询文章成功', { article })
  } catch (error) {
    failure(res, error)
  }
})
```

清除缓存需要在删除、恢复、更新单条文章中添加，并且需要加上对应的 id，改造之前的函数

```js
async function clearCache(id = null) {
  const keys = await getKeyByPattern('articles:*')
  
  if (keys.length > 0) {
    await delKey(keys)
  }

  if (id) {
    const keys = Array.isArray(id) ? id.map(i => `article:${i}`) : [`article:${id}`]
    await delKey(keys)
  }
}
```

删除、恢复、更新中加上

```js
await clearCache(id)
```

`routes/courses.js` 查询列表中多了一个 categoryId 直接加上

```js
router.get('/', async function(req, res, next) {
  try {
    const query = req.query;
    const page = Math.abs(Number(query.page)) || 1;
    const pageSize = Math.abs(Number(query.pageSize)) || 10;
    const offset = (page - 1) * pageSize;
    const categoryId = Number(query.categoryId)

    if (!categoryId) {
      throw new BadRequest('获取课程列表失败，分类 ID 不能为空')
    }

    const cacheKey = `courses:${categoryId}:${page}:${pageSize}`
    let data = await getKey(cacheKey)
    if (data) {
      return success(res, '查询课程列表成功', data)
    }

    const condition = {
      attributes: { exclude: ['CategoryId', 'UserId', 'content'] },
      where: { categoryId: query.categoryId },
      order: [['id', 'DESC']],
      limit: pageSize,
      offset
    }
    const { count: total, rows: courses } = await Course.findAndCountAll(condition)
    data = {
      courses,
      pagination: {
        total,
        page,
        pageSize
      }
    }
    await setKey(cacheKey, data)
    success(res, '查询课程列表成功', data)
  } catch (error) {
    failure(res, error)
  }
})
```

`routes/courses.js` 查询详情合并在一起清理缓存那么只是一个模型变了其他的也变，不太行，因此还需要分开

```js
router.get('/:id', async function(req, res, next) {
  try {
    const { id } = req.params

    let course = await getKey(`course:${id}`)
    if (!course) {
      const condition = {
        attributes: { exclude: ['CategoryId', 'UserId'] }
      }
      course = await Course.findByPk(id, condition)
      if (!course) {
        throw new NotFound(`ID 为 ${id} 的课程不存在`)
      }
      await setKey(`course:${id}`, course)
    }

    let category = await getKey(`category:${course.categoryId}`)
    if (!category) {
      category = await course.getCategory({
        attributes: ['id', 'name']
      })
      await setKey(`category:${course.categoryId}`, category)
    }

    let user = await getKey(`user:${course.userId}`)
    if (!user) {
      user = await User.findByPk(course.userId, {
        attributes: { exclude: ['password'] }
      })
      await setKey(`user:${course.userId}`, user)
    }

    let chapters = await getKey(`chapters:${course.id}`)
    if (!chapters) {
      chapters = await Chapter.findAll({
        attributes: { exclude: ['CourseId', 'content'] },
        where: { courseId: course.id },
        order: [['rank', 'ASC'], ['id', 'DESC']]
      })
      await setKey(`chapters:${course.id}`, chapters)
    }

    success(res, '查询课程详情成功', { course, category, user, chapters })
  } catch (error) {
    failure(res, error)
  }
})
```

`routes/chapters.js` 查询章节详情同样

```js
router.get('/:id', async function(req, res, next) {
  try {
    const { id } = req.params
    let chapter = await getKey(`chapter:${id}`)
    if (!chapter) {
      chapter = await Chapter.findByPk(id, {
        attributes: { exclude: ['CourseId'] }
      })
      if (!chapter) {
        throw new NotFound(`ID 为 ${id} 的章节不存在`)
      }
      await setKey(`chapter:${id}`, chapter)
    }

    let course = await getKey(`course:${chapter.courseId}`)
    if (!course) {
      course = await Chapter.findByPk(chapter.courseId, {
        attributes: { exclude: ['CourseId', 'UserId'] }
      })
      await setKey(`course:${chapter.courseId}`, course)
    }

    let user = await getKey(`user:${course.userId}`)
    if (!user) {
      user = await User.findByPk(course.userId, {
        attributes: { exclude: ['password'] }
      })
      await setKey(`user:${course.userId}`, user)
    }

    let chapters = await getKey(`chapters:${course.id}`)
    if (!chapters) {
      chapters = await Chapter.findAll({
        attributes: { exclude: ['CourseId', 'content'] },
        where: { courseId: course.id },
        order: [['rank', 'ASC'], ['id', 'DESC']]
      })
      await setKey(`chapters:${course.id}`, chapters)
    }

    success(res, '查询章节成功', { chapter, course, user, chapters })
  } catch (error) {
    failure(res, error)
  }
})
```

`routes/users.js`

```js
router.get('/me', async function(req, res, next) {
  try {
    let user = await getKey(`user:${req.userId}`)
    if (!user) {
      user = await getUser(req)
      await setKey(`user:${req.userId}`, user)
    }
    success(res, '查询当前用户信息成功', { user })
  } catch (error) {
    failure(res, error)
  }
})
```

清除缓存

```js
router.put('/info', async function(req, res, next) {
  try {
    const body = {
      nickname: req.body.nickname,
      sex: req.body.sex,
      company: req.body.company,
      introduce: req.body.introduce,
      avatar: req.body.avatar,
    }

    const user = await getUser(req)
    await user.update(body)
    await clearCache(user)

    success(res, '更新用户信息成功', { user })
  } catch (error) {
    failure(res, error)
  }
})

router.put('/account', async function(req, res, next) {
  try {
    const body = {
      email: req.body.email,
      username: req.body.username,
      current_password: req.body.current_password,
      password: req.body.password,
      password_confirmation: req.body.password_confirmation
    }

    if (!body.current_password) {
      throw new BadRequest('请输入当前密码')
    }
    if (!body.password) {
      throw new BadRequest('请输入新密码')
    }
    if (body.password !== body.password_confirmation) {
      throw new BadRequest('两次输入的密码不一致')
    }

    const user = await getUser(req, true)

    const isPasswordValid = bcrypt.compareSync(body.current_password, user.password)
    if (!isPasswordValid) {
      throw new BadRequest('当前密码不正确')
    }
    if (bcrypt.compareSync(body.password, user.password)) {
      throw new BadRequest('新密码不能与当前密码相同')
    }

    await user.update(body)
    delete user.dataValues.password
    await clearCache(user)
    
    success(res, '更新用户账号成功', { user })
  } catch (error) {
    failure(res, error)
  }
})

async function clearCache(user) {
  await delKey(`user:${user.id}`)
}
```

`routes/admin/users.js`

```js
router.put('/:id', async function(req, res, next) {
  try {
    const body = filterBody(req)
    const user = await getUser(req)

    await user.update(body)
    await clearCache(user)
    
    success(res, '更新用户成功', { user })
  } catch (error) {
    failure(res, error)
  }
})

async function clearCache(user) {
  await delKey(`user:${user.id}`)
}
```

修改之前的 `routes/admin/categories.js`

```js
async function clearCache(category = null) {
  await delKey(cacheKey)

  if (category) {
    await delKey(`category:${category.id}`)
  }
}
```

更新和删除分类都需要修改成

```js
await clearCache(category)
```

`routes/admin/courses.js`

```js
async function clearCache(course = null) {
  let keys = await getKeyByPattern('courses:*')
  if (keys.length > 0) {
    await delKey(keys)
  }

  if (course) {
    await delKey(`course:${course.id}`)
  }
}

// 创建课程
const course = await Course.create(body)
await clearCache()

// 更新和删除
await course.update(body)
// await course.destroy()
await clearCache(course)
```

`routes/admin/chapters.js`

```js
async function clearCache(chapter) {
  await delKey(`chapters:${chapter.courseId}`)
  await delKey(`chapter:${chapter.id}`)
}

// 创建、更新、删除章节
await clearCache(chapter)
```

## 清空所有缓存

`utils/redis.js` 中添加并导出

```js
const flushAll = async () => {
  if (!client) await redisClient()

  await client.flushAll()
}
```

`routes/admin/settings.js`

```js
router.get('/flush-all', async function(req, res, next) {
  try {
    await flushAll()
    success(res, '清除所有缓存成功', null)
  } catch (error) {
    failure(res, error)
  }
})
```
