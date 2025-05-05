# Chinese Search Optimization

## 安装必要插件

**注意：以下所有操作都是基于你使用 Docker 管理 Mastodon 的基础上的**

在 `./elasticsearch` 下执行：

```bash
export ES_VERSION=7.17.4     #版本号与elasticsearch版本号一致
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v${ES_VERSION}/elasticsearch-analysis-ik-${ES_VERSION}.zip
wget https://github.com/medcl/elasticsearch-analysis-stconvert/releases/download/v${ES_VERSION}/elasticsearch-analysis-stconvert-${ES_VERSION}.zip
```

新建文件 `docker-entrypoint-es-plugins.sh`，写入：

```bash
#!/bin/bash
# setting up prerequisites

export ES_VERSION=7.17.4     #版本号与elasticsearch版本号一致

yes | elasticsearch-plugin install file:/usr/share/elasticsearch/data/elasticsearch-analysis-ik-${ES_VERSION}.zip
elasticsearch-plugin install file:/usr/share/elasticsearch/data/elasticsearch-analysis-stconvert-${ES_VERSION}.zip

exec /usr/local/bin/docker-entrypoint.sh
```

## 修改 docker-compose.yml

在 `es` 部分添加以下内容：

```yml
    entrypoint: /usr/share/elasticsearch/data/docker-entrypoint-es-plugins.sh
```

最后应该看起来是这样：

```yml
......
  es:
    restart: always
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.4
.......
        soft: 65536
        hard: 65536
    entrypoint: /usr/share/elasticsearch/data/docker-entrypoint-es-plugins.sh
    ports:
.......
```

## 部署搜索

执行以下指令：

```bash
docker compose down    # 关闭 Mastodon 所有服务
cd elasticsearch       # 进入 elasticsearch 文件夹（如果 elasticsearch/nodes 存在的话）
rm -rf nodes           # 删除nodes文件夹
cd ..                  # 返回 Mastodon 主文件夹
docker compose up -d   # 重新启动 Mastodon 所有服务
```

此时可运行 `docker logs mastodon_es_1 -f -t`查看日志，看两个插件是否安装成功，有无 Fail / Error 等报错。正常情况下，应该能在开头部分找到 `Installed analysis-ik` 和 `Installed analysis-stconvert` 两行。

## 更新搜索

执行以下指令：

```
docker-compose run --rm web bin/tootctl search deploy
```

重新生成索引。

## 参考

[进阶魔改：修改字数上限、媒体上限、投票上限、添加自定义主题、界面用语、非登陆用户有限显示、优化中文搜索，附阻止本站嘟文流入某站点方法 - 技术小白搭建Mastodon站点指南](https://pullopen.github.io/%E8%BF%9B%E9%98%B6%E9%AD%94%E6%94%B9/2020/11/14/mastodon-modify.html#docker%E5%AE%89%E8%A3%85%E4%BC%98%E5%8C%96%E4%B8%AD%E6%96%87%E6%90%9C%E7%B4%A22022-04-25%E6%96%B0%E5%A2%9E)
