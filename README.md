## Medical Atomizer on ventilators

- MCU: STM32C031G6U6![image](https://github.com/user-attachments/assets/a0be71a1-305c-4ab3-ba47-a368890d5140)

- MCU change to: STM32G031G6U6
 ---

## 注意事项
- 由于硅胶垫圈的原因，雾化片震动效果可能不好，更换后，雾化率提升，故降低boost电路电压，R29降为270kΩ
- 经过软件工程师ADC采集数据情况，C18电容NC效果较好
- 保持C20 NC

## 医用雾化器-雾化专用芯片设计注意事项

- 三脚电感值，测试

- 供电输入5V不耐冲击，加TVS等防护
- 高压网络layout和器件封装，参考雾化片手册注意事项
- 雾化片功率检测方式

## 测试数据
### 2024.12.17
- 采样点电流：
  - 有水：133mV
  - 空杯：159mV
  - 无杯：78mV
- MOS电压：
  - 有水：72~90V
  - 空杯：68V
  - 无杯：212V（150V的TVS钳位到160V）
  


---
---

<!-- 板子背景，更改前的状态，遇到的问题，做的更改，当前设计简述 -->


## 背景信息

- **硬件设计**:
  - 使用Microchip公司的`PIC16F1578-I/SS`微控制器单元(MCU)。
  - 编程环境为`MPLAB X IDE v5.50`，使用`xc8` C语言编译器。
  - 硬件仿真器为`PICKit3`。
  - 通过输出PWM信号驱动MOS管来雾化药物。
  - 与呼吸机通过三根信号线进行通讯，包括使能信号`EN_Fog`、状态反馈信号`NebSt`和设备识别信号`Dev_NC`。

- **软件设计**:
  - 需要实现的功能包括与呼吸机的通信、状态反馈、扫频以确定最佳雾化频率、无水检测等。
  - 雾化器能够通过反馈信号告知呼吸机其工作状态。

## 更改前的状态

- 雾化器能够正常执行基本的雾化操作，但雾化率不理想[需由王工补充]
- 能够与呼吸机进行通信并反馈工作状态，但偶尔无法启动雾化功能
- 已经实现了扫频功能以确定最佳雾化频率，但扫频功能容易误识别

## 遇到的问题

1. **对于PI膜雾化片**:
   - 在剩余约0.5~1ml液体时会误判为无水状态，导致提前停机。
   - 这种误报警现象稳定出现，影响正常使用。

4. **不锈钢雾化片兼容性问题**:
   - 新的不锈钢雾化片与原有雾化片特性不同，需要重新调试扫频、水杯未连接、无水停机策略等来适应新雾化片。

3. **雾化片扫频不准**
   - 当前已经实现了扫频功能，但通过示波器查看扫频波形，若存在多个功率较大的频点时容易误识别，故而不工作在雾化片的最佳频点，需要更精确的电路。

4. **指示灯问题**:
   - 初始化时，工作指示灯的亮度不足，可能是软件控制的问题。

5. **LED状态问题**:
   - 当使能信号被取消后，LED会锁定在红色状态，无法回到初始化状态，除非重新启动雾化器。

这些问题需要进一步的软件优化和调试来解决，以确保雾化器的稳定运行和提高用户体验。

## 问题更改及改进

1. MCU
   - 原MCU为microchip的PIC16F1578-I/SS，IDE使用MPLAB X IDE，配置复杂，且软件通用性差，与公司生态不一致
   - 考虑到性能、价格、封装、生态等因素，将microchip MCU换成ST的STM32C031G6U6，价格近似，性能有质的提升，且外设满足所有功能，精度更高，封装尺寸小，更便于布局。与原厂FAE和各经销商确认过，供货长期且稳定，若量大价格可以更低
   - STM32C0系列与STM32G系列多种型号封装互相通用，后期若需要修改，也有更多选择，而无需修改PCB
   - 使用STM32CubeMX，使用图形化配置 + HAL库的方式，实现快速开发，满足易维护的要求

2. MOSFET
   - 原雾化电路驱动的LC电路可达到100V以上，但原NMOS耐压值不足，本次选择onsemi(安森美)家的FDN86246，耐压可达150V，Id 1.6A，均满足设计需求且余量充足
   - 封装更小，节省紧张的板空间

3. 扫频功能
   - 增加雾化驱动电路的电压和电流检测电路
   - 电压检测通过分压方式；电流检测通过检流电阻和运放电路的方式
   - 相较原简易电阻分压测电流方式，精度更高，且layout采用差分走线，避免layout造成的采样不准确的问题
   - 将电压和电流检测电路结合，配合软件逻辑，实现扫频功能。