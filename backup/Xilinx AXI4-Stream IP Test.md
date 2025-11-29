> 添加 IP

<img width="508" height="130" alt="Image" src="https://github.com/user-attachments/assets/7474d678-ddb6-49d0-bd95-5040d6c56f95" />

> 完成连线

<img width="529" height="133" alt="Image" src="https://github.com/user-attachments/assets/10ef4177-901d-4541-9307-d53c814fad70" />

> Master IP 参数配置

<img width="670" height="278" alt="Image" src="https://github.com/user-attachments/assets/fe2f6228-f087-4855-a5aa-38e13faba304" />

> Slave IP 参数配置

<img width="660" height="275" alt="Image" src="https://github.com/user-attachments/assets/cec4bfe3-b343-440b-ab00-d82ba262a7a8" />

> 自动创建顶层文件

<img width="389" height="252" alt="Image" src="https://github.com/user-attachments/assets/3c3364a1-bd3f-4892-a55f-33fe9a5fed6c" />

> 仿真文件

```verilog
`timescale 1ns / 1ns

module axis_top_sim();                                                //可随便定义仿真模块名字

    reg m00_axis_aclk_0;                                               //定义寄存器驱动时钟
    reg m00_axis_aresetn_0;                                         //定义寄存器驱动复位信号

    initial begin                                                                //定义时钟信号初始值
        m00_axis_aclk_0 = 1'b0;
    end

    always begin                                                              //产生时钟信号  5个周期反转一次
        #5 m00_axis_aclk_0 = ~m00_axis_aclk_0;
    end

    system_wrappertem system_wrapper_inst (          //例化自动生成的HDL文件
        .m00_axis_aclk_0    (m00_axis_aclk_0),
        .m00_axis_aresetn_0 (m00_axis_aresetn_0)
    );

    initial begin
        m00_axis_aresetn_0 = 1'b0;                               //仿真开始时，复位
        #100;                                                                     //100个周期后，消除复位信号
        m00_axis_aresetn_0 = 1'b1;
    end

endmodule

```






