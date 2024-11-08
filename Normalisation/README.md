


```c
integer shift_count;
reg [7:0] final_exp;
wire [8:0] exp_sum = exp_a + exp_b - 8'd127 ;
reg [22:0] normalized_mant;
always @(*) begin
    // Start with the input exponent
    final_exp = exp_sum;
    shift_count = 0;

    // Check for leading 1 in the 48-bit product, shifting left if needed
    while (product[47 - shift_count] == 1'b0 && shift_count < 24) begin
        shift_count = shift_count + 1;
    end

    // Adjust the normalized mantissa and exponent based on shift count
    if (shift_count < 24) begin
        // Select the appropriate mantissa bits after shifting
        normalized_mant = mant_result[46 - shift_count -: 23];
        // Adjust the exponent to account for the shift
        final_exp = exp_sum - shift_count;
    end else begin
        // Edge case if the product is zero or too small to normalize
        normalized_mant = 23'b0;
        final_exp = 8'b0;
    end
end

```
