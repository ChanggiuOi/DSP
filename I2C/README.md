#I2C

## 1. Giao thức I2C

### 1.1. Giao thức I2C là gì?

- I2C là giao thức nối tiếp đồng bộ, cho phép vdk giao tiếp với nhiều thiết bị ngoại vi và dữ liệu được truyền theo từng bit 1. I2C chỉ sử dụng 2 dây dẫn: đường truyền dữ liệu (SDA) và đường xung nhịp đồng bộ (SCL)

![alt](Anh1.png)

- Trong 1 sơ đồ giao tiếp I2C:

	+ SDA: đường truyền đẻ master và slave gửi và nhận dữ liệu
	
	+ SCL: đường mang tín hiệu xung nhịp

	+ GND: nối chung 2 thiết bị xuống đất để lấy điện áp tham chiếu
	
### 1.2. Message

- Với I2C, dữ liệu được truyền trong các tin nhắn. Các tin nhắn này được chia thành các khung dữ liệu

![alt](Anh2.png)

	+ Điều kiện Start: Đường SDA kéo điện áo từ HIGH xuống LOW trước khi SCL chuyển từ HIGH xuống LOW
	
	![alt](Anh4.png)
	
	+ Điều kiện Stop: Đường SDA kéo điện áo từ LOW lên HIGH sau khi SCL chuyển từ LOW lên HIGH
	
	![alt](Anh5.png)
	
	+ Address Frame (Khung địa chỉ): Một chuỗi 7 hoặc 10 bit duy nhất cho mỗi slave để xác định slave khi master muốn giao tiếp với nó
	
	+ Read/Write Bit: Một bit chỉ định của master gửi đến slave
	
		_ bit 0: (Write) Ghi dữ liệu (master gửi dữ liệu slave)
		
		_ bit 1: (Read) Đọc dữ liệu (master yêu cầu dữ liệu từ slave -> slave gửi dữ liệu cho master)
		
	+ ACK/NACK Bit: Một bit xác nhận xem đã gửi thành công dữ liệu chưa	
		
		_ bit 0: ACK (đã nhận thành công dữ liệu)
		
		_ bit 1: NACK
		
	![alt](Anh6.png)
		
	+ Data Frame (Khung dữ liệu): Một chuỗi 8 bit chứa dữ liệu mong muốn nhận/gửi từ master/slave
	
### 1.2.1. Start and Stop Conditions

![alt](Anh8.png)

### 1.2.2. Data Validity and Byte Format

![alt](Anh9.png)

- SDA(đường dữ liệu nối tiếp) hoạt động phụ thuộc vào SCL (đường xung nhịp nối tiếp) khi SCL kéo lên cao thì SDA mới truyền được dữ liệu

- SCL luôn do Master tạo (nên mới có trường hợp khi FPGA làm master thì phải tính 1 loại xung nhịp mới dựa theo xung clk)

### 1.2.3. ACK and NACK

![alt](Anh10.png)

- ACK khi SDA kéo xuống thấp

- NACK khi SDA kéo lên cao

### 1.2.4. Ghi dữ liệu vào Slave

![alt](Anh11.png)

### 1.2.5. Đọc dữ liệu trong Slave

![alt](Anh12.png)

- Repeated START giống tín hiệu START bình thường, chỉ khác ở chỗ không cần STOP trước đó và Bus vẫn được MASTER giữ

- Việc sử dụng Repeated START nhằm chuyển đổi giữa 2 trạng thái R và W mà Master không quên địa chỉ cần mong muốn trước đó (thay vì dùng STOP rồi sang START khiến bus giải phóng, không nhớ địa chỉ)

//### 1.3. Một master với nhiều slave

//![alt](Anh3.png)

//- Một master có thể đọc/ghi dữ liệu từ nhiều slave bằng cách xác định đúng adddress frame trong Message

//### 1.4. Nhiều master với nhiều slave

//![alt](Anh7.png)

//- Nếu 2 master Start tại cùng 1 thời điểm, muốn yêu cầu cùng đọc/ghi dữ liệu tại cùng 1 slave sẽ xảy ra Arbitration (phân xử bus)

//	+ Vì I2C cùng open-drain, nên master yêu cầu Write luôn thắng (được truyền dữ liệu trước)
	
//	+ Master thua sẽ dừng điểu khiển SDA, chuyển sang chỉ theo dõi bus chờ Master thắng truyền xong, khi bus rảnh sẽ truyền lại mà KHÔNG CẦN RESET BUS VÀ DỮ LIỆU 
	
### 1.5. Tri-sate buffer (được FPGA sử dụng khi giao tiếp I2C với các ngoại vi)

![alt](Anh13.png)

- Trong đó:

	+ X: Dữ liệu Master muốn xuất ra
	
	+ Y: chân vật lú SDA nối ra ngoài
	
	+ EN: Tín hiệu cho phép FPGA điều khiển SDA

- Tri-State sẻ dụng khi nhiều thiết bị cùng chia sẻ một đường tín hiệu (bus)

#### 1.5.1. Một Master - Một Slave (FPGA là Master)

- EN = 1: FPGA đang truyền dữ liệu (START, địa chỉ, dữ liệu, ACK của Master), nên Y = X.

- EN = 0: FPGA nhả SDA về trạng thái Z để Slave gửi ACK hoặc dữ liệu; lúc này FPGA chỉ đọc giá trị trên SDA.

	+ Slave điều khiển đường SDA khi muốn gửi ACK hoặc gửi dữ liệu cho Master (bit R/W=1)

## 2. I2C 

- Code

```
module I2C(
	input wire clk,
	input wire rst,
	output reg i2c_sda,
	output reg i2c_scl);
	
	localparam STATE_IDLE = 0;
	localparam STATE_START = 1;
	localparam STATE_ADDR = 2;
	localparam STATE_RW = 3;
	localparam STATE_ACK = 4;
	localparam STATE_DATA = 5;
	localparam STATE_NACK = 6;
	localparam STATE_STOP = 7;
	
	reg [2:0] state;
	reg [6:0] addr;
	reg [7:0] count;
	reg [7:0] data;
	
	always @(posedge clk) begin
		if(rst) begin
			state <= 0;
			i2c_sda <= 1'b1;
			i2c_scl <= 1'b1;
			data <= 8'hA5; //example Data = 10100101
			addr <= 7'h50; //dia chi mac dinh cua mot so thiet bi i2c (EEPROM) = 1010000
			count <= 8'd0;
		end
		
		else begin
			case(state)
				STATE_IDLE: begin 
					i2c_sda <= 1'b1;
					i2c_scl <= 1'b1;
					state <= STATE_START;
				end
				
				STATE_START: begin
					i2c_scl <= 1'b1;
					i2c_sda <= 1'b0;
					state <= STATE_ADDR;
					count <= 6;
				end
				
				STATE_ADDR: begin
					i2c_scl <= 1'b0;
					i2c_sda <= addr[count];
					if(count==0) state <= STATE_RW;
					else count <= count-1;
				end
				
				STATE_RW: begin
					i2c_scl <= 1'b0;
					i2c_sda <= 1'b0; //Write
					state <= STATE_ACK;
				end
				
				STATE_ACK: begin
					state <= STATE_DATA;
					count <= 7;
				end
				
				STATE_DATA: begin
					i2c_scl <= 1'b0;
					i2c_sda <= data[count];
					if(count==0) state <= STATE_NACK;
					else count <= count - 1;
				end
				
				STATE_NACK: begin
					state <= STATE_STOP;
				end
				
				STATE_STOP: begin
					i2c_scl <= 1'b1;
					i2c_sda <= 1'b1;
					state <= STATE_IDLE;
				end
				
				default: 
					state <= STATE_IDLE;
			endcase
		end
	end

endmodule 
					
```

-TB

```
module I2C_tb;

    reg clk;
    reg rst;

    wire i2c_sda;
    wire i2c_scl;
	 
    I2C dut(
        .clk(clk),
        .rst(rst),
        .i2c_sda(i2c_sda),
        .i2c_scl(i2c_scl));

    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    initial begin
        rst = 1;

        #20;
        rst = 0;

        #400;// Chạy mô phỏng thêm

        $stop;
    end

endmodule
```
	
