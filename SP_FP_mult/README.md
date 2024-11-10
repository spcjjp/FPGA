```c
module single_precission_fp_multiplier(
    input [31:0] a,
    input [31:0] b,
    output [31:0] result,
    output [47:0] mant_result
);


wire sign_a = a[31];
wire sign_b = b[31];
wire [7:0] exp_a = a[30:23];
wire [7:0] exp_b = b[30:23];
wire [22:0] mant_a = a[22:0];
wire [22:0] mant_b = b[22:0];


wire [23:0] full_mant_a = {1'b1, mant_a};
wire [23:0] full_mant_b = {1'b1, mant_b};

// Connect to the Radix4BoothMultiplier
wire [30:0] product;           // Product output from Radix4BoothMultiplier
wire [47:0] shifted_product;    // Shifted product output from Radix4BoothMultiplier
        // Final mantissa result

// Instantiate Radix4BoothMultiplier
Radix4BoothMultiplier booth_multiplier (
    .a(full_mant_a),            // Using full mantissa a
    .b(full_mant_b),            // Using full mantissa b
    .product(product),
    .shifted_product(shifted_product)
);

// DSP_1 instance to combine with shifted product
DSP_1 dsp (
    .a(full_mant_a),            // Using full mantissa a
    .b(full_mant_b[16:0]),      // Using lower 17 bits of full mantissa b for DSP_1
    .shifted_product(shifted_product),
    .result(mant_result)        // Result from the DSP operation
);




// Normalization and rounding
/*wire normalize = mant_result[47]; 
wire [22:0] normalized_mant = normalize ? mant_result[46:24] : mant_result[45:23];

// Exponent calculation
wire [8:0] exp_sum = exp_a + exp_b - 8'd127 + {8'd0, normalize};
wire [7:0] final_exp = (exp_sum[8] || exp_sum == 0) ? 8'd0 : exp_sum[7:0];
*/

   /*input [47:0] product;                // 48-bit intermediate product
    input [7:0] exponent_in;     */       // Input exponent
  
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


// Sign calculation
wire final_sign = sign_a ^ sign_b;

// Final result
assign result = {final_sign, final_exp, normalized_mant};

endmodule

// Radix-4 Booth Multiplier Module
module Radix4BoothMultiplier(
    input [23:0] a,       // 24-bit operand
    input [23:0] b,       // Full 24-bit input, but only using b[23:17]
    output reg [30:0] product,       // Adjusted product width to accommodate result
    output reg [47:0] shifted_product // Shifted product for further operations
);

    reg [6:0] x; // Only use b[23:17] from b
    reg [2:0] booth_code; // 3-bit Booth encoding for each group
    reg [30:0] partial_product[0:3]; // Partial products array for 4 radix-4 groups
    wire [30:0] pp_sum;  // Sum of partial products after compression
    wire [30:0] pp_carry; // Carry of partial products after compression
    integer i;

    // Assign x to b[23:17] bits of b
    always @(*) begin
        x = b[23:17];
    end

    // Booth Encoding and Partial Product Generation for 7-bit multiplier
    always @(*) begin
        for (i = 0; i < 4; i = i + 1) begin
            booth_code = {x[2*i+1], x[2*i], (i == 0) ? 1'b0 : x[2*i-1]}; // 3 bits for encoding
            case (booth_code)
                3'b000, 3'b111: partial_product[i] = 0;
                3'b001, 3'b010: partial_product[i] = a; // +a
                3'b011: partial_product[i] = a << 1;    // +2a
                3'b100: partial_product[i] = ~(a << 1) + 1; // -2a
                3'b101, 3'b110: partial_product[i] = ~a + 1; // -a
                default: partial_product[i] = 0;
            endcase
        end
    end

    // Compression using 4:2 and 3:2 counters for 7-bit partial products
    wire [30:0] sum[0:2];
    wire [30:0] carry[0:2];

    // 4:2 Counter
    FourTwoCounter u_counter1 (
        .a(partial_product[0]),
        .b(partial_product[1]),
        .c(partial_product[2]),
        .d(partial_product[3]),
        .sum(sum[0]),
        .carry(carry[0])
    );

    // 3:2 Counter
    ThreeTwoCounter u_counter2 (
        .a(sum[0]),
        .b(carry[0]),
        .c(31'b0), // Zero-padding to match 31-bit width
        .sum(sum[1]),
        .carry(carry[1])
    );

    // 2:2 Counter
    TwoTwoCounter u_counter3 (
        .a(sum[1]),
        .b(carry[1]),
        .sum(pp_sum),
        .carry(pp_carry)
    );

    // Summing compressed products with pipelined addition
    always @(*) begin
        product = pp_sum + pp_carry;
        shifted_product = product << 17; 
    end

endmodule


module FourTwoCounter(
    input [30:0] a, b, c, d,
    output [30:0] sum,
    output [30:0] carry
);
    assign sum = a ^ b ^ c ^ d;
    assign carry = (a & b) | (c & d) | (a & c) | (b & d);
endmodule


module ThreeTwoCounter(
    input [30:0] a, b, c,
    output [30:0] sum,
    output [30:0] carry
);
    assign sum = a ^ b ^ c;
    assign carry = (a & b) | (b & c) | (c & a);
endmodule


module TwoTwoCounter(
    input [30:0] a, b,
    output [30:0] sum,
    output [30:0] carry
);
    assign sum = a ^ b;
    assign carry = a & b;
endmodule


module DSP_1 (
    input wire [23:0] a,
    input wire [16:0] b,
    input wire [47:0] shifted_product,
    output [47:0] result
);

    wire [40:0] product; 

    // Multiplication operation
  //  assign product = a * b;
    
    dsp_macro_0 your_instance_name (
  .CLK(CLK),  // input wire CLK
  .A(a),      // input wire [23 : 0] A
  .B(b),      // input wire [16 : 0] B
  .C(shifted_product),      // input wire [47 : 0] C
  .P(result)      // output wire [47 : 0] P
);

    
   /* always @(*) begin
        result = product + shifted_product;
    end*/

endmodule

```
![WhatsApp Image 2024-11-10 at 09 54 06_a4dbb93c](https://github.com/user-attachments/assets/c713f28b-6edc-40b6-bf5a-b15a2df01ddc)
