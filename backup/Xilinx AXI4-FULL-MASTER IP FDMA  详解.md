> FDMA是基于 AXI4 总线协议定制的一个 DMA 控制器。

此 IP 可以实现用 FPGA 代码直接读写 PL 的 DDR 或 ZYNQ/ZYNQMP SOC PS 的 DDR 或BRAM。


> FDMA 的写时序

<img width="1057" height="297" alt="Image" src="https://github.com/user-attachments/assets/0e5d6b54-4618-4dce-ba5f-2a8da081e789" />

fdma_wready = 1， fdma_wbusy=0 表示总线非忙，可以进行新的 FDMA 传输。

使fdma_wreq=1，设置fdma burst 的起始地址和 fdma_wsize （需要传输的数据大小）。

fdma_wvalid=1 ，给出待发送数据。

最后一个数传输完成后，fdma_wvalid 和fdma_wbusy = 0。

> AXI4 最大的 burst lenth 是 256，封装后， fdma_size 可以任意大小，ip 自动控制每次Burst 长度，简化了 AXI4 总线的使用

> FDMA 的读时序

<img width="1097" height="291" alt="Image" src="https://github.com/user-attachments/assets/e32806dc-7d84-4a6f-8bdd-1f2c0e36c499" />

fdma_rready = 1， fdma_rbusy=0 时 FDMA 总线非忙，可以进行次新的 FDMA 传输。

使fdma_rreq=1，设置fdma burst 的起始地址和 fdma_rsize （需要传输的数据大小）(以 bytes 为单位)。

当 fdma_rvalid=1 ，给出待发送数据。

最后一个数写完后，fdma_rvalid 和 fdma_rbusy = 0。

> FDMA IP 的封装

使用编写好的 uiFDMA.v 进行封装

<img width="837" height="142" alt="Image" src="https://github.com/user-attachments/assets/2e49ab1f-df9d-45b2-b606-6db29f71c367" />

> 添加 uiFDMA.v 源码

<img width="858" height="357" alt="Image" src="https://github.com/user-attachments/assets/80f529b5-65c8-49e6-8ad8-09a3ed15b3a2" />

> 创建 IP

<img width="862" height="442" alt="Image" src="https://github.com/user-attachments/assets/de41cd0c-355a-490a-b49e-ca38f0d40f73" />

<img width="857" height="436" alt="Image" src="https://github.com/user-attachments/assets/f6698c41-af62-4c4f-bd51-dc685a9eda14" />

<img width="843" height="674" alt="Image" src="https://github.com/user-attachments/assets/b5ba4cb7-cb65-474b-9857-df9e7673d2f2" />

<img width="884" height="415" alt="Image" src="https://github.com/user-attachments/assets/b18d3816-5c28-4f73-886a-22bd6b6edcec" />

按住 shift 全选后，右击弹出菜单后选择 Create Interface Definition

<img width="870" height="405" alt="Image" src="https://github.com/user-attachments/assets/b962b324-79e2-4b3c-8524-41f84e300125" />

接口定义为 slave，命名为 FDMA_S

<img width="866" height="490" alt="Image" src="https://github.com/user-attachments/assets/1438b573-ea62-460e-9db1-a4e85951bba5" />

uisrc/03_ip/uifdma 路径下多出 2 个文件，这个两个文件就是自定义的总线接口

<img width="756" height="336" alt="Image" src="https://github.com/user-attachments/assets/f4da0149-6b1b-4338-8211-0bf391669a8f" />

看到封装后的总线

<img width="878" height="395" alt="Image" src="https://github.com/user-attachments/assets/641a880e-7afe-443f-a32f-2b983a3f08d6" />

可以改名字

<img width="1057" height="609" alt="Image" src="https://github.com/user-attachments/assets/f211b701-0937-4db2-aec7-2c2f25c5d64b" />

封装好的接口

<img width="1059" height="482" alt="Image" src="https://github.com/user-attachments/assets/54123b33-094d-4a4f-a858-4ce56717f602" />

<img width="1078" height="525" alt="Image" src="https://github.com/user-attachments/assets/fe33808f-a691-4b52-8f80-28650e13aef2" />



