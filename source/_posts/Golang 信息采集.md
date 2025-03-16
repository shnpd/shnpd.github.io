---
title: Golang 信息采集
toc: true
date: 2022-01-09 01:04:43
tags: go语言
categories: Golang
---

​​点击阅读更多查看文章内容<!--more-->


# Golang信息采集
>#### 项目完整内容及使用方法已上传至GitHub，点击[传送门](https://github.com/shn-1/Go_InfoCollect)即可查看

**Go语言的部分硬件信息采集可以通过gopsutil库来实现**
gopsutil库是python中的psutil库在Golang上的移植版，主要用于收集主机的各种信息，包括网络信息，进程信息，硬件信息等
[项目地址](https://github.com/shirou/gopsutil)
[官方文档](https://pkg.go.dev/github.com/shirou/gopsutil)
具体的引用方法网上有很多教程，这里不再赘述

**还有一部分linux信息的采集通过调用linux的命令，经过管道回显实现的**
[具体使用方法](https://blog.csdn.net/weixin_40592935/article/details/80888212)

**输出形式均为JSON文件**
# 设备信息
通过调用linux的dmidecode命令获取设备信息，然后经过管道获取回显，放到bytes内，输出。
此命令需要管理员权限
```go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"os/exec"
	"strings"
)

func executive_DeviceOrder(attr string) []byte {
	//name 设备名
	//manufacturer 设备厂商
	//serial_number 设备编码
	//version 设备型号
	var cmd *exec.Cmd
	if attr == "name" {
		cmd = exec.Command("/bin/bash", "-c", "sudo dmidecode -s system-product-name")
	} else
	if attr == "manufacturer" {
		cmd = exec.Command("/bin/bash", "-c", "sudo dmidecode -s system-manufacturer")
	} else
	if attr == "serial_number" {
		cmd = exec.Command("/bin/bash", "-c", "sudo dmidecode -s system-serial-number")
	} else
	if attr == "version" {
		cmd = exec.Command("/bin/bash", "-c", "sudo dmidecode -s system-version")
	} else {
		fmt.Println("输入有误！")
		return nil
	}

	//创建获取命令输出管道
	stdout, err := cmd.StdoutPipe()
	if err != nil {
		fmt.Printf("Error:can not obtain stdout pipe for command:%s\n", err)
		return nil
	}
	//执行命令
	if err := cmd.Start(); err != nil {
		fmt.Println("Error:The command is err", err)
		return nil
	}
	//读取所有输出
	bytes, err := ioutil.ReadAll(stdout)
	if err != nil {
		fmt.Println("ReadAll Stdout:", err.Error())
		return nil
	}
	if err := cmd.Wait(); err != nil {
		fmt.Println("wait:", err.Error())
		return nil
	}
	return bytes
	//fmt.Printf("stdout:\n\n%s", bytes)
}

type Device struct {
	Name         string `json:"name"`
	Manufacturer string `json:"manufacturer"`
	SerialNumber string `json:"serial_number"`
	Version      string `json:"version"`
}

func Getdevice() {

	//name 设备名
	//manufacturer 设备厂商
	//serial_number 设备编码
	//version 设备型号
	name := string(executive_DeviceOrder("name"))
	manu := string(executive_DeviceOrder("manufacturer"))
	numb := string(executive_DeviceOrder("serial_number"))
	version := string(executive_DeviceOrder("version"))

	//去掉末尾换行符
	name = strings.TrimRight(name, "\n")
	manu = strings.TrimRight(manu, "\n")
	numb = strings.TrimRight(numb, "\n")
	version = strings.TrimRight(version, "\n")

	device := Device{
		Name:         name,
		Manufacturer: manu,
		SerialNumber: numb,
		Version:      version,
	}
	deviceJson, err := json.Marshal(device)
	if err != nil {
		panic(err)
	}
	WriteFile("device.json", deviceJson)
}
```

# CPU

```go
//获得CPU有关信息
package main

import (
	"encoding/json"
	"github.com/shirou/gopsutil/cpu"
	"time"
)

type Cpu struct {
	Info          []cpu.InfoStat  `json:"info"`
	LogicalCount  int             `json:"logical_count"`
	PhysicalCount int             `json:"physical_count"`
	Usage         []float64       `json:"usage"`
	Time          []cpu.TimesStat `json:"time"`
}

func getCpu() {
	//cpu基本信息
	cpuInfos, err := cpu.Info()
	if err != nil {
		panic(err)
	}
	//cpu数量;true逻辑核心数量，false物理核心数量
	cpuLogicalCount, err := cpu.Counts(true)
	if err != nil {
		panic(err)
	}
	cpuPhysicalCount, err := cpu.Counts(false)
	if err != nil {
		panic(err)
	}
	//cpu利用率;true为每个cpu，false为总的cpu
	cpuUsage, err := cpu.Percent(time.Second, true)
	if err != nil {
		panic(err)
	}
	//cpu有关时间信息;true为每个cpu，false为总的cpu
	cpuTime, err := cpu.Times(true)
	cpu := Cpu{
		Info:  cpuInfos,
		LogicalCount: cpuLogicalCount,
		PhysicalCount: cpuPhysicalCount,
		Usage: cpuUsage,
		Time:  cpuTime,
	}
	cpuJson, err := json.Marshal(cpu)
	if err != nil {
		panic(err)
	}
	WriteFile("cpu.json", cpuJson)
}

```
> 通过读取/proc/cpuinfo获取CPU的有关信息

> cpu基本信息输出各项的含义：
> processor　：系统中逻辑处理核的编号。对于单核处理器，则课认为是其CPU编号，对于多核处理器则可以是物理核、或者使用超线程技术虚拟的逻辑核
vendor_id　：CPU制造商     
cpu family　：CPU产品系列代号
model　　　：CPU属于其系列中的哪一代的代号
model name：CPU属于的名字及其编号、标称主频
stepping　  ：CPU属于制作更新版本
cpu MHz　  ：CPU的实际使用主频
cache size   ：CPU二级缓存大小
physical id   ：单个CPU的标号
siblings       ：单个CPU逻辑物理核数
core id        ：当前物理核在其所处CPU中的编号，这个编号不一定连续
cpu cores    ：该逻辑核所处CPU的物理核数
apicid          ：用来区分不同逻辑核的编号，系统中每个逻辑核的此编号必然不同，此编号不一定连续
fpu             ：是否具有浮点运算单元（Floating Point Unit）
fpu_exception  ：是否支持浮点计算异常
cpuid level   ：执行cpuid指令前，eax寄存器中的值，根据不同的值cpuid指令会返回不同的内容
wp             ：表明当前CPU是否在内核态支持对用户空间的写保护（Write Protection）
flags          ：当前CPU支持的功能
bogomips   ：在系统内核启动时粗略测算的CPU速度（Million Instructions Per Second）
clflush size  ：每次刷新缓存的大小单位
cache_alignment ：缓存地址对齐单位
address sizes     ：可访问地址空间位数
power management ：对能源管理的支持，有以下几个可选支持功能：
　ts：　　temperature sensor
　fid：　  frequency id control
　vid：　 voltage id control
　ttp：　 thermal trip
　tm：
　stc：
　100mhzsteps：
　hwpstate：
CPU信息中flags各项含义：
fpu： Onboard (x87) Floating Point Unit
vme： Virtual Mode Extension
de： Debugging Extensions
pse： Page Size Extensions
tsc： Time Stamp Counter: support for RDTSC and WRTSC instructions
msr： Model-Specific Registers
pae： Physical Address Extensions: ability to access 64GB of memory; only 4GB can be accessed at a time though
mce： Machine Check Architecture
cx8： CMPXCHG8 instruction
apic： Onboard Advanced Programmable Interrupt Controller
sep： Sysenter/Sysexit Instructions; SYSENTER is used for jumps to kernel memory during system calls, and SYSEXIT is used for jumps： back to the user code
mtrr： Memory Type Range Registers
pge： Page Global Enable
mca： Machine Check Architecture
cmov： CMOV instruction
pat： Page Attribute Table
pse36： 36-bit Page Size Extensions: allows to map 4 MB pages into the first 64GB RAM, used with PSE.
pn： Processor Serial-Number; only available on Pentium 3
clflush： CLFLUSH instruction
dtes： Debug Trace Store
acpi： ACPI via MSR
mmx： MultiMedia Extension
fxsr： FXSAVE and FXSTOR instructions
sse： Streaming SIMD Extensions. Single instruction multiple data. Lets you do a bunch of the same operation on different pieces of input： in a single clock tick.
sse2： Streaming SIMD Extensions-2. More of the same.
selfsnoop： CPU self snoop
acc： Automatic Clock Control
IA64： IA-64 processor Itanium.
ht： HyperThreading. Introduces an imaginary second processor that doesn’t do much but lets you run threads in the same process a  bit quicker.
nx： No Execute bit. Prevents arbitrary code running via buffer overflows.
pni： Prescott New Instructions aka. SSE3
vmx： Intel Vanderpool hardware virtualization technology
svm： AMD “Pacifica” hardware virtualization technology
lm： “Long Mode,” which means the chip supports the AMD64 instruction set
tm： “Thermal Monitor” Thermal throttling with IDLE instructions. Usually hardware controlled in response to CPU temperature.
tm2： “Thermal Monitor 2″ Decrease speed by reducing multipler and vcore.
est： “Enhanced SpeedStep”


>cpu有关时间输出各项的含义：
>user:用户态的CPU时间
nice：低优先级程序所占用的用户态的cpu时间。
system：系统态的CPU时间
idle：CPU空闲的时间（不包含IO等待）
iowait：等待IO响应的时间
irq：处理硬件中断的时间
softirq：处理软中断的时间
steal:其他系统所花的时间（个人理解是针对虚拟机）
guest：运行时间为客户操作系统下的虚拟CPU控制（个人理解是访客控制CPU的时间）
guest_nice：低优先级程序所占用的用户态的cpu时间。（访客的）
# 磁盘
## 1. 获取磁盘分区信息

```go
//获取磁盘分区
DiskParti, err := disk.Partitions(false)
```
## 2.获取磁盘序列号

```go
//得到磁盘序列号
func GetSerialNumber(name string) string {
	//获取路径为name的磁盘的序列号
	sn := disk.GetDiskSerialNumber(name)
	return sn
}
```
## 3.获取磁盘标签

```go
//得到磁盘标签
func GetLabel(name string) string {
	label := disk.GetLabel(name)
	return label
}
```
## 4.获取磁盘使用情况

```go
//获取磁盘使用情况
func GetUsage(path string) *disk.UsageStat {
	usage, err := disk.Usage(path)
	if err == nil {
		return usage
	} else {
		return nil
	}
}
```
## 5.获取磁盘IO信息

```go
//得到磁盘IO信息
func GetIO(name string) map[string]disk.IOCountersStat {
	IO, err := disk.IOCounters(name)
	if err == nil {
		return IO
	} else {
		return nil
	}
}
```
# 内存
## 获取内存有关信息

```go
package main

import (
	"encoding/json"
	"github.com/shirou/gopsutil/mem"
)

type Memory struct {
	SwapMemory      *mem.SwapMemoryStat      `json:"swap_memory"`
	VirtualMemory   *mem.VirtualMemoryStat   `json:"virtual_memory"`
	VirtualMemoryEx *mem.VirtualMemoryExStat `json:"virtual_memory_ex"`
}

func getMemory() {
	swapInfo, err := mem.SwapMemory()
	if err != nil {
		panic(err)
	}
	memInfo, err := mem.VirtualMemory()
	if err != nil {
		panic(err)
	}
	memExinfo, err := mem.VirtualMemoryEx()
	if err != nil {
		panic(err)
	}

	memory := Memory{
		SwapMemory:      swapInfo,
		VirtualMemory:   memInfo,
		VirtualMemoryEx: memExinfo,
	}

	memoryJson, err := json.Marshal(memory)
	if err != nil {
		panic(err)
	}
	WriteFile("Memory.json", memoryJson)
}

//获得内存有关信息
func GetVirtualMemInfo() *mem.VirtualMemoryStat {
	memInfo, _ := mem.VirtualMemory()
	return memInfo
}

func GetVirtualMemExInfo() *mem.VirtualMemoryExStat {
	memInfo, _ := mem.VirtualMemoryEx()
	return memInfo
}

//
//func main(){
//	fmt.Println(GetVirtualMemExInfo())
//	fmt.Println(GetVirtualMemInfo())
//}

```
包括总内存、已用内存、可用内存、内存占用比例等信息
# 网卡

```go
package main

import (
	"encoding/json"
	"net"
)

type netInterface struct {
	Index        int       `json:"index"`
	MTU          int       `json:"mtu"`
	Name         string    `json:"name"`
	HardwareAddr string    `json:"hardware_addr"`
	Flags        net.Flags `json:"flags"`
}

//网络接口信息采集
func GetNetInterface() {
	interfaces, err := net.Interfaces()
	if err != nil {
		panic(err)
	}
	var netInterfaces []netInterface
	for _, n := range interfaces {
		temp := netInterface{
			Index:        n.Index,
			MTU:          n.MTU,
			Name:         n.Name,
			HardwareAddr: n.HardwareAddr.String(),
			Flags:        n.Flags,
		}
		netInterfaces = append(netInterfaces, temp)
	}
	netInterfacesJson, err := json.Marshal(netInterfaces)
	if err != nil {
		panic(err)
	}
	WriteFile("NetInterface.json", netInterfacesJson)
}

```

# 进程
## 1.获取进程pid

```go
//获取进程id
func ProcessId() (pid []int32) {
	pids, _ := process.Pids()
	for _, p := range pids {
		pid = append(pid, p)
	}
	return pid
}
```
## 2.获取进程名

```go
//获取进程名
func ProcessName() (pname []string) {
	pids, _ := process.Pids()
	for _, pid := range pids {
		pn, _ := process.NewProcess(pid)
		pName, _ := pn.Name()
		pname = append(pname, pName)
	}
	return pname
}
```
## 3.获取进程的内存占用

```go
//内存占用
func ProcessMemory() (pmemory []string) {
	pids, _ := process.Pids()
	for _, pid := range pids {
		pn, _ := process.NewProcess(pid)
		pmry, _ := pn.MemoryInfoEx()
		pMemory := pmry.String()
		pmemory = append(pmemory, pMemory)
	}
	return pmemory
}
```
## 4. 获取进程的CPU占用
```go
//CPU占用
func ProcessCpu() (pcpu []float64) {
	pids, _ := process.Pids()
	for _, pid := range pids {
		pn, _ := process.NewProcess(pid)
		pCpu, _ := pn.CPUPercent()
		pcpu = append(pcpu, pCpu)
	}
	return pcpu
}
```

# 日志
## 1.系统日志
通过Linux命令行获取系统日志信息，然后经过管道获取回显，放到bytes内，返回。
```go
func Syslog() interface{} {  
//读取系统日志  
     cmd := exec.Command("/bin/bash", "-c","cat /var/log/syslog")  
//创建获取命令输出管道  
   stdout, err := cmd.StdoutPipe()  
   if err != nil {  
      fmt.Printf("Error:can not obtain stdout pipe for command:%s\n", err)  
      return nil  
   }  
//执行命令  
   if err := cmd.Start(); err != nil {  
      fmt.Println("Error:The command is err,", err)  
      return nil  
   }  
//读取所有输出  
   bytes, err := ioutil.ReadAll(stdout)  
   if err != nil {  
      fmt.Println("ReadAll Stdout:", err.Error())  
      return nil  
   }  
  
   if err := cmd.Wait(); err != nil {  
      fmt.Println("wait:", err.Error())  
      return nil  
   }  
  
//返回日志结果，类型为[]byte  
   return bytes  
} 
```



## 2.安全日志
由于不同版本的Ubuntu中安全日志存放的形式不同，所以这里只查看了auth，回显用户登录信息以及记录通过sudo执行管理员权限的操作。原理同系统日志
```go
func seculog() interface{}{  
//读取安全日志  
   cmd := exec.Command("/bin/bash", "-c","cat /var/log/auth.log")  
//创建获取命令输出管道  
   stdout, err := cmd.StdoutPipe()  
   if err != nil {  
      fmt.Printf("Error:can not obtain stdout pipe for command:%s\n", err)  
      return nil  
   }  
//执行命令  
   if err := cmd.Start(); err != nil {  
      fmt.Println("Error:The command is err,", err)  
      return nil  
   }  
//读取所有输出  
   bytes, err := ioutil.ReadAll(stdout)  
   if err != nil {  
      fmt.Println("ReadAll Stdout:", err.Error())  
      return nil  
   }  
  
   if err := cmd.Wait(); err != nil {  
      fmt.Println("wait:", err.Error())  
      return nil  
   }  
//返回安全日志内容  
   return bytes  
}  
```
# 网络信息
## 1.获取本机IP地址

```go
func GetIpAddrs() map[string]string {
	mpIp := make(map[string]string)
	//获取网络接口
	ifaces, err := net.Interfaces()
	if err != nil {
		fmt.Print(fmt.Errorf("localAddresses: %+v\n", err.Error()))
		return nil
	}
	//遍历网络接口
	for _, i := range ifaces {
		addrs, err := i.Addrs()
		if err != nil {
			fmt.Print(fmt.Errorf("localAddresses: %+v\n", err.Error()))
			continue
		}
		//获取网络接口的地址
		for _, a := range addrs {
			mpIp[i.Name] = a.String()
			//fmt.Printf("%v : %s \n", i.Name, a.String())
		}
	}
	return mpIp
}
```

## 2.获取本机MAC地址

```go
func GetMacAddrs() map[string]string {
	mpMac := make(map[string]string)
	netInterfaces, err := net.Interfaces()
	if err != nil {
		panic(err)
	}
	for _, netInterface := range netInterfaces {
		macAddr := netInterface.HardwareAddr.String()
		if len(macAddr) == 0 {
			continue
		}
		mpMac[netInterface.Name] = macAddr
	}
	return mpMac
}
```
## 3.获取本机网关信息
```go

type rtInfo struct {
	Dst net.IPNet
	Gateway, PrefSrc net.IP
	OutputIface uint32
	Priority uint32
}
type GateWay struct {
	InterfaceName string `json:"interface_name"`
	GateWay       string `json:"gate_way"`
	Ip            string `json:"ip"`
}
type routeSlice []*rtInfo

type router struct {
	ifaces []net.Interface
	addrs []net.IP
	v4 routeSlice
}

func getRouteInfo() (*router, error) {
	rtr := &router{}

	tab, err := syscall.NetlinkRIB(syscall.RTM_GETROUTE, syscall.AF_INET)
	if err != nil {
		return nil, err
	}

	msgs, err := syscall.ParseNetlinkMessage(tab)
	if err != nil {
		return nil, err
	}

	for _, m := range msgs {
		switch m.Header.Type {
		case syscall.NLMSG_DONE:
			break
		case syscall.RTM_NEWROUTE:
			rtmsg := (*syscall.RtMsg)(unsafe.Pointer(&m.Data[0]))
			attrs, err := syscall.ParseNetlinkRouteAttr(&m)
			if err != nil {
				return nil, err
			}
			routeInfo := rtInfo{}
			rtr.v4 = append(rtr.v4, &routeInfo)
			for _, attr:= range attrs {
				switch attr.Attr.Type {
				case syscall.RTA_DST:
					routeInfo.Dst.IP = net.IP(attr.Value)
					routeInfo.Dst.Mask = net.CIDRMask(int(rtmsg.Dst_len), len(attr.Value)*8)
				case syscall.RTA_GATEWAY:
					routeInfo.Gateway = net.IPv4(attr.Value[0], attr.Value[1], attr.Value[2], attr.Value[3])
				case syscall.RTA_OIF:
					routeInfo.OutputIface = *(*uint32)(unsafe.Pointer(&attr.Value[0]))
				case syscall.RTA_PRIORITY:
					routeInfo.Priority = *(*uint32)(unsafe.Pointer(&attr.Value[0]))
				case syscall.RTA_PREFSRC:
					routeInfo.PrefSrc = net.IPv4(attr.Value[0], attr.Value[1], attr.Value[2], attr.Value[3])
				}
			}
		}
	}

	sort.Slice(rtr.v4, func(i, j int) bool {
		return rtr.v4[i].Priority < rtr.v4[j].Priority
	})

	ifaces, err := net.Interfaces()
	if err != nil {
		return nil, err
	}

	for i, iface := range ifaces {

		if i != iface.Index - 1 {
			break
		}

		if iface.Flags & net.FlagUp == 0{
			continue
		}
		rtr.ifaces = append(rtr.ifaces, iface)
		ifaceAddrs, err := iface.Addrs()
		if err != nil {
			return nil, err
		}
		var addrs net.IP
		for _, addr := range ifaceAddrs {
			if inet, ok := addr.(*net.IPNet); ok {
				if v4 := inet.IP.To4(); v4 != nil {
					if addrs == nil {
						addrs = v4
					}
				}
			}
		}
		rtr.addrs = append(rtr.addrs, addrs)
	}
	return rtr, nil
}


func getGateWay() []GateWay {
	newRoute, err := getRouteInfo()
	if err != nil {
		fmt.Println(err)
		return nil
	}
	var gateWays []GateWay
	for _, rt := range newRoute.v4 {
		var gateway GateWay
		if rt.Gateway != nil {
			gateway.InterfaceName = newRoute.ifaces[rt.OutputIface-1].Name
			gateway.GateWay = rt.Gateway.String()
			gateway.Ip = newRoute.addrs[rt.OutputIface-1].String()
			gateWays = append(gateWays, gateway)
		}
	}
	return gateWays
}
```
# 防火墙

这里的防火墙信息包括iptable和firewall两种防火墙
iptables防火墙通过"service iptables status"和"sudo iptables -L"命令来获取其状态和规则
firewall防火墙通过"firewall-cmd --state"和"firewall-cmd --list-all"命令来获得其状态和规则
其中获取iptables规则以及获取firewall状态的命令需要管理员权限

## 1. iptables防火墙

```go
//iptables防火墙
func Getiptables() (string, string) {
	//iptables防火墙状态
	cmd := exec.Command("/bin/bash", "-c", "service iptables status")
	//iptables防火墙规则
	cmd2 := exec.Command("/bin/bash", "-c", "sudo iptables -L")
	//创建获取命令输出管道
	stdout, err := cmd.StdoutPipe()
	stdout2, err2 := cmd2.StdoutPipe()
	if err != nil {
		fmt.Printf("Error:can not obtain stdout pipe for command:%s\n", err)
		return "", ""
	}
	if err2 != nil {
		fmt.Printf("Error:can not obtain stdout pipe for command:%s\n", err2)
		return "", ""
	}
	//执行命令
	if err := cmd.Start(); err != nil {
		fmt.Println("Error:The command is err", err)
		return "", ""
	}
	if err2 := cmd2.Start(); err2 != nil {
		fmt.Println("Error:The command is err", err2)
		return "", ""
	}
	//读取所有输出
	bytes, err := ioutil.ReadAll(stdout)
	bytes2, err2 := ioutil.ReadAll(stdout2)
	if err != nil {
		fmt.Println("ReadAll Stdout:", err.Error())
		return "", ""
	}
	if err2 != nil {
		fmt.Println("ReadAll Stdout:", err2.Error())
		return "", ""
	}

	if err2 := cmd2.Wait(); err2 != nil {
		fmt.Println("wait:", err2.Error())
		return "", ""
	}

	return string(bytes), string(bytes2)
}
```

## 2.firewall防火墙
```go
//firewall防火墙
func Getfirewall() (string, string) {
	//firewall防火墙状态
	cmd := exec.Command("/bin/bash", "-c", "firewall-cmd --state")
	//firewall防火墙规则
	cmd2 := exec.Command("/bin/bash", "-c", "firewall-cmd --list-all")
	//创建获取命令输出管道
	stdout, err := cmd.StdoutPipe()
	stdout2, err2 := cmd2.StdoutPipe()
	if err != nil {
		fmt.Printf("Error:can not obtain stdout pipe for command:%s\n", err)
		return "", ""
	}
	if err2 != nil {
		fmt.Printf("Error:can not obtain stdout pipe for command:%s\n", err2)
		return "", ""
	}
	//执行命令
	if err := cmd.Start(); err != nil {
		fmt.Println("Error:The command is err", err)
		return "", ""
	}
	if err2 := cmd2.Start(); err2 != nil {
		fmt.Println("Error:The command is err", err2)
		return "", ""
	}
	//读取所有输出
	bytes, err := ioutil.ReadAll(stdout)
	bytes2, err2 := ioutil.ReadAll(stdout2)
	if err != nil {
		fmt.Println("ReadAll Stdout:", err.Error())
		return "", ""
	}
	if err2 != nil {
		fmt.Println("ReadAll Stdout:", err2.Error())
		return "", ""
	}
	if err2 := cmd2.Wait(); err2 != nil {
		fmt.Println("wait:", err2.Error())
		return "", ""
	}
	return string(bytes), string(bytes2)
}
```

# 主机、操作系统、内核信息
```go
/*type InfoStat struct {
	Hostname             string `json:"hostname"`
	Uptime               uint64 `json:"uptime"`
	BootTime             uint64 `json:"bootTime"`
	Procs                uint64 `json:"procs"`           // number of processes
	OS                   string `json:"os"`              // ex: freebsd, linux
	Platform             string `json:"platform"`        // ex: ubuntu, linuxmint
	PlatformFamily       string `json:"platformFamily"`  // ex: debian, rhel
	PlatformVersion      string `json:"platformVersion"` // version of the complete OS
	KernelVersion        string `json:"kernelVersion"`   // version of the OS kernel (if available)
	KernelArch           string `json:"kernelArch"`      // native cpu architecture queried at runtime, as returned by `uname -m` or empty string in case of error
	VirtualizationSystem string `json:"virtualizationSystem"`
	VirtualizationRole   string `json:"virtualizationRole"` // guest or host
	HostID               string `json:"hostid"`             // ex: uuid
}*/
func getHostInfo() *host.InfoStat {
	id, err := host.Info()
	if err == nil {
		return id
	}else{
		return nil
	}
}
```
# 服务
## 1.获取正在运行的系统服务以及全部系统服务
通过Linux命令行获取服务，然后经过管道获取回显，放到bytes内，返回。

```go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"os/exec"
)

type Service struct {
	RunningService string `json:"running_service"`
	AllService     string `json:"all_service"`
}

func executive_ServiceOrder(attr string) string {
	//running:获取正在运行的系统服务
	//all:获取全部的系统服务
	var cmd *exec.Cmd
	if attr == "running" {
		cmd = exec.Command("/bin/bash", "-c", "service --status-all | grep +")
	} else if attr == "all" {
		cmd = exec.Command("/bin/bash", "-c", "service --status-all")
	} else {
		fmt.Println("输入有误！")
		return ""
	}
	stdout, err := cmd.StdoutPipe()
	if err != nil {
		panic(err)
	}
	//执行命令
	if err := cmd.Start(); err != nil {
		panic(err)
	}
	//读取所有输出
	bytes, err := ioutil.ReadAll(stdout)
	if err != nil {
		panic(err)
	}
	if err := cmd.Wait(); err != nil {
		panic(err)
	}
	return string(bytes)
}

func GetService() {
	//running:获取正在运行的系统服务
	//all:获取全部的系统服务
	runningService := executive_ServiceOrder("running")
	allService := executive_ServiceOrder("all")
	service := Service{
		RunningService: runningService,
		AllService:     allService,
	}
	serviceJson, err := json.Marshal(service)
	if err != nil {
		panic(err)
	}
	WriteFile("Service.json", serviceJson)
}
```
# 流量
## 获取数据量

通过Linux命令行进行更新并获取数据量，然后经过管道获取回显，放到bytes内，返回。

```go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"os/exec"
)

type DataAmount struct {
	DataAmount string `json:"data_amount"`
}

func GetDataAmount() {
	cmd1 := exec.Command("/bin/bash", "-c", "vnstat -u")
	cmd := exec.Command("/bin/bash", "-c", "vnstat")

	//分别执行更新命令和读取数据量命令
	stdout, err := cmd.StdoutPipe()
	stdout2, err2 := cmd1.StdoutPipe()
	stdout2 = stdout2
	if err != nil {
		fmt.Printf("Error:can not obtain stdout pipe for command:%s\n", err)
		return
	}
	if err2 != nil {
		fmt.Printf("Error:can not obtain stdout pipe for command:%s\n", err2)
		return
	}

	//执行命令
	if err2 := cmd1.Start(); err2 != nil {
		fmt.Println("Error:The command is err,", err2)
		return
	}

	if err := cmd.Start(); err != nil {
		fmt.Println("Error:The command is err,", err)
		return
	}
	//读取所有输出
	bytes, err := ioutil.ReadAll(stdout)
	if err != nil {
		fmt.Println("ReadAll Stdout:", err.Error())
		return
	}

	if err := cmd.Wait(); err != nil {
		fmt.Println("wait:", err.Error())
		return
	}
	//返回结果
	dataAmount := DataAmount{DataAmount: string(bytes)}
	dataAmountJson, err := json.Marshal(dataAmount)
	if err != nil {
		panic(err)
	}
	WriteFile("DataAmount.json", dataAmountJson)
}
  
```
# 应用软件信息
分别通过"rpm -qa"、"dpkg -l"、"yum list installed"命令获取deb包、rpm包和yum方法安装的软件的信息
```go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"os/exec"
)

type AppInfo struct {
	DebInstall string `json:"deb_install"`
	rpmInstall string `json:"rpm_install"`
	yumInstall string `json:"yum_install"`
}

func GetAppInfo() {
	//deb安装
	cmd := exec.Command("/bin/bash", "-c", "dpkg -l")
	//rpm安装
	cmd2 := exec.Command("/bin/bash", "-c", "rpm -qa")
	//yum安装
	cmd3 := exec.Command("/bin/bash", "-c", "yum list installed")
	//创建获取命令输出管道
	stdout, err := cmd.StdoutPipe()
	stdout2, err2 := cmd2.StdoutPipe()
	stdout3, err3 := cmd3.StdoutPipe()
	if err != nil {
		fmt.Printf("Error:can not obtain stdout pipe for command:%s\n", err)
		return
	}
	if err2 != nil {
		fmt.Printf("Error:can not obtain stdout pipe for command:%s\n", err2)
		return
	}
	if err3 != nil {
		fmt.Printf("Error:can not obtain stdout pipe for command:%s\n", err3)
		return
	}
	//执行命令
	if err := cmd.Start(); err != nil {
		fmt.Println("Error:The command is err", err)
		return
	}
	if err2 := cmd2.Start(); err2 != nil {
		fmt.Println("Error:The command is err", err2)
		return
	}
	if err3 := cmd3.Start(); err3 != nil {
		fmt.Println("Error:The command is err", err3)
		return
	}
	//读取所有输出
	bytes, err := ioutil.ReadAll(stdout)
	bytes2, err2 := ioutil.ReadAll(stdout2)
	bytes3, err3 := ioutil.ReadAll(stdout3)
	if err != nil {
		fmt.Println("ReadAll Stdout:", err.Error())
		return
	}
	if err2 != nil {
		fmt.Println("ReadAll Stdout:", err2.Error())
		return
	}
	if err3 != nil {
		fmt.Println("ReadAll Stdout:", err3.Error())
		return
	}
	if err := cmd.Wait(); err != nil {
		fmt.Println("wait:", err.Error())
		return
	}
	if err2 := cmd2.Wait(); err2 != nil {
		fmt.Println("wait:", err2.Error())
		return
	}
	if err3 := cmd3.Wait(); err3 != nil {
		fmt.Println("wait:", err3.Error())
		return
	}

	appInfo := AppInfo{
		DebInstall: string(bytes),
		rpmInstall: string(bytes2),
		yumInstall: string(bytes3),
	}
	appInfoJson, err := json.Marshal(appInfo)
	WriteFile("Application.json", appInfoJson)
}
```

