# pahole

pahole（poke-a-hole）是用于显示和操纵数据结构布局的工具。

pahole显示以调试信息格式编码的数据结构布局，支持 DWARF，CTF和BTF。

pahole指定的文件必须具有相关的调试信息。一种方法是在编译时指定`-g`选项。

默认情况下，pahole显示出指定的文件所有具名结构体的布局。如果没有指定文件，pahole会在`/sys/kernel/btf/vmlinux`里查找。比如：
```c
// pahole task_struct
struct task_struct {
        struct thread_info         thread_info;          /*     0    24 */

        /* XXX last struct has 4 bytes of padding */

        unsigned int               __state;              /*    24     4 */
        ...
}
```

选项：
* -C, --class_name=CLASS_NAMES：只显示指定名字的结构体
* -E, --expand_types：展开结构体成员

更多信息参考pahole man page