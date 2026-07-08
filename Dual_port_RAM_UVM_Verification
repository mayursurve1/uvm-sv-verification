# Dual_Port RAM Implementation

## DUT Dual Port Ram
---
```systemverilog
module dual_port_ram 
    (   
        input  logic       clk,         // clock
        input  logic       wr_en,       // write enable for port 0
        input  logic [7:0] data_in,     // Input data to port 0.
        input  logic [3:0] addr_in_0,   // address for port 0
        input  logic [3:0] addr_in_1,   // address for port 1
        input  logic       port_en_0,   // enable port 0.
        input  logic       port_en_1,   // enable port 1.
        output logic [7:0] data_out_0,   // output data from port 0.
        output logic [7:0] data_out_1     // output data from port 1.
    );
  
logic [7:0] ram[0:15];
  
always_ff @(posedge clk)
begin
    if (port_en_0 == 1 && wr_en == 1) 
        ram[addr_in_0] <= data_in;
end

assign data_out_0 = port_en_0 ? ram[addr_in_0] : 'z;   
assign data_out_1 = port_en_1 ? ram[addr_in_1] : 'z;   

endmodule
```


##Interface

```systemverilog

interface ram_if();
  
  logic  clk;      
  logic  wr_en;     
  logic [7:0]data_in;    
  logic [3:0]addr_in_0;   
  logic [3:0]addr_in_1;   
  logic  port_en_0;   
  logic   port_en_1;   
  logic [7:0] data_out_0;
  logic [7:0] data_out_1;
  
endinterface
```
---
##Verification 
---
```systemverilog
`include "uvm_macros.svh"

import uvm_pkg::*;

class trans extends uvm_sequence_item;
  

   rand bit  wr_en;     
   rand bit[7:0]data_in;    
   rand bit[3:0]addr_in_0;   
   rand bit[3:0]addr_in_1;   
   rand bit  port_en_0;   
   rand bit  port_en_1;   
  
  
  bit[7:0] data_out_0;
  bit [7:0] data_out_1;
  
  

  function new(string name = "trans");
    super.new(name);
  endfunction
  
  `uvm_object_utils_begin(trans)
  
  `uvm_field_int(wr_en,UVM_DEFAULT)
  `uvm_field_int(data_in, UVM_DEFAULT)
  `uvm_field_int(addr_in_0, UVM_DEFAULT)
  `uvm_field_int(addr_in_1, UVM_DEFAULT)
  `uvm_field_int(port_en_0, UVM_DEFAULT)
  `uvm_field_int(port_en_1, UVM_DEFAULT)
  
  `uvm_field_int(data_out_0, UVM_DEFAULT)
  `uvm_field_int(data_out_1, UVM_DEFAULT)
  `uvm_object_utils_end
  
  
  
  
  constraint c1{
   addr_in_0 inside {[0:15]};
    addr_in_1 inside {[0:15]};
  }
  constraint c2{
    wr_en==1;
    port_en_0==1;
    port_en_1==1;
}
  
endclass:trans




class gnr extends uvm_sequence#(trans);
  `uvm_object_utils(gnr)
  
  trans t;

  function new(string name = "gnr");
    super.new(name);    
  endfunction
  
  virtual task body();
    t = trans::type_id::create("t");
    repeat(20) begin
      start_item(t);
      if (!t.randomize()) begin
        `uvm_error("gnr", "Randomization failed")
      end
      finish_item(t);
      `uvm_info("gnr", $sformatf(" Generated transaction:\n%s", t.sprint()), UVM_LOW)
    end
  endtask
endclass: gnr





class driver extends uvm_driver#(trans);
  `uvm_component_utils(driver)
  
  function new(string name = "driver", uvm_component parent = null);
    super.new(name, parent);
  endfunction
  
     trans tr;
     virtual ram_if rif;

   virtual function void build_phase(uvm_phase phase);
    
         super.build_phase(phase);
    
         tr = trans::type_id::create("tr", this);
       if(!uvm_config_db #(virtual ram_if )::get(this, "", "rif", rif)) 
        `uvm_error("DRV", "Unable to access uvm_config_db");

    endfunction
  
  virtual task run_phase(uvm_phase phase);
    forever begin
      seq_item_port.get_next_item(tr);
      
      
      
      rif.wr_en=tr.wr_en;
      rif.data_in=tr.data_in;
      
      rif.addr_in_0=tr.addr_in_0;
      rif.addr_in_1=tr.addr_in_1;
      rif.port_en_0=tr.port_en_0;
      rif.port_en_1=tr.port_en_1;

       @(posedge rif.clk);
     
      `uvm_info("drv", $sformatf(" %s", tr.sprint() ), UVM_NONE)
      seq_item_port.item_done();
     end
   endtask
endclass: driver




class monitor extends uvm_monitor;
  `uvm_component_utils(monitor)
  
  uvm_analysis_port #(trans) send;
  trans tr1;
  virtual ram_if rif;
  
  function new(string name = "monitor", uvm_component parent = null);
    super.new(name, parent);
    send = new("send", this);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    tr1 = trans::type_id::create("tr1", this);
    if (!uvm_config_db #(virtual ram_if)::get(this, "", "rif", rif)) 
      `uvm_error("mon", "Unable to access uvm_config_db");
  endfunction
  
  virtual task run_phase(uvm_phase phase);
    forever begin
      
      
      @(posedge rif.clk)
      
    
      tr1.wr_en=rif.wr_en;
      tr1.data_in=rif.data_in;
      
      tr1.addr_in_0=rif.addr_in_0;
      tr1.addr_in_1=rif.addr_in_1;
      tr1.port_en_0=rif.port_en_0;
       tr1.port_en_1=rif.port_en_1;

      tr1.data_out_0=rif.data_out_0;
      tr1.data_out_1=rif.data_out_1;
      
     
      `uvm_info("MON", $sformatf(" %s", tr1.sprint()   ), UVM_NONE)
      send.write(tr1);
    end
  endtask 
endclass




class sb extends uvm_scoreboard;
  `uvm_component_utils(sb)
  
  trans tr2;
  uvm_analysis_imp #(trans, sb) recv;
  
  logic [7:0] exp_mem[0:15];
  bit valid[16];
  function new(string name = "sb", uvm_component parent = null);
    super.new(name, parent);
    recv = new("recv", this);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    tr2 = trans::type_id::create("tr2", this);
  endfunction

  virtual function void write(trans t);
    tr2 = t;
     
    `uvm_info("SCO", $sformatf(" wr enable is %b , data  is %b , address for port 1 %b and for port 2 is %d",tr2.wr_en,tr2.data_in,tr2.addr_in_0,tr2.addr_in_1), UVM_NONE)
    
    if(tr2.port_en_0 && tr2.wr_en)
      begin
      exp_mem[tr2.addr_in_0] = tr2.data_in;
       valid[tr2.addr_in_0] =1;
    end
       
    if(tr2.port_en_0 && valid[tr2.addr_in_0])
      begin
        if(exp_mem[tr2.addr_in_0]==tr2.data_out_0)
          `uvm_info("sb",$sformatf(" test pass  expected data is %b and  output is %b ",exp_mem[tr2.addr_in_0],tr2.data_out_0),UVM_NONE)
         else
           `uvm_error("sb",$sformatf(" test failed expected data is %b and  output is %b ",exp_mem[tr2.addr_in_0],tr2.data_out_0))
        end  
           
    
     
        if(tr2.port_en_1 && valid[tr2.addr_in_1])
        begin
          if(exp_mem[tr2.addr_in_1]== tr2.data_out_1)
            `uvm_info("sb",$sformatf("test pass expected data is %b and  output is %b ",exp_mem[tr2.addr_in_1],tr2.data_out_1),UVM_NONE)
         else
           `uvm_error("sb",$sformatf(" test failed expected data is %b and  output is %b ",exp_mem[tr2.addr_in_1],tr2.data_out_1))
         end  
  
    
  endfunction
endclass: sb
      
      
      


class agent extends uvm_agent;
  `uvm_component_utils(agent)
 
  monitor m;
  driver d;
  uvm_sequencer #(trans) seqr;

  function new(string name = "agent", uvm_component parent = null);
    super.new(name, parent);
  endfunction
 
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    m = monitor::type_id::create("m",this);
    d = driver::type_id::create("d",this);
    seqr = uvm_sequencer #(trans)::type_id::create("seqr", this);
  endfunction
 
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    
    d.seq_item_port.connect(seqr.seq_item_export);
    
  endfunction
endclass: agent

    


    
class env extends uvm_env;
  `uvm_component_utils(env)
 
  sb s;
  agent a;
 
  function new(string name = "env", uvm_component parent = null);
    super.new(name, parent);
  endfunction
 
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    s = sb::type_id::create("s", this);
    a = agent::type_id::create("a", this);
  endfunction
 
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    a.m.send.connect(s.recv);
  endfunction
endclass
    
    


    

class test extends uvm_test;
  `uvm_component_utils(test)
 
  gnr gen;
  env e;
 
  function new(string name = "test", uvm_component parent = null);
    super.new(name, parent);
  endfunction
 
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase); 
    gen = gnr::type_id::create("gen");
    e = env::type_id::create("e", this);
  endfunction
 
  virtual task run_phase(uvm_phase phase);
    phase.raise_objection(this);
    gen.start(e.a.seqr);
    #1000;
    phase.drop_objection(this);
  endtask
endclass



    
    
module RAM();
  
  ram_if rif();
 
dual_port_ram dut(.clk(rif.clk),.wr_en(rif.wr_en),.data_in(rif.data_in),.addr_in_0(rif.addr_in_0),.addr_in_1(rif.addr_in_1),.port_en_0(rif.port_en_0),.port_en_1(rif.port_en_1),.data_out_0(rif.data_out_0),.data_out_1(rif.data_out_1));
  
  
  initial begin
    $dumpfile("dump.vcd");
    $dumpvars;
  end
  
  initial 
    rif.clk=0;
    
    always #5 rif.clk=~rif.clk;

  initial begin
    uvm_config_db #(virtual ram_if)::set(null, "uvm_test_top.e.a*", "rif", rif);
  
    run_test("test");
  end
endmodule

```
---



