﻿<table class="sphinxhide" width="100%">
 <tr width="100%">
    <td align="center"><img src="https://raw.githubusercontent.com/Xilinx/Image-Collateral/main/xilinx-logo.png" width="30%"/><h1>AI Engine Development</h1>
    <a href="https://www.xilinx.com/products/design-tools/vitis.html">See Vitis™ Development Environment on xilinx.com</br></a>
    <a href="https://www.xilinx.com/products/design-tools/vitis/vitis-ai.html">See Vitis™ AI Development Environment on xilinx.com</a>
    </td>
 </tr>
</table>

# Packet Stream Based AI Engine Kernels

Packet Stream-based AI Engine kernels allow fine-grain control over how packets are generated and consumed in the kernels. This section explains how to code AI Engine kernels with packet stream interfaces (`input_pktstream` and `output_pktstream`). The connection in the graph is also described. 

The PL side and PS side of this example is the same as [Window Based AI Engine Kernels](./window_based_aie_kernel.md). Refer to:
* [Packet Format](./window_based_aie_kernel.md/#Packet-Format)
* [Example PL Kernels for Packet Switching](./window_based_aie_kernel.md/#Example-PL-Kernels-for-Packet-Switching)
* [Example PS code for Packet Switching](./window_based_aie_kernel.md/#Example-PS-code-for-Packet-Switching)

### Packet Stream Interfaces and Operations
Two stream types are provided to denote streaming data, consisting of packetized interleaving of several different streams. These types are listed in the following table.

Input Stream Types | Output Stream Types
------------ | -------------
input_pktstream | output_pktstream

A data packet consists of a one word (32-bit) packet header, followed by some number of data words where the last data word has the TLAST field denoting the end-of-packet. The following operations are used to read and advance input packet streams and write and advance output packet streams.

    int32 readincr(input_pktstream *w);
    int32 readincr(input_pktstream *w, bool &tlast);
    void writeincr(output_pktstream *w, int32 value);
    void writeincr(output_pktstream *w, int32 value, bool tlast);

The API with the `TLAST` argument reads or writes the end-of-packet condition if the packet size is not fixed.

The AI Engine tools provide the built-in function `writeHeader` to generate a packet header for packets originating from the AI Engine kernel and writes them to the output. 

    void writeHeader(output_pktstream *str, unsigned int pcktType, unsigned int ID);
    void writeHeader(output_pktstream *str, unsigned int pcktType, unsigned int ID, bool tlast);

The AI Engine tools also provide the built-in function `getPacketid` to get the packet ID for the packet stream interface. The index for `getPacketid` only applies if the packet stream feeds into a `pktsplit`. In this example, each AI Engine kernel output sees only one logical stream (0 for index).

    uint32_t getPacketid(input_pktstream * in, int index);
    uint32_t getPacketid(output_pktstream * out, int index);   
    
Change working directory to `pktstream_aie`. Review the AI Engine kernels (`aie/aie_core1.cpp`, ... , `aie/aie_core4.cpp`). The code for `aie_core1` (`aie/aie_core1.cpp`) is as follows.

	void aie_core1(input_pktstream *in,output_pktstream *out){
		readincr(in);//read header and discard
		uint32 ID=getPacketid(out,0);//for output pktstream
		writeHeader(out,pktType,ID); //Generate header for output
	
		bool tlast;
		for(int i=0;i<8;i++){
			int32 tmp=readincr(in,tlast);
			tmp+=1;
			writeincr(out,tmp,i==7);//TLAST=1 for last word
		}
	}

It can be seen that the input packet header is discarded. The output header is generated by `writeHeader`, and the packet ID for the header is obtained by `getPacketid`. `TLAST` equals 1 for the last word in the packet.

### Construct Graph for Packet Stream Kernels
Review how the graph is constructed in `aie/graph.h`. 

	using namespace adf;
	class mygraph: public adf::graph {
	private:
	  adf:: kernel core[4];
	
	  adf:: pktsplit<4> sp;
	  adf:: pktmerge<4> mg;
	public:
	  adf::input_plio  in;
	  adf::output_plio  out;
	  mygraph() {
	    core[0] = adf::kernel::create(aie_core1);
	    core[1] = adf::kernel::create(aie_core2);
	    core[2] = adf::kernel::create(aie_core3);
	    core[3] = adf::kernel::create(aie_core4);
	    adf::source(core[0]) = "aie_core1.cpp";
	    adf::source(core[1]) = "aie_core2.cpp";
	    adf::source(core[2]) = "aie_core3.cpp";
	    adf::source(core[3]) = "aie_core4.cpp";
	
		in=input_plio::create("Datain0", plio_32_bits,  "data/input.txt");
		out=output_plio::create("Dataout0", plio_32_bits,  "data/output.txt");
	
	    sp = adf::pktsplit<4>::create();
	    mg = adf::pktmerge<4>::create();
	    for(int i=0;i<4;i++){
	    	adf::runtime<ratio>(core[i]) = 0.9;
	    	adf::connect<adf::pktstream > (sp.out[i], core[i].in[0]);
	        adf::connect<adf::pktstream > (core[i].out[0], mg.in[i]);
	    }
	
	    adf::connect<adf::pktstream> (in.out[0], sp.in[0]);
	    adf::connect<adf::pktstream> (mg.out[0], out.in[0]);
	  }
	};

Note that the connection type for the `input_pktstream` and `output_pktstream` interfaces are `adf::pktstream`. So, it uses `adf::connect<adf::pktstream>` to connect the AI Engine kernel and `pktsplit.out` / `pktmerge.in`. 

Note that `input_pktstream` is read as integer input. It needs to be `reinterpret_cast` to other types if needed. 

### Run the AI Engine Simulator, HW Emulation, and HW Flows

Run the AI Engine simulator with the following make command.

    make aiesim
    
Run HW emulation with the following make command (it will build the HW system and host application) :

    make run_hw_emu
    
Hint: If the keyboard is accidentally hit and stops the system booting automatically, type boot at the **Versal>** prompt to resume the system booting.

After Linux has booted, run the following commands at the Linux prompt (this is only for HW cosim).

    mount /dev/mmcblk0p1 /mnt
    cd /mnt
    export XILINX_XRT=/usr
    export XCL_EMULATION_MODE=hw_emu
    ./host.exe a.xclbin
    
To exit QEMU press Ctrl+A, x

To run in hardware, first build the system and application using the following make command.

    make package TARGET=hw
    
After Linux has booted, run the following commands at the Linux prompt.

    mount /dev/mmcblk0p1 /mnt
    cd /mnt
    export XILINX_XRT=/usr
    ./host.exe a.xclbin
    
The host code is self-checking. It checks the correctness of output data. If the output data is correct, after the run has completed, it will print:

    TEST PASSED

### Conclusion
In this tutorial you learned about:

* Building the window interface or packet stream interface to AI Engine kernels
* Constructing the packet switching graph
* Writing PL kernels to perform packet switching

# Support

GitHub issues will be used for tracking requests and bugs. For questions go to [forums.xilinx.com](http://forums.xilinx.com/).

# License

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License.

You may obtain a copy of the License at [http://www.apache.org/licenses/LICENSE-2.0]( http://www.apache.org/licenses/LICENSE-2.0 )


Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

<p align="center"><sup>XD029 | &copy; Copyright 2020-2022 Xilinx, Inc.</sup></p>
