# 网络面板

测试您的网速，多出口查询您的ip地址

支持定量完成，支持多线程，适配iOS后台运行。

如果你不了解如何打包vite项目，请[点此下载](https://github.com/ljxi/NetworkPanel/archive/refs/heads/gh-pages.zip)解压文件部署到服务器根目录即可

或者使用`Docker`部署
```bash
docker run -d --rm -p 8080:80 netart/network-panel:latest
```

或者使用腾讯云提供的一键部署

[![Use EdgeOne Pages to deploy](https://cdnstatic.tencentcs.com/edgeone/pages/deploy.svg)](https://edgeone.ai/pages/new?repository-url=https%3A%2F%2Fgithub.com%2Fljxi%2FNetworkPanel&repository-name=NetworkPanel&project-name=network-panel&build-command=pnpm%20run%20build&install-command=pnpm%20i&output-directory=dist&root-directory=.%2F)

[Demo](https://ljxi.github.io/NetworkPanel/)

这是vue3重写版本，旧版本在old分支，这次重写，主要增加了以下特性：

1. 支持添加自定义节点
2. 启动之后更改节点与线程数立即生效（旧版本需要重新启动）
3. 线程数和后台开关状态保存
4. 更友好的界面
5. 榜单功能(需要后端支持)

## 本次修改内容

- 修复了项目依赖安装问题，确保项目可以正常运行。
- 重构了IP地址查询功能，使其不依赖原项目未开源的后端API，现在可以独立运行。
- 移除了对未开源后端服务的依赖，包括排行榜和数据上传功能。
- 优化了ECharts图表的初始化逻辑，解决了潜在的尺寸获取警告。
- 配置了Vite代理，为将来集成不支持CORS的第三方API提供了基础。
