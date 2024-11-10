```c
integer shift_count;
reg [7:0] final_exp;
wire [8:0] exp_sum = exp_a + exp_b - 8'd127 ;
reg [22:0] normalized_mant;
always @(*) begin
    
    final_exp = exp_sum;
    shift_count = 0;
   while (product[47 - shift_count] == 1'b0 && shift_count < 24) begin
        shift_count = shift_count + 1;
    end
  if (shift_count < 24) begin
       normalized_mant = mant_result[46 - shift_count -: 23];
       final_exp = exp_sum - shift_count;
    end else begin
       
        normalized_mant = 23'b0;
        final_exp = 8'b0;
    end
end
```
