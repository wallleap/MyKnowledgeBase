---
title: 在 Rails 项目中使用 Docker 和 GitLab CI 高效构建镜像
date: 2024-08-16T11:15:00+08:00
updated: 2024-08-21T10:32:32+08:00
dg-publish: false
source: //ruby-china.org/topics/38766
---

> 原文地址：[https://blog.wildcat.io/2019/06/rails-with-docker-part-1-zh/](https://blog.wildcat.io/2019/06/rails-with-docker-part-1-zh/)，英文版：[https://blog.wildcat.io/2019/06/rails-with-docker-part-1/](https://blog.wildcat.io/2019/06/rails-with-docker-part-1/)。转载请注明原文出处。

## 背景

Docker 是令人惊艳的 DevOps 工具，通过使用它可以节省大把时间，即使对个人项目来说也是如此，然而，把 Rails 项目有效地迁移到 Docker 技术栈上并不是一件容易的事情。通常我们会遇到如下问题：

- 最终镜像体积过大。
- 构建时间太长。安装项目依赖时会首先构建很多工具和库，这会消耗很多时间。
- 不太容易复用 Docker 景象来进行测试。

在这片文章里，你可以看到一个逐步 Docker 化 Rails 项目的比较完整流程，基于 GitLab CI。这个教程，对于 Rails 的 DevOps 来说，可能不是百分百完美。但是我认为这可以是一个不错的开始 😊。

## 文章太长，不想看怎么办？

如果你非常熟悉 Rails 和 Docker，或者你甚至尝试了多次这个技术栈的话，请直接跳过第一和第二节，或者直接前往 GitHub ([https://github.com/imWildCat/rails-docker-demo](https://github.com/imWildCat/rails-docker-demo)) 或者 GitLab 仓库 ([https://gitlab.com/imWildCat/docker-rails-demo](https://gitlab.com/imWildCat/docker-rails-demo))。如果你对 Docker 和 Gitlab CI 有丰富的经验，直接阅读问末尾总结部分也是一个不错的选择。

## 第 1 节：开始使用 Rails 和 Docker

> 请注意，Rails 6.0.0 稳定版现在还没发布 (2019-06-29)。请使用这个命令 `gem install --pre rails` 安装 rc1 版本。

从一个简单而且干净的项目开始总是极好的。我们可以使用下面的命令来创建一个 Rails 项目：

```
rails new -d postgresql --webpack react rails_docker_demo
```

在这个阶段，我们不需要写 Rails 代码。直接进入 Docker 部分吧。

所以，`Dockerfile` 应该是什么样子呢？

```
FROM ruby:2.6.3-alpine3.8

# Install alpine packages
RUN apk add --no-cache \
  build-base \
  busybox \
  ca-certificates \
  cmake \
  curl \
  git \
  tzdata \
  gnupg1 \
  graphicsmagick \
  libffi-dev \
  libsodium-dev \
  nodejs \
  yarn \
  openssh-client \
  postgresql-dev \
  tzdata

# Define WORKDIR
WORKDIR /app

# Use bunlder to avoid exit with code 1 bugs while doing integration test
RUN gem install bundler -v 2 --no-doc

# Copy dependency manifest
COPY Gemfile Gemfile.lock /app/

# Install Ruby dependencies
RUN bundle update --bundler
RUN bundle install --jobs $(nproc) --retry 3 --without development test \
      && rm -rf /usr/local/bundle/bundler/gems/*/.git /usr/local/bundle/cache/

# Copy JavaScript dependencies
COPY package.json yarn.lock /app/

# Install JavaScript dependencies
RUN yarn install

# Define basic environment variables
ENV NODE_ENV production
ENV RAILS_ENV production
ENV RAILS_LOG_TO_STDOUT true

# Copy source code
COPY . /app/

# Build front-end assets
RUN bundle exec rails webpacker:verify_install
RUN SECRET_KEY_BASE=nein bundle exec rails assets:precompile

RUN chmod +x ./bin/entrypoint.sh

# Define entrypoint
ENTRYPOINT ["./bin/entrypoint.sh"]
```

你也可以在项目仓库的 `v1-base-case` 下（[GitHub](https://github.com/imWildCat/rails-docker-demo/tree/v1-base-case) 或者 [GitLab](https://gitlab.com/imWildCat/docker-rails-demo/tree/v1-base-case)），找到 `bin/entrypoint.sh` 的内容。

现在我们来看 `.gitlab-ci.yml`:

```
variables:
  # Prevent any locale errors
  LC_ALL: C.UTF-8
  LANG: en_US.UTF-8
  LANGUAGE: en_US.UTF-8

build_image:
  stage: build
  image: docker:stable
  services:
    - docker:dind
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE:latest
  before_script:
    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY
  script:
    - docker pull $IMAGE_TAG || echo "No pre-built image found."
    - docker build --cache-from $IMAGE_TAG -t $IMAGE_TAG . || docker build -t $IMAGE_TAG . # Use cache for building if possible
    - docker push $IMAGE_TAG
```

这个 CI 配置文件使用 `--cahce-from` 做了一些缓存，但是它没有特别大的作用。一旦我们修改了 alpine 的依赖库，整个流程都会被重新构建。在改进缓存之前，我们不如先部署这个 Rails 镜像试试。到目前位置，构建好的 Docker 镜像还不能被直接部署。

> 构建时间：
>
> - Rails 项目：5 分 51 秒（从零开始，估计时间)

有关这部分的详细改动，请阅读这个 Merge Request：[https://gitlab.com/imWildCat/docker-rails-demo/merge_requests/1](https://gitlab.com/imWildCat/docker-rails-demo/merge_requests/1).

### 第 1.1 节：关于部署的改进

对于部署来说，我们都需要做哪些工作呢？

- 一个反向代理，比如 nginx
- 数据库配置
- 密钥或者 `master.key`
- （可选）任务队列，比如 sidekiq

因为这个话题不是很契合这篇文章的主题，我想用一个 PR 来展示改动：

- [https://gitlab.com/imWildCat/docker-rails-demo/merge_requests/3](https://gitlab.com/imWildCat/docker-rails-demo/merge_requests/3)

部署大概步骤如下：

- 下载并修改 `docker-compose.yml` 到你的宿主机。
- （可选）把 `master.key` 移动到宿主机的指定位置（`docker-compose.yml` 有设定）。
- 拉取镜像：`sudo docker-compose pull`。
- 启动服务：`sudo docker-compose up`。
- （可选）在停止服务后，你可以执行 `sudo docker-compose down` 删除之前创建的所有资源。

我们也可以了解以下内容：

- 我们也可以对 Rails 服务设置 `SECRET_KEY_BASE` 这个环境变量。
- nginx 服务器的端口可以被暴露出来，或者也可以使用宿主机级别的反向代理比如 Traefik。

> 构建时间：
>
> - Rails 项目：5:09（从零开始，估算时间），1:47（使用缓存，估算时间）
> - 反向代理：0:43（估算时间）

在部署之后，你会看到 404 页面。因为我们没有写任何业务逻辑，所以这没问题。

## 第 2 节：Multi-stage 构建

在前一节构建的景象里，有一个非常明显的问题，就是镜像太大了（大概 617MB）。所以，我们可以做些什么来减小这个 Docker 镜像的体积呢？

首先，让我们来看看 `Dockerfile`。这里大概有三部分：

- 安装系统依赖
- 根据 `Gemfile` 和 `package.json` 安装项目依赖
- 构建前端资源，设置基础环境

不错的事情是，我们可以利用 Docker 的 [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)。在这个教程里，我不会太深入探讨这个特性，但是它有很大的潜力，值得探索。

在这个章节里，我们需要把原来的 `Dockerfile` 分为两部分：

- `Dockerfile.builder`：安装依赖
- `Dockerfile`：构建最终 Docker 镜像

我们应该把 `Dockerfile` 里在最终构建 Rails 应用之前的代码移动到 `Dockerfile.builder` 里：

```
# Stage for dependencies installation
FROM ruby:2.6.3-alpine3.8 as builder

# Install alpine packages
RUN apk add --no-cache \
  build-base \
  busybox \
  ca-certificates \
  cmake \
  curl \
  git \
  tzdata \
  gnupg1 \
  graphicsmagick \
  libffi-dev \
  libsodium-dev \
  nodejs \
  yarn \
  openssh-client \
  postgresql-dev \
  tzdata

# Define WORKDIR
WORKDIR /app

# Use bunlder to avoid exit with code 1 bugs while doing integration test
RUN gem install bundler -v 2 --no-doc

# Copy dependency manifest
COPY Gemfile Gemfile.lock /app/

# Install Ruby dependencies
RUN bundle update --bundler
RUN bundle install --jobs $(nproc) --retry 3 --without development test

# Copy JavaScript dependencies
COPY package.json yarn.lock /app/

# Install JavaScript dependencies
RUN yarn install
```

我们也需要更新 `Dockerfile`，使用两个 stages，由此我们可以减小最终的镜像体积。

```
# ARG: https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact
ARG BUILDER_IMAGE_TAG
FROM $BUILDER_IMAGE_TAG as builder

# Define basic environment variables
ENV NODE_ENV production
ENV RAILS_ENV production
ENV RAILS_LOG_TO_STDOUT true

# Copy source code
COPY . /app/

# Build front-end assets
RUN bundle exec rails webpacker:verify_install
RUN SECRET_KEY_BASE=nein bundle exec rails assets:precompile

RUN rm -rf node_modules

FROM ruby:2.6.3-alpine3.8 as deploy

RUN apk add --no-cache \
  ca-certificates \
  curl \
  tzdata \
  gnupg1 \
  graphicsmagick \
  libsodium-dev \
  nodejs \
  postgresql-dev \
  bash

# Define basic environment variables
ENV NODE_ENV production
ENV RAILS_ENV production
ENV RAILS_LOG_TO_STDOUT true
# Defined for future testing
ENV RAILS_SERVE_STATIC_FILES true

WORKDIR /var/www/app

COPY --from=builder /usr/local/bundle/ /usr/local/bundle/
COPY --from=builder /app/ /var/www/app/
# We will copy the files in to /app/public while app is starting.
# Otherwise, the asset files may not be updated if we use named volume.
COPY --from=builder /app/public /var/www/app/public_temp

RUN chmod +x ./bin/entrypoint.sh

# Define entrypoint
ENTRYPOINT ["./bin/entrypoint.sh"]
```

然后，我们也需要更新 CI 的配置 `.gitlab-ci.yml`：

```
# ...
build_image:
  stage: build
  image: docker:stable
  services:
    - docker:dind
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
    BUILDER_IMAGE_TAG: $CI_REGISTRY_IMAGE/builder:latest
  before_script:
    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY
  script:
    - docker pull $BUILDER_IMAGE_TAG || echo "No pre-built image found."
    - docker build --cache-from $BUILDER_IMAGE_TAG -t $BUILDER_IMAGE_TAG -f Dockerfile.builder . || docker build -t $BUILDER_IMAGE_TAG -f Dockerfile.builder . 
    - docker push $BUILDER_IMAGE_TAG
    - docker pull $IMAGE_TAG || echo "No pre-built image found."
    - docker build --cache-from $IMAGE_TAG --build-arg BUILDER_IMAGE_TAG=${BUILDER_IMAGE_TAG} -t $IMAGE_TAG . || docker build --build-arg BUILDER_IMAGE_TAG=${BUILDER_IMAGE_TAG} -t $IMAGE_TAG . # Use cache for building if possible
    - docker push $IMAGE_TAG
# ...
```

请注意我们在这里使用了：[build-time variables (--build-arg)](https://docs.docker.com/engine/reference/commandline/build/#set-build-time-variables---build-arg)。

另外，在 `bin/entrypoint.sh` 里也用了一点小技巧来处理静态资源：

```
rm -rf public/* # Remove assets in named volume
cp -r public_temp/* public/ # Copy new files from new image
```

通过这些改进，我们可以把最终镜像大小大概从 617MB 减小到 **221MB** (**64.1%**)。时间消耗大致相同。

关于细节，可以阅读这个 Merge Request: [https://gitlab.com/imWildCat/docker-rails-demo/merge_requests/1](https://gitlab.com/imWildCat/docker-rails-demo/merge_requests/4).

## 第 3 节：按需的 multi-stage 构建

在引入 multi-stage 构建后，你可能想知道，我们是否可以只在有必要的情况下构建 builder。因为对于绝大多数 commits 和 PRs 来说，我们不会改动依赖。只有业务逻辑是经常变动的。我们可以利用 GitLab CI 的 `only` 配置（[https://docs.gitlab.com/ee/ci/yaml/#onlyexcept-basic](https://docs.gitlab.com/ee/ci/yaml/#onlyexcept-basic)）和‘stage’特性。他们非常好用。

首先，我们可以在 `.gitlab-ci.yml` 文件头部添加 `stages` 设定。

```
stages:
  - prebuild
  # - test # For future work
  - build
  # - deploy # For future work
```

然后，builder 的构建逻辑需要被移动到一个新的 stage `prebuild` 里：

```
construct_builder:
  stage: prebuild
  image: docker:stable
  services:
    - docker:dind
  only:
    changes:
      - Dockerfile
      - Dockerfile.builder
      - Gemfile
      - Gemfile.lock
      - package.json
      - yarn.lock
  variables:
    BUILDER_IMAGE_TAG: $CI_REGISTRY_IMAGE/builder:latest
  before_script:
    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY
  script:
    - docker pull $BUILDER_IMAGE_TAG || echo "No pre-built image found."
    - docker build --cache-from $BUILDER_IMAGE_TAG -t $BUILDER_IMAGE_TAG -f Dockerfile.builder . || docker build -t $BUILDER_IMAGE_TAG -f Dockerfile.builder . 
    - docker push $BUILDER_IMAGE_TAG
```

请注意，为了简单，我们这里继续使用 `latest` 标签。你可能需要根据需求改动它。

所以我们也可以继续简化 `build_image`：

```
build_image:
  # ...
  script:
    - docker pull $BUILDER_IMAGE_TAG
    - docker build --build-arg BUILDER_IMAGE_TAG=${BUILDER_IMAGE_TAG} -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
```

类似地，我们也需要在 `build_reverse_proxy` 里也使用这个方法。它的 stage 需要被改为 `prebuild`，以备 `construct_builder` 构建失败。在这个特殊情况里，如果对反向代理做了任何改动，`build_reverse_proxy` 不会被触发了。

```
build_reverse_proxy:
  stage: prebuild
  # ...
  only:
    changes:
      - misc/reverse_proxy/**/*
      - misc/reverse_proxy/Dockerfile
  # ...
```

最后，我们终于可以只在必须的情况下，构建这些基础镜像了。构建 `builder` 这个基础镜像会花很多时间。通过这些工作，我们可以剩下很多 GitLab CI 月度配额。实际上，我们自己的时间也节约了很多。

> 构建时间
>
> - Builder（只会在依赖改变时触发）：4:38（估算）
> - Rails 应用：02:14（估算），节约了 **50%** 的时间，相对于上一节。
> - 反向代理（只会在依赖改变时触发）：没有变化。

详情可阅读这个 Merge Request：[https://gitlab.com/imWildCat/docker-rails-demo/merge_requests/1](https://gitlab.com/imWildCat/docker-rails-demo/merge_requests/5).

## 总结

实际上，这个教程用了如下的技术来加速构建和减少最终镜像大小：

- `--cahce-from` option: [https://docs.docker.com/engine/reference/commandline/build/](https://docs.docker.com/engine/reference/commandline/build/)
- Multi-stage build: [https://docs.docker.com/develop/develop-images/multistage-build/](https://docs.docker.com/develop/develop-images/multistage-build/)
- Build-time variables (`--build-arg`): [https://docs.docker.com/engine/reference/commandline/build/#set-build-time-variables---build-arg](https://docs.docker.com/engine/reference/commandline/build/#set-build-time-variables---build-arg)
- GitLab CI: `only`: [https://docs.gitlab.com/ee/ci/yaml/#onlyexcept-basic](https://docs.gitlab.com/ee/ci/yaml/#onlyexcept-basic)

通过使用这些技术，我们不仅可以节约构建时间，也可以降低最终构建镜像的大小，从而获得愉悦的开发和部署体验。

### 未来的话题

这里本来应该有两个或者更多关于测试 和 lint 的话题，不过我准备延后发布这些内容。因为前三节花费了我太多时间。实际上，在 Docker Swarm 上使用 [Traefik](https://traefik.io/) 和 [Portainer](https://www.portainer.io/) webhooks 可以完全自动化这个 DevOps 流程。我想以后的空闲时间再来写这些。

### 小提示

- 如果你不希望在你的开发机器上使用 Docker，[VS Code Remote Development](https://code.visualstudio.com/docs/remote/remote-overview) 是一个非常棒的工具，可以在远程机器上使用。

---

催更~ 等有空想实现在 [https://github.com/jasl/cybros_core](https://github.com/jasl/cybros_core) 上

---

很久之前想用 multistage，以为原生扩展二进制会散落在人间，原来已经统一了。

[https://github.com/docker-library/ruby/blob/master/2.6/alpine3.10/Dockerfile#L122](https://github.com/docker-library/ruby/blob/master/2.6/alpine3.10/Dockerfile#L122)

这就好办了。。。

---

发现一个问题，似乎这样的 `entrypoint.sh` 就没法用 `docker run -it` 进入容器内部了

可以的，`docer exec -it bash` 大概是这样。

---

最后发现，用 alpine 和 ubuntu，最终镜像尺寸差不多，还是 ubuntu 用起来更方便。

Ubuntu 的鏡像解壓後有上百兆，如果你的應用和依賴加起來有幾百兆，那用 Alpine 確實意義不大。

---

看到一篇也很详细的 Dockerize Rails app 的文章 [https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development](https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development)

---

小项目 k3s 或者 Swarm 大项目 k8s
