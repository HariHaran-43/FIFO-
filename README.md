# FIFO-
Design a FIFO and done verification using class based test bench architecture.
module full_adder_1bit (

   input  a,

   input  b,

   input  cin,

   output sum,

   output cout

);

 

   wire axb;

   wire axb_cin;

   wire ab;

 

   xor (axb, a, b);

   xor (sum, axb, cin);

 

   and (ab, a, b);

   and (axb_cin, axb, cin);

   or  (cout, ab, axb_cin);

 

endmodule

 

module mux2_1 (

   input  d0,

   input  d1,

   input  sel,

   output y

);

 

   wire nsel;

   wire w0, w1;

 

   not (nsel, sel);

   and (w0, d0, nsel);

   and (w1, d1, sel);

   or  (y, w0, w1);

 

endmodule

 

module mux2_1_4bit (

   input  [3:0] d0,

   input  [3:0] d1,

   input        sel,

   output [3:0] y

);

 

   wire nsel;

 

   not (nsel, sel);

 

   genvar i;

   generate

       for (i = 0; i < 4; i = i + 1) begin : MUX_BITS

           wire w0, w1;

           and (w0, d0[i], nsel);

           and (w1, d1[i], sel);

           or  (y[i], w0, w1);

       end

   endgenerate

 

endmodule

 

 

 

module cla_4bit (

   input  [3:0] A,

   input  [3:0] B,

   input        Cin,

   output [3:0] Sum,

   output       Cout,

   output       PG,

   output       GG

);

 

   wire [3:0] P, G;

   wire C1, C2, C3;

 

   // Bit propagate & generate

   xor (P[0], A[0], B[0]);

   xor (P[1], A[1], B[1]);

   xor (P[2], A[2], B[2]);

   xor (P[3], A[3], B[3]);

 

   and (G[0], A[0], B[0]);

   and (G[1], A[1], B[1]);

   and (G[2], A[2], B[2]);

   and (G[3], A[3], B[3]);

 

   // Carry lookahead inside block

   // C1

   wire p0c;

   and (p0c, P[0], Cin);

   or  (C1, G[0], p0c);

 

   // C2

   wire p1c;

   and (p1c, P[1], C1);

   or  (C2, G[1], p1c);

 

   // C3

   wire p2c;

   and (p2c, P[2], C2);

   or  (C3, G[2], p2c);

 

   // Cout

   wire p3c;

   and (p3c, P[3], C3);

   or  (Cout, G[3], p3c);

 

   // Sum

   xor (Sum[0], P[0], Cin);

   xor (Sum[1], P[1], C1);

   xor (Sum[2], P[2], C2);

   xor (Sum[3], P[3], C3);

 

   // Group propagate: all P must be 1

   wire p01, p23;

   and (p01, P[0], P[1]);

   and (p23, P[2], P[3]);

   and (PG, p01, p23);

 

   // Group generate

   wire g3p, g2p, g1p;

   wire p3g2, p3p2g1, p3p2p1g0;

 

   and (p3g2, P[3], G[2]);

   and (p3p2g1, P[3], P[2], G[1]);

   and (p3p2p1g0, P[3], P[2], P[1], G[0]);

 

   or (GG, G[3], p3g2, p3p2g1, p3p2p1g0);

 

endmodule

 

module cla_60bit (

   input  [59:0] A,

   input  [59:0] B,

   input         Cin,

   output [59:0] Sum,

   output        C60

);

 

   wire [14:0] PG, GG;

   wire [15:0] Cblk;

 

   assign Cblk[0] = Cin;

 

   genvar i;

 

   // Instantiate 15 CLA-4 blocks

   generate

       for (i = 0; i < 15; i = i + 1) begin : CLA4_BLOCKS

           cla_4bit U (

               .A   (A[i*4+3 : i*4]),

               .B   (B[i*4+3 : i*4]),

               .Cin (Cblk[i]),

               .Sum (Sum[i*4+3 : i*4]),

               .Cout(),

               .PG  (PG[i]),

               .GG  (GG[i])

           );

       end

   endgenerate

 

   // Second-level carry lookahead (block carries)

   genvar j;

   generate

       for (j = 0; j < 15; j = j + 1) begin

           wire pgc;

           and (pgc, PG[j], Cblk[j]);

           or  (Cblk[j+1], GG[j], pgc);

       end

   endgenerate

 

   assign C60 = Cblk[15];

 

endmodule

 

 

module ripple_adder_4bit (

   input  [3:0] A,

   input  [3:0] B,

   input        Cin,

   output [3:0] Sum,

   output       Cout

);

 

   wire c1, c2, c3;

 

   full_adder_1bit f0 (A[0], B[0], Cin, Sum[0], c1);

   full_adder_1bit f1 (A[1], B[1], c1,  Sum[1], c2);

   full_adder_1bit f2 (A[2], B[2], c2,  Sum[2], c3);

   full_adder_1bit f3 (A[3], B[3], c3,  Sum[3], Cout);

 

endmodule

 

module bec_4bit (

   input  [3:0] X,

   output [3:0] Y

);

 

   wire w1, w2;

 

   // Y0 = ~X0

   not (Y[0], X[0]);

 

   // Y1 = X1 ^ X0

   xor (Y[1], X[1], X[0]);

 

   // w1 = X1 & X0

   and (w1, X[1], X[0]);

   // Y2 = X2 ^ (X1 & X0)

   xor (Y[2], X[2], w1);

 

   // w2 = X2 & X1 & X0

   and (w2, X[2], w1);

   // Y3 = X3 ^ (X2 & X1 & X0)

   xor (Y[3], X[3], w2);

 

endmodule

 

module csla_4bit (

   input  [3:0] A,

   input  [3:0] B,

   input        Sel,      // from C60

   output [3:0] Sum,

   output       Cout

);

 

   wire [3:0] s0, s1;

   wire c0, c1;

 

   ripple_adder_4bit R0 (A, B, 1'b0, s0, c0);

    bec_4bit BEC (

       .X (s0),

       .Y (s1)

   );

 

   genvar i;

   generate

      mux2_1_4bit SUM_MUX (

       .d0 (s0),

       .d1 (s1),

       .sel(Sel),

       .y  (Sum)

   );

 

   endgenerate

 

   mux2_1 mc (c0, c1, Sel, Cout);

 

endmodule

 

module hybrid_adder_64bit_add_sub (

   input  [63:0] A,

   input  [63:0] B,

   input         op,        // 0 = ADD, 1 = SUB

   output [63:0] Sum,

   output        Cout

);

 

   wire [63:0] B_mod;

   wire carry_mid;

 

   // ----------------------------------

   // B modification: B XOR op

   // ----------------------------------

   genvar i;

   generate

       for (i = 0; i < 64; i = i + 1) begin : B_XOR_OP

           xor (B_mod[i], B[i], op);

       end

   endgenerate

 

   // ----------------------------------

   // Lower 60-bit CLA

   // cin = op (2's complement)

   // ----------------------------------

   cla_60bit CLA_LSB (

       .A   (A[59:0]),

       .B   (B_mod[59:0]),

       .Cin (op),

       .Sum (Sum[59:0]),

       .C60 (carry_mid)

   );

 

   // ----------------------------------

   // Upper 4-bit CSLA

   // carry_mid selects correct path

   // ----------------------------------

   csla_4bit CSLA_MSB (

       .A   (A[63:60]),

       .B   (B_mod[63:60]),

       .Sel (carry_mid),

       .Sum (Sum[63:60]),

       .Cout(Cout)

   );

 

endmodule

 

 

module mux4_1 (

   input  d0,   // AND

   input  d1,   // OR

   input  d2,   // XOR

   input  d3,   // NOT

   input  s0,

   input  s1,

   output y

);

 

   wire ns0, ns1;

   wire w0, w1, w2, w3;

 

   not (ns0, s0);

   not (ns1, s1);

 

   and (w0, d0, ns1, ns0);

   and (w1, d1, ns1, s0);

   and (w2, d2, s1,  ns0);

   and (w3, d3, s1,  s0);

 

   or  (y, w0, w1, w2, w3);

 

endmodule

 

module logic_unit_64bit (

   input  [63:0] A,

   input  [63:0] B,

   input         s0,

   input         s1,

   output [63:0] Y

);

 

   wire [63:0] and_ab;

   wire [63:0] or_ab;

   wire [63:0] xor_ab;

   wire [63:0] not_a;

 

   genvar i;

   generate

       for (i = 0; i < 64; i = i + 1) begin : LOGIC_BITS

 

           // AND

           and (and_ab[i], A[i], B[i]);

 

           // OR

           or  (or_ab[i],  A[i], B[i]);

 

           // XOR

           xor (xor_ab[i], A[i], B[i]);

 

           // NOT

           not (not_a[i],  A[i]);

 

           // Select result

           mux4_1 M (

               .d0(and_ab[i]),

               .d1(or_ab[i]),

               .d2(xor_ab[i]),

               .d3(not_a[i]),

               .s0(s0),

               .s1(s1),

               .y (Y[i])

           );

       end

   endgenerate

 

endmodule

 

 

module lsl_64bit (

   input  [63:0] A,

   input  [5:0]  shamt,

   output [63:0] Y

);

 

   wire [63:0] s1, s2, s3, s4, s5;

 

   genvar i;

 

   // shift by 1

   generate

       for (i=0; i<64; i=i+1) begin

           if (i==0)

               mux2_1 m (A[i], 1'b0, shamt[0], s1[i]);

           else

               mux2_1 m (A[i], A[i-1], shamt[0], s1[i]);

       end

   endgenerate

 

   // shift by 2

   generate

       for (i=0; i<64; i=i+1) begin

           if (i<2)

               mux2_1 m (s1[i], 1'b0, shamt[1], s2[i]);

           else

               mux2_1 m (s1[i], s1[i-2], shamt[1], s2[i]);

       end

   endgenerate

 

   // shift by 4

   generate

       for (i=0; i<64; i=i+1) begin

           if (i<4)

               mux2_1 m (s2[i], 1'b0, shamt[2], s3[i]);

           else

               mux2_1 m (s2[i], s2[i-4], shamt[2], s3[i]);

       end

   endgenerate

 

   // shift by 8

   generate

       for (i=0; i<64; i=i+1) begin

           if (i<8)

               mux2_1 m (s3[i], 1'b0, shamt[3], s4[i]);

           else

               mux2_1 m (s3[i], s3[i-8], shamt[3], s4[i]);

       end

   endgenerate

 

   // shift by 16

   generate

       for (i=0; i<64; i=i+1) begin

           if (i<16)

               mux2_1 m (s4[i], 1'b0, shamt[4], s5[i]);

           else

               mux2_1 m (s4[i], s4[i-16], shamt[4], s5[i]);

       end

   endgenerate

 

   // shift by 32

   generate

       for (i=0; i<64; i=i+1) begin

           if (i<32)

               mux2_1 m (s5[i], 1'b0, shamt[5], Y[i]);

           else

               mux2_1 m (s5[i], s5[i-32], shamt[5], Y[i]);

       end

   endgenerate

 

endmodule

 

 

module right_shifter_64bit (

   input  [63:0] A,

   input  [5:0]  shamt,

   input         arith,   // 0 = LSR, 1 = ASR

   output [63:0] Y

);

 

   wire sign;

   buf (sign, A[63]);

 

   wire fill;

   mux2_1 f (1'b0, sign, arith, fill);

 

   wire [63:0] s1, s2, s3, s4, s5;

 

   genvar i;

 

   // shift by 1

   generate

       for (i=0; i<64; i=i+1) begin

           if (i==63)

               mux2_1 m (A[i], fill, shamt[0], s1[i]);

           else

               mux2_1 m (A[i], A[i+1], shamt[0], s1[i]);

       end

   endgenerate

 

   // shift by 2

   generate

       for (i=0; i<64; i=i+1) begin

           if (i>61)

               mux2_1 m (s1[i], fill, shamt[1], s2[i]);

           else

               mux2_1 m (s1[i], s1[i+2], shamt[1], s2[i]);

       end

   endgenerate

 

   // shift by 4

   generate

       for (i=0; i<64; i=i+1) begin

           if (i>59)

               mux2_1 m (s2[i], fill, shamt[2], s3[i]);

           else

               mux2_1 m (s2[i], s2[i+4], shamt[2], s3[i]);

       end

   endgenerate

 

   // shift by 8

   generate

       for (i=0; i<64; i=i+1) begin

           if (i>55)

               mux2_1 m (s3[i], fill, shamt[3], s4[i]);

           else

               mux2_1 m (s3[i], s3[i+8], shamt[3], s4[i]);

       end

   endgenerate

 

   // shift by 16

   generate

       for (i=0; i<64; i=i+1) begin

           if (i>47)

               mux2_1 m (s4[i], fill, shamt[4], s5[i]);

           else

               mux2_1 m (s4[i], s4[i+16], shamt[4], s5[i]);

       end

   endgenerate

 

   // shift by 32

   generate

       for (i=0; i<64; i=i+1) begin

           if (i>31)

               mux2_1 m (s5[i], fill, shamt[5], Y[i]);

           else

               mux2_1 m (s5[i], s5[i+32], shamt[5], Y[i]);

       end

   endgenerate

 

endmodule

 

 

module shift_unit_64bit (

   input  [63:0] A,

   input  [5:0]  shamt,

   input         s0,

   input         s1,

   output [63:0] Y

);

 

   wire [63:0] lsl_out;

   wire [63:0] lsr_asr_out;

 

   lsl_64bit LSL (

       .A(A),

       .shamt(shamt),

       .Y(lsl_out)

   );

 

   right_shifter_64bit RSH (

       .A(A),

       .shamt(shamt),

       .arith(s1),   // s1=1 → ASR

       .Y(lsr_asr_out)

   );

 

   genvar i;

   generate

       for (i=0; i<64; i=i+1) begin

           // s0 selects left or right

           mux2_1 m (lsl_out[i], lsr_asr_out[i], s0, Y[i]);

       end

   endgenerate

 

endmodule

 

 

module alu_top_64bit (

   input         clk,

   input         rst,

 

   input  [63:0] A,

   input  [63:0] B,

   input  [5:0]  shamt,

   input  [3:0]  opcode,

 

   output reg [63:0] RESULT,

   output reg        COUT

);

​

   // =====================================================

   // INPUT REGISTERS

   // =====================================================

   reg [63:0] A_r, B_r;

   reg [5:0]  shamt_r;

   reg [3:0]  opcode_r;

 

   always @(posedge clk or posedge rst) begin

       if (rst) begin

           A_r      <= 64'd0;

           B_r      <= 64'd0;

           shamt_r  <= 6'd0;

           opcode_r <= 4'd0;

       end else begin

           A_r      <= A;

           B_r      <= B;

           shamt_r  <= shamt;

           opcode_r <= opcode;

       end

   end

 

   // =====================================================

   // INTERNAL WIRES (UNCHANGED LOGIC BELOW)

   // =====================================================

   wire [63:0] arith_out;

   wire [63:0] logic_out;

   wire [63:0] shift_out;

 

   wire op;

   wire logic_s0, logic_s1;

   wire shift_s0, shift_s1;

 

   wire sel_arith;

   wire sel_logic;

   wire sel_shift;

 

   wire n0, n1, n2, n3;

 

   not (n0, opcode_r[0]);

   not (n1, opcode_r[1]);

   not (n2, opcode_r[2]);

   not (n3, opcode_r[3]);

 

   buf (op, opcode_r[0]);

 

   wire is_add, is_sub;

   and (is_add, n3, n2, n1, n0);

   and (is_sub, n3, n2, n1, opcode_r[0]);

   or  (sel_arith, is_add, is_sub);

 

   and (logic_and, n3, n2, opcode_r[1], n0);

   and (logic_or,  n3, n2, opcode_r[1], opcode_r[0]);

   and (logic_xor, n3, opcode_r[2], n1, n0);

   and (logic_not, n3, opcode_r[2], n1, opcode_r[0]);

 

   or (sel_logic, logic_and, logic_or, logic_xor, logic_not);

 

   buf (logic_s1, opcode_r[2]);

   buf (logic_s0, opcode_r[0]);

 

   and (shift_lsl, n3, opcode_r[2], opcode_r[1], n0);

   and (shift_lsr, n3, opcode_r[2], opcode_r[1], opcode_r[0]);

   and (shift_asr, opcode_r[3], n2, n1, n0);

 

   or (sel_shift, shift_lsl, shift_lsr, shift_asr);

 

   and (shift_s1, opcode_r[3], n2, n1, n0);

 

   or  (shift_s0,

        (n3 & opcode_r[2] & opcode_r[1] & opcode_r[0]),

        (opcode_r[3] & n2 & n1 & n0)

   );

 

   // =====================================================

   // DATAPATH BLOCKS (UNCHANGED)

   // =====================================================

   hybrid_adder_64bit_add_sub ADDER (

       .A   (A_r),

       .B   (B_r),

       .op  (op),

       .Sum (arith_out),

       .Cout(COUT_c)

   );

 

   logic_unit_64bit LOGIC (

       .A (A_r),

       .B (B_r),

       .s0(logic_s0),

       .s1(logic_s1),

       .Y (logic_out)

   );

 

   shift_unit_64bit SHIFT (

       .A    (A_r),

       .shamt(shamt_r),

       .s0   (shift_s0),

       .s1   (shift_s1),

       .Y    (shift_out)

   );

 

   // =====================================================

   // FINAL MUX (UNCHANGED)

   // =====================================================

   wire [63:0] RESULT_c;

   wire        COUT_c;

 

   genvar i;

   generate

       for (i = 0; i < 64; i = i + 1) begin : FINAL_MUX

           wire t1;

           mux2_1 m1 (arith_out[i], logic_out[i], sel_logic, t1);

           mux2_1 m2 (t1, shift_out[i], sel_shift, RESULT_c[i]);

       end

   endgenerate

 

   // =====================================================

   // OUTPUT REGISTERS

   // =====================================================

   always @(posedge clk or posedge rst) begin

       if (rst) begin

           RESULT <= 64'd0;

           COUT   <= 1'b0;

       end else begin

           RESULT <= RESULT_c;

           COUT   <= COUT_c;

       end

   end

 

endmodule*/

 

module full_adder_1bit (

   input  a,

   input  b,

   input  cin,

   output sum,

   output cout

);

 

   wire axb;

   wire axb_cin;

   wire ab;

 

   xor (axb, a, b);

   xor (sum, axb, cin);

 

   and (ab, a, b);

   and (axb_cin, axb, cin);

   or  (cout, ab, axb_cin);

 

endmodule

 

module mux2_1 (

   input  d0,

   input  d1,

   input  sel,

   output y

);

 

   wire nsel;

   wire w0, w1;

 

   not (nsel, sel);

   and (w0, d0, nsel);

   and (w1, d1, sel);

   or  (y, w0, w1);

 

endmodule

 

module mux2_1_4bit (

   input  [3:0] d0,

   input  [3:0] d1,

   input        sel,

   output [3:0] y

);

 

   wire nsel;

 

   not (nsel, sel);

 

   genvar i;

   generate

       for (i = 0; i < 4; i = i + 1) begin : MUX_BITS

           wire w0, w1;

           and (w0, d0[i], nsel);

           and (w1, d1[i], sel);

           or  (y[i], w0, w1);

       end

   endgenerate

 

endmodule

 

 

 

module cla_4bit (

   input  [3:0] A,

   input  [3:0] B,

   input        Cin,

   output [3:0] Sum,

   output       Cout,

   output       PG,

   output       GG

);

 

   wire [3:0] P, G;

   wire C1, C2, C3;

 

   // Bit propagate & generate

   xor (P[0], A[0], B[0]);

   xor (P[1], A[1], B[1]);

   xor (P[2], A[2], B[2]);

   xor (P[3], A[3], B[3]);

 

   and (G[0], A[0], B[0]);

   and (G[1], A[1], B[1]);

   and (G[2], A[2], B[2]);

   and (G[3], A[3], B[3]);

 

   // Carry lookahead inside block

   // C1

   wire p0c;

   and (p0c, P[0], Cin);

   or  (C1, G[0], p0c);

 

   // C2

   wire p1c;

   and (p1c, P[1], C1);

   or  (C2, G[1], p1c);

 

   // C3

   wire p2c;

   and (p2c, P[2], C2);

   or  (C3, G[2], p2c);

 

   // Cout

   wire p3c;

   and (p3c, P[3], C3);

   or  (Cout, G[3], p3c);

 

   // Sum

   xor (Sum[0], P[0], Cin);

   xor (Sum[1], P[1], C1);

   xor (Sum[2], P[2], C2);

   xor (Sum[3], P[3], C3);

 

   // Group propagate: all P must be 1

   wire p01, p23;

   and (p01, P[0], P[1]);

   and (p23, P[2], P[3]);

   and (PG, p01, p23);

 

   // Group generate

   wire g3p, g2p, g1p;

   wire p3g2, p3p2g1, p3p2p1g0;

 

   and (p3g2, P[3], G[2]);

   and (p3p2g1, P[3], P[2], G[1]);

   and (p3p2p1g0, P[3], P[2], P[1], G[0]);

 

   or (GG, G[3], p3g2, p3p2g1, p3p2p1g0);

 

endmodule

 

module cla_60bit (

   input  [59:0] A,

   input  [59:0] B,

   input         Cin,

   output [59:0] Sum,

   output        C60

);

 

   wire [14:0] PG, GG;

   wire [15:0] Cblk;

 

   assign Cblk[0] = Cin;

 

   genvar i;

 

   // Instantiate 15 CLA-4 blocks

   generate

       for (i = 0; i < 15; i = i + 1) begin : CLA4_BLOCKS

           cla_4bit U (

               .A   (A[i*4+3 : i*4]),

               .B   (B[i*4+3 : i*4]),

               .Cin (Cblk[i]),

               .Sum (Sum[i*4+3 : i*4]),

               .Cout(),

               .PG  (PG[i]),

               .GG  (GG[i])

           );

       end

   endgenerate

 

   // Second-level carry lookahead (block carries)

   genvar j;

   generate

       for (j = 0; j < 15; j = j + 1) begin

           wire pgc;

           and (pgc, PG[j], Cblk[j]);

           or  (Cblk[j+1], GG[j], pgc);

       end

   endgenerate

 

   assign C60 = Cblk[15];

 

endmodule

 

 

module ripple_adder_4bit (

   input  [3:0] A,

   input  [3:0] B,

   input        Cin,

   output [3:0] Sum,

   output       Cout

);

 

   wire c1, c2, c3;

 

   full_adder_1bit f0 (A[0], B[0], Cin, Sum[0], c1);

   full_adder_1bit f1 (A[1], B[1], c1,  Sum[1], c2);

   full_adder_1bit f2 (A[2], B[2], c2,  Sum[2], c3);

   full_adder_1bit f3 (A[3], B[3], c3,  Sum[3], Cout);

 

endmodule

 

module bec_4bit (

   input  [3:0] X,

   output [3:0] Y

);

 

   wire w1, w2;

 

   // Y0 = ~X0

   not (Y[0], X[0]);

 

   // Y1 = X1 ^ X0

   xor (Y[1], X[1], X[0]);

 

   // w1 = X1 & X0

   and (w1, X[1], X[0]);

   // Y2 = X2 ^ (X1 & X0)

   xor (Y[2], X[2], w1);

 

   // w2 = X2 & X1 & X0

   and (w2, X[2], w1);

   // Y3 = X3 ^ (X2 & X1 & X0)

   xor (Y[3], X[3], w2);

 

endmodule

 

module csla_4bit (

   input  [3:0] A,

   input  [3:0] B,

   input        Sel,      // from C60

   output [3:0] Sum,

   output       Cout

);

 

   wire [3:0] s0, s1;

   wire c0, c1;

 

   ripple_adder_4bit R0 (A, B, 1'b0, s0, c0);

    bec_4bit BEC (

       .X (s0),

       .Y (s1)

   );

 

   genvar i;

   generate

      mux2_1_4bit SUM_MUX (

       .d0 (s0),

       .d1 (s1),

       .sel(Sel),

       .y  (Sum)

   );

 

   endgenerate

  wire c1;

   or (c1, c0, Sel);

 

   mux2_1 mc (c0, c1, Sel, Cout);

 

endmodule

 

module hybrid_adder_64bit_add_sub (

   input  [63:0] A,

   input  [63:0] B,

   input         op,        // 0 = ADD, 1 = SUB

   output [63:0] Sum,

   output        Cout

);

 

   wire [63:0] B_mod;

   wire carry_mid;

 

   // ----------------------------------

   // B modification: B XOR op

   // ----------------------------------

   genvar i;

   generate

       for (i = 0; i < 64; i = i + 1) begin : B_XOR_OP

           xor (B_mod[i], B[i], op);

       end

   endgenerate

 

   // ----------------------------------

   // Lower 60-bit CLA

   // cin = op (2's complement)

   // ----------------------------------

   cla_60bit CLA_LSB (

       .A   (A[59:0]),

       .B   (B_mod[59:0]),

       .Cin (op),

       .Sum (Sum[59:0]),

       .C60 (carry_mid)

   );

 

   // ----------------------------------

   // Upper 4-bit CSLA

   // carry_mid selects correct path

   // ----------------------------------

   csla_4bit CSLA_MSB (

       .A   (A[63:60]),

       .B   (B_mod[63:60]),

       .Sel (carry_mid),

       .Sum (Sum[63:60]),

       .Cout(Cout)

   );

 

endmodule

 

 

module mux4_1 (

   input  d0,   // AND

   input  d1,   // OR

   input  d2,   // XOR

   input  d3,   // NOT

   input  s0,

   input  s1,

   output y

);

 

   wire ns0, ns1;

   wire w0, w1, w2, w3;

 

   not (ns0, s0);

   not (ns1, s1);

 

   and (w0, d0, ns1, ns0);

   and (w1, d1, ns1, s0);

   and (w2, d2, s1,  ns0);

   and (w3, d3, s1,  s0);

 

   or  (y, w0, w1, w2, w3);

 

endmodule

 

module logic_unit_64bit (

   input  [63:0] A,

   input  [63:0] B,

   input         s0,

   input         s1,

   output [63:0] Y

);

 

   wire [63:0] and_ab;

   wire [63:0] or_ab;

   wire [63:0] xor_ab;

   wire [63:0] not_a;

 

   genvar i;

   generate

       for (i = 0; i < 64; i = i + 1) begin : LOGIC_BITS

 

           // AND

           and (and_ab[i], A[i], B[i]);

 

           // OR

           or  (or_ab[i],  A[i], B[i]);

 

           // XOR

           xor (xor_ab[i], A[i], B[i]);

 

           // NOT

           not (not_a[i],  A[i]);

 

           // Select result

           mux4_1 M (

               .d0(and_ab[i]),

               .d1(or_ab[i]),

               .d2(xor_ab[i]),

               .d3(not_a[i]),

               .s0(s0),

               .s1(s1),

               .y (Y[i])

           );

       end

   endgenerate

 

endmodule

 

 

module lsl_64bit (

   input  [63:0] A,

   input  [5:0]  shamt,

   output [63:0] Y

);

 

   wire [63:0] s1, s2, s3, s4, s5;

 

   genvar i;

 

   // shift by 1

   generate

       for (i=0; i<64; i=i+1) begin

           if (i==0)

               mux2_1 m (A[i], 1'b0, shamt[0], s1[i]);

           else

               mux2_1 m (A[i], A[i-1], shamt[0], s1[i]);

       end

   endgenerate

 

   // shift by 2

   generate

       for (i=0; i<64; i=i+1) begin

           if (i<2)

               mux2_1 m (s1[i], 1'b0, shamt[1], s2[i]);

           else

               mux2_1 m (s1[i], s1[i-2], shamt[1], s2[i]);

       end

   endgenerate

 

   // shift by 4

   generate

       for (i=0; i<64; i=i+1) begin

           if (i<4)

               mux2_1 m (s2[i], 1'b0, shamt[2], s3[i]);

           else

               mux2_1 m (s2[i], s2[i-4], shamt[2], s3[i]);

       end

   endgenerate

 

   // shift by 8

   generate

       for (i=0; i<64; i=i+1) begin

           if (i<8)

               mux2_1 m (s3[i], 1'b0, shamt[3], s4[i]);

           else

               mux2_1 m (s3[i], s3[i-8], shamt[3], s4[i]);

       end

   endgenerate

 

   // shift by 16

   generate

       for (i=0; i<64; i=i+1) begin

           if (i<16)

               mux2_1 m (s4[i], 1'b0, shamt[4], s5[i]);

           else

               mux2_1 m (s4[i], s4[i-16], shamt[4], s5[i]);

       end

   endgenerate

 

   // shift by 32

   generate

       for (i=0; i<64; i=i+1) begin

           if (i<32)

               mux2_1 m (s5[i], 1'b0, shamt[5], Y[i]);

           else

               mux2_1 m (s5[i], s5[i-32], shamt[5], Y[i]);

       end

   endgenerate

 

endmodule

 

 

module right_shifter_64bit (

   input  [63:0] A,

   input  [5:0]  shamt,

   input         arith,   // 0 = LSR, 1 = ASR

   output [63:0] Y

);

 

   wire sign;

   buf (sign, A[63]);

 

   wire fill;

   mux2_1 f (1'b0, sign, arith, fill);

 

   wire [63:0] s1, s2, s3, s4, s5;

 

   genvar i;

 

   // shift by 1

   generate

       for (i=0; i<64; i=i+1) begin

           if (i==63)

               mux2_1 m (A[i], fill, shamt[0], s1[i]);

           else

               mux2_1 m (A[i], A[i+1], shamt[0], s1[i]);

       end

   endgenerate

 

   // shift by 2

   generate

       for (i=0; i<64; i=i+1) begin

           if (i>61)

               mux2_1 m (s1[i], fill, shamt[1], s2[i]);

           else

               mux2_1 m (s1[i], s1[i+2], shamt[1], s2[i]);

       end

   endgenerate

 

   // shift by 4

   generate

       for (i=0; i<64; i=i+1) begin

           if (i>59)

               mux2_1 m (s2[i], fill, shamt[2], s3[i]);

           else

               mux2_1 m (s2[i], s2[i+4], shamt[2], s3[i]);

       end

   endgenerate

 

   // shift by 8

   generate

       for (i=0; i<64; i=i+1) begin

           if (i>55)

               mux2_1 m (s3[i], fill, shamt[3], s4[i]);

           else

               mux2_1 m (s3[i], s3[i+8], shamt[3], s4[i]);

       end

   endgenerate

 

   // shift by 16

   generate

       for (i=0; i<64; i=i+1) begin

           if (i>47)

               mux2_1 m (s4[i], fill, shamt[4], s5[i]);

           else

               mux2_1 m (s4[i], s4[i+16], shamt[4], s5[i]);

       end

   endgenerate

 

   // shift by 32

   generate

       for (i=0; i<64; i=i+1) begin

           if (i>31)

               mux2_1 m (s5[i], fill, shamt[5], Y[i]);

           else

               mux2_1 m (s5[i], s5[i+32], shamt[5], Y[i]);

       end

   endgenerate

 

endmodule

 

 

module shift_unit_64bit (

   input  [63:0] A,

   input  [5:0]  shamt,

   input         s0,

   input         s1,

   output [63:0] Y

);

 

   wire [63:0] lsl_out;

   wire [63:0] lsr_asr_out;

 

   lsl_64bit LSL (

       .A(A),

       .shamt(shamt),

       .Y(lsl_out)

   );

 

   right_shifter_64bit RSH (

       .A(A),

       .shamt(shamt),

       .arith(s1),   // s1=1 → ASR

       .Y(lsr_asr_out)

   );

 

   genvar i;

   generate

       for (i=0; i<64; i=i+1) begin

           // s0 selects left or right

           mux2_1 m (lsl_out[i], lsr_asr_out[i], s0, Y[i]);

       end

   endgenerate

 

endmodule

 

 

module alu_top_64bit (

   input         clk,

   input         rst,

 

   input  [63:0] A,

   input  [63:0] B,

   input  [5:0]  shamt,

   input  [3:0]  opcode,

 

   output reg [63:0] RESULT,

   output reg        COUT

);

​

   // =====================================================

   // INPUT REGISTERS

   // =====================================================

   reg [63:0] A_r, B_r;

   reg [5:0]  shamt_r;

   reg [3:0]  opcode_r;

 

   always @(posedge clk or posedge rst) begin

       if (rst) begin

           A_r      <= 64'd0;

           B_r      <= 64'd0;

           shamt_r  <= 6'd0;

           opcode_r <= 4'd0;

       end else begin

           A_r      <= A;

           B_r      <= B;

           shamt_r  <= shamt;

           opcode_r <= opcode;

       end

   end

 

   // =====================================================

   // INTERNAL WIRES (UNCHANGED LOGIC BELOW)

   // =====================================================

   wire [63:0] arith_out;

   wire [63:0] logic_out;

   wire [63:0] shift_out;

 

   wire op;

   wire logic_s0, logic_s1;

   wire shift_s0, shift_s1;

 

   wire sel_arith;

   wire sel_logic;

   wire sel_shift;

 

   wire n0, n1, n2, n3;

 

   not (n0, opcode_r[0]);

   not (n1, opcode_r[1]);

   not (n2, opcode_r[2]);

   not (n3, opcode_r[3]);

 

   buf (op, opcode_r[0]);

 

   wire is_add, is_sub;

   and (is_add, n3, n2, n1, n0);

   and (is_sub, n3, n2, n1, opcode_r[0]);

   or  (sel_arith, is_add, is_sub);

 

   and (logic_and, n3, n2, opcode_r[1], n0);

   and (logic_or,  n3, n2, opcode_r[1], opcode_r[0]);

   and (logic_xor, n3, opcode_r[2], n1, n0);

   and (logic_not, n3, opcode_r[2], n1, opcode_r[0]);

 

   or (sel_logic, logic_and, logic_or, logic_xor, logic_not);

 

   buf (logic_s1, opcode_r[2]);

   buf (logic_s0, opcode_r[0]);

 

   and (shift_lsl, n3, opcode_r[2], opcode_r[1], n0);

   and (shift_lsr, n3, opcode_r[2], opcode_r[1], opcode_r[0]);

   and (shift_asr, opcode_r[3], n2, n1, n0);

 

   or (sel_shift, shift_lsl, shift_lsr, shift_asr);

 

   and (shift_s1, opcode_r[3], n2, n1, n0);

 

   or  (shift_s0,

        (n3 & opcode_r[2] & opcode_r[1] & opcode_r[0]),

        (opcode_r[3] & n2 & n1 & n0)

   );

 

   // =====================================================

   // DATAPATH BLOCKS (UNCHANGED)

   // =====================================================

   hybrid_adder_64bit_add_sub ADDER (

       .A   (A_r),

       .B   (B_r),

       .op  (op),

       .Sum (arith_out),

       .Cout(COUT_c)

   );

 

   logic_unit_64bit LOGIC (

       .A (A_r),

       .B (B_r),

       .s0(logic_s0),

       .s1(logic_s1),

       .Y (logic_out)

   );

 

   shift_unit_64bit SHIFT (

       .A    (A_r),

       .shamt(shamt_r),

       .s0   (shift_s0),

       .s1   (shift_s1),

       .Y    (shift_out)

   );

 

   // =====================================================

   // FINAL MUX (UNCHANGED)

   // =====================================================

   wire [63:0] RESULT_c;

   wire        COUT_c;

 

   genvar i;

   generate

       for (i = 0; i < 64; i = i + 1) begin : FINAL_MUX

           wire t1;

           mux2_1 m1 (arith_out[i], logic_out[i], sel_logic, t1);

           mux2_1 m2 (t1, shift_out[i], sel_shift, RESULT_c[i]);

       end

   endgenerate

 

   // =====================================================

   // OUTPUT REGISTERS

   // =====================================================

   always @(posedge clk or posedge rst) begin

       if (rst) begin

           RESULT <= 64'd0;

           COUT   <= 1'b0;

       end else begin

           RESULT <= RESULT_c;

           COUT   <= COUT_c;

       end

   end

 

endmodule
