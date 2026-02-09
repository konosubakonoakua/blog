---
number: 162
title: "[fpga] vivado tcl workflow"
state: open
url: https://github.com/konosubakonoakua/blog/issues/162
author: konosubakonoakua
createdAt: 2025-03-11T08:10:09Z
updatedAt: 2025-12-31T16:59:42Z
labels: []
---

# 162 [fpga] vivado tcl workflow

- https://github.com/rajivbishwokarma/rsig
- https://github.com/juliancaba/xilinx-tcl/
- https://projectf.io/posts/vivado-tcl-build-script/
- https://aidan-mcnay.github.io/vivado-tutorial/overview/
- example

  <details>
  
  ```tcl
  # Vivado流程脚本（Block Design作为顶层）
  set project_name "bd_project"
  set project_dir "./vivado_prj"
  set part "xc7z020clg400-1"  # 替换实际器件型号
  
  # 创建工程
  create_project $project_name $project_dir -part $part -force
  
  # 添加设计文件（非BD文件）
  add_files [glob -nocomplain ./src/*.v ./src/*.vhd]
  update_compile_order -fileset sources_1
  
  # 加载并重建Block Design
  set bd_loaded 0
  foreach bd_script [glob -nocomplain ./bd/*.tcl] {
      source $bd_script
      regenerate_bd_layout
      validate_bd_design
      save_bd_design
      set bd_loaded 1
  }
  
  # 强制设置BD为顶层（关键修改）
  if {$bd_loaded} {
      # 生成并添加BD wrapper
      make_wrapper -files [get_files *.bd] -top
      add_files -norecurse [get_files */hdl/*_wrapper.v]  # 自动查找生成的wrapper
      
      # 显式设置BD wrapper为顶层
      set bd_wrapper [file rootname [file tail [get_files -norecurse *.bd]]]_wrapper
      set_property TOP $bd_wrapper [current_fileset]
      update_compile_order -fileset sources_1
  } else {
      error "No Block Design loaded! Please check bd/ directory"
  }
  
  # 添加约束文件
  add_files -fileset constrs_1 [glob -nocomplain ./constraints/*.xdc]
  
  # 标准流程执行
  launch_runs synth_1
  wait_on_run synth_1
  launch_runs impl_1 -to_step write_bitstream
  wait_on_run impl_1
  
  # 输出文件生成
  write_bitstream -force "${project_dir}/${project_name}.bit"
  close_project
  ```
  </details>

