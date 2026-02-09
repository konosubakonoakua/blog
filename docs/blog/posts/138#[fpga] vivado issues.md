---
number: 138
title: "[fpga] vivado issues"
state: open
url: https://github.com/konosubakonoakua/blog/issues/138
author: konosubakonoakua
createdAt: 2025-02-13T02:48:39Z
updatedAt: 2025-06-16T01:49:06Z
labels: []
---

# 138 [fpga] vivado issues

## AXI BRAM Controller depth not matched with Block Memory Generator (2019.1, 2024.2)
<details>

![AXI BRAM Controller depth not matched with Block Memory Generator](../assets/138/1.png)

> ERROR: [IP_Flow 19-3478] Validation failed for parameter 'Disable Collision Warnings(Disable_Collision_Warnings)' with value 'false' for BD Cell 'axi_bram_ctrl_0_bram'. User configuration exceeds BRAM count in the selected device

Solution: 
> Set `Block Memory Generator` as `Stand Alone` Mode, then set `Write/Read Width` & `Write Depth` manually, finally, set `Mode` back to `BRAM Controller`.

> Sometimes, instead directly setting width to 512, setting to 128/256 first, then back to 512 may work.

I asked on AMD forum: https://adaptivesupport.amd.com/s/question/0D54U000092F4C1SAK/vivado-20191-axi-bram-controller-data-width-not-propagated-to-block-memory-generator?language=en_US

They give me a workaround:
```
Set BRAM Controller DW = 32 & Address Range = 4k
Validate bd
Set BRAM Controller DW = 512 & Address Range = 256k
Validate bd --> no more validation errors
```
If still no okay, remove block mem then add again.

Again, if you are using a True dual-port BRAM, you maybe encounter still the propagation issue.
For example, you use bram controller to control portA, then you want to control port B using custom rtl codes, you will fail on validation because the unmatched issue again.
Here, I found a new workaround to solve issue with the port B. We need to create a new bram controller connected to port B, then set all parameters using method above. After we pass the validation, remove the new bram controller form port B, connect our own RTL codes to port B, try validating, wish your success.

https://www.bilibili.com/video/BV1oTPvefEEZ/

### A new workaround (manually set config `Write_Width_A` parameter)
- manually set `Write_Width_A` for `blk_mem_gen`, save, valid.
  ![Image](../assets/138/2.png)
- make external BRAM_PORTB, set MEM_WIDTH manually, save, valid
![Image](../assets/138/3.png)

</details>

---

## Comments

### konosubakonoakua · 2025-02-25T08:03:34Z

## Clocking Wizard external pin freq not matched with clk_in1
- you must set the external pin freq when making clk_in1 external
  <details>
  
  ![Image](../assets/138/4.png)
  </details>


### konosubakonoakua · 2025-02-25T08:06:43Z

## Cound not find external pin port of Clocking Wizard in Schematic
- Maybe because the `design_1_wrapper` does not contain the port for clocking wizard, Try to remove it then recreate one.


### konosubakonoakua · 2025-02-28T10:11:33Z

## Could not generate `x4` spi mcs file
<details>

- [Vivado生成X1或X4位宽mcs文件并固化到flash](https://blog.csdn.net/m0_66578894/article/details/145341631)
  ```tcl
  set_property CFGBVS VCCO [current_design]
  设置 FPGA 配置引脚的参考电压源为 VCCO，确保与外部器件的电平兼容。
   
  set_property CONFIG_VOLTAGE 3.3 [current_design]
  设置 FPGA 配置接口的工作电压为 3.3V，需与外部设备（如 SPI Flash）的工作电压匹配。
   
  set_property BITSTREAM.GENERAL.COMPRESS true [current_design]
  启用比特流压缩，减小比特流文件大小，加快配置速度。
   
  set_property BITSTREAM.CONFIG.CONFIGRATE 50 [current_design]
  设置配置时钟频率为 50 MHz，需与配置器件的时钟速率能力相匹配。
   
  set_property BITSTREAM.CONFIG.SPI_BUSWIDTH 4 [current_design]
  设置 SPI 配置模式为 Quad SPI（四线模式），以提高比特流传输速度。
   
  set_property BITSTREAM.CONFIG.SPI_FALL_EDGE yes [current_design]
  指定 SPI 时钟在下降沿采样数据。需要与外部 SPI Flash 的工作模式匹配。
  ```

</details>


### konosubakonoakua · 2025-03-10T02:30:46Z

## ERROR: [Labtools 27-3165] End of startup status: LOW
- check powersupply
- check VREF connection (not sure)
- check bootmode (jtag mode)
- check done pin (if with led attached)
  - [how-to-connect-done-signal-to-cpld](https://adaptivesupport.amd.com/s/question/0D52E00006hpTXtSAM/how-to-connect-done-signal-to-cpld-?language=en_US)
    - Red LED has a lower voltage drop, use blue or green LEDs instead.
    - Remove LED
  - [FPGA configuration process successfully completes (DONE PIN)](https://wiki.trenz-electronic.de/pages/viewpage.action.dir/pageId=3924933)



### konosubakonoakua · 2025-03-13T01:40:24Z

## [Opt 31-67] Problem: A LUT2 cell in the design is missing a connection on input pin I1
- https://adaptivesupport.amd.com/s/article/72980?language=en_US
  1. open Elaborated Design
  2. `select_objects [get_cells {your LUT cell name}]`
  3. `F4` or `schematic`
  4. ` Expand Cone > To Leaf Cells. `
  5. Trace through as many hierarchies as needed to find where it is disconnected (n/c).
  6. goto source
- Check your IP core internal outputs, does them really output something?
  - e.g. I have forget to assign reg content to port
- Check your `Block Design`, do you really connect the signal?
  - e.g. I have forget to connect global `rst` signal to modules

### konosubakonoakua · 2025-03-18T09:59:37Z

## AXI DMA stuck at `XAxiDma_Busy` due to wrong block design SMC connection
If you connect AXI DMA wrongly, you will get wrong address range, the correct connection & range are showing below.
<details>

![Image](../assets/138/5.png)

![Image](../assets/138/6.png)

</details>

### konosubakonoakua · 2025-03-20T09:28:34Z

## vivado speed comparison on win/linux
<details>

- https://www.bilibili.com/video/BV14EfnYtEzS

![Image](../assets/138/7.png)

</details>

### konosubakonoakua · 2025-03-31T07:02:58Z

## [Labtools 27-3303] Incorrect bitstream assigned to device. Bitstream was generated for part xcvu37p-fsvh2892-1-e, target device (with IDCODE revision 0) is compatible with es1 revision bitstreams

```tcl
set_param xicom.use_bitstream_version_check false
```

### konosubakonoakua · 2025-04-01T07:02:25Z

## System Memory Recommendations
https://www.amd.com/en/products/software/adaptive-socs-and-fpgas/vivado/vivado-buy.html#tabs-413944f675-item-9598720e6a-tab

my settings: 6 threads, 20GB for zynqMP

### konosubakonoakua · 2025-04-14T01:13:51Z

## language server crashes very often
- [Vivado hangs due to sigasi cache utilizing significant space](https://adaptivesupport.amd.com/s/article/76976)
- don't use `sigasi`
- or try to use another cache path: `Settings -> Tool Settings -> Text Editor -> Syntax Checking -> Cache Settings`

### konosubakonoakua · 2025-04-14T01:14:44Z

### 55854 - Vivado - What can I do to resolve a Vivado crash, exception, or abnormal program termination?
- https://adaptivesupport.amd.com/s/article/55854


### konosubakonoakua · 2025-05-22T05:18:33Z

## switch vivado version
- by registry: https://github.com/konosubakonoakua/vivtool/tree/main/switcher
- by dynamic switcher: 
- by `.bat` scripts

### konosubakonoakua · 2025-05-30T09:38:13Z

## ~move vivado from `C:` to `D:`~
> maybe break the disk optimization performed by vivado installer (shouldn't turn disk optim on)
<details>

```bash
robocopy C:\Xilinx\ D:\tools\Xilinx /E /MT:32 /COPYALL /DCOPY:DAT /SECFIX /TIMFIX /LOG:xilinx_copy.log /NP
```
```bash
rmdir /s /q C:\Xilinx
```
```bash
mklink /j <target> <source>
```
```bash
scoop bucket add ssyinternals
scoop install junction
```
```bash
junction -d <j_link_path>
```
</detais>

### konosubakonoakua · 2025-06-13T09:57:38Z

## X_INTERFACE_INFO
### bram
<details>

<summary> bram x_interface_info vhdl </summary>

```vhdl
entity save_data_brm_ctr is
  port (
    clk       : in    std_logic;
    reset_n   : in    std_logic;
    read_done : in    std_logic_vector(31 downto 0);

    interlock      : in    std_logic;
    adc_valid      : in    std_logic;
    wr_en          : out   std_logic_vector(0 downto 0);
    bram_addr      : out   std_logic_vector(31 downto 0);
    lock_bram_addr : out   std_logic_vector(12 downto 0);
    en             : out   std_logic;
    led            : out   std_logic;
--    dout          : in    std_logic_vector(511 downto 0);
--    din           : out   std_logic_vector(511 downto 0);
    write_done     : out   std_logic
  );



end entity save_data_brm_ctr;

architecture behavioral of save_data_brm_ctr is
  
    ATTRIBUTE X_INTERFACE_INFO : STRING;
    ATTRIBUTE X_INTERFACE_PARAMETER : STRING;

    ATTRIBUTE X_INTERFACE_INFO of en: SIGNAL is "xilinx.com:interface:bram:1.0 BRAM_PORT EN";
--    ATTRIBUTE X_INTERFACE_INFO of dout: SIGNAL is "xilinx.com:interface:bram:1.0 BRAM_PORT  DOUT";
--    ATTRIBUTE X_INTERFACE_INFO of din: SIGNAL is "xilinx.com:interface:bram:1.0 BRAM_PORT  DIN";
    ATTRIBUTE X_INTERFACE_INFO of wr_en: SIGNAL is "xilinx.com:interface:bram:1.0 BRAM_PORT  WE";
    ATTRIBUTE X_INTERFACE_INFO of bram_addr: SIGNAL is "xilinx.com:interface:bram:1.0 BRAM_PORT  ADDR";
    ATTRIBUTE X_INTERFACE_INFO of adc_valid: SIGNAL is "xilinx.com:interface:bram:1.0 BRAM_PORT  CLK";
    ATTRIBUTE X_INTERFACE_INFO of reset_n: SIGNAL is "xilinx.com:interface:bram:1.0 BRAM_PORT  RST";
  
    attribute X_INTERFACE_PARAMETER of en   : signal is "MASTER_TYPE BRAM_CTRL, MEM_WIDTH 512";
--    attribute X_INTERFACE_PARAMETER of dout   : signal is "MASTER_TYPE BRAM_CTRL, MEM_WIDTH 512";
--    attribute X_INTERFACE_PARAMETER of din   : signal is "MASTER_TYPE BRAM_CTRL, MEM_WIDTH 512";
    attribute X_INTERFACE_PARAMETER of wr_en   : signal is "MASTER_TYPE BRAM_CTRL, MEM_WIDTH 512";
    attribute X_INTERFACE_PARAMETER of bram_addr   : signal is "MASTER_TYPE BRAM_CTRL, MEM_WIDTH 512";
    attribute X_INTERFACE_PARAMETER of adc_valid   : signal is "MASTER_TYPE BRAM_CTRL, MEM_WIDTH 512";
    attribute X_INTERFACE_PARAMETER of reset_n   : signal is "MASTER_TYPE BRAM_CTRL, MEM_WIDTH 512";
```
- https://www.taterli.com/10000
- https://www.adiuvoengineering.com/post/microzed-chronicles-ip-integrator-hdl
- https://github.com/C2A2-at-Florida-Atlantic-University/OFDM_UnderwaterModem/blob/dbd5887aa4570af5d3e09f8144b55719e5a5feb6/Xilinx/Vivado/modules/rtl/adc_raw_sample_save.vhd
</details>


### konosubakonoakua · 2025-06-16T01:34:25Z

## apply ps preset by tcl
<details>

```tcl
proc apply_ps_presets { cell presetFile } {
    source $presetFile
    set presets [apply_preset 0]
    foreach {k v} $presets {
        if { ![info exists preset_list] } {
            set preset_list [dict create $k $v]
        } else {
            dict set preset_list $k $v
        }
    }

    set_property -dict $preset_list $cell 
}

proc create_ps { parentCell cellName } {
    global script_folder
    # Add Zynq processing system
    set ps_cell [ create_bd_cell -type ip -vlnv xilinx.com:ip:processing_system7 $cellName ]
    apply_ps_presets $ps_cell $script_folder/BlackBoard_ps_presets.tcl
    apply_bd_automation -rule xilinx.com:bd_rule:processing_system7 -config {make_external "FIXED_IO, DDR" Master "Disable" Slave "Disable" } $ps_cell 
}

# Create processing system
create_ps $parentCell "processing_system7_0"
```
</details>

<details>

<summary> ps config file content </summary>

```tcl
proc getPresetInfo {} {
  return [dict create name {zu9-v3} description {zu9-v3}  vlnv xilinx.com:ip:zynq_ultra_ps_e:3.3 display_name {zu9-v3} ]
}

proc validate_preset {IPINST} { return true }


proc apply_preset {IPINST} {
  return [dict create \
    CONFIG.PSU_VALUE_SILVERSION {3}  \
    CONFIG.PSU__USE__DDR_INTF_REQUESTED {0}  \
    CONFIG.PSU__EN_AXI_STATUS_PORTS {0}  \
]}
```

</detials>


### konosubakonoakua · 2025-06-16T01:34:35Z

## project management
- https://wiki.trenz-electronic.de/pages/viewpage.action.dir/pageId=3925279
