树莓派-家庭NAS(1)   https://www.jianshu.com/p/9be7ada37863
树莓派-家庭NAS(2)   https://www.jianshu.com/p/91405ca824b8
树莓派-家庭NAS(3)   https://www.jianshu.com/p/80777ed85246

树莓派端服务安装过程可以分为几个步骤：
1. 安装树莓派操作系统。在网上可以找到各种类型的这里不再赘述。
2. 挂载硬盘。
3. 安装Docker。
4. 启动Docker镜像。

下面按步骤说明。

## 挂载硬盘

* #### 安装ntfs-3g
    ``` 
    $ sudo apt-get install ntfs-3g 
    ```
* #### 挂载硬盘
    获取硬盘描述符：
    ``` 
     $ sudo fdisk -l 
    ```   
    ```
    Disk /dev/sdb: 1.8 TiB, 2000365289472 bytes, 3906963456 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x7a911a43

    Device     Boot      Start        End    Sectors  Size Id Type
    /dev/sdb1               63 1048578614 1048578552  500G  7 HPFS/NTFS/exFAT
    /dev/sdb2       1048578615 2097157229 1048578615  500G  7 HPFS/NTFS/exFAT
    dev/sdb3       2097157230 3906959804 1809802575  863G  7 HPFS/NTFS/exFAT
    ```
    获取硬盘的blkid：
    ``` 
    $ sudo blkid /dev/sdb1
    ```
    ```
    /dev/sdb1: LABEL="M-fM-^VM-0M-eM-^JM- M-eM-^MM-7" UUID="565CF4865CF461E3" TYPE="ntfs" PARTUUID="7a911a43-01"
    ```
    修改配置文件/etc/fstab:
    ```(bash)
    $ sudo cat /etc/fstab cat /etc/fstab 
    ```
    ```
    proc            /proc           proc    defaults          0       0
    PARTUUID=c0edcca8-01  /boot           vfat    defaults          0       2
    PARTUUID=c0edcca8-02  /               ext4    defaults,noatime  0       1
    # a swapfile is not a swap partition, no line here
    #   use  dphys-swapfile swap[on|off]  for that
    PARTUUID=7a911a43-01  /mnt/share/1               ntfs-3g    defaults,noatime  0       1
    PARTUUID=7a911a43-02  /mnt/share/2               ntfs-3g    defaults,noatime  0       1
    PARTUUID=7a911a43-03  /mnt/share/3               ntfs-3g    defaults,noatime  0       1
    ```
* #### 重启树莓派：
    ``` 
    sudo restart 
    ```

## 安装Docker
* #### 安装Docker软件
    ```
    $ sudo curl -sSL https://get.docker.com | sh
    ```
* #### 配置Docker镜像源：
    ```
    $ sudo nano /etc/docker/daemon.json
    ```
    ```
    {
      "registry-mirrors": ["https://registry.docker-cn.com"]
    }
    ```
* #### 启动Docker服务：
    ```
    $ sudo systemctl restart docker.service 
    $ sudo systemctl enable docker.service
    ```

## 启动Docker
* #### 启动Samba服务：
    ```
    cat docker_samba.sh 
    #!/bin/bash
    docker run -d \
      -p 137:137/udp \
      -p 138:138/udp \
      -p 139:139 \
      -p 445:445 \
      -p 445:445/udp \
      --restart='always' \
      --hostname 'filer' \
      -v /mnt/share:/share/ \    
      --name samba docker.io/dastrasmue/rpi-samba \
      -s "Public:/share/:rw:"
    ```
* #### 启动NextCloud服务：
    ```
    cat docker_nextcloud.sh 
    #! /bin/bash
    docker run -d -p 10024:80 \
        -v /mnt/share/3/nextcloud/nextcloud:/var/www/html \
        -v /mnt/share/3/nextcloud/apps:/var/www/html/custom_apps \
        -v /mnt/share/3/nextcloud/config:/var/www/html/config \
        -v /mnt/share/3/nextcloud/data:/var/www/html/data \
        -v /mnt/share/3/nextcloud/theme:/var/www/html/themes/ \
        -v /mnt/share/3/nextcloud/util.php:/var/www/html/lib/private/legacy/util.php \
        docker.io/arm32v7/nextcloud
    ```

## 检查效果
* #### Docker情况：
    ```
    $ sodu docker ps
    ```
    ```
    CONTAINER ID        IMAGE                                          COMMAND                CREATED             STATUS              PORTS                                                                                                                            NAMES
    a98033083c4e        docker.io/arm32v7/nextcloud:latest             "/entrypoint.sh apac   2 days ago          Up 2 days           0.0.0.0:10024->80/tcp                                                                                                            backstabbing_pike      
    43a39a0de681        docker.io/dastrasmue/rpi-samba:latest          "/run.sh -s Public:/   4 weeks ago         Up 3 days           137/tcp, 138/tcp, 0.0.0.0:137->137/udp, 0.0.0.0:138->138/udp, 0.0.0.0:139->139/tcp, 0.0.0.0:445->445/tcp, 0.0.0.0:445->445/udp   samba
    ```
* #### 浏览器打开：
![NextCloud](https://upload-images.jianshu.io/upload_images/2454595-a56891afad3112eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

哈哈，好了。下面就可以使用这个工具在公网上访问我们的家用NAS服务器了。
