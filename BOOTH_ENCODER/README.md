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

```
