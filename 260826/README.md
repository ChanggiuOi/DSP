# ELE-D24-NguyenThiKieuTrang - Báo cáo công việc 26/08/2026

# A. Công việc đã làm

1. Tìm hiểu về STM32

2. GPIO

# B. Báo cáo chi tiết

## 1. Tìm hiểu về STM32

### 1.1. STM32 là gì?

- STM32 là một họ vi điều khiển 32-bit (MCU - Microcontroller Unit) do STMicroelectronics phát triển.

- Các dòng STM32 sử dụng các nhân xử lý thuộc họ ARM Cortex-M, vd: Cortex - M0/M0+, Cortex-M3,...

- STM32 không phải là tên của một vi điều khiển cụ thể mà là một họ vi điều khiển, gồm nhiều dòng như: STM32F0, STM32F1...

###  1.2. Các thành phần cơ bản bên trong STM32

- Một vi điều khiển STM32 có thể bao gồm:

	+ CPU: thực thi các lệnh của chương trình.

	+ Flash: lưu chương trình sau khi được nạp vào vi điều khiển.

	+ SRAM: lưu các biến và dữ liệu tạm thời trong quá trình chương trình chạy.

	+ GPIO: giao tiếp tín hiệu số với bên ngoài.

	+ Timer: đếm thời gian, tạo ngắt, PWM...

	+ ADC: chuyển đổi tín hiệu Analog sang Digital.

	+ DAC: chuyển đổi Digital sang Analog (chỉ có trên một số dòng).

	+ UART/USART: giao tiếp nối tiếp.

	+ SPI: giao tiếp nối tiếp đồng bộ tốc độ cao.

	+ I2C: giao tiếp với cảm biến, IC ngoại vi...

	+ DMA: hỗ trợ truyền dữ liệu giữa bộ nhớ và ngoại vi mà không cần CPU xử lý từng dữ liệu.

	+ NVIC: quản lý các nguồn ngắt.

	+ RCC: quản lý và cấp clock cho CPU và các ngoại vi.
	
### 1.3. Clk trong STM32

- STM32 là mạch số đồng bộ nên cần xung clock để hoạt động.

- Ngoài CPU, các ngoại vi như GPIO, Timer, UART, ADC... cũng cần được cấp clock trước khi sử dụng.

- Khối quản lý clock của STM32 được gọi là: RCC - Reset and Clock Control

- Ví dụ muốn sử dụng GPIOA trên STM32F1 bằng SPL: RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);

## 2. GPIO

### 2.1. Sơ lược lý thuyết

- GPIO (General purpose I/O ports) tạm hiểu và nơi giao tiếp chung giữa tín hiệu ra và tín hiệu vào

- Ở STM32 thì các chân GPIO chia làm nhiều port, vd: PortA, PortB,...

	+ Mỗi Port thường có 16 chân đánh số từ 0->15 tương ứng với mỗi chân là 1 bit (nhiều dòng có 32 chân)
	
	+ Môi chân có 1 chức năng khác nhau như analog input, external interrupt,... hay đơn thuần là xuất tín hiệu on/off ở mức 0,1
	
### 2.2. Các mode GPIO của STM32

- Input floating: Cấu hình chân I/O là ngõ vào và để nổi

- Input pull/up: Cấu hình chân I/O là ngõ vào, có trở kéo lên nguồn

- Input pull down: Cấu hình chân I/O là ngõ vào, có trở kéo xuống GND

- Analog: Cấu hinhf chân I/O là Analog, dùng cho các mode có sử dụng ADC hoặc DAC

- Output open-drain: Ở chế độ Open-Drain:

	+ Xuất 0: trans kéo chân GPIO xuống GND
	
	+ Xuất 1: trans ngắt và chân GPIO ở trạng thái trở kháng cao (Z)
	
	=> Do đó thường cần điện trở pull-up bên ngoài hoặc nội bộ tùy ứng dụng
	
	VD: bus I2C

- Output push-pull: Ở chế độ push-pull (output=0->Low, output=1->High)

	+ Xuất 0: chân được kéo xuống gần GND
	
	+ Xuất 1: chân được kéo lên gần VDD

- Alternate function push-pull: Chân GPIO được điều khiển bởi một ngoại vi bên trong STM32 thay vì phần mềm điều khiển GPIO thông thường, vd: USART TX, Timer PWM, SPI...

### 2.3. Các bước lập trình GPIO cơ bản

- Do mỗi chân vdk có nhiều chức năng, nên muốn sử dụng chức năng khác với mặc định thì ta phải cấu hình cho chân đó

- Và để các chân này hoạt động được thì ta phải cấp xung clk cho chân đó

- B1: Cấp xung clk cho ngoại vi 

	+ Lệnh RCC_APB2PeriphClockCmd(A,B);  
	
	  A: RCC_APB2PeripH_X;
	  
	  B: ENABLE or DISABLE
	
	+ Thư viện sử dụng: stm32f10x_rcc.h 
	
- B2: Cấu hình chân
	
	+ Lệnh cấu hình chân: GPIO_InitTypeDef A;
	
		Khai báo biến A thuộc kiểu dữ liệu GPIO_InitTypeDef;
		
		Thư viện sử dụng: stm32f10x_gpio.h
		
		Các tham số gồm: GPIO_Mode; GPIO_PIN; GPIO_Speed;
		
	+ GPIO_Init(GPIOx;&A); : Cấu hình GPIOx theo các thông số được lưu trong biến ADC
	
### 2.4. Một số lệnh thường dùng

- GPIO_SetBits(A,B);

	+ lệnh xuất mức "1" ra một chân vdk
	
	+ A: GPIOx (chọn port GPIO muốn điều khiển)
	
	+ B: GPIO_Pin (Chọn chân thuộc port đó)
	
- GPIO_ResetBits(A,B);

	+ lệnh xuất mức "0" ra một chân vdk
	
	+ A: GPIOx
	
	+ B: GPIO_Pin

- Ghi trực tiếp trạng thái: PIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET); hoặc: GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_RESET);

- Đọc một chân Input: GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_0); Giá trị trả về:

	+ Bit_SET: chân đang ở mức 1
	
	+ Bit_RESET: chân đang ở mức 0
	
	+ VD: ``` if(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_0) == Bit_RESET) 
		  { 
		  // Nút được nhấn nếu sử dụng Input Pull-Up 
		  }```
		  
### 2.5. Ví dụ

- Cấu hình một LED Output, sử dụng PA0

```
#include "stm32f10x.h" 
int main(void) 
{ 
	GPIO_InitTypeDef GPIO_InitStructure; 
	
	// 1. Cap clock cho GPIOA 
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE); 
	
	// 2. Cau hinh PA0 
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0; 
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP; 
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_2MHz; 
	
	GPIO_Init(GPIOA, &GPIO_InitStructure); 
	
	// 3. Xuat muc HIGH tai PA0 GPIO_SetBits(GPIOA, GPIO_Pin_0); 
	
	while(1) 
	{ 
	} 
}
```