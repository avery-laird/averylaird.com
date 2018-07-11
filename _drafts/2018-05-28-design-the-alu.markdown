---
layout: post
title: "Kermit: ALU simulation"
categories: hardware computer kermit ALU
---

The Arithmetic Logic Unit (ALU), as the name implies, performs many of the arithmetic and logic functions for the computer. Kermit just has one main ALU, which is capable of addition, subtraction, logical shifts left and right (by a variable amount), bitwise AND, OR, and XOR, and zero detection. 

Addition and subtract is trival using one or more CLAs, and a two's compliment representation of integers. To support shifts of up to 32 bits, we need $$\log_2{2^5} = 5$$ control lines, and a barrel shifter. The rest of the logical functions are more interesting. The simplest solution would be to have hardware for each operation, which would use $$32 * 4 = 128$$ gates. This assumes we use three gates for each bit, plus a multiplexor to select the output. Is this simple method feasible? Is it possible to combine some of the functionality in the hardware?

Let's introduce a control line `LSEL`, which is a two bit line to control the logic function we want (AND, OR, or XOR). We get the following truth table:

$$
\begin{array}{c c l l | r}
LSEL_0 & LSEL_1 & A & B & C \\
\hline 
0      & 0       & A & B & A \land B \\
0      & 1       & A & B & A \lor B \\
1      & 0       & A & B & A \oplus B \\
1      & 1       & \times & \times & \times \\
\end{array}
$$

We can describe the desired behaviour for one bit in a truth table, and try to simplify the expression using a K-map. We get:

$$
C = LSEL_0 \bar{A} B + LSEL_1 B + \bar{LSEL_0} A B + LSEL_1 A \bar{B} + LSEL_0 A \bar{B}
$$

Despite some additional simplifications we could make (like combining the last two terms), this expression will still end up using at least $$8 + 3 = 11$$ gates for each bit, including the hardware to support a multiplexor (which is embedded in the logic). In reality, the true number of components needed depends heavily on the hardware I use. For example, if I use a quad 2-input XOR chip, then I only need $$8$$ of them to implement XOR. I won't worry about the hardware implementation for now, and just focus on the logic.

A CLA can be generalized quite easily for any bit $$i$$:

$$
\begin{align}
S_i &= P_i \oplus C_i \\
C_i &= G_{i-1} + P_{i-1} \cdot C_{i-1} \\
G_i &= A_i \cdot B_i \\
P_i &= A_i \oplus B_i \\
\end{align}
$$

Where $$S_i$$ is the sum, $$C_i$$ is the carry, $$G_i$$ is the "generate" value (whether two bits will generate a carry), and $$P_i$$ is the "propogate" value (whether two bits will propogate a carry). There's a [good introduction to CLAs](https://en.wikipedia.org/wiki/Carry-lookahead_adder) on wikipedia.

This logic can be easily translated (almost verbatim) into Verilog, resulting in [pretty clear](#appendix-code) (if verbose) code.

I'm "cheating" a little bit with some of the other logic functions, but only because they are easy to implement in parallel --- in the case of the adder, I specifically wanted to simulate a CLA implementation.

When I did some looking around online, it seemed the most common logic chips in DIP packaging came with four gates with two ports each. With this in mind, I made some chips to simulate this, and implemented the CLA using quad gate packages:


![](/static/kermit/ALU/together.png)

Subtraction is only partially implemented, as `FSEL[0]` is the first carry-in bit. To introduce subtraction, another 8 `XOR` gates (the `CD4070BC`) would have to be added. All in all, we have:

* $$16$$ quad `OR` chips
* $$8 + 8 + 8 = 24$$ quad `XOR` chips
* $$8$$ quad `AND` chips

For a grand total of $$48$$ chips, which is not outrageous. The data sheet for both chips never showed dimensions larger than 8mm, which seems quite small to me. If true, I should have no problem fitting them on a decent size PCB. For example, assuming they have more like a 2cm x 1cm footprint, 6 rows of 8 chips: $$2*6 + 1*8 = 12\text{cm} \times 8\text{cm}$$. Allocating even more space (depending on how sophisticated I want the PCB to be) would be unlikely to exceed $$12\text{cm} \times 12\text{cm}$$, which I would still consider reasonable. 

After all this, I found a 4-bit CLA chips with a rather small propogation delay. The data sheet for the DM74LS283 (4-bit adder) claims that typical add times for two 16-bit words is 45ns. The add time for two 32-bit words is unlikely to be more than double this: we have to wait 45ns for the $$16^{\text{th}}$$ carry bit, and it takes another 45ns to add the other two half-words. Assuming we have a usable result from the adder after 100ns, the fastest we could run the clock is at $$10$$Mhz (assuming, for now, the ALU is the component with the longest propagation time). If I can, I would like to make my own, CLA, even if chaining adders is easier (and cheaper).

According to a timing simulation for the full CLA implementation, the longest path through the adder/subtractor is either `FSEL[0]` (the carry-in bit) or `A[0]` to `R[31]`. The two paths are:

* `FSEL[0] -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4070BC -> Cout`
* `A[0] -> CD4070BC -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4081BC -> quad_or -> CD4070BC -> Cout`

Obviously the times depend heavily on how fast we can make the gates. The longest path is comprised of $$18$$ gates: to get performance comparable to $$8$$ $$4$$-bit CLAs, we'd need an average propagation time of $$100/18 = 5.55$$ns per chip. This is achievable, but how much do I want to spend? We want to minimize:

$$
16 \cdot C_{\text{OR}} + 24 \cdot C_{\text{XOR}} + 8 \cdot C_{\text{AND}} = C_T \\
8 \cdot t_{POR} + 2 \cdot t_{PXOR} + 8 \cdot t_{PAND} = t_T
$$

Where $$C_T$$ is the total cost, and $$t_T$$ is the total propagation delay for the longest path through the CLA. I also don't want packages of less than 2 gates per package, otherwise it's too many components for me to place. We can see that, although the `XOR` gates far outnumber the others, they aren't used heavily in any one path: so we might be able to pay a bit less for a slower chip. This also benefits the cost minimization, because we have to buy a lot of `XOR` chips.

Likewise, it's best to spend more on `OR` and `AND` gates, because the signal travels through a lot of them. Ideally, (all things being equal) we want to pay a little bit less for `OR` gates, since we have to buy twice as many `OR` gates than `AND` gates. However, in order of importance, I kept the following things in mind (based on this simple cost analysis model):

* Pay **least** for slow `XOR` gates
* Pay **more** for faster `OR` gates
* Pay **most** for fastest `AND` gates 

For the `AND` gate chip, I found the `74LVC08A` (Nexperia) to be very promising, with the additional bonus of supporting $$4$$ gates per chip. This halves the needed `AND` chips to $$4$$. 

For the `OR` gate chip, the `74ALVC32` family seems good. It also offers $$4$$ gates per chip, lowering my cost and work at the same time (total `OR` chips now $$8$$). Texas Instruments and Nexperia both manufacture very similar versions of this chip, and they both seem to perform about the same on paper (and cost about the same).

Finally, the `XOR` gate chip. Nexperia also manufactures a fast dual gate chip, but I found the `SN74LCC86A` made by Texas Instruments, which offers $$4$$ gates per chip. This brings down the `XOR` chip count to $$12$$. 

Using the maximum propagation times for each chip, we get $$4.1\text{ns}$$ (`AND`), $$2.8\text{ns}$$ (`OR`), and $$4.4\text{ns}$$ (`XOR`), all operating at $$3.6\text{V}$$. This is well below the required $$5.55\text{ns}/\text{chip}$$ average. I think it also may be safe to assume that typical propagation times will be much lower than the maximum.

Based on these hardware choices, we know have $$4 + 8 + 12 = 24$$ chips on the board, half of the original estimated count.

The next thing to consider is the shifting logic, which will also eat up area. The minimum number of multiplexers needed to shift a 32-bit word using a barrel shifter is $$32 \cdot \log{32} = 160$$, assuming a cascade of $$2x1$$ multiplexers. 

# Appendix (Code)

{% highlight verilog %}
module ALU(
    input wire [31:0] A,
    input wire [31:0] B,
    input wire [2:0] FSEL,
    input wire [4:0] SHMT,
    output reg [31:0] R,
    output wire Z, // zero
    output wire N, // negative
    output wire P  // positive
   );
	
wire [31:0] Bp;
assign Bp = B ^ {32{FSEL[0]}};

wire [31:0] I;

assign Z = (R == 0);
assign N = R[31];
assign P = (R > 0);

always @ * begin
    case(FSEL)
        3'b000,                 // A+B
        3'b001: R = I;          // A-B
        3'b010: R = A&B;        // A&B
        3'b011: R = A|B;        // A|B
        3'b100: R = A^B;        // A^B
        3'b101: R = A << SHMT;  // A << SHMT
        3'b110: R = A >> SHMT;  // A >> SHMT
    endcase
end

// carry
wire C1,C2,C3,C4,C5,C6,C7,
     C8,C9,C10,C11,C12,C13,
     C14,C15,C16,C17,C18,C19,
     C20,C21,C22,C23,C24,C25,
     C26,C27,C28,C29,C30,C31,
     C32;
// generate
wire G0,G1,G2,G3,G4,G5,G6,G7,G8,
     G9,G10,G11,G12,G13,G14,G15,G16,G17,
     G18,G19,G20,G21,G22,G23,G24,G25,G26,
     G27,G28,G29,G30,G31;
// propogate
wire P0,P1,P2,P3,P4,P5,P6,P7,P8,
     P9,P10,P11,P12,P13,P14,P15,P16,P17,
     P18,P19,P20,P21,P22,P23,P24,P25,P26,
     P27,P28,P29,P30,P31;

assign G0 = A[0] & Bp[0];
assign G1 = A[1] & Bp[1];
assign G2 = A[2] & Bp[2];
assign G3 = A[3] & Bp[3];
assign G4 = A[4] & Bp[4];
assign G5 = A[5] & Bp[5];
assign G6 = A[6] & Bp[6];
assign G7 = A[7] & Bp[7];
assign G8 = A[8] & Bp[8];
assign G9 = A[9] & Bp[9];
assign G10 = A[10] & Bp[10];
assign G11 = A[11] & Bp[11];
assign G12 = A[12] & Bp[12];
assign G13 = A[13] & Bp[13];
assign G14 = A[14] & Bp[14];
assign G15 = A[15] & Bp[15];
assign G16 = A[16] & Bp[16];
assign G17 = A[17] & Bp[17];
assign G18 = A[18] & Bp[18];
assign G19 = A[19] & Bp[19];
assign G20 = A[20] & Bp[20];
assign G21 = A[21] & Bp[21];
assign G22 = A[22] & Bp[22];
assign G23 = A[23] & Bp[23];
assign G24 = A[24] & Bp[24];
assign G25 = A[25] & Bp[25];
assign G26 = A[26] & Bp[26];
assign G27 = A[27] & Bp[27];
assign G28 = A[28] & Bp[28];
assign G29 = A[29] & Bp[29];
assign G30 = A[30] & Bp[30];
assign G31 = A[31] & Bp[31];

assign P0 = A[0] ^ Bp[0];
assign P1 = A[1] ^ Bp[1];
assign P2 = A[2] ^ Bp[2];
assign P3 = A[3] ^ Bp[3];
assign P4 = A[4] ^ Bp[4];
assign P5 = A[5] ^ Bp[5];
assign P6 = A[6] ^ Bp[6];
assign P7 = A[7] ^ Bp[7];
assign P8 = A[8] ^ Bp[8];
assign P9 = A[9] ^ Bp[9];
assign P10 = A[10] ^ Bp[10];
assign P11 = A[11] ^ Bp[11];
assign P12 = A[12] ^ Bp[12];
assign P13 = A[13] ^ Bp[13];
assign P14 = A[14] ^ Bp[14];
assign P15 = A[15] ^ Bp[15];
assign P16 = A[16] ^ Bp[16];
assign P17 = A[17] ^ Bp[17];
assign P18 = A[18] ^ Bp[18];
assign P19 = A[19] ^ Bp[19];
assign P20 = A[20] ^ Bp[20];
assign P21 = A[21] ^ Bp[21];
assign P22 = A[22] ^ Bp[22];
assign P23 = A[23] ^ Bp[23];
assign P24 = A[24] ^ Bp[24];
assign P25 = A[25] ^ Bp[25];
assign P26 = A[26] ^ Bp[26];
assign P27 = A[27] ^ Bp[27];
assign P28 = A[28] ^ Bp[28];
assign P29 = A[29] ^ Bp[29];
assign P30 = A[30] ^ Bp[30];
assign P31 = A[31] ^ Bp[31];

assign C1 = G0 | P0 & FSEL[0];
assign C2 = G1 | P1 & C1;
assign C3 = G2 | P2 & C2;
assign C4 = G3 | P3 & C3;
assign C5 = G4 | P4 & C4;
assign C6 = G5 | P5 & C5;
assign C7 = G6 | P6 & C6;
assign C8 = G7 | P7 & C7;
assign C9 = G8 | P8 & C8;
assign C10 = G9 | P9 & C9;
assign C11 = G10 | P10 & C10;
assign C12 = G11 | P11 & C11;
assign C13 = G12 | P12 & C12;
assign C14 = G13 | P13 & C13;
assign C15 = G14 | P14 & C14;
assign C16 = G15 | P15 & C15;
assign C17 = G16 | P16 & C16;
assign C18 = G17 | P17 & C17;
assign C19 = G18 | P18 & C18;
assign C20 = G19 | P19 & C19;
assign C21 = G20 | P20 & C20;
assign C22 = G21 | P21 & C21;
assign C23 = G22 | P22 & C22;
assign C24 = G23 | P23 & C23;
assign C25 = G24 | P24 & C24;
assign C26 = G25 | P25 & C25;
assign C27 = G26 | P26 & C26;
assign C28 = G27 | P27 & C27;
assign C29 = G28 | P28 & C28;
assign C30 = G29 | P29 & C29;
assign C31 = G30 | P30 & C30;
assign C32 = G31 | P31 & C31;

assign I[0] = FSEL[0]^P0;
assign I[1] = C1^P1;
assign I[2] = C2^P2;
assign I[3] = C3^P3;
assign I[4] = C4^P4;
assign I[5] = C5^P5;
assign I[6] = C6^P6;
assign I[7] = C7^P7;
assign I[8] = C8^P8;
assign I[9] = C9^P9;
assign I[10] = C10^P10;
assign I[11] = C11^P11;
assign I[12] = C12^P12;
assign I[13] = C13^P13;
assign I[14] = C14^P14;
assign I[15] = C15^P15;
assign I[16] = C16^P16;
assign I[17] = C17^P17;
assign I[18] = C18^P18;
assign I[19] = C19^P19;
assign I[20] = C20^P20;
assign I[21] = C21^P21;
assign I[22] = C22^P22;
assign I[23] = C23^P23;
assign I[24] = C24^P24;
assign I[25] = C25^P25;
assign I[26] = C26^P26;
assign I[27] = C27^P27;
assign I[28] = C28^P28;
assign I[29] = C29^P29;
assign I[30] = C30^P30;
assign I[31] = C31^P31;

endmodule
{% endhighlight %}
