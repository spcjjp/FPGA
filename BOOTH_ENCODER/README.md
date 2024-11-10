

![Screenshot 2024-11-03 230340](https://github.com/user-attachments/assets/822baf47-196b-4f9d-a721-dd1e0b0dfbff)




```c
module Radix4BoothMultiplier(
    input [23:0] a,      
    input [23:0] b,      
    output reg [30:0] product,       
    output reg [47:0] shifted_product 
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

    // Booth Encoding and Partial Product Generation for 7-bit multiplier
    always @(*) begin
        for (i = 0; i < 4; i = i + 1) begin
            booth_code = {x[2*i+1], x[2*i], (i == 0) ? 1'b0 : x[2*i-1]}; 
            case (booth_code)
                3'b000, 3'b111: partial_product[i] = 0;
                3'b001, 3'b010: partial_product[i] = a; 
                3'b011: partial_product[i] = a << 1;   
                3'b100: partial_product[i] = ~(a << 1) + 1; 
                3'b101, 3'b110: partial_product[i] = ~a + 1; 
                default: partial_product[i] = 0;
            endcase
        end
    end



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
        .c(31'b0), 
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

    always @(posedge clk) begin
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
```
