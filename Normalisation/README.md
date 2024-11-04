![Screenshot 2024-11-04 180656](https://github.com/user-attachments/assets/33d63425-d325-4071-be82-85a866579a2a)



```c
// Mantissa Normalisation
wire normalize = mant_result[47]; 
wire [22:0] normalized_mant = normalize ? mant_result[46:24] : mant_result[45:23];

// Exponent calculation
wire [8:0] exp_sum = exp_a + exp_b - 8'd127 + {8'd0, normalize};
wire [7:0] final_exp = (exp_sum[8] || exp_sum == 0) ? 8'd0 : exp_sum[7:0];
```
