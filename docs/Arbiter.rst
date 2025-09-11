MIG & Cache Interface
=====================

1. I/O Ports
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
     - Reset caches at low
   * - input
     - init_addr_i
     - 32
     - Provide DRAM initial values
   * - input
     - init_data_i
     - 128
     - Initial values
   * - input
     - init_end_i
     - 1
     - Initialization is done

.. list-table:: Dcache Ports
   :header-rows: 1
   :widths: 10 20 10 60

   * - I/O
     - name
     - width
     - purpose
   * - output
     - d_rd_data_o
     - 256
     - Data Dcache needs
   * - output
     - d_rd_end_o
     - 1
     - Read data process is done
   * - input
     - d_wr_data_i
     - 256
     - Data to store into DRAM
   * - input
     - d_req_wr_i
     - 1
     - Dcache requests to write to DRAM
   * - input
     - d_req_rd_i
     - 1
     - Dcache requests to read from DRAM
   * - input
     - d_addr_i
     - 32
     - Address for DRAM write/read
   * - output
     - d_mem_rdy_o
     - 1
     - Dcache can start to read/write DRAM

.. list-table:: Icache Ports
   :header-rows: 1
   :widths: 10 20 10 60

   * - I/O
     - name
     - width
     - purpose
   * - output
     - i_rd_data_o
     - 256
     - Data Icache needs
   * - output
     - i_rd_end_o
     - 1
     - Read data process is done
   * - input
     - i_req_rd_i
     - 1
     - Icache requests to read from DRAM
   * - input
     - i_addr_i
     - 32
     - Address for DRAM write/read
   * - output
     - i_mem_rdy_o
     - 1
     - Icache can start to read/write DRAM

.. list-table:: MIG Ports
   :header-rows: 1
   :widths: 10 20 10 60

   * - I/O
     - name
     - width
     - purpose
   * - output
     - app_addr
     - 27
     - DRAM address (128MB DRAM → 27 bits)
   * - output
     - app_cmd
     - 1
     - Decide read or write
   * - output
     - app_en
     - 1
     - Command enable
   * - input
     - app_rdy
     - 32
     - MIG is ready to accept new command
   * - input
     - init_mem_rdy
     - 1
     - High after DRAM starts working
   * - output
     - app_wdf_data
     - 128
     - Data to write into DRAM
   * - output
     - app_wdf_end
     - 1
     - High for 1 cycle at end of data
   * - output
     - app_wdf_wren
     - 1
     - Write command valid
   * - input
     - app_wdf_rdy
     - 1
     - DRAM can accept write
   * - output
     - app_wdf_mask
     - 1
     - Which bytes to write (default: all zeros)
   * - input
     - app_rd_data
     - 128
     - Data from DRAM
   * - input
     - app_rd_data_end
     - 1
     - High when data read back from DRAM
   * - input
     - app_rd_data_valid
     - 1
     - Handshake indicating data from DRAM is valid

2. Description
--------------

This interface resolves competition between I-cache and D-cache and also serves as a data channel when the CPU is first activated.

