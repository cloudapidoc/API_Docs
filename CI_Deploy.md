# CI 部署

目前使用的是 [Travis CI](https://travis-ci.org/huobiapi/docs)，由于 `slate` 原生不支持多语言多版本独立部署，此项目改为用 `临时目录` 中转部署文件的方案部署。与原生的部署方案相比，此方案便于部署单个分支，不会对其他分支造成影响。

## 部署脚本

### 部署过程
- 修改 `./deploy.sh`，实现在 `./gh-pages` 文件夹下 `checkout origin/gh-pages` 分支，再将 build 所生成的静态页面文件全部粘贴过来。

## 配置
- 修改 `./.travis.yml`
  ```yml
  sudo: false

    language: ruby

    rvm:
    - 2.4.0

    branches:
    only:
        - v1_cn
        - v1_en
        - v2_cn
        - v2_en
        # new branch add here

    cache: bundler

    script: 
    - ./deploy.sh
  ```