// 1. 独立AGU与扇出压制：强迫Vivado在布局时复制该加法器节点，降低布线延迟
(* max_fanout = "16" *) wire [31:0] mem_agu_addr = A + imme_ex_i;
assign mem_agu_addr_ex_o = mem_agu_addr;
// 2. JALR目标地址复用AGU结果，砍掉一层MUX延迟
assign pc_jalr = {mem_agu_addr[31:1], 1'b0};
// 3. 剥离专用分支比较器，直接使用前递数据并行比较
wire branch_eq          = (A == ALU_DB);
wire branch_lt_signed   = ($signed(A) < $signed(ALU_DB));
assign jump_flag = (beq_ex_i & branch_eq) | (blt_ex_i & branch_lt_signed) ;


// 减法转加法：A - B = A + (~B) + 1
wire is_sub = (ALU_CTL == `SUB) || (|ALU_compare_ctrl);
wire [31:0] adder_b = is_sub ? (~ALU_B) : ALU_B;
wire [32:0] adder_result = {1'b0, ALU_A} + {1'b0, adder_b} + is_sub;
// 极速判大：利用加法器状态标志位 (N, V, C) 瞬间得出比较结果
wire N = adder_result[31];
wire V = (ALU_A[31] == adder_b[31]) && (adder_result[31] != ALU_A[31]);
wire C = adder_result[32]; 
wire cmp_signed   = N ^ V;  // 有符号比较
wire cmp_unsigned = ~C;     // 无符号比较



always @(posedge clk) begin // 放弃negedge，拥抱posedge，时序预算翻倍
    if(W_en & (Rd_addr != 5'd0))
        regs[Rd_addr] <= Wr_data;
end
// 同周期内部前递 (Internal Bypass)：解决同一周期的读写冲突
wire rs1_is_rd = W_en && (Rs1_addr == Rd_addr) && (Rd_addr != 5'd0);
assign Rs_data1 = (Rs1_addr == 5'd0) ? `zeroword :
                  (rs1_is_rd       ) ? Wr_data   : 
                  regs[Rs1_addr];




assign Rs1_addr_id_o = rs1_valid_id_i ? Rs1_addr : 5'd0;        //利用valid信号来判断是否需要研究源寄存器   
assign Rs2_addr_id_o = rs2_valid_id_i ? Rs2_addr : 5'd0;
wire rs1_hazard = (Rd_id_ex_i == Rs1_id_i);                     //判断Rs1和Rs2是否发生碰撞）
wire rs2_hazard = (Rd_id_ex_i == Rs2_id_i);      



// 1. 冲突侦测：记录发生同地址读写冲突时的 4 位字节写掩码
always@(posedge clk) begin
    if ((|W_en) == `WriteEnable && R_en == `ReadEnable && ram_wr_addr == ram_rd_addr) begin
        hazard_flag <= 1'b1;
        hazard_wstrb <= W_en; // 精确记录哪个字节被写入了
    end else begin
        hazard_flag <= 1'b0;
        hazard_wstrb <= 4'b0000;
    end
end
always @(posedge clk) begin // 将待写入的数据打一拍，用于下个周期的前递拼接
    w_data_temp <= Wr_mem_data;
end
// 2. 核心修复：按字节精细前递拼接 (Byte-Level Forwarding)
assign mem_rd_data_o[7:0]   = (hazard_flag && hazard_wstrb[0]) ? w_data_temp[7:0]   : r_data_temp[7:0];
assign mem_rd_data_o[15:8]  = (hazard_flag && hazard_wstrb[1]) ? w_data_temp[15:8]  : r_data_temp[15:8];
assign mem_rd_data_o[23:16] = (hazard_flag && hazard_wstrb[2]) ? w_data_temp[23:16] : r_data_temp[23:16];
assign mem_rd_data_o[31:24] = (hazard_flag && hazard_wstrb[3]) ? w_data_temp[31:24] : r_data_temp[31:24];


logic rst_n_d1;
logic cpu_safe_rst_n;

//原始复位是高电平有效，转为低电平有效的原始信号
logic raw_rst_n;
assign raw_rst_n = ~w_clk_rst;

//使用双触发器打两拍同步
always_ff@(posedge w_cpu_clk or negedge raw_rst_n)begin
    if(!raw_rst_n)begin
        rst_n_d1 <= 1'b0;
        cpu_safe_rst_n <= 1'b0;         //异步拍死：复位来了立即拉低
    end else begin
        rst_n_d1 <= 1'b1;
        cpu_safe_rst_n <= rst_n_d1;     //同步释放：时钟沿到来时才拉高
    end
end




function automatic logic [31:0] gray_to_bin(input logic [31:0]gray);
    integer i;
    begin
        gray_to_bin[31] = gray[31];
        for (i = 30; i>= 0; i=i - 1) begin
        gray_to_bin[i] = gray_to_bin[i + 1] ^ gray[i];
        end
    end
endfunction
logic[15:0]cnt 1ms;
logiccnt_ms_bin;[31:0]
logic[31:0] cnt_ms_gray;
logic cnt_enable_cnt_d1, cnt_enable_cnt_d2;
logic [31:0] cnt_gray_cpu_d1, cnt_gray_cpu_d2;
// CPU->counter CDC: synchronize level control into cnt_clk domain.
always_ff @(posedge cnt_clk) begin
    if (rst) begin
        cnt enable cnt d1 <= 1'b0;
        cnt enable cnt d2 <= 1'be;
        end else begin
    end
    cnt_enable_cnt_d1 <= cnt_enable_cpu;
    cnt enable cnt d2 <= cnt enable cnt d1;
end




// Counter->CPU CDC: Gray code allows safe multi-bit crossing.
always_ff @(posedge cpu_clk) begin
     if (rst) begin
         cnt_gray_cpu_d1 <= 32'de;
         cnt_gray_cpu_d2 <= 32'de;
     end else begin
         cnt_gray_cpu_d1 <= cnt_ms_gray;
        cnt_gray_cpu_d2 <= cnt_gray_cpu_d1;
     end
 end
