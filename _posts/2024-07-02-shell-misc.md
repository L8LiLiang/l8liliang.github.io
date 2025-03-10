---
layout: article
tags: Shell
title: Shell Misc
mathjax: true
key: Linux
---

[soruce](https://blog.csdn.net/Free_time_/article/details/107428503)
{:.info} 

[soruce](https://www.cnblogs.com/stono/p/13844552.html)
{:.info} 

## grep 零宽断言
```
grep高级用法——断言
先行断言 x(?=y)
当x的右边是y是匹配成功

后发断言 (?<=y)x
当x的左边是y时匹配成功

负向零宽先行断言 x(?!y)
x右边不是y时匹配成功

负向零宽后发断言 (?<!y)x
x左边不是y时匹配成功

需要使用-P选项

[root@liali-vm1 job_scheduler]# grep  -P -o "(?<=SKIP_DRIVER\=\").*?(?=\")" -R list/
list/sriov.list:bnx2x
list/sriov.list:igb
list/sriov.list:bnx2x
list/sriov.list:bnx2x
list/sriov.list:bnx2x
list/sriov.list:cxgb4
list/sriov.list:be2net
list/sriov.list:be2net
list/sriov.list:qlcnic bnx2x liquidio
list/sriov.list:qlcnic bnx2x liquidio
list/sriov.list:qlcnic bnx2x liquidio
list/sriov.list:qlcnic bnx2x liquidio
list/sriov.list:mlx4_en
list/sriov.list:qlcnic bnx2x cxgb4 liquidio
list/sriov.list:qlcnic bnx2x sfc ixgbe igb mlx4_en be2net cxgb4 liquidio
list/sriov.list:mlx4_en cxgb4 liquidio
list/sriov.list:nfp
list/sriov.list:cxgb4 nfp
list/sriov.list:be2net cxgb4
list/sriov.list:be2net cxgb4
list/sriov.list:liquidio cxgb4 igb be2net bnx2x mlx4_en sfc ice
list/sriov.list:bnxt_en cxgb4
list/sriov.list:mlx4_en
list/qinq.list:cxgb4 be2net bnx2x igb ionic ixgbe mlx4_en nfp sfc bnxt_en qede
list/qinq.list:cxgb4 be2net bnx2x igb ionic ixgbe mlx4_en nfp sfc bnxt_en qede
list/qinq.list:cxgb4 be2net bnx2x igb ionic ixgbe mlx4_en nfp sfc bnxt_en qede

```

## grep \b
```
https://unix.stackexchange.com/questions/22700/what-does-b-mean-in-a-grep-pattern

\b in a regular expression means "word boundary".

With this grep command, you are searching for all words i in the file linux.txt. i can be at the beginning of a line or at the end, or between two space characters in a sentence.

More precisely, not only space, but any non-word character. Just like the -w --word-regexp switch does: grep -w "i" linux.txt. For example a line like "<i>italic</i>" also matches. 

```

[soruce](https://www.junmajinlong.com/shell/bash_source/)
{:.info} 

## BASH_SOURCE & $0
```
${BASH_SOURCE[0]}等价于$BASH_SOURCE,永远表示当前的代码文件（就是BASH_SORUCE这个代码所在的文件）
$0表示的是，当前执行的是哪个程序。

一个shell(bash)脚本有两种执行方式：

直接执行，类似于执行二进制程序
source加载，类似于加载库文件
$0保存了被执行脚本的程序名称。注意，它保存的是以二进制方式执行的脚本名而非以source方式加载的脚本名称。

例如，执行a.sh时，a.sh中的$0的值是a.sh，如果a.sh执行b.sh，b.sh中的$0的值是b.sh，如果a.sh中source b.sh，则b.sh中的$0的值为a.sh。

除了$0，bash还提供了一个数组变量BASH_SOURCE，该数组保存了bash的SOURCE调用层次。这个层次是怎么体现的，参考下面的示例。

执行shell脚本a.sh时，shell脚本的程序名a.sh将被添加到BASH_SOURCE数组的第一个元素中，即${BASH_SOURCE[0]}的值为a.sh，这时${BASH_SOURCE[0]}等价于$0。

---- 当在a.sh中执行b.sh时：
# a.sh中的$0和BASH_SOURCE
$0 --> "a.sh"
${BASH_SOURCE[0]} -> "a.sh"

# b.sh中的$0和BASH_SOURCE
$0 --> "b.sh"
${BASH_SOURCE[0]} -> "b.sh"


---- 当在a.sh中source b.sh时：
# a.sh中的$0和BASH_SOURCE
$0 --> "a.sh"
${BASH_SOURCE[0]} -> "a.sh"

# b.sh中的$0和BASH_SOURCE
$0 --> "a.sh"
${BASH_SOURCE[0]} -> "b.sh"
${BASH_SOURCE[1]} -> "a.sh"

---当在a.sh中source b.sh时，如果b.sh中还执行了source c.sh，那么：
# a.sh中的$0和BASH_SOURCE
$0 --> "a.sh"
${BASH_SOURCE[0]} -> "a.sh"

# b.sh中的$0和BASH_SOURCE
$0 --> "a.sh"
${BASH_SOURCE[0]} -> "b.sh"
${BASH_SOURCE[1]} -> "a.sh"

# c.sh中的$0和BASH_SOURCE
$0 --> "a.sh"
${BASH_SOURCE[0]} -> "c.sh"
${BASH_SOURCE[1]} -> "b.sh"
${BASH_SOURCE[2]} -> "a.sh"

---使用脚本来验证一下BASH_SOURCE和$0。在x.sh中source y.sh，在y.sh中source z.sh：
#~ /tmp/x.sh
cat >/tmp/x.sh<<'eof'
source y.sh
echo "x.sh: ${BASH_SOURCE[@]}"
eof

#~ /tmp/y.sh
cat >/tmp/y.sh<<'eof'
source z.sh
echo "y.sh: ${BASH_SOURCE[@]}"
eof

#~ /tmp/z.sh
cat >/tmp/z.sh<<'eof'
echo "z.sh: ${BASH_SOURCE[@]}"
eof

---执行x.sh输出结果：
$ bash /tmp/x.sh
z.sh: z.sh y.sh /tmp/x.sh
y.sh: y.sh /tmp/x.sh
x.sh: /tmp/x.sh
```

## awk next & continue
```
next 是跳过当前行（awk自身是列循环和行循环的结合）；

continue是跳过当前循环（跳过列循环）；

(base) [b20223040323@admin1 test2]$ ls
a.txt
(base) [b20223040323@admin1 test2]$ cat a.txt                  ## 测试文本
001 002 003 004 005 006 007 008 009 010
011 012 013 014 015 016 017 018 019 020
021 022 023 024 025 026 027 028 029 030
031 032 033 034 035 036 037 038 039 040
041 042 043 044 045 046 047 048 049 050
051 052 053 054 055 056 057 058 059 060
061 062 063 064 065 066 067 068 069 070
071 072 073 074 075 076 077 078 079 080
081 082 083 084 085 086 087 088 089 090
091 092 093 094 095 096 097 098 099 100
(base) [b20223040323@admin1 test2]$ awk '{if(NR % 2 == 0) {next}; print $0}' a.txt
001 002 003 004 005 006 007 008 009 010                         ## 遇到偶数行，直接跳过
021 022 023 024 025 026 027 028 029 030
041 042 043 044 045 046 047 048 049 050
061 062 063 064 065 066 067 068 069 070
081 082 083 084 085 086 087 088 089 090


(base) [b20223040323@admin1 test2]$ cat a.txt                      ## 测试文件
001 002 003 004 005 006 007 008 009 010
011 012 013 014 015 016 017 018 019 020
021 022 023 024 025 026 027 028 029 030
031 032 033 034 035 036 037 038 039 040
041 042 043 044 045 046 047 048 049 050
051 052 053 054 055 056 057 058 059 060
061 062 063 064 065 066 067 068 069 070
071 072 073 074 075 076 077 078 079 080
081 082 083 084 085 086 087 088 089 090
091 092 093 094 095 096 097 098 099 100                             ## continue跳过当前的loop
(base) [b20223040323@admin1 test2]$ awk '{for(i = 1; i <= NF; i++) {if(i % 2 == 0) {continue} else {printf("%s ", $i)}}; printf("\n")}' a.txt
001 003 005 007 009
011 013 015 017 019
021 023 025 027 029
031 033 035 037 039
041 043 045 047 049
051 053 055 057 059
061 063 065 067 069
071 073 075 077 079
081 083 085 087 089
091 093 095 097 099
```

## Disable Kernel Parameter for Sending ICMP Redirects
```
https://unix.stackexchange.com/questions/57941/linux-always-send-icmp-redirect
https://docs.datadoghq.com/security/default_rules/xccdf-org-ssgproject-content-rule-sysctl-net-ipv4-conf-default-send-redirects/

echo 0 | tee /proc/sys/net/ipv4/conf/*/send_redirects

ipv6: https://www.quora.com/How-do-I-make-Linux-stop-sending-IPv6-ICMP-redirects
```

## enable ip forward
```
echo 1 > /proc/sys/net/ipv4/ip_forward
echo 1 > /proc/sys/net/ipv6/conf/all/forwarding 
```
