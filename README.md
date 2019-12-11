# Kubectl Plugin Demo

## 使用方式

```bash
curl -O https://raw.githubusercontent.com/kuberenetes-learning/kubectl-plugin-demo/master/kubectl-foo
sudo chmod +x ./kubectl-foo

sudo mv ./kubectl-foo /usr/local/bin

kubectl foo
```

## Kubectl Plugin

### Kebernetes源码解析

参考 [source_code.md](source_code.md)

### 要求

1. 可执行文件
2. 名称以`kubectl-`开头,命令以`-`来分割，比如`kubectl gpu version`的可执行文件名称是`kubectl-gpu-version`。命令名称中如果包含`-`，用`_`表示，比如`kubectl gpu-version`的可执行文件名称是`kubectl-gpu_version`
3. 可执行文件位于`PATH`路径下
4. 不能和`kubectl`命令或者子命令重复，比如`kubectl version`

### 注意

1. Kubectl查找时按照最长匹配原则，比如存在`kubectl-gpu`和`kubectl-gpu-version`,那么输入`kubectl gpu verson`时执行的是`kubectl-gpu-version`。

2. 运行命令名称重复，但是只有一个会执行，具体哪个可以通过`kubectl plugin list`查看

```bash
kubectl plugin list
The following kubectl-compatible plugins are available:

/usr/local/bin/plugins/kubectl-foo
/usr/local/bin/moreplugins/kubectl-foo
  - warning: /usr/local/bin/moreplugins/kubectl-foo is overshadowed by a similarly named plugin: /usr/local/bin/plugins/kubectl-foo

error: one plugin warning was found
```
