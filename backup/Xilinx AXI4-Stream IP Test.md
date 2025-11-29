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

> AXI Master替换代码


```verilog
`timescale 1 ns / 1 ps

module maxis_v1_0_M00_AXIS #(
    // 定义DATA位宽
    parameter integer C_M_AXIS_TDATA_WIDTH = 32,
    // 在 INIT_COUNTER 状态下 等待多长时间开始尝试第一次发送数据
    parameter integer C_M_START_COUNT      = 32
) (
    // 时钟 复位
    input wire M_AXIS_ACLK,
    input wire M_AXIS_ARESETN,

    // AXI-S信号
    output wire M_AXIS_TVALID,
    input wire M_AXIS_TREADY，

    output wire [C_M_AXIS_TDATA_WIDTH-1 : 0] M_AXIS_TDATA,
    output wire [(C_M_AXIS_TDATA_WIDTH/8)-1 : 0] M_AXIS_TSTRB,
    output wire M_AXIS_TLAST
    
);


    // 定义每次发送的数据包的长度
    localparam NUMBER_OF_OUTPUT_WORDS = 8;

    //函数clogb2  计算 WAIT_COUNT_BITS值
    function integer clogb2(input integer bit_depth);
        begin
            for (clogb2 = 0; bit_depth > 0; clogb2 = clogb2 + 1)
                bit_depth = bit_depth >> 1;
        end
    endfunction

    // WAIT_COUNT_BITS 等待计数器的位宽
    localparam integer WAIT_COUNT_BITS = clogb2(C_M_START_COUNT - 1);

    // bit_num 寻址 NUMBER_OF_OUTPUT_WORDS 数量所需的最少位数。
    localparam bit_num = clogb2(NUMBER_OF_OUTPUT_WORDS);


 
    // 读寄存器 (用于生成输出数据)
    reg [bit_num-1:0] read_pointer;
    // 等待计数器。主设备在发起传输前会等待用户定义的时钟周期数。
    reg [WAIT_COUNT_BITS-1 : 0] count;

 
    always @(posedge M_AXIS_ACLK) begin
        if (!M_AXIS_ARESETN) begin
            read_pointer <= 0;
        end else if (tx_en) begin
            // 仅当一次传输发生时，读指针才增加
            read_pointer <= read_pointer + 1;
        end
    end



    // AXI Stream 内部信号
    wire axis_tvalid;
    wire axis_tlast;
    wire tx_en;
    wire tx_done;

    //================================================================
    // 端口连接与赋值 (I/O Connections assignments)
    //================================================================
    assign M_AXIS_TVALID = axis_tvalid;
    assign M_AXIS_TDATA  = read_pointer; // 注意: 此处将读指针的值作为数据发送
    assign M_AXIS_TLAST  = axis_tlast;
    assign M_AXIS_TSTRB  = {(C_M_AXIS_TDATA_WIDTH/8){1'b1}}; // 所有字节选通始终有效





    // 定义状态机的状态
    parameter [1:0] IDLE         = 2'b00, // 初始/空闲状态
                    INIT_COUNTER = 2'b01, // 初始化计数器状态
                    SEND_STREAM  = 2'b10; // 发送流数据状态

   // 状态机变量
    reg [1:0] mst_exec_state;
 
    // 控制状态机实现
    // IDLE（空闲）、INIT_COUNTER（初始化计数器）、 SEND_STREAM（发送数据）

    always @(posedge M_AXIS_ACLK) begin //每次上升沿开始
        if (!M_AXIS_ARESETN) begin     //有复位信号

            mst_exec_state <= IDLE;  // 状态切换到IDLE
            count          <= 0;      //计数归零

        end else begin    //不复位时的操作

            case (mst_exec_state) //读mst_exec_state值 切换状态机
                

                //IDLE 空闲状态
                IDLE: begin 
                    mst_exec_state <= INIT_COUNTER;  //下一个时钟切换到INIT_COUNTER状态
                end


                //INIT_COUNTER  计数器延时状态
                INIT_COUNTER: begin   
                    if (count == C_M_START_COUNT - 1) begin   //计数满足延迟切换时间，切换到SEND_STREAM状态
                        mst_exec_state <= SEND_STREAM; 
                    end else begin //不满足条件 ，mst_exec_state计数+1
                        count <= count + 1;   
                        mst_exec_state <= INIT_COUNTER; //还是原来状态
                    end
                end


                 //SEND_STREAM  发送数据状态
                SEND_STREAM: begin  
                    if (tx_done) begin
                        mst_exec_state <= IDLE;
                    end else begin
                        mst_exec_state <= SEND_STREAM;
                    end
                end
                  
               
                //其他状态
                default: begin  
                    mst_exec_state <= IDLE;
                end
            endcase
        end
    end


    //产生 AXI Stream 信号
    // 当状态机处于 SEND_STREAM 状态且尚未发送完所有数据字时，tvalid 置为有效。
    assign axis_tvalid = (mst_exec_state == SEND_STREAM) && (read_pointer < NUMBER_OF_OUTPUT_WORDS);

    // 当主设备和从设备都准备好进行传输时，tx_en 置为有效。
    assign tx_en = M_AXIS_TREADY && axis_tvalid;

    // 在传输数据包的最后一个字时，tlast 置为有效。
    assign axis_tlast = (read_pointer == NUMBER_OF_OUTPUT_WORDS - 1) && tx_en;

    // 当最后一次传输完成时，tx_done 置为有效。
    assign tx_done = axis_tlast;

 

endmodule

```

>  Tcl Console 输入 reset_project 更新IP

<img width="701" height="160" alt="Image" src="https://github.com/user-attachments/assets/47297007-0d7c-4c42-a35e-2b1d24519d47" />

<img width="567" height="128" alt="Image" src="https://github.com/user-attachments/assets/8f0f226a-393a-44db-b39b-ce9787d9db37" />

<img width="964" height="178" alt="Image" src="https://github.com/user-attachments/assets/d181e212-ffca-41c2-8e2e-ba4710b3d8da" />





