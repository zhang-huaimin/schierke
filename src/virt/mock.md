# mock

## mock配置文件

* 设置编译路径： `config_opts['basedir'] = '/var/cache/anyway/builddir'`
* 设置源：
```
[build]
name=build
baseurl=file:///var/cache/anyway/rpm_repo/
```


## 生成srpm包

```shell
mock -r build.cfg --buildsrpm --spec SPECS/rust.spec --sources SOURCES/
```

## 编译

```shell
mock -v -r build.cfg --rebuild --no-clean SRPMS/rust-xxx.src.rpm
# ro
nohup  mock -v -r build.cfg --rebuild --no-clean SRPMS/rust-xxx.src.rpm > buildlog 2>&1 &
```
