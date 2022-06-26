### 遇到的一些问题

[1] ImportError: cannot import name BPF；原因是得安装bpfcc

https://GitHub.com/iovisor/bcc/issues/2278

[2] 





### 另外一种解决办法

Create iptables rule per process/service; 用iptables的owner模块，但是得低版本的kernel（2.3～2.6，在4.+的kernel中owner模块没有查看pid的选项）