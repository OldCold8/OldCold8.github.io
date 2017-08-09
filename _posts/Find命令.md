---
layout: post
title: "Find命令"
date:  2017-08-07
description: "Find查找命令的使用"
tag: 博客 
---

##正文
* find搜索：实时查找工具，通过遍历指定路径完成文件查找
   * 工作特点：
            查找速度略慢
            精确查找
            实时查找
            可能只搜索用户具备读取和执行权限的目录
   * find [OPTION]... [查找路径] [查找条件] [处理动作]
           查找路径：指定具体目标路径；默认为当前目录
           查找条件：指定的查找标准，可以文件名、大小、类型、权限等标准进行；默认为找出指定路径下的所有文件
           处理动作：对符合条件的文件做操作，默认输出至屏幕
* 查找条件
   * 指搜索层级：
          -maxdepth level 最大搜索目录深度,指定目录为第1级
          -mindepth level 最小搜索目录深度
   * 根据文件名和inode查找：
          -name "文件名称"：支持使用glob *, ?, [], [^]
          -iname "文件名称"：不区分字母大小写
          -inum n 按inode号查找
          -samefile name 相同inode号的文件
          -links n 链接数为n的文件
          -regex "PATTERN"：以PATTERN匹配整个文件路径字符串，而不仅仅是文件名称
   * 根据属主、属组查找：
          -user USERNAME：查找属主为指定用户(UID)的文件
          -group GRPNAME: 查找属组为指定组(GID)的文件
          -uid UserID：查找属主为指定的UID号的文件
          -gid GroupID：查找属组为指定的GID号的文件
          -nouser：查找没有属主的文件
          -nogroup：查找没有属组的文件
   *  find示例
          find -name snow.png  ：搜索名为snow.png的文件
          find -iname snow.png ：不分大小写地搜索名为snow.png、Snow.png、SNOW.PNG等等的文件
          find / -name “*.txt”
          find /var –name “*log*”
          find -user joe -group joe ：搜索被用户joe 以及组群joe所拥有的文件
          find -user joe -not -group joe
          find -user joe -o -user jane
          find -not \( -user joe -o -user jane \)
          find / -user joe -o -uid 500
      找出/tmp目录下，属主不是root，且文件名不以f开头的文件
            find /tmp \( -not -user root -a -not -name 'f*' \) -ls
            find /tmp -not \( -user root -o -name 'f*' \) –ls
        排除目录:示例：查找/etc/下，除/etc/sane.d目录的其它所有.conf后缀的文件
            find /etc -path ‘/etc/sane.d’ -a -prune-o -name “*.conf”
            find /etc \(–path ‘/etc/sane.d’ –o –path ’/etc/fonts’ \)-a prune –o name “*.conf” 
   * 根据文件类型查找： 
         -type TYPE: 
            f: 普通文件  
            d: 目录文件 
            l: 符号链接文件  
            s：套接字文件  
            b: 块设备文件 
            c: 字符设备文件 
            p: 管道文件
   * 根据文件大小来查找：
         -size [+|-] #UNIT 常用单位：k, M, G，c（byte）
                #UNIT: (#-1, #] 如：6k 表示(5k,6k]
                -#UNIT：[0,#-1] 如：-6k 表示[0,5k]
                +#UNIT：(#,∞)   如：+6k 表示(6k,∞)
   * 根据时间戳：
         以“天”为单位； 
            -atime [+|-]#,
                   #: [#,#+1)
                   +#: [#+1,∞]
                   -#: [0,#)
            -mtime 
            -ctime
         以“分钟”为单位： 
            -amin 
            -mmin 
            -cmin
         ##注释： atime：文件读取时间
                 mtime：文件修改时间
                 ctime：状态修改时间（属性）
   * 根据权限查找：
         -perm [/|-]MODE
               MODE: 精确权限匹配
               /MODE：任何一类(u,g,o)对象的权限中只要能一位匹配即可，或关系，+ 从centos7开始淘汰
               -MODE：每一类对象都必须同时拥有指定权限，与关系
               0 表示不关注
        示例
            find -perm 755 会匹配权限模式恰好是755的文件
            只要当任意人有写权限时，find -perm +222就会匹配（"+"centos6支持, centos7支持"/"）
            只有当每个人都有写权限时，find -perm -222才会匹配
            只有当其它人（other）有写权限时，find -perm -002才会匹配（用户和用户组的0不表示0权限 表示不关注）
   * 组合条件：
         与：-a
         或：-o
         非：-not, ! 
   * 德·摩根定律：
         (非 A) 或 (非 B) = 非(A 且 B)
         (非 A) 且 (非 B) = 非(A 或 B)
         示例：
            !A -a !B = !(A -o B)
            !A -o !B = !(A -a B)
* 处理动作
      -print：默认的处理动作，显示至屏幕
      -ls：类似于对查找到的文件执行“ls -l”命令
      -delete：删除查找到的文件
      -fls file：查找到的所有文件的长格式信息保存至指定文件中
      -ok COMMAND {} \; 对查找到的每个文件执行由COMMAND指定的命令，对于每个文件执行命令之前，都会交互式要求用户确认
      -exec COMMAND {} \; 对查找到的每个文件执行由COMMAND指定的命令
      {}: 用于引用查找到的文件名称自身
      find传递查找到的文件至后面指定的命令时，查找到所有符合3条件的文件一次性传递给后面的命令

* 参数替换xargs
  * 由于很多命令不支持管道|来传递参数，而日常工作中有这个必要，所以就有了xargs命令
  xargs用于产生某个命令的参数，xargs 可以读入 stdin 的数据，并且以空格符或回车符将 stdin 的数据分隔成为arguments
  * 注意：文件名或者是其他意义的名词内含有空格符的情况
  有些命令不能接受过多参数，命令执行可能会失败，xargs可以解决
  * 示例：
        ls f* |xargs rm
        find /sbin -perm +700 |ls -l 这个命令是错误的
        find /sbin -perm +7000 | xargs ls –l
        find和xargs格式：find | xargs COMMAND
* find示例
      find -name “*.conf” -exec cp {} {}.orig \;  备份配置文件，添加.orig这个扩展名
      find /tmp -ctime +3 -user joe -ok rm {} \;  提示删除存在时间超过３天以上的joe的临时文件
      find ~ -perm -002 -exec chmod o-w {} \;     在你的主目录中寻找可被其它用户写入的文件
      find /data –type f -perm 644 -name “*.sh” –exec chmod 755 {} \; data目录下权限为644的普通文件找到并修改权限为755
      find /home –type d -ls 找到并列出家目录下的目录文件
* 练习：
    * 查找/var目录下属主为root，且属组为mail的所有文件 
          find /var \( -user root -a -group mail \)
    * 查找/var目录下不属于root、lp、gdm的所有文件
          find /var -not \( -user root -a -user lp -a -user gdm \)
    * 查找/var目录下最近一周内其内容修改过，同时属主不为root，也不是postfix的文件
          find /var -mtime -7 -not \( -user root -a -user postfix \)
    * 查找当前系统上没有属主或属组，且最近一个周内曾被访问过的文件
          find -atime -7 \( -nouser -o -nogroup \)
    * 查找/etc目录下大于1M且类型为普通文件的所有文件
          find -type f -size +1M
    * 查找/etc目录下所有用户都没有写权限的文件
          find /etc ! -perm /222
    * 查找/etc目录下至少有一类用户没有执行权限的文件
          find /etc ! -perm -111
    * 查找/etc/init.d目录下，所有用户都有执行权限，且其它用户有写权限的文件
          find /etc/init.d/ -perm -113 -ls
