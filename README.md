# bark-server-lzcapp

每天定时检查 Bark Server 稳定版本，并支持 `v*` tag 和手动触发懒猫官方商店与喵喵私有商店发布。

容器通过包内 `startup.sh` 直接启动 Bark Server，绕过上游会写入只读 `/etc/timezone` 的入口脚本。帮助页面使用 `/help` 静态 upstream，并通过 response inject 将 404 跳转到帮助页。
