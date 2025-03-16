Fundamentally, the problem of binary register multiplication comes down to the problem of adding together a series of shifted values based on a conditional jump.

I have access to 64 full adders, which is enough for four x16 adders. Initially, 3 will suffice.

Essentially, my plan is to accelerate the shift multiplier we use in the C program. We can do this by operating on 4 bits of the operand in one cycle. This gives us 4 numbers to add together (because of the adder limit we will limit ourselves to a 16-bit output), which requires 3x16 adders. The last adder we can save to save one cycle later.

Note that, as stated earlier, we are limiting ourselves to a 16-bit output. This is limiting, but a 32-bit output would require far longer to execute. Each output to the regfile would require an extra 16-bit register in the ALU and another clock cycle. It would also take far longer to add the final numbers together, as each addition would take 3 cycles. In short, the program would probably take something like 15 or 20 extra cycles to compute, although it could be done using the same principles as I have used with this circuit.

In DPDECODE, I have modified the outputs to also pass out MOVOPC, or MOV opcode, which can be used by the ALU as a MUX select for the outputs of the new instruction(s) I have defined. This input (MUXed with op2sel to ensure that standard MOV instructions still work) enables the extra 7 instructions.

In the ALU, I have two registers that store the values of the multiplier operands. This is necessary because we cannot take as inputs and store to different registers due to the restrictions of MOVCn.

The block itself selects 4 bits of the multiplicand and does the appropriate multiplication with shifts, then shifts the result to account for the position of the bits. This gets saved to a register, and the 4 registers written to can be summed together at the end for the final result.

My testing shows that this component works for signed multiplication as well as unsigned multiplication, so long as the end value fits within 16 bits, which is the critical problem with my implementation. This could be fixed with more time and potentially more registers and adders.

Note that only 3 adders are used in the initial block. The final adder is used within the ALU to save on a general-purpose register for the final block of 4 bits; we can add straight into a register that already has a saved value on the final cycle of multiplication to save both the use of a general-purpose register (e.g. for subroutines) and an additional cycle at the end of execution.

test_16bit.txt contains an unoptimised test of signed multiplication; changing ```MOV R3, R0, R4``` to ```MOV R2, R0, R5``` would allow for the removal of ADD R0, R3 at the end of the program and free up R3 for general use.

The test_16bit files inside the ISSIE project folder are not necessarily optimal, as I changed the .ram file for debugging purposes.

This implementation can achieve register multiplication in 7 cycles (one to initialise the registers, 4 for computing the multiplication, and 2 for adding together the remaining additions), which is faster than the naive conversion of the C program.

Now, onto discussing how to convert this to a 32-bit output. First, we would need to place a register on the output to capture the MS 16 bits of a 32-bit output and save the output of the multiplier component over two cycles. Inside the component, we would need either complex cross-register addition or larger adder blocks (this could be optimised by using 20, 24 and 28 bit adder blocks for each subsequent addition to save on full adders). Outside the component, we would need to use the 3-cycle 32-bit addition covered in class with shifts for each extra addition, and the final-addition-register-trick in the 16-bit ALU would not work. This would not be achievable with this approach within the confines of this challenge, and would take much longer (probably around 14 cycles longer at least) and would require either extra delay or more full adders inside the NEWMULT block.