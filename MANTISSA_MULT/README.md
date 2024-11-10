```c

`timescale 1ns / 1ps

module Radix4BoothMultiplier(
    input clk,
    input [23:0] a,                 
    input [23:0] b,                 
    output reg [30:0] product,      
    output reg [47:0] shifted_product,
    output reg [47:0] result,
    output reg [40:0] out           
);

    reg [6:0] x;                    
    reg [2:0] booth_code;          
    reg [30:0] partial_product[0:3];
    wire [30:0] pp_sum;             
    wire [30:0] pp_carry;           
    integer i;
    
    always @(*) begin
        x = b[23:17];
    end

    always @(*) begin
        for (i = 0; i < 4; i = i + 1) begin
            booth_code = {x[2*i+1], x[2*i], (i == 0) ? 1'b0 : x[2*i-1]}; // 3 bits for encoding
            case (booth_code)
                3'b000, 3'b111: partial_product[i] = 0;
                3'b001, 3'b010: partial_product[i] = a;          // +a
                3'b011: partial_product[i] = a << 1;             // +2a
                3'b100: partial_product[i] = ~(a << 1) + 1;      // -2a
                3'b101, 3'b110: partial_product[i] = ~a + 1;     // -a
                default: partial_product[i] = 0;
            endcase
        end
    end

    // Compression using 4:2 and 3:2 counters for partial products
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

    
    always @(*) begin
        product <= pp_sum + pp_carry;
        shifted_product <= product << 17; 
    end
    
    
    reg [16:0] y;
    
    always @(*) begin
        y <= b[16:0];
    end


 
       wire [40:0] temp;

    dsp_macro_0 satya (
  //.CLK(CLK),  // input wire CLK
  .A(a),      // input wire [23 : 0] A
  .B(y),      // input wire [16 : 0] B
  .P(temp)      // output wire [40 : 0] P
);

      always @(*)
      begin
        out <= temp;
        end
    // Final summation
    always @(*) begin

        result <= out + shifted_product;
    end
endmodule

// 4:2 Counter Module
module FourTwoCounter(
    input [30:0] a, b, c, d,
    output [30:0] sum,
    output [30:0] carry
);
    assign sum = a ^ b ^ c ^ d;
    assign carry = (a & b) | (c & d) | (a & c) | (b & d);
endmodule

// 3:2 Counter Module
module ThreeTwoCounter(
    input [30:0] a, b, c,
    output [30:0] sum,
    output [30:0] carry
);
    assign sum = a ^ b ^ c;
    assign carry = (a & b) | (b & c) | (c & a);
endmodule

// 2:2 Counter Module
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
    output reg [47:0] result
);

    wire [40:0] product; // Intermediate product of a and b

    // Multiplication operation
   // assign product = a * b;
    
    dsp_macro_0 your_instance_name (
  .CLK(CLK),  // input wire CLK
  .A(a),      // input wire [23 : 0] A
  .B(b),      // input wire [16 : 0] B
  .P(product)      // output wire [40 : 0] P
);

    // Addition with shifted_product
    always @(*) begin
        result = product + shifted_product;
    end

endmodule
```

![Screenshot 2024-11-08 022708](https://github.com/user-attachments/assets/313386c5-09c7-48e2-9a2a-35624a270f3b)
