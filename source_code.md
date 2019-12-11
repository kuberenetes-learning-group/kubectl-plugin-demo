# Kubernetes 源码解析

> [pkg/kubectl/cmd/cmd.go Line 310](https://github.com/kubernetes/kubernetes/blob/30a5db136fd7343bfb53ecf31f38b4b3ff9e9a7e/pkg/kubectl/cmd/cmd.go#L310)
```golang
func NewDefaultKubectlCommandWithArgs(pluginHandler PluginHandler, args []string, in io.Reader, out, errout io.Writer) *cobra.Command {
	cmd := NewKubectlCommand(in, out, errout)

	if pluginHandler == nil {
		return cmd
	}

	if len(args) > 1 {
		cmdPathPieces := args[1:]

		if _, _, err := cmd.Find(cmdPathPieces); err != nil { //如果找到Kubectl子命令，直接退出
			if err := HandlePluginCommand(pluginHandler, cmdPathPieces); err != nil { // 查找Plugin命令
				fmt.Fprintf(errout, "%v\n", err)
				os.Exit(1)
			}
		}
	}

	return cmd
}

```


> [pkg/kubectl/cmd/cmd.go Line 398](https://github.com/kubernetes/kubernetes/blob/30a5db136fd7343bfb53ecf31f38b4b3ff9e9a7e/pkg/kubectl/cmd/cmd.go#L398)
```golang
func HandlePluginCommand(pluginHandler PluginHandler, cmdArgs []string) error {
	remainingArgs := []string{} // all "non-flag" arguments

	for idx := range cmdArgs {
		if strings.HasPrefix(cmdArgs[idx], "-") { // 跳过flag参数
			break
		}
		remainingArgs = append(remainingArgs, strings.Replace(cmdArgs[idx], "-", "_", -1)) // 替换其中的“-”为_“”
	}

	foundBinaryPath := ""

	// attempt to find binary, starting at longest possible name with given cmdArgs
	for len(remainingArgs) > 0 {
		path, found := pluginHandler.Lookup(strings.Join(remainingArgs, "-")) // 在Path中寻找可执行文件(用-连接)
		if !found {
			remainingArgs = remainingArgs[:len(remainingArgs)-1] // 没找到缩短名称
			continue
		}

		foundBinaryPath = path
		break
	}

	if len(foundBinaryPath) == 0 {
		return nil
	}

	// invoke cmd binary relaying the current environment and args given
	if err := pluginHandler.Execute(foundBinaryPath, cmdArgs[len(remainingArgs):], os.Environ()); err != nil { // 调用Plugin可执行文件
		return err
	}

	return nil
}

```