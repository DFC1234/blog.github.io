> FDMA是基于 AXI4 总线协议定制的一个 DMA 控制器。

此 IP 可以实现用 FPGA 代码直接读写 PL 的 DDR 或 ZYNQ/ZYNQMP SOC PS 的 DDR 或BRAM。


> FDMA 的写时序

<img width="1057" height="297" alt="Image" src="https://github.com/user-attachments/assets/0e5d6b54-4618-4dce-ba5f-2a8da081e789" />

fdma_wready = 1， fdma_wbusy=0 表示总线非忙，可以进行新的 FDMA 传输。

使fdma_wreq=1，写入fdma burst 的起始地址和 fdma_wsize （需要传输的数据大小）。

fdma_wvalid=1 ，给出有效的数据，写入 AXI 总线。

当最后一个数传输完成后，fdma_wvalid 和fdma_wbusy = 0。

> AXI4 最大的 burst lenth 是 256，封装后， fdma_size 可以任意大小，ip 自动控制每次Burst 长度，简化了 AXI4 总线的使用

