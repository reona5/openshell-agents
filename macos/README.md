# macOS(Docker Desktop)

`sandbox create` が次のエラーで失敗する場合は、gateway の設定を変える。

```text
ControlSupervisorStartFailed: Docker supervisor exited before becoming ready
... Startup configuration fetch failed after 5 attempts: failed to connect to OpenShell server
```

```sh
cp /opt/homebrew/var/openshell/gateway.toml{,.bak}
cp macos/gateway.toml /opt/homebrew/var/openshell/gateway.toml
brew services restart openshell
```

もう 1 つの方法として、Docker Desktop の Settings で host networking を有効にしてもよい。
