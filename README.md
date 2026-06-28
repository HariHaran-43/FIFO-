# FIFO-
Design a FIFO and done verification using class based test bench architecture.

module fifo (
    input        clk,
    input        rst,
    input        wr_en,
    input        rd_en,
    input  [7:0] din,
    output reg [7:0] dout,
    output       full,
    output       empty
);

    reg [7:0] mem [0:3];
    reg [1:0] wr_ptr;
    reg [1:0] rd_ptr;
    reg [2:0] count;

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            wr_ptr <= 2'b00;
            rd_ptr <= 2'b00;
            count  <= 3'b000;
            dout   <= 8'b00;

            for (int i = 0; i < 4; i++)
                mem[i] <= 8'b00;
        end

        else if (wr_en && !rd_en && !full) begin
            mem[wr_ptr] <= din;
            wr_ptr <= wr_ptr + 1;
            count  <= count + 1;
        end

        else if (rd_en && !wr_en && !empty) begin
            dout   <= mem[rd_ptr];
            rd_ptr <= rd_ptr + 1;
            count  <= count - 1;
        end

        else if (wr_en && rd_en && !full && !empty) begin
            mem[wr_ptr] <= din;
            dout        <= mem[rd_ptr];

            wr_ptr <= wr_ptr + 1;
            rd_ptr <= rd_ptr + 1;
        end
    end

    assign empty = (count == 0);
    assign full  = (count == 4);

endmodule
