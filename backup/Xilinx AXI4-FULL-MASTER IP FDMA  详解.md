> FDMA是基于 AXI4 总线协议定制的一个 DMA 控制器。

此 IP 可以实现用 FPGA 代码直接读写 PL 的 DDR 或 ZYNQ/ZYNQMP SOC PS 的 DDR 或BRAM。


> FDMA 的写时序

<img width="1057" height="297" alt="Image" src="https://github.com/user-attachments/assets/0e5d6b54-4618-4dce-ba5f-2a8da081e789" />

fdma_wready 设置为 1， fdma_wbusy=0 的时候代表 FDMA 的总线非忙，可以进行一次新的 FDMA 传输。
这个时候可以设置 fdma_wreq=1，同时设置 fdma burst 的起始地址和 fdma_wsize （本次需要传输的数据大小）。
当 fdma_wvalid=1 的时候需要给出有效的数据，写入 AXI 总线。当最后一个数写完后，fdma_wvalid 和fdma_wbusy 变为 0。
AXI4 总线最大的 burst lenth 是 256，而经过封装后，用户接口的 fdma_size 可以任意大小的，fdma ip 内部代码控制每次 AXI4 总线的 Burst 长度，这样极大简化了 AXI4 总线协议的使用。