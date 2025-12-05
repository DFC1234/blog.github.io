# user 空间
把设备内存直接映射到用户空间
# uixdmairq  IP使用
此ip配合xdma驱动处理中断

此ip提供AXI-ilte接口 ，上位机通过访问user空间地址 ，读写IP的寄存器

此ip输入一个user_irq_req_i信号 ，提供中断 ，输出到xdma ip 。

在上位机响应xdma中断时，上位机在中断里面写入 uixdmairq  IP的寄存器


