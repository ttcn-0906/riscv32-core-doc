Dcache
======

1. I/O port
-----------

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
     - 18
     - A number to recognize whether the data we want is on the specific address or not
   * - input
     - idx_i
     - 9
     - To address which cache line we want
   * - input
     - word_ofs_i
     - 3
     - Operating a specific word in a specific data block
   * - input
     - mask_i
     - 4
     - A word is 4 bytes in 32-bit CPU and mask_i is a filter to choose which bytes need to be operated
   * - input
     - cpu_req_wr
     - 1
     - CPU wants to write data to cache
   * - input
     - cpu_req_rd
     - 1
     - CPU wants to read data from cache
   * - output
     - cpu_data_o
     - 32
     - Turn specific data back to CPU
   * - output
     - dcache_rdy_o
     - 1
     - A one way handshake telling CPU if this cache is available or not
   * - input
     - invalidate_i
     - 1
     - Invalidate specific cacheline
   * - input
     - flush_i
     - 1
     - Flush specific cacheline
   * - input
     - writeback_i
     - 1
     - Write all of the data back to DRAM

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
   * - output
     - wr_mem_end_o
     - 1
     - Get high when the data is last one
   * - input
     - rd_mem_end_i
     - 1
     - Tell cache reading process is done
   * - output
     - req_wr_mem
     - 1
     - Cache requests to write data to memory
   * - output
     - req_rd_mem
     - 1
     - Cache requests to read data from memory
   * - output
     - mem_addr_o
     - 32
     - Tell DRAM which address cache wants to write or read
   * - output
     - mem_data_o
     - 256
     - Data for DRAM

2. Description
--------------

This is a 32kB 2-way cache. Its data storage is constructed by BRAM IP, and each block size is 32 bits. It uses LRU as the replacement policy when a data miss occurs.
