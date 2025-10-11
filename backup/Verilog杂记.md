# 三八译码

## 主模块

```verilog
module decoder3_8( 
    a,
    b,
    c,
    out
);

    input a; // 输入端口A
    input b; // 输入端口B
    input c; // 输入端口C
    
    output [7:0] out; // 输出端口

    reg [7:0] out;
    
    always @(a, b, c)
    begin
        case({a, b, c})
            3'b000: out = 8'b0000_0001;
            3'b001: out = 8'b0000_0010;
            3'b010: out = 8'b0000_0100;
            3'b011: out = 8'b0000_1000;
            3'b100: out = 8'b0001_0000;
            3'b101: out = 8'b0010_0000;
            3'b110: out = 8'b0100_0000;
            3'b111: out = 8'b1000_0000;
        endcase    
    end

endmodule
```  
### 分析
模块名字叫decoder3_8  在仿真文件里使用decoder3_8 u1  对其例化
开头使用一下的格式列出总体信号列表
```  
module decoder3_8( 
		a,
		b,
		c,
		out
	);
```  
然后列出输入输出端口  设置寄存器
``` 
        input a;
	input b;
	input c;
	output [7:0]out;
	reg [7:0]out;
``` 
译码器核心部分
``` 
always@(a,b,c)
	begin
		case({a,b,c})
			3'b000:out = 8'b0000_0001;
			3'b001:out = 8'b0000_0010;
			3'b010:out = 8'b0000_0100;
			3'b011:out = 8'b0000_1000;
			3'b100:out = 8'b0001_0000;
			3'b101:out = 8'b0010_0000;
			3'b110:out = 8'b0100_0000;
			3'b111:out = 8'b1000_0000;
		endcase	
	end
``` 
组合逻辑 用case判断a,b,c在不同时候的输入 ，返回对应的值 
一个case块要 一个endcase块 


一个always块要一个begin 一个end




