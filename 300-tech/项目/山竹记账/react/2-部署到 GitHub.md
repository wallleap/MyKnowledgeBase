本地运行命令
```zsh
npm run build
```

/dist 文件夹中修改下 index.html，将用到的路径都改成仓库名+原地址
```html
<script scr="repo/……"></script>
```

之后我们只要把 dist 目录上传到 GitHub 即可
来到 GitHub，新建一个仓库 `save-money`
本地创建 ssh-key，添加到 GitHub
```zsh
cd dist # dist 目录
git init
git add .
git commit -m "publish"
git remote add origin git@…… # 自己的仓库地址
git status # 看下自己的分支 master 还是 main
git push -f origin master:master # 自己分支
```

上传完成之后，到 GitHub 进入仓库，Settings-Pages，进行部署

如果不想手动加 repo，可以去 vite 官网搜索 `publish base path`

运行命令
```zsh
cd .. # 项目目录
npm run build --  --base=repo-name # 不要忘记 --
```

或者在 package.json 中修改，如果修改了仓库名，记得把 repo-name 修改为新的
```zsh
"build": "tsc && vite build --base=repo-name"
```

新建 /bin/deploy_to_github.sh 输入如下内容
```zsh
#!/usr/bin/env bash
rm -rf dist
npm run build
cd dist
git init
git add .
git commit -m deploy
git remote add origin git@……
git push -f origin master:master
cd .. # 或者 cd -
```

下次就可以直接运行
```zsh
sh ./bin/deploy_to_github.sh
```

给这个文件权限
```zsh
chmod +x bin/deploy_to_github.sh
```

就可以不需要用 sh
```zsh
bin/deploy_to_github.sh
```