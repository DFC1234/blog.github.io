<img width="810" height="542" alt="Image" src="https://github.com/user-attachments/assets/f4bcac51-eeff-4d9f-935b-18d81d8025a9" />

创建的文件放在 FPGA 工程目录的 pro\01_rtl 目录下 vivado自动识别到文件地址

<img width="550" height="380" alt="Image" src="https://github.com/user-attachments/assets/d1a56740-9009-4056-998d-ebecec484f0d" />

继续弹出的对话框中，可以设置端口，现在什么都不做。里面的moudle自动和.v文件相同 后续要调用这个

<img width="481" height="339" alt="Image" src="https://github.com/user-attachments/assets/5ba75a34-8beb-438b-a125-1963e593e1e1" />

创建完成后可以看到 Design Source的新文件

<img width="475" height="402" alt="Image" src="https://github.com/user-attachments/assets/d7d1024d-010e-44af-b828-8ed8a458ac3d" />




`
timescale 1ns / 1ns //仿真单位/仿真精度

module run_led              //定义模块名称

(

output [1:0] O_led         //输出 两位O_led 

);

assign O_led = 2'b0111; //assign赋值 O_led为后两位数字

endmodule                     //结束模块
1
`

> 流水灯实现

`timescale 1ns / 1ns
module run_led#
(                                                                                             //#定义预设值
parameter T_INR_CNT_SET = 32'd99_999_999             //分频时钟计数，初始时钟过快，人眼观察会导致 LED 常亮
)                                                                                             //设置分频系数，降低流水灯的变化速度，该参数可以由上层调用时修改
(
 input I_sysclk,                                     //系统时钟信号
 input I_rstn,                                        //全局复位
);

reg[32:0] t_cnt;                                   //定义寄存器
                        
always @(posedge I_sysclk or negedge I_rstn)begin     //系统时钟的上升沿触发以及复位的下降沿触发
   if(I_rstn==1'b0)                                //复位信号
     t_cnt <= 0;
   else if(t_cnt == T_INR_CNT_SET)  // t_cnt 达到目标时清零
     t_cnt <= 0;
   else
     t_cnt <= t_cnt + 1'b1;                    //else计数+1
end
endmodule

> 总体

`timescale 1ns / 1ns
module run_led#
(
parameter T_INR_CNT_SET = 32'd99_999_999 
) 
(
input I_sysclk, 
input I_rstn, 
output [3:0] O_led                   //LED 灯输出
);

reg [3:0] led_r;                        //LED 的寄存器，存储 LED 信号状态
reg [32:0] t_cnt;                      //计数器寄存器
                                                  //寄存器值仅在模块内部可访问 
assign O_led = led_r;              //加一个assign将寄存器内信号输出 

always @(posedge I_sysclk or negedge I_rstn)begin
if(I_rstn==1'b0) 
t_cnt <= 0;
else if(t_cnt == T_INR_CNT_SET) 
t_cnt <= 0;
else
t_cnt <= t_cnt + 1'b1; 
end

always @(posedge I_sysclk or negedge I_rstn)begin 
if(I_rstn==1'b0)
led_r <= 4'b0111;                  //设置 LED 寄存器的初始状态 <=为赋值
else if( t_cnt == 0)               //计数器寄存器达到预定值  开始一个循环
led_r <= {led_r[0],led_r[3:1]}; //LED 寄存器将最低为左移至最高位，(移位操作)
                                                   //{} 是拼接操作符，用于将多个信号或位拼接成一个新的信号。
end

endmodule

添加管脚约束文件
.xdc 文件存放到工程目录的 uisrc\04_pin 目录下，方便管理。
添加管脚约束有三种方法，分别是新建 XDC PIN 脚约束文件、添加已经写好的约束文件、综合后添加管脚约束，


#系统时钟周期约束
create_clock -period 10.000 -name sysclk [get_ports I_sysclk]
#时钟管脚物理，物理约束为具体的芯片管脚号约束
set_property PACKAGE_PIN V23 [get_ports I_sysclk]
#电平属性为 LVCMOS12,代表了 1V2 的 IO BANK,电平约束不会改版实际的 IO BANK 电平，如果电平约束和实际的 BANK 电平不匹配，可能会导致工作异常
set_property IOSTANDARD LVCMOS12 [get_ports I_sysclk]
#复位管脚约束，这里绑定到按键输入
set_property PACKAGE_PIN W19 [get_ports I_rstn]
#复位输入的电平约束为 1V8 的 IO BANK 电平
set_property IOSTANDARD LVCMOS12 [get_ports I_rstn]
#绑定 led 输出管脚到 FPGA IO 上
set_property PACKAGE_PIN AE18 [get_ports {O_led[1]}]
set_property PACKAGE_PIN AC16 [get_ports {O_led[0]}]
set_property IOSTANDARD LVCMOS12 [get_ports {O_led[*]}]
#对 bit 大小进行压缩，可以节省程序存储空间
set_property BITSTREAM.GENERAL.COMPRESS true [current_design]

综合

打开RTL 原理图

<img width="372" height="296" alt="Image" src="https://github.com/user-attachments/assets/94adeb56-189a-4669-8fd8-37323e6bd585" />

<img width="387" height="333" alt="Image" src="https://github.com/user-attachments/assets/0b872d5d-6d1b-4aac-988c-da8b3606229d" />

点击IO配置配置引脚

> RTL 仿真

编写 testbench 为了对使用的硬件描述语言设计的电路进行验证

`timescale                            //仿真单位/仿真精度
module Test_bench();        //通常 testbench 没有输入与输出端口
       信号或变量声明定义

       逻辑设计中输入对应 reg 型
       逻辑设计中输出对应 wire 型

       使用 initial 或 always 语句产生激励 语句产生激励
       
        例化设计模块

endmodule


`timescale 1ns / 1ns
module tb_run_led();
















> 添加ILA进行在线分析

<img width="792" height="409" alt="Image" src="https://github.com/user-attachments/assets/a37e7467-b8fe-4a8d-9c09-d9025019665d" />

<img width="767" height="440" alt="Image" src="https://github.com/user-attachments/assets/2ae8ec61-bbea-4367-994d-2779e4fd5818" />


> 视频驱动编写（VTC）

### CRT时序

         CRT 显示器扫描是一行一行进行的 在扫描完一行有效数据段之后不会立马返回，而是会继续向右扫描一段区域，这个区域称为右边界区，不在有效的显示范围内，可以理解为显示器右边的黑边。（左边同理）「行扫描」
        扫描过了右侧边界区域后，并不会自动回到最左边准备下一行，需要行同步信号脉冲（horizontal sync pulse，HSYC）。再没有收到HSYC之前，需要关闭来消隐，消隐是为了不影响已经点亮的像素点。当收到 HSYC 信号，电子枪会在一定时间内从最右侧回到显示屏的最左侧，这个回去的过程需要耗费一定的时间，这个时间就称为 horizontal back porch。当 HSYC 信号结束以后，电子枪重新打开，显示新的一行图像数据。「列扫描」
      扫描完有效行的图像后会继续向下扫描一段区域，这个区域称为下边界区域（vertical bottom border），该区域不在有效的显示范围内，可以理解为显示器下边的黑边。同样的，显示器上边也有这样一段黑边，在扫描有效数据之前，电子枪扫描到的这段区域不会显示图像, 这个区域称为上边界区域（vertical top border）。「上下黑框」
        扫描一场图像到达屏的最下方后，不会自动回到最上边，需要有一个通知信号，即场同步信号脉冲（vertical sync pulse，VSYC）。在没有收到场同步信号脉冲之前，需要关闭电子枪以实现消隐，这段时间就称为 vertical front porch，消隐是为了不影响已经点亮的像素点。当收到 VSYC 信号，开始回去 ，回去的过程需要耗费一定的时间，这个时间就称为vertical back porch。当 VSYC 信号结束以后，电子枪就会重新打开，就可以显示新的一行图像数据了。「换图像」
        
## 像素时钟

       像素时钟（Pixel clock，pclk），一个基准时钟 clk 对应一个像素点，根据行时序和场时序，显示一帧图像需要的基准时钟数 N(CLK) = H_FrameEnd *V_FrameEnd，（所有像素），像素时钟 = H_FrameEnd * V_FrameEnd * 帧率。

<img width="676" height="569" alt="Image" src="https://github.com/user-attachments/assets/68e564a6-fdbc-407a-9109-5bbf797cde00" />

<img width="511" height="223" alt="Image" src="https://github.com/user-attachments/assets/7bf07bd2-a4ee-431c-b4d6-70bc32032bb4" />

<img width="796" height="347" alt="Image" src="https://github.com/user-attachments/assets/376140d6-3c31-4afd-aebf-d07e90ab122d" />

      对于分辨率 1920*1080*60 的分辨率通常采用 148.5MHZ 的像素时钟。H_FrameSize* V_FrameSize*帧率 =
2,200* 1,125*60= 148,500,000，在时钟晶振足够精确的情况下，可以确保帧率以恒定 60fps 输出。

<img width="776" height="233" alt="Image" src="https://github.com/user-attachments/assets/dfbd0a36-27b5-4aff-b916-524e8273b4d3" />

## VTC 控制器程序设计












       














