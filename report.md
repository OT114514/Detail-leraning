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
