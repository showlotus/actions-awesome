# actions-awesome

## Release NPM

> 参考：https://docs.github.com/zh/actions/publishing-packages/publishing-nodejs-packages#publishing-packages-to-the-npm-registry

### use npm

```yml
name: Release NPM with npm

on:
  push:
    tags:
      # 每当创建一个 tag 时，触发当前工作流
      - '*'
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # Setup .npmrc file to publish to npm
      - uses: actions/setup-node@v4
        with:
          node-version: '20.x'
          registry-url: 'https://registry.npmjs.org'
      - run: npm ci
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### use pnpm

```yml
# 需要在 package.json 中配置 version 和 packageManager（指定 pnpm 版本）
# 例如，`packageManager: pnpm@8.10.0` 指定 pnpm 版本为 8.10.0
name: Release NPM with pnpm

on:
  push:
    tags:
      # 每当新创建一个以 v 开头的 tag 时，触发当前工作流
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      id-token: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install pnpm
        uses: pnpm/action-setup@v2

      - name: Set node
        uses: actions/setup-node@v4
        with:
          node-version: lts/*
          cache: pnpm
          registry-url: 'https://registry.npmjs.org'

      - run: npx changelogithub
        continue-on-error: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Install Dependencies
        run: pnpm i

      - name: PNPM build
        run: pnpm run build

      - name: Publish to NPM
        # 发布工作区下的所有包
        run: pnpm -r publish --access public --no-git-checks
        # 指定发布工作区下的某个包
        # run: pnpm --filter ./packages/package-name publish --access public --no-git-checks
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
          NPM_CONFIG_PROVENANCE: true
```

## Publish Github Pages

```yml
name: deploy

on:
  push:
    branches: [master] # master 分支有 push 时触发
    paths-ignore: # 下列文件的变更不触发部署，可以自行添加
      - README.md

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      # 下载源码
      # 这一步就是检出你的仓库并下载里面的代码到runner中,actions/checkout@v2是官方自己造的轮子，直接拿来用就行
      - name: Checkout
        uses: actions/checkout@v2

      # 打包构建
      - name: Build
        uses: actions/setup-node@master
        with:
          node-version: '20.x'
      - run: npm install # 安装依赖
      - run: npm run build # 打包

      # 部署到 GitHub pages
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3 # 使用部署到 GitHub pages 的 action
        with:
          publish_dir: ./dist # 部署打包后的 dist 目录
          github_token: ${{ secrets.DEPLOY_SECRET }} # secret 名
          user_name: ${{ secrets.MY_USER_NAME }}
          user_email: ${{ secrets.MY_USER_EMAIL }}
          commit_message: 自动部署 # 部署时的 git 提交信息，自由填写
```
