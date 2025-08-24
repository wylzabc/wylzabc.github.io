记录了修改Linux系统默认启动内核的几种方式。

# Ubuntu 24.04/22.04

### GRUB启动项

查看系统中有哪些内核版本及其索引：
`grep "menuentry" /boot/grub/grub.cfg` 命令输出中每行为实例内核启动菜单中的索引项，从0开始编号。例如，对于下面的Ubuntu 24.04实例，有6.8.0-71-generic和6.8.0-54-generic两个内核版本，grub菜单中有两层菜单结构。

![Image](https://github.com/user-attachments/assets/83357809-c8ec-4350-b080-570437dc6cea)

- `menuentry 'Ubuntu'﻿`
    - 默认启动项索引号为`0`，启动项菜单标题为`Ubuntu`。唯一标识符menuentry_id_option为`gnulinux-simple-48906949-0bb0-44c4-9dc2-2134ae62d9b8`。
- `submenu 'Advanced options for Ubuntu'﻿`
    - 子菜单索引号为`1`，子菜单标题为`Advanced options for Ubuntu`。唯一标识符为`gnulinux-advanced-48906949-0bb0-44c4-9dc2-2134ae62d9b8`。 
    - `menuentry 'Ubuntu, with Linux 6.8.0-71-generic'﻿`
        - 启动项索引号为`"1>0"`，启动项菜单标题为`'Ubuntu, with Linux 6.8.0-71-generic'`，即6.8.0-71-generic内核启动项。唯一标识符为`gnulinux-6.8.0-71-generic-advanced-48906949-0bb0-44c4-9dc2-2134ae62d9b8`。 
    - `menuentry 'Ubuntu, with Linux 6.8.0-71-generic (recovery mode)'﻿`
        - 启动项索引号为`"1>1"`，启动项菜单标题为`Ubuntu, with Linux 6.8.0-71-generic (recovery mode)`，即6.8.0-71-generic内核的恢复模式启动项。唯一标识符为`gnulinux-6.8.0-71-generic-recovery-48906949-0bb0-44c4-9dc2-2134ae62d9b8`。
    - `menuentry 'Ubuntu, with Linux 6.8.0-54-generic'﻿`
        - 启动项索引号为`"1>2"`，启动项菜单标题为`Ubuntu, with Linux 6.8.0-54-generic`，即6.8.0-54-generic内核启动项。唯一标识符为`gnulinux-6.8.0-54-generic-advanced-48906949-0bb0-44c4-9dc2-2134ae62d9b8`。
    - `menuentry 'Ubuntu, with Linux 6.8.0-54-generic (recovery mode)'﻿`
        - 启动项索引号为`"1>3"`，启动项菜单标题为`Ubuntu, with Linux 6.8.0-54-generic (recovery mode)`，即6.8.0-54-generic内核的恢复模式启动项。唯一标识符为`gnulinux-6.8.0-54-generic-recovery-48906949-0bb0-44c4-9dc2-2134ae62d9b8`。

注意：
1. `"X>Y"` 中X为主菜单序号，Y为子菜单序号，>前后无空格‌。需要带上双引号。 
2. ﻿`menuentry_id_option`是唯一标识符，格式为 `gnulinux-simple-<磁盘UUID>`、`gnulinux-<kernel-version>-advanced-<磁盘UUID>`或者`gnulinux-<kernel-version>-recovery-<磁盘UUID>`。 
3. 可以采用启动项菜单标题方式指定默认启动项。例如，`"1>2"`可以改为`"Advanced options for Ubuntu>Ubuntu, with Linux 6.8.0-54-generic"`。 
4. 可以采用唯一标识符方式指定默认启动项。例如，`"1>2"`可以改为`"gnulinux-advanced-48906949-0bb0-44c4-9dc2-2134ae62d9b8>gnulinux-6.8.0-54-generic-advanced-48906949-0bb0-44c4-9dc2-2134ae62d9b8"`。

## 修改启动项

以上图Ubuntu 24.04实例为例，修改为以6.8.0-54-generic内核启动。

<meta charset="UTF-8"><meta charset="UTF-8"><div class="mp-table-align"><div class="mp-table-wrapper"><div data-slate-node="element" style="padding-left:0px" class="mp-table-container">
方法 | 方法1：修改/etc/default/grub持久修改启动项 | 方法2：使用saved_entry方法持久修改启动项 | 方法3：使用next_entry临时修改启动项
-- | -- | -- | --
是否持久化 | 持久化 | 持久化 | 临时修改一次
是否需要更新grub.cfg | 每次修改都需要执行update-grub命令更新grub.cfg | 只在第一次在/etc/default/grub中设置GRUB_DEFAULT=saved时执行update-grub命令更新grub.cfg。后续使用grub-set-default或者grub-editenv命令直接更新grubenv设置。 | 不需要更新grub.cfg。直接使用grub-reboot或者grub-editenv命令更新
优点 | 适用于全局配置，可指定任意菜单项 | 更安全，避免直接编辑配置文件导致的语法错误 | 方便进行临时修改
缺点 | 每次更新启动项都要更新grub.cfg | 需要设置GRUB_DEFAULT=saved，否则grubenv设置不会生效 | 非持久化修改
使用场景 | 需要深度定制启动顺序（如调整子菜单优先级）需兼容非saved模式的特殊配置（如固定索引启动） | 多系统环境下快速切换默认启动项脚本自动化管理（如内核升级后自动设置新内核为默认项） | 测试新内核或修复启动问题
修改方式 | 修改GRUB_DEFAULT配置，并执行update-grub | 若GRUB_DEFAULT未设置为saved，需修改需GRUB_DEFAULT=saved，并执行update-grub﻿使用grub-set-default命令修改grubenv中saved_entry设置 | 只需执行grub-reboot命令修改grubenv中next_entry设置

</div><div class="mp-table-serialize-flag"><br/></div></div></div><span data-morpho-doc-data='{"token":"eyJhbGciOiJkaXIiLCJlbmMiOiJBMjU2R0NNIiwiYXBwSWQiOjEsInVpZCI6InBjeHlvcDV3YTYiLCJkb2NJZCI6IjI1S3BFM2RvSkJmNjlEIn0..n94SMFJDna8AMwbP.0myqnjNxG0I4y1yMbEiKMtQrfHJWw8Gm-R0zdyZngbhZ5TJarUaHjQNCzIjAbwjmzZ4C_9zqJU1_BBtJxNiBqpL1OTzLxp2tPFW83hfrxrcqtxJBmBOhmRKk1DWvMInFxXKd_Ve7mMHX3EY58NMYfP-2hoiQK_-Bwlg8iFV1kQHYVZt2_Ui5wHBDkNwGNhIVJRzQh9julA_Rger4i8rWg_nX8Q.MF05KaxgCnZxoAJ16VaEnQ","appId":"1"}' class='mp-morpho-clipboard-doc-data'>﻿</span>

### 方法1：修改/etc/default/grub持久修改启动项

操作步骤：

1. 修改`/etc/default/grub`，将GRUB_DEFAULT配置项设置为`"1>2"`、`"Advanced options for Ubuntu>Ubuntu, with Linux 6.8.0-54-generic"`或者`"gnulinux-advanced-48906949-0bb0-44c4-9dc2-2134ae62d9b8>gnulinux-6.8.0-54-generic-advanced-48906949-0bb0-44c4-9dc2-2134ae62d9b8"`。
<img width="960" height="401" alt="Image" src="https://github.com/user-attachments/assets/f3ce0b3c-91c8-44bd-8cbc-74bbd23c8bdb" />

2. 执行`update-grub` 。之后可以在`/boot/grub/grub.cfg` 配置文件中看到default被设置为`"1>2"` 。
<img width="568" height="592" alt="Image" src="https://github.com/user-attachments/assets/10f7d5d5-80d6-44a2-a147-6a337dfb424e" />

3. 重启实例。在vnc中可以看到GRUB图形界面中选中`Advanced options for Ubuntu` 子菜单。之后，加载6.8.0-54-generic内核。登录实例，`uname -r` 查看当前启动的内核版本。

<img width="952" height="566" alt="Image" src="https://github.com/user-attachments/assets/513e0fc7-fc13-4c25-8f2a-4cebbd7561ec" />

<img width="500" height="152" alt="Image" src="https://github.com/user-attachments/assets/0b2b028b-45fc-40f1-b38a-c3b1cf69663d" />

### 方法2：使用`saved_entry` 方法持久修改启动项

记录用户上次选择的启动项，需配合`GRUB_DEFAULT=saved` 使用，实现持久化记忆功能。
操作步骤：
1. 修改`/etc/default/grub` ，将GRUB_DEFAULT配置为`"saved"` ，之后执行`update-grub` 。检查`/boot/grub/grub.cfg` 文件，可以看到default被设置为`"${saved_entry}"` 

```bash
# 修改/etc/default/grub
GRUB_DEFAULT="saved"
GRUB_SAVEDEFAULT="true"  # 可选：自动保存新选择
sudo update-grub
```

<img width="602" height="751" alt="Image" src="https://github.com/user-attachments/assets/da8f9d2a-bb44-4010-b20f-8954f6ac7a11" />

2. 使用`grub-set-default` 命令指定内核启动。检查`/boot/grub/grubenv` 文件，看到`saved_entry` 被设置为`1>2` 
```
grub-set-default "1>2"
```

<img width="1649" height="165" alt="Image" src="https://github.com/user-attachments/assets/77429748-b7e9-4976-88a7-b51e129c317b" />

3. 重启实例。在vnc中可以看到GRUB图形界面中选中`Advanced options for Ubuntu`子菜单。之后，加载6.8.0-54-generic内核。

### 方法3：使用`next_entry`临时修改启动项

临时设置下一次启动的菜单项（仅生效一次），重启后恢复原配置，常用于测试新内核或修复启动问题。
无需配合`GRUB_DEFAULT=saved`使用。
操作步骤：
1. 执行`grub-reboot "1>2"`命令，设置下次临时使用6.8.0-54-generic内核启动。使用`grub-editenv list`命令可以看到`next_entry=1>2`设置。

<img width="960" height="131" alt="Image" src="https://github.com/user-attachments/assets/8fec8d6f-e471-4efd-9f99-3637fbd629d2" />

<img width="960" height="93" alt="Image" src="https://github.com/user-attachments/assets/26e04077-fa45-4d39-8fa6-a5f4044524a6" />

2. 重启实例。在vnc中可以看到GRUB图形界面中选中`Advanced options for Ubuntu`子菜单。之后，加载6.8.0-54-generic内核。登录实例，`uname -r`查看当前启动的内核版本。

# Rocky Linux 9/8、CentOS 8

## GRUB启动项

Rocky Linux 默认使用`GRUB_DEFAULT=saved`设置。GRUB菜单启动项在`/boot/loader/entries/`目录下。如下图，Rocky Linux 9.4实例有两个内核5.14.0-427.13.1.el9_4.x86_64和5.14.0-427.42.1.el9_4.x86_64。

```
[root@image-test ~]# ls /boot/loader/entries/
63d587c2144840dcbb5dff9795a00d50-0-rescue.conf  63d587c2144840dcbb5dff9795a00d50-5.14.0-427.13.1.el9_4.x86_64.conf  63d587c2144840dcbb5dff9795a00d50-5.14.0-427.42.1.el9_4.x86_64.conf
[root@image-test ~]# cd /boot/loader/entries/
[root@image-test entries]# ls
63d587c2144840dcbb5dff9795a00d50-0-rescue.conf  63d587c2144840dcbb5dff9795a00d50-5.14.0-427.13.1.el9_4.x86_64.conf  63d587c2144840dcbb5dff9795a00d50-5.14.0-427.42.1.el9_4.x86_64.conf
[root@image-test entries]# cat 63d587c2144840dcbb5dff9795a00d50-0-rescue.conf 
title Rocky Linux (0-rescue-63d587c2144840dcbb5dff9795a00d50) 9.4 (Blue Onyx)
version 0-rescue-63d587c2144840dcbb5dff9795a00d50
linux /boot/vmlinuz-0-rescue-63d587c2144840dcbb5dff9795a00d50
initrd /boot/initramfs-0-rescue-63d587c2144840dcbb5dff9795a00d50.img
options root=UUID=b8b13ace-e420-4f42-b36f-6ee51a44fda1 ro net.ifnames=0 console=tty0 console=ttyS0,115200 intel_idle.max_cstate=1 nopti nospectre_v2 nospec_store_bypass_disable 
grub_users $grub_users
grub_arg --unrestricted
grub_class rocky
[root@image-test entries]# cat 63d587c2144840dcbb5dff9795a00d50-5.14.0-427.13.1.el9_4.x86_64.conf
title Rocky Linux (5.14.0-427.13.1.el9_4.x86_64) 9.4 (Blue Onyx)
version 5.14.0-427.13.1.el9_4.x86_64
linux /boot/vmlinuz-5.14.0-427.13.1.el9_4.x86_64
initrd /boot/initramfs-5.14.0-427.13.1.el9_4.x86_64.img $tuned_initrd
options root=UUID=b8b13ace-e420-4f42-b36f-6ee51a44fda1 ro net.ifnames=0 console=tty0 console=ttyS0,115200 intel_idle.max_cstate=1 nopti nospectre_v2 nospec_store_bypass_disable 
grub_users $grub_users
grub_arg --unrestricted
grub_class rocky
[root@image-test entries]# cat 63d587c2144840dcbb5dff9795a00d50-5.14.0-427.42.1.el9_4.x86_64.conf
title Rocky Linux (5.14.0-427.42.1.el9_4.x86_64) 9.4 (Blue Onyx)
version 5.14.0-427.42.1.el9_4.x86_64
linux /boot/vmlinuz-5.14.0-427.42.1.el9_4.x86_64
initrd /boot/initramfs-5.14.0-427.42.1.el9_4.x86_64.img $tuned_initrd
options root=UUID=b8b13ace-e420-4f42-b36f-6ee51a44fda1 ro net.ifnames=0 console=tty0 console=ttyS0,115200 intel_idle.max_cstate=1 nopti nospectre_v2 nospec_store_bypass_disable 
grub_users $grub_users
grub_arg --unrestricted
grub_class rocky
```

可以通过grubby命令获取启动项信息，并设置默认启动项。

```
# grubby --info=ALL
index=0
kernel="/boot/vmlinuz-5.14.0-427.42.1.el9_4.x86_64"
args="ro net.ifnames=0 console=tty0 console=ttyS0,115200 intel_idle.max_cstate=1 nopti nospectre_v2 nospec_store_bypass_disable"
root="UUID=b8b13ace-e420-4f42-b36f-6ee51a44fda1"
initrd="/boot/initramfs-5.14.0-427.42.1.el9_4.x86_64.img $tuned_initrd"
title="Rocky Linux (5.14.0-427.42.1.el9_4.x86_64) 9.4 (Blue Onyx)"
id="63d587c2144840dcbb5dff9795a00d50-5.14.0-427.42.1.el9_4.x86_64"
index=1
kernel="/boot/vmlinuz-5.14.0-427.13.1.el9_4.x86_64"
args="ro net.ifnames=0 console=tty0 console=ttyS0,115200 intel_idle.max_cstate=1 nopti nospectre_v2 nospec_store_bypass_disable"
root="UUID=b8b13ace-e420-4f42-b36f-6ee51a44fda1"
initrd="/boot/initramfs-5.14.0-427.13.1.el9_4.x86_64.img $tuned_initrd"
title="Rocky Linux (5.14.0-427.13.1.el9_4.x86_64) 9.4 (Blue Onyx)"
id="63d587c2144840dcbb5dff9795a00d50-5.14.0-427.13.1.el9_4.x86_64"
index=2
kernel="/boot/vmlinuz-0-rescue-63d587c2144840dcbb5dff9795a00d50"
args="ro net.ifnames=0 console=tty0 console=ttyS0,115200 intel_idle.max_cstate=1 nopti nospectre_v2 nospec_store_bypass_disable"
root="UUID=b8b13ace-e420-4f42-b36f-6ee51a44fda1"
initrd="/boot/initramfs-0-rescue-63d587c2144840dcbb5dff9795a00d50.img"
title="Rocky Linux (0-rescue-63d587c2144840dcbb5dff9795a00d50) 9.4 (Blue Onyx)"
id="63d587c2144840dcbb5dff9795a00d50-0-rescue"
```

与Ubuntu系统类似，Rocky Linux系统也可以通过修改saved_entry为特定的index、id等来更新系统默认启动内核。

## 修改默认启动项

以上述Rocky Linux 9.4实例为例，修改为以5.14.0-427.13.1.el9_4.x86_64内核启动。

<meta charset="UTF-8"><meta charset="UTF-8"><div class="mp-table-align"><div class="mp-table-wrapper"><div data-slate-node="element" style="padding-left:0px" class="mp-table-container">
方法 | 方法1：grubby持久化修改启动内核（推荐） | 方法2：修改/etc/default/grub持久修改启动项 | 方法3：使用next_entry临时修改启动项
-- | -- | -- | --
是否持久化 | 持久化 | 持久化 | 临时修改一次
是否需要更新grub.cfg | 系统/etc/default/grub默认配置了GRUB_DEFAULT=saved无需更改grub.cfg。若改成了其它配置，需要重新改为RUB_DEFAULT=saved，并更新grub.cfg。后续使用grubby命令直接更新grubenv设置。 | 每次修改都需要执行grub2-mkconfig命令更新grub.cfg | 不需要更新grub.cfg。直接使用grub2-reboot命令更新
优点 | 更安全，避免直接编辑配置文件导致的语法错误 | 适用于全局配置，可指定任意菜单项 | 方便进行临时修改
缺点 | 需要设置GRUB_DEFAULT=saved，否则grubenv设置不会生效 | 每次更新启动项都要更新grub.cfg | 非持久化修改
使用场景 | 多系统环境下快速切换默认启动项脚本自动化管理（如内核升级后自动设置新内核为默认项） | 需要深度定制启动顺序（如调整子菜单优先级）需兼容非saved模式的特殊配置（如固定索引启动） | 测试新内核或修复启动问题
修改方式 | 若GRUB_DEFAULT未设置为saved，需修改需GRUB_DEFAULT=saved，并执行grub2-mkconfig -o /boot/grub2/grub.cfg命令使用grubby命令修改grubenv中saved_entry设置 | 修改GRUB_DEFAULT配置，并执行grub2-mkconfig -o /boot/grub2/grub.cfg命令 | 只需执行grub-reboot命令修改grubenv中next_entry设置

</div><div class="mp-table-serialize-flag"><br/></div></div></div><span data-morpho-doc-data='{"token":"eyJhbGciOiJkaXIiLCJlbmMiOiJBMjU2R0NNIiwiYXBwSWQiOjEsInVpZCI6InBjeHlvcDV3YTYiLCJkb2NJZCI6IjI1S3BFM2RvSkJmNjlEIn0..n94SMFJDna8AMwbP.0myqnjNxG0I4y1yMbEiKMtQrfHJWw8Gm-R0zdyZngbhZ5TJarUaHjQNCzIjAbwjmzZ4C_9zqJU1_BBtJxNiBqpL1OTzLxp2tPFW83hfrxrcqtxJBmBOhmRKk1DWvMInFxXKd_Ve7mMHX3EY58NMYfP-2hoiQK_-Bwlg8iFV1kQHYVZt2_Ui5wHBDkNwGNhIVJRzQh9julA_Rger4i8rWg_nX8Q.MF05KaxgCnZxoAJ16VaEnQ","appId":"1"}' class='mp-morpho-clipboard-doc-data'>﻿</span>

### 方法1：grubby持久化修改启动内核

在Rocky Linux系统中，/etc/default/grub默认设置了`GRUB_DEFAULT=saved`。可以直接通过grubby命令修改启动内核。

操作步骤：
1. 使用grubby命令修改默认启动项为5.14.0-427.13.1.el9_4.x86_64内核。可通过`grubby --set-default="/boot/vmlinuz-5.14.0-427.13.1.el9_4.x86_64"`或者`grubby --set-default-index=1`命令操作。

<img width="960" height="94" alt="Image" src="https://github.com/user-attachments/assets/96caa8c5-70e4-4250-83c1-603ae044ff61" />

2. 重启实例，可以从vnc界面看到GRUB界面中从5.14.0-427.13.1.el9_4.x86_64内核进入系统。

### 方法2：修改`/etc/default/grub`持久修改启动项

在Rocky Linux系统中，将`/etc/default/grub`中`GRUB_DEFAULT`设置为指定内核index或者id。更新grub.cfg。此时，grubby设置的`saved_entry`启动项将不再起作用。

操作步骤：
1. 修改`/etc/default/grub`中`GRUB_DEFAULT`为`"1"`、`"63d587c2144840dcbb5dff9795a00d50-5.14.0-427.13.1.el9_4.x86_64"`或者`"Rocky Linux (5.14.0-427.13.1.el9_4.x86_64) 9.4 (Blue Onyx)"`。
2. 更新grub.cfg：`grub2-mkconfig -o /boot/grub2/grub.cfg`
3. 重启实例，可以从vnc界面看到GRUB界面中从5.14.0-427.13.1.el9_4.x86_64内核进入系统。

### 方法3：使用next_entry临时修改启动项
临时设置下一次启动的菜单项（仅生效一次），重启后恢复原配置，常用于测试新内核或修复启动问题。
同Ubuntu系统类似，在Rocky Linux系统中，可以使用`grub2-reboot`命令临时设置下次启动内核。
操作步骤：
1. 执行`grub2-reboot "1"`命令、`grub2-reboot "63d587c2144840dcbb5dff9795a00d50-5.14.0-427.13.1.el9_4.x86_64"`命令或者`grub2-reboot "Rocky Linux (5.14.0-427.13.1.el9_4.x86_64) 9.4 (Blue Onyx)"`命令。检查`/boot/grub2/grubenv`文件中`next_entry`字段被设置为指定启动项。

<img width="960" height="127" alt="Image" src="https://github.com/user-attachments/assets/a2f384b1-0749-41f3-8c98-1aeb8e58bb3e" />

2. 重启实例。在vnc中可以看到GRUB图形界面中选中`Rocky Linux (5.14.0-427.13.1.el9_4.x86_64) 9.4 (Blue Onyx)`菜单选项。之后，加载5.14.0-427.13.1.el9_4.x86_64内核。登录实例看到`next_entry`字段设置被清空。

<img width="960" height="129" alt="Image" src="https://github.com/user-attachments/assets/6c4d3716-c45d-41e1-8cec-c98e3f4c08fe" />

# 其他

## grub-editenv

> ~# grub-editenv --help
Usage: grub-editenv [OPTION...] FILENAME COMMAND
>
> Tool to edit environment block.
>
> Commands:
  create                     Create a blank environment block file.
  list                       List the current variables.
  set [NAME=VALUE ...]       Set variables.
  unset [NAME ...]           Delete variables.
>
> Options:
  -?, --help                 give this help list
      --usage                give a short usage message
  -v, --verbose              print verbose messages.
  -V, --version              print program version
>
> If FILENAME is `-', the default value /boot/grub/grubenv is used.
>
> There is no `delete' command; if you want to delete the whole environment
block, use `rm /boot/grub/grubenv'.
>
> Report bugs to <bug-grub@gnu.org>.

使用技巧：
```
# 创建空白grubenv文件，覆盖/boot/grub/grubenv设置
grub-editenv create
# 创建一个空白grubenv文件
grub-editenv /yourpath/grubenv create
# 查看grubenv设置列表
grub-editenv list
# 临时设置下一次驱动使用"1>2"启动菜单项
grub-editenv /boot/grub/grubenv set next_entry="1>2"
# 设置持久化启动项"1>2"，需确保/etc/default/grub中设置了GRUB_DEFAULT=saved，并且已更新/boot/grub/grub.cfg文件
grub-editenv /boot/grub/grubenv set saved_entry="1>2"
# 取消next_entry设置
grub-editenv /boot/grub/grubenv unset next_entry
```

## grub-set-default

> ~# grub-set-default --help
Usage: grub-set-default [OPTION] MENU_ENTRY
Set the default boot menu entry for GRUB.
This requires setting GRUB_DEFAULT=saved in /etc/default/grub.
>
>  -h, --help              print this message and exit
  -V, --version           print the version information and exit
  --boot-directory=DIR    expect GRUB images under the directory DIR/grub
                          instead of the /boot/grub directory
>
> MENU_ENTRY is a number, a menu item title or a menu item identifier.
>
> Report bugs to <bug-grub@gnu.org>.

使用技巧：
```
# 需确保/etc/default/grub中设置了GRUB_DEFAULT=saved，并且已更新/boot/grub/grub.cfg文件
# 设置持久化启动项"1>2"
grub-set-default "1>2"
# 检查
grub-editenv list
```

## grub-reboot

>~# grub-reboot -h
Usage: grub-reboot [OPTION] MENU_ENTRY
Set the default boot menu entry for GRUB, for the next boot only.
  -h, --help              print this message and exit
  -V, --version           print the version information and exit
  --boot-directory=DIR    expect GRUB images under the directory DIR/grub
                          instead of the /boot/grub directory
>
>MENU_ENTRY is a number, a menu item title or a menu item identifier. Please note that menu items in submenus or sub-submenus require specifying the submenu components and then the menu item component. The titles should be separated using the greater-than character (>) with no extra spaces. Depending on your shell some characters including may need escaping. More information about this is available in the GRUB Manual in the section about the 'default' command. 
>
>NOTE: In cases where GRUB cannot write to the environment block, such as when it is stored on an MDRAID or LVM device, the chosen boot menu entry will remain the default even after reboot. 
>
>Report bugs to <bug-grub@gnu.org>.

使用技巧：

```
# 临时设置下一次驱动使用"1>2"启动菜单项
grub-reboot "1>2"
# 检查
grub-editenv list
cat /boot/grub/grubenv
```

## grubby

>~# grubby --help
>Usage: grubby [OPTION...]
>      --add-kernel=kernel-path            add an entry for the specified kernel
>      --args=args                         default arguments for the new kernel or new arguments for kernel being updated)
>      --bad-image-okay                    don't sanity check images in boot entries (for testing only)
>  -c, --config-file=path                  path to grub config file to update ("-" for stdin)
>      --copy-default                      use the default boot entry as a template for the new entry being added; if the default is not a linux image, or if >the kernel referenced by the default image does not exist, the
>                                          first linux entry whose kernel does exist is used as the template
>      --default-kernel                    display the path of the default kernel
>      --default-index                     display the index of the default kernel
>      --default-title                     display the title of the default kernel
>      --env=path                          path for environment data
>      --grub2                             configure grub2 bootloader
>      --info=kernel-path                  display boot information for specified kernel
>      --initrd=initrd-path                initrd image for the new kernel
>  -i, --extra-initrd=initrd-path          auxiliary initrd image for things other than the new kernel
>      --make-default                      make the newly added entry the default boot entry
>      --remove-args=STRING                remove kernel arguments
>      --remove-kernel=kernel-path         remove all entries for the specified kernel
>      --set-default=kernel-path           make the first entry referencing the specified kernel the default
>      --set-default-index=entry-index     make the given entry index the default entry
>      --title=entry-title                 title to use for the new kernel entry
>      --update-kernel=kernel-path         updated information for the specified kernel
>      --zipl                              configure zipl bootloader
>  -b, --bls-directory                     path to directory containing the BootLoaderSpec fragment files
>      --no-etc-grub-update                don't update the GRUB_CMDLINE_LINUX variable in /etc/default/grub
>
>Help options:
>  -h, --help                              Show this help message

使用技巧：
```
# 查看所有启动项信息
grubby --info=ALL
# 查看默认启动项内核路径
grubby --default-kernel
# 查看默认启动项索引
grubby --default-index
# 查看默认启动项标题
grubby --default-title
# 查看指定内核的启动项信息
grubby --info=<kernel-path>
# 使用kernel-path设置默认启动内核
grubby --set-default=<kernel-path>
# 使用entry-index设置默认启动内核
grubby --set-default-index=<kernel-index>
```

## GRUB选项

**GRUB_DEFAULT**

>     The default menu entry.  This may be a number, in which case it identifies the Nth entry in the generated menu counted from zero, or the title of a menu entry, or the special string ‘saved’.  Using the id may be useful if you want to set a menu entry as the default even though there may be a variable number of entries before it.
>
>     For example, if you have:
>
>     menuentry 'Example GNU/Linux distribution' --class gnu-linux --id example-gnu-linux {
>        ...
>     }
>
>     then you can make this the default using:
>
>          GRUB_DEFAULT=example-gnu-linux
>
>     Previously it was documented the way to use entry title.  While this still works it's not recommended since titles often contain unstable device names and may be translated
>
>     If you set this to ‘saved’, then the default menu entry will be that saved by ‘GRUB_SAVEDEFAULT’ or ‘grub-set-default’.  This relies on the environment block, which may not be available in all situations (*note Environment block::).
>
>     The default is ‘0’.

**GRUB_SAVEDEFAULT**

>    If this option is set to ‘true’, then, when an entry is selected,
     save it as a new default entry for use by future runs of GRUB. This
     is only useful if ‘GRUB_DEFAULT=saved’; it is a separate option
     because ‘GRUB_DEFAULT=saved’ is useful without this option, in
     conjunction with ‘grub-set-default’.  Unset by default.  This
     option relies on the environment block, which may not be available
     in all situations (*note Environment block::).





