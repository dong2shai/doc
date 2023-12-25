# <center> BusyBox构建rootfs </center>


##  编译busybox 
---
1. 下载最新的busybox   
>  下载并解压 [BusyBox 官网](https://busybox.net)
```
 #tar -xvf busybox-1.36.1.tar.bz2 
```
2. 编译busybox
> busybox 的编译和linux kernel 编译方式类似通过执行:    
```
[busybox-1.36.1]# make menuconfig
```

## 构建rootfs
---
1. 创建rootfs需要的目录   
>   
```   
[busybox-1.36.1]# mkdir rootfs    
[busybox-1.36.1]# mkdir rootfs/etc    
[busybox-1.36.1]# mkdir rootfs/dev    
[busybox-1.36.1]# mkdir rootfs/init.d    
```

2. 安装rootfs需要的工具   
>       
```     
[busybox-1.36.1]# make install CONFIG_PREFIX=./rootfs
```

3. 创建init文件     
>      
```  
[busybox-1.36.1]# rm -rf rootfs/linuxrc     
[busybox-1.36.1]#  ln -s rootfs/bin/busybox rootfs/init 
```
4. 创建控制台节点       
>       
```
[busybox-1.36.1]# mknod -m 600 rootfs/dev/console c 5 1
```

5. 创建inittab脚本并写入    
>        
```
[busybox-1.36.1]# vim rootfs/etc/inittabi      
::sysinit:/etc/init.d/rcS
::askfirst:-/bin/sh
::restart:/sbin/init
::ctrlaltdel:/sbin/reboot
::shutdown:/bin/umount -a -r
::shutdown:/sbin/swapoff -a
```

6. 创建rcS脚本并写入    
>              
```
[busybox-1.36.1]# vim  rootfs/etc/init.d/rcS
#!/bin/sh
export PATH=/sbin:/bin:/usr/bin;/usr/sbin;
export HOSTNAME=node1
echo "node1 minisystem start..."
``` 
```
[busybox-1.36.1]# chmod a+x rootfs/etc/init.d/rcS
```

7. 构建     
>       
```
[busybox-1.36.1]# find ./rootfs | cpio -o -H newc | gzip > rootfs.img
```
