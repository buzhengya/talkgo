## 高CPU问题5.28-5.29
### 内容
1. 找到高CPU的进程，并且定位其性能瓶颈。
2. CPU高但找不到对应高CPU的进程，考虑短时进程。

### 定位进程占用高CPU的情况
1. 查找哪个进程导致了高CPU。top显示所有进程的CPU使用百分比，pidstat显示所有或者指定进程的CPU使用情况，并且含具体的user和system使用百分比。
2. 定位到具体的进程后，通过perf工具具体查找热点函数（通常占百分之八十CPU的代码只占所有代码中的20%）。找到热点函数后，查看热点函数是否具有优化空间。
### 机器总的CPU高但却找不到占用CPU的进程的情况
1. 当部署服务器的性能没有达到预期时，去部署的机器通过top，pidstat查看机器的状态，看到机器总的CPU很高，但是却没有看到热点进程。
2. 总CPU高但找不到热点进程，此时考虑是否是有大量的短时进程，快速创建，快速死亡，因此top，pidstat这种间隔统计工具统计不到。因为在这个间隔中，进程已经消亡，所以无法统计到这些进程。
3. pstree(process tree)打印进程的调用关系。execsnoop追踪短时进程。
### 收获
1. CPU高的特殊情况--短时进程。这种情况下，常用的统计工具，top，pidstat等无法定位到热点进程。
2. 代码编写时的考虑错误反馈机制。代码编写时，难免会遇到不可预估的错误，此时需要有正确的，及时的反馈机制来保证错误出现时，我们能快速定位到问题所在。比如例子中的进程创建和执行失败，要么应该有ERROR日志，或者ASSERT机制，保证顶用失败时，可以通过错误日志或者ASSERT触发邮件，告知进程的异常情况。