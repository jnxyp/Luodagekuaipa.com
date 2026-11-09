# 罗大哥快爬网

原首页和图片由官方 Nginx 镜像托管，域名 `luodagekuaipa.com` 通过 DNSPod CNAME 指向 `cn-hk-docker.jnxyp.net.`，TTL 600。

## 部署

- 本地项目：`D:\ProjectsLocal\luodagekuaipa.com`，沿用本仓库 `master`。
- 服务器：SSH 别名 `cn-hk-docker`，项目 `/home/ubuntu/docker_projects/luodagekuaipa.com`。
- `compose.yml` 使用固定 digest 的官方 `nginx:1.30.6-alpine`，无需构建镜像，容器 `luodagekuaipa-website-1`。
- 首页、图片和 Nginx 配置分别只读 bind mount；仅开放 `127.0.0.1:18087 → 80`，内存上限 64 MiB，日志 json-file 20m × 3，带健康检查。
- Nginx 只提供首页和图片，关闭目录列表与隐藏文件访问，配置文件不挂载到网站目录。
- 宿主机 Caddy 提供自动 HTTPS，配置参考 `site.caddy`，实际文件 `~/docker_projects/.caddy/luodagekuaipa/site.caddy`，ubuntu:caddy 0640，目录 0750。
- 项目和 Caddy 配置纳入服务器既有 Restic 备份范围，沿用原定时计划。

服务器为配置副本，更新时从本地上传 `compose.yml`、`nginx.conf`、`index.html`、图片和 `site.caddy`。修改网页或图片后立即生效；修改 Compose 或 Nginx 配置后执行：

```sh
cd /home/ubuntu/docker_projects/luodagekuaipa.com
docker compose config --quiet
docker compose up -d --wait
docker compose exec -T website nginx -t
# 已运行容器的 Nginx 配置变更需重新加载
docker compose exec -T website nginx -s reload
```

修改 Caddy 时，以 caddy 用户校验配置后重载系统服务。不要将 .git、部署文件或服务器凭据复制到网站目录。

## 迁移与回退

首页与图片来自原 GitHub Pages 仓库，迁移前与 `185.199.109.153` 上保留 Host/SNI 的 HTTPS 响应逐字节比对一致。首页保留博客入口；音乐站改为删除线文字“罗德尼音乐网”，标注“（已倒闭）”并移除链接。

DNSPod 根记录 ID `2446530423`。需要回退时，确认 GitHub Pages 仍可用后，将该根记录改回原 A 记录 `185.199.109.153`。原 GitHub Pages 配置及 `CNAME` 文件保留。
