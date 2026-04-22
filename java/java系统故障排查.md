## 排查思路

| 问题场景 | 排查步骤与命令 |
| :--- | :--- |
| **系统启动慢** | `jps -v` (看参数) -> `jstat -gcutil` (看 GC 是否频繁) |
| **CPU 飙高** | `top -H -p <pid>` (找线程) -> `printf "%x" <tid>` (转十六进制) -> `jstack <pid> \| grep <hex>` (定位代码) |
| **内存泄漏/OOM** | `jmap -heap` (看各区占用) -> `jmap -dump:live...` (导出快照) -> **MAT 分析** |
| **接口响应慢/死锁** | `jstack <pid>` (看线程是否 blocked 或 waiting) -> 搜索 `waiting on` / `locked` |
| **频繁 Full GC** | `jstat -gcutil <pid> 1000` (持续观察 O/U 比例和 FGC 次数) |