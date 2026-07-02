LOGIC相关的微指令

| 名称     | Bit 数 | Bit 位 | 含义                                                         |
| -------- | ------ | ------ | ------------------------------------------------------------ |
| 指令有效 | 1      | 0      | ‘b1：指令有效‘b0：指令无效，相关逻辑 IDLE，目标寄存器不更新  |
| 指令类型 | 4      | 1-4    | ‘b0000：and‘b0001：or‘b0010：xor‘b0011：not‘b0100：copy‘b0101：select_great‘b0110：select_equal‘b0111：select_less‘b1000：logic_left_shift‘b1001：arith_left_shift‘b1010：rotate_left_shift‘b1011：logic_right_shift‘b1100：arith_right_shift‘b1101：rotate_right_shift‘b1110：get_first_one‘b1111：get_first_zero |
| 数据精度 | 1      | 5      | ‘b0：16bit‘b1：32bit                                         |

