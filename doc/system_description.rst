System Description
==================

This section briefly describes the software components used, thread diagram and resource usage details.

Software Architecture
---------------------

.. figure:: images/s2e_threads.png
    
    Core Usage
    
In order to achieve the desired data bridging, this application essentially maps each of the configured UART to a telnet socket and maintains application buffers (FIFOs) for each of such mapping. Whenever there is a UART data available, the application fills the appropriate UART buffer and notifies TCP client thread to consume this data. Similarly whenever there is data from Ethernet packets, XTCP server notfies TCP client about data availability and the client stores this data into respective application buffers. This data is then consumed by UART (client) handler.

Cores
~~~~~

The design uses seven logical cores as described below.

The multi-UART component comprises two logical cores, one acting as a transmit (TX) server for up to 8 uarts, and the other acting as a receive (RX) server for up to 8 uarts.

UART_Handler is an application core that interfaces with the UART RX and TX servers. It handles UART configuration requests, facilitates to store the UART data received from UART RX server into respective application buffers and transfers the data received from TCP clients to UART TX server.

TCP_Handler is another application core that initializes and manages the application. It interfaces with the flash module for UART configuration storage and recovery, and handles all the xtcp application events (application data and UI configuration requests) received from the xtcp server module. UDP discovery management, web server handling, telnet data extraction are all implemented in this logical core.

The XTCP server runs on a single logical core and connects to the Ethernet MAC component which uses a single logical core. It uses XC channel to communicate to clients (TCP_Handler in this case) using XTCP events. 

The Flash core handles flash data sotrage and retrieval requests from TCP_Handler core based on the application dynamics such as start-up or UI driven requests.

Buffering
~~~~~~~~~

Buffering for the TX server is managed by the UART_Handler Core. Data is transferred to the UART TX logical core via a shared memory interface.

There is no buffering provided by the UART RX server. The UART_Handler core is able to respond to the received data in real time and store them in buffer(s) available in the  TCP_Handler core via token notifcations.

Communication Model
~~~~~~~~~~~~~~~~~~~

The ``sc_multi_uart`` module utilises a combination of shared memory and channel communication. Channel communication is used on both the RX and TX servers to pause the logical core and subsequently release the logical core when required for reconfiguration. The primary means of data transfer for both the RX and TX logical cores is shared memory. The RX logical core utilises a channel to notify any client (UART_Handler in this case) of available data - this means that events can be utilised within an application to avoid the requirement for polling for received data.

XTCP module and flash core connects to TCP_handler client using their repective XC channels.


Software components used
------------------------

.. list-table::
 :header-rows: 1

 * - Component
   - Description
 * - sc_ethernet
   - Two thread version of the ethernet component implementing 10/100 Mii ethernet mac and filters
 * - sc_xtcp
   - Micro TCP/IP stack for use with sc_ethernet component
 * - sc_multi_uart
   - Component for implementing multiple serial device communication
 * - sc_util
   - General utility modules for developing for XMOS devices
 * - sc_website
   - Component framework for Embedded web site development
 * - xcommon
   - Common application framework for XMOS software

Resource Usage
++++++++++++++

.. figure:: images/resource_usage.jpg
    
    Resource Usage
    
