### Basic ALU SV Verification

### DUT
```system verilog
module alu (
  input  logic [3:0] a,
  input  logic [3:0] b,
  input  logic [1:0] sel,
  output logic [3:0] y
  
);
  

always_comb begin
  case (sel)
    2'b00: y = a + b;
    2'b01: y = a - b;
    2'b10: y = a & b;
    2'b11: y = a | b;
    default: y = 0;
  endcase
end

endmodule
```
---
### Interface
---
```system verilog

interface alu_if;
  logic clk;
  logic [3:0] a;
  logic [3:0] b;
  logic [1:0] sel;
  logic [3:0] y;
endinterface
```
---
### Verification
---
```system verilog 
class transaction;

  rand logic [3:0] a;
  rand logic [3:0] b;
  rand logic [1:0] sel;
  logic [3:0] y;

endclass

```
---
```system verilog
class generator;

  mailbox gen2drv;

  function new(mailbox gen2drv);
    this.gen2drv = gen2drv;
  endfunction

  task run();

    transaction tr;

    repeat(20) begin
      tr = new();
      tr.randomize();
      gen2drv.put(tr);
    end

  endtask

endclass

```
---
```system verilog
class driver;

  mailbox gen2drv;
  virtual alu_if aif;

  function new(mailbox gen2drv, virtual alu_if aif);

    this.gen2drv = gen2drv;
    this.aif = aif;

  endfunction


  task run();

    transaction tr;

    forever begin

      gen2drv.get(tr);

      @(negedge aif.clk);

      aif.a   <= tr.a;
      aif.b   <= tr.b;
      aif.sel <= tr.sel;

    end

  endtask

endclass
```
---
```system verilog
class monitor;

  mailbox mon2scr;
  virtual alu_if aif;

  function new(mailbox mon2scr, virtual alu_if aif);

    this.mon2scr = mon2scr;
    this.aif = aif;

  endfunction


  task run();

    transaction tr;

    forever begin

      tr = new();

      @(posedge aif.clk);

      tr.a   = aif.a;
      tr.b   = aif.b;
      tr.sel = aif.sel;
      tr.y   = aif.y;

      mon2scr.put(tr);

    end

  endtask

endclass

```
---
```system verilog
class scoreboard;

  mailbox mon2scr;

  function new(mailbox mon2scr);

    this.mon2scr = mon2scr;

  endfunction


  task run();

    transaction tr;
    logic [3:0] exp;
    int count = 0;

    forever begin

      mon2scr.get(tr);

      case(tr.sel)

        2'b00: exp = tr.a + tr.b;
        2'b01: exp = tr.a - tr.b;
        2'b10: exp = tr.a & tr.b;
        2'b11: exp = tr.a | tr.b;

      endcase


      if(exp == tr.y)

        $display("PASS a=%0d b=%0d sel=%0d y=%0d",
                  tr.a, tr.b, tr.sel, tr.y);

      else

        $display("FAIL exp=%0d act=%0d",
                  exp, tr.y);


      count++;

      if(count == 20)
        $finish;

    end

  endtask

endclass
```
---
```system verilog
class env;

  generator gen;
  driver drv;
  monitor mon;
  scoreboard scb;

  mailbox gen2drv;
  mailbox mon2scr;

  virtual alu_if aif;


  function new(virtual alu_if aif);

    this.aif = aif;

    gen2drv = new();
    mon2scr = new();

    gen = new(gen2drv);
    drv = new(gen2drv, aif);
    mon = new(mon2scr, aif);
    scb = new(mon2scr);

  endfunction


  task run();

    fork

      gen.run();
      drv.run();
      mon.run();
      scb.run();

    join

  endtask

endclass

```
---
```system verilog
module tb;

  alu_if aif();

  env e;

  alu  dut(.a(aif.a), .b(aif.b),.sel(aif.sel),.y(aif.y));

  initial begin
    aif.clk = 0;
    forever #5 aif.clk = ~aif.clk;
  end

  initial begin
    e = new(aif);
    e.run();
  end

endmodule
```
---
