# DSP (mặc định 32 bit)

##1. Cộng 2 số nhị phân có dấu 
```
module adder_fixedpoint(
	input [31:0] a,
	input [31:0] b,
	output reg [32:0] y);
	
	wire sign_a; //dau cua a
	wire sign_b; //dau cua b
	wire [30:0] mag_a; //tri tuyet doi cua a
	wire [30:0] mag_b; //tri tuyet doi cua b
	
	reg sign_y;
	reg [31:0] mag_y;
	
	assign sign_a=a[31];
	assign sign_b=b[31];
	assign mag_a=a[30:0];
	assign mag_b=b[30:0];
	
	always @(*) begin
		if (sign_a==sign_b) begin //neu a va b cung dau
			sign_y=sign_a;
			mag_y={1'b0,mag_a}+{1'b0,mag_b};
		end
		else begin
			if(mag_a>=mag_b) begin
				sign_y=sign_a;
				mag_y={1'b0,mag_a}-{1'b0,mag_b};
			end
			else begin
				sign_y=sign_b;
				mag_y={1'b0,mag_b}-{1'b0,mag_a};
			end
		end
		
		if (mag_y==32'd0)
			y=33'd0;
		else
			y={sign_y,mag_y};
	end
endmodule
```

## 2. Trừ 2 số nhị phân có dấu
```
module sub_fixedpoint(
	input [31:0] a,
	input [31:0] b,
	output [32:0] y);
	
	wire [31:0] b_neg; //so doi cua b
	
	assign b_neg=(b[30:0]==31'd0) ? 32'd0 : {~b[31],b[30:0]};
	
	adder_fixedpoint sub(a,b_neg,y);

endmodule 
```

## 3. Nhân 2 số nhị phân có dấu
```
module mul_fixedpoint(
	input [31:0] a,b,
	output [63:0] y);

	wire sign_a,sign_b,sign_y; //dau cua a,b,ket qua y
	
	wire [30:0] mag_a,mag_b; //tri tuyet doi a,b
	
	wire [61:0] mag_mul; //ket qua nhan 31 bit x 31 bit=62 bit
	wire [61:0] mag_mul_scale;
	wire [62:0] mag_y; //ket qua cua bus y
	
	assign sign_a=a[31];
	assign sign_b=b[31];
	
	assign mag_a=a[30:0];
	assign mag_b=b[30:0];
	
	assign sign_y = sign_a ^ sign_b; //dau cua output
	
	assign mag_mul=mag_a * mag_b; //gia tri tuyet doi cua output
	assign mag_mul_scale = mag_mul >> 16;
	
	assign mag_y={1'b0,mag_mul_scale};
	
	assign y=(mag_mul_scale == 62'd0) ? 64'd0 : ({sign_y,mag_y});
	
endmodule
```

## 4. Chia 2 số nhị phân có dấu
```
module div_fixedpoint(
	input [31:0] a,b,
	output [31:0] y,
	output div_by_zero);
	
	wire sign_a,sign_b,sign_y; //dau a,b, ket qua y
	
	wire [30:0] mag_a,mag_b; //gia tri tuyet doi cua a va b
	
	wire [63:0] bitexp; // số 64 bit chia so 32 bit còn 32 bit
	wire [63:0] div_mag_full;
	wire [30:0] dive_mag;
	
	assign sign_a=a[31];
	assign sign_b=b[31];
	assign mag_a=a[30:0],mag_b=b[30:0];
	
	assign sign_y=sign_a ^ sign_b;
	
	assign div_by_zero=(mag_b==31'b0); // bao loi khi chia cho 0
	
	assign bitexp = {17'd0,mag_a,16'd0}; // dich trai 16 bit (2 mu 16)
	
	assign div_mag_full=div_by_zero ? 64'd0 : (bitexp/mag_b);
	
	assign div_mag = div_mag_full[30:0]; //ketqua cuoi cung
	
	assign y=(div_by_zero || div_mag==31'd0) ?  32'd0 : {sign_y,div_mag};
	
endmodule
```

## 5. FP_main
```
module FP (a,b,op,add_result,sub_result,mul_result,div_result, div_by_zero,result);
    input  [31:0] a,b;
    input  [1:0]  op; 

    output [32:0] add_result,sub_result;
    output [63:0] mul_result;
    output [31:0] div_result;
    output        div_by_zero;

    output reg [63:0] result;
	 parameter add=2'd0,sub=2'd1,mul=2'd2,div=2'd3;

    adder_fixedpoint U1(a,b,add_result);
	 sub_fixedpoint U2(a,b,sub_result);
	 mul_fixedpoint U3(a,b,mul_result);
    div_fixedpoint U4(a,b,div_result,div_by_zero);

    always @(*) begin
        case (op)
            add: result = {31'd0, add_result};
            sub: result = {31'd0, sub_result};
            mul: result = mul_result;
            div: result = {32'd0, div_result};
            default: result = 64'd0;
        endcase
    end

endmodule
```