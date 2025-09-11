Icache
======

1. I/O ports
------------

.. list-table:: System Ports
   :header-rows: 1
   :widths: 10 20 10 60

   * - I/O
     - name
     - width
     - purpose
   * - input
     - clk
     - 1
     - Timing
   * - input
     - rst_n
     - 1
     - Reset dcache at low
   * - input
     - mem_init_complete_i
     - 1
     - Inform cache if DRAM is working

.. list-table:: CPU Ports
   :header-rows: 1
   :widths: 10 20 10 60

   * - I/O
     - name
     - width
     - purpose
   * - input
     - tag_i
     - 19
     - A number to recognize whether the data we want is on the specific address or not
   * - input
     - idx_i
     - 8
     - To address which cache line we want
   * - input
     - ofs_i
     - 5
     - Operating a specific word in a specific data block (The last two bits must be zero)
   * - input
     - invalidate_i
     - 1
     - Invalidate specific cacheline
   * - output
     - icache_rdy_o
     - 1
     - Tell CPU if I-cache is available
   * - output
     - cpu_inst_o
     - 32
     - Output particular instruction

.. list-table:: Arbiter Ports
   :header-rows: 1
   :widths: 10 20 10 60

   * - I/O
     - name
     - width
     - purpose
   * - input
     - mem_rdy_i
     - 1
     - A handshake signal to inform cache if it can send new instruction to arbiter
   * - input
     - mem_data_i
     - 256
     - Data from arbiter (DRAM)
   * - input
     - rd_mem_end_i
     - 1
     - Tell cache reading process is done
   * - output
     - req_rd_mem_o
     - 1
     - Cache requests to read data from memory
   * - output
     - mem_addr_o
     - 32
     - Tell DRAM which address cache wants to write or read

2. Description
--------------

This is a 16kB 2-way cache. Its data storage is constructed by BRAM IP, and each block size is 32 bits. It uses FIFO as the replacement policy when a data miss occurs.
