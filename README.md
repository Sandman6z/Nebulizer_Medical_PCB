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
  

