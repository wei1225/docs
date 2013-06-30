#ovs吞吐量测试

1. 把仪表接到绿网服务器的两个网口上

```
	note:可以用`ip link show`来查看网口通断，进而判断所接网口和ethx的关系
```
2. 删除ovs上的所有流表  
	命令是`ovs-ofctl del-flows <switch名>`，例如`ovs-ofctl dump-flows s1`
3. 添加两条流表，使得AB两个口的流量互相转发  
	命令是`ovs-ofctl add-flow <switch名> in_port=<ovs端口号>,actions=output:<ovs端口号>`，例如`ovs-ofctl add-flow br0 inport=1,actions=output:2`

```
	note: 
		1. 可以用`ovs-vsctl show`来查看所有的ovs
		2. 可以用`ovs-dpctl show <switch名>`来查看某个ovs上端口名和端口号的对应
		3. 可以用`ovs-ofctl dump-flows <switch名>`来查看某个ovs的流表
```
4. 使用仪表所带软件进行测试