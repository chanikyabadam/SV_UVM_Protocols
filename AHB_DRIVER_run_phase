//Writing a AHB Driver run_phase with INCR burst.
//Doesn't support WRAP and custom burst length. 

task run_phase(uvm_phase phase);
  transaction t;
  int burst_len;

  forever begin
    seq_item_port.get_next_item(t);

    if(!t.rstn) begin
      seq_item_port.item_done();
      continue;
    end

    case (t.hburst)
      3'b000: burst_len = 1;   // SINGLE
      3'b001: burst_len = 4;   // INCR4
      3'b010: burst_len = 8;   // INCR8
      3'b011: burst_len = 16;  // INCR16
      default: burst_len = 1;  // default single
    endcase

    for(int a = 0; a <= burst_len; a++) begin
      // Address phase
      @(posedge clk);
      if(a < burst_len) begin
        i.HADDR  <= t.haddr + (a * (1 << t.hsize));
        i.HSIZE  <= t.hsize;
        i.HWRITE <= t.hwrite;
        i.HBURST <= t.hburst;
        i.HTRANS <= (a == 0) ? 2'b10 : 2'b11;
      end
      else begin
        i.HTRANS <= 2'b00;
      end

      // Data phase
      if(a > 0) begin
        if(t.hwrite) begin
          i.HWDATA <= t.hwdata[a-1];
          wait(i.hready);
          t.hresp = i.hresp;
        end
        else begin
          wait(i.hready);
          t.hrdata[a-1] = i.hrdata;
          t.hresp = i.hresp;
        end
      end
    end

    @(posedge clk);
    i.HTRANS <= 0;
    i.HWDATA <= '0;
    seq_item_port.item_done();
  end
endtask
