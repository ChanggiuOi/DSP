# FPGA Test 

## 1. Blink led bằng giao thức UART

- Lap là khối Uart_Tx truyền tín hiệu (lệnh) cho FPGA (Tích hợp sẵn led) là khối Uart_Rx để tắt/bật/nháy led theo yêu cầu

```
module uart_blink (
    input clk,          
    input rst_n,        
    input uart_rx,      
    output reg led      
);
    parameter CLK_FREQ  = 50000000;
    parameter BAUD_RATE = 115200;
    parameter BIT_TICK  = CLK_FREQ / BAUD_RATE;

    reg [15:0] tick_cnt; //bien dem so chu ki clk (FPGA) de tinh thoi gian 1 bit du lieu 
    reg [3:0]  bit_cnt; //bien dem soluong bit
    reg [8:0]  rx_shift_reg; //thanh ghi nhan du lieu
    reg        is_receiving; //co trang thai bao dang nhan du lieu (co <=0 thi dang trang thai idle, cho START Bit
                             //                                        <=1 thi dang nhan data bit, sao cho tick_cnt du 8bit
    reg        rx_data_ready; // co trang thai bao da nhan xong du lieu
    reg [7:0]  rx_data; // thanh ghi chua 8 bit data nhan duoc tu uart_tx
    
    reg uart_rx_d1, uart_rx_d2;
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            uart_rx_d1 <= 1'b1;
            uart_rx_d2 <= 1'b1;
        end else begin
            uart_rx_d1 <= uart_rx;
            uart_rx_d2 <= uart_rx_d1;
        end
    end

    always @(posedge clk or negedge rst_n) begin //( THEO CHUAN 8N1)
        if (!rst_n) begin
            tick_cnt      <= 0;
            bit_cnt       <= 0;
            is_receiving  <= 0;
            rx_data_ready <= 0;
            rx_data       <= 8'h00;
        end else begin
            rx_data_ready <= 1'b0; 
            
            if (!is_receiving) begin
                if (uart_rx_d2 == 1'b0) begin
                    is_receiving <= 1'b1;
                    tick_cnt     <= BIT_TICK / 2; // Lay mau tai giua bit_tick
                    bit_cnt      <= 0;
                end
            end else begin
                if (tick_cnt < BIT_TICK - 1) begin
                    tick_cnt <= tick_cnt + 1;
                end else begin
                    tick_cnt <= 0;
                    if (bit_cnt < 8) begin
                        rx_shift_reg <= {uart_rx_d2, rx_shift_reg[8:1]};
                        bit_cnt      <= bit_cnt + 1;
                    end else begin
                        // Đã nhận đủ 8 bit dữ liệu
                        rx_data       <= rx_shift_reg[7:0];
                        rx_data_ready <= 1'b1;
                        is_receiving  <= 0;
                    end
                end
            end
        end
    end
    
    reg [1:0] mode;
    reg [25:0] blink_cnt; // thay doi bien nay giup led nhay nhanh/cham

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            mode <= 2'b00;
        end else if (rx_data_ready) begin
            case (rx_data)
                8'h30: mode <= 2'b00; // '0' tat
                8'h31: mode <= 2'b01; // '1' bat
                8'h62: mode <= 2'b10; // 'b' Nhap nhay
                default: ;
            endcase
        end
    end

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            blink_cnt <= 0;
        end else begin
            blink_cnt <= blink_cnt + 1;
        end
    end

    always @(*) begin //Hien thi Led tren module FPGA
        case (mode)
            2'b00:   led = 1'b0;          // Tắt (hoặc 1 tùy phần cứng active-low/high)
            2'b01:   led = 1'b1;          // Bật
            2'b10:   led = blink_cnt[24]; // Nhấp nháy theo xung đếm
            default: led = 1'b0;
        endcase
    end

endmodule
```

## 2. Giao thức UART cho ESP32 và FPGA