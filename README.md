Download Link: https://assignmentchef.com/product/solved-cmu15441-project-2-tcp-in-the-wild
<br>



<h1>1        Overview</h1>

In class, we have been talking about TCP, the default transport protocol on the Internet. TCP serves many purposes: it provides reliable, in-order delivery of bytes, it makes sure the sender does not send too fast and overwhelm the receiver (flow control), and it makes sure the sender does not send too fast and overwhelm the network (congestion control). It also aims to be fair: when multiple senders share the same link, they should receive roughly the same proportion of bandwidth.

In class, we discussed TCP Reno. A variant of TCP Reno, NewReno used to be the standard TCP on the Internet. But these are just one of many congestion control algorithms (CCAs). Companies use different TCPs depending on the context, for example, one TCP for data centers and another for serving web content. If you are curious learn more about TCPs in the real world, here is are some fun CCAs to read about: Microsoft and Google use very different approaches to TCP in their datacenters. DCTCP is a TCP for datacenters designed by Microsoft. TIMELY is a TCP for datacenters designed by Google. For web content, the most common algorithm – and the default in Linux servers – is called Cubic. Akamai, the largest content distribution network in the world, uses a proprietary TCP called FastTCP.

In this project, you will demonstrate your understanding of the TCP basics by implementing TCP Reno. You will then use your own engineering skills to design a modern TCP for the datacenter, the Internet, or Earth to Moon communication.

<h2>1.1        Checkpoints and Deadlines</h2>

The timeline for the project is below, including two checkpoints. To help you pace your work, remember that checkpoints represent a date by which you should easily have completed the required functionality. Given the timeline, you can see that this means you should get started now! The late policy is explained on the course website.

<table width="408">

 <tbody>

  <tr>

   <td width="86"><strong>Date</strong></td>

   <td width="322"><strong>Description</strong></td>

  </tr>

  <tr>

   <td width="86">February 19</td>

   <td width="322">Project released.</td>

  </tr>

  <tr>

   <td width="86">March 4</td>

   <td width="322"><strong>Checkpoint 1</strong>: Data Serialization and transmission</td>

  </tr>

  <tr>

   <td width="86">March 22</td>

   <td width="322"><strong>Checkpoint 2</strong>: TCP Reno implementation</td>

  </tr>

  <tr>

   <td width="86">March 29</td>

   <td width="322"><strong>Final Project Deadline by 11:59 P.M.</strong></td>

  </tr>

 </tbody>

</table>

<h1>2        TCP Re-Hash</h1>

TCP is a network layer protocol that enables different devices to communicate. There are a variety of different TCP protocols such as Reno, New Reno, Cubic, and more. For this project we are focusing on Reno. As a reminder the basic setup of Reno is the following.

You have an initiator (client) and a listener (server). The listener is waiting to receive a connection from the initiator. After a connection is received, they perform a TCP handshake to initate the connection. Afterwards, they can perform read and writes to each other. From the application layer, reads and writes to the socket are buffered before being sent over the network. This means that multiple reads or writes might be combined into a single packet or the opposite, that a single read or write to a socket might be split into many packets.

In order to establish reliable data transfer, TCP has to manage lots of different variables and data structures. Here is an example of some of the details you’ll need to track.

Let’s say we have sockets A and B, and that socket A wants to start sending data to socket B. A will store the data in a buffer that it will pull from for sending packets. Socket A will then create packets using data from the buffer and send as many as it is allowed to send based on the congestion control algorithm used. As Socket B receives packets, it stores the data transmitted in a buffer. This buffer helps with maintaining the principle of in-order data transmission. B sends ACKs as responses to A to notify A that various bytes have been received up to a certain point. B is also tracking the next byte requested by the application reading from the socket, and when it receives this byte, will forward as much data in order that it can to the applications buffer. For example, if B is looking for byte number 400, it will not write any other bytes into the application’s buffer until it receives byte number 400. After receiving byte number 400, it will write in as many other bytes as it can. (If B had bytes 400-1000 then all would be written to the application buffer at the time of receiving the packet with byte 400. As packets are ACK’d, socket A will release memory used for storing data as they no longer need to hold onto it. Finally either side can initiate closing the connection where the close handshake begins.

As both sides (initiator and listener) can both send and receive, you’ll be tracking a lot of data and information. It’s important to write down everything each side knows while writing your implementation and to utilize interfaces to keep your code module and re-usable.

<h1>3        Project specification</h1>

<h2>3.1        Background</h2>

This project will consist of three checkpoints. The first checkpoint will have you build the basics of TCP: reliable, in-order delivery and flow control. In the second checkpoint, you will implement the Reno CCA and demonstrate to us that it is correct. In the third and final checkpoint, you will design and implement your own CCA for one of three scenarios: a datacenter, the Internet, or Earth to Moon communication.

<h2>3.2        What are you actually turning in</h2>

You are implementing the cmu tcp.h interface. Your code will be tested by us creating other C files that will utilize your interface to perform communications. The starter code has an example of how we might perform the tests, we have a client.c and server.c which utilize the sockets to send information back and forth. You can add additional helper functions to cmu tcp or change the implementation of the 4 core functions (socket, close, read and write), however you cannot change the function signature of the 4 core functions. Further, we will be utilizing grading.h to help us test your code. We may change any of the values for the variables present in the file to make sure you aren’t hard coding anything. Namely, we will be fluctuating the packet length, and the initial window variables.

Additionally, for each checkpoint you will need to provide a graph showing the number of packets in flight (or unacked packets) vs time. We have provided you with a samplefile in the test directory that you can transfer and graph. We have also provided a python file called gen graph.py to help you generate the graph. It should be setup to monitor packets sent to and from the sender’s perspective – you may need to change and update the gen graph.py script in order to provide a quality graph.

<h1>4        Checkpoint 1</h1>

In this checkpoint, you will implement windowed transmission, where more than one packet can be “on the wire” at the same time. You will start by extending a simple “stop and wait” implementation that we provide.

<h2>4.1        Learning Objectives</h2>

With this section, you will learn to think about data packetization and reconstruction, TCP windowing, and packet retransmission. You will also continue to strengthen your C programming skills.

<h2>4.2        Starter Code</h2>

The following files have been provided for you to use:

<ul>

 <li>cmu packet.h: this file describes the basic packet format and header. You are not allowed to modify this file until the final submission! The scripts that we provide to help you graph your packet traces rely on this file being unchanged.</li>

 <li>h: these are variables that we will use to test your implementation, please do not make any changes here as we will be replacing it when running tests.</li>

 <li>c: this is the starter code for the server side of your transport protocol.</li>

 <li>c: this is the starter code for the client side of your transport protocol.</li>

 <li>cmu tcp.c: this contains the main socket functions required of your TCP socket including reading, writing, opening and closing.</li>

 <li>c: this file contains the code used to emulate the buffering and sending of packets. This is where you should spend most of your time. gen graph.py: Python script that takes in a pcap file and graphs your sequence numbers by time.</li>

 <li>cmu packet.h All the communication between your server and client will use UDP as the underlying protocol. All packets will begin with the common header described in cmu packet.h as follows:

  <ul>

   <li>Course Number [4 bytes]</li>

   <li>Source Port [2 bytes]</li>

   <li>Destination Port [2 bytes]</li>

   <li>Sequence Number [4 bytes]</li>

   <li>Acknowledgement Number [4 bytes]</li>

   <li>Header Length [2 bytes]</li>

   <li>Packet Length [2 bytes]</li>

   <li>Flags [1 byte]</li>

   <li>Advertised Window [2 bytes]</li>

   <li>Extension length [2 bytes]</li>

   <li>Extension Data [You Decide]</li>

  </ul></li>

</ul>

All multi-byte integer fields must be transmitted in network byte order. ntoh, hton, and friends will be very important functions for you to call! All integers must be unsigned, and the course number should be set to 15441 (the scripts rely on this). You are not allowed to change any of the fields in the header, with the exception of the extension data which you may want to modify in Checkpoint 3. Additionally, plen cannot exceed 1400 in order to prevent packets from being broken into parts.

You can verify that your headers are sent correctly using wireshark or tcpdump. You can view packet data sent including the full Ethernet frames. When viewing your packet you should see something similar to the below image; in this case the payload starts at 0x0020. The course number – 15441- shows up in hex as 0x00003C51.

<h2>4.3        Checkpoint 1 Tasks</h2>

<ol>

 <li>TCP Handshakes – Implement TCP start and end handshakes before data transmission starts and ends [1]. This should happen in the constructor and destructor for cmu socket.</li>

 <li>Flow Control – You will notice that data transfer is very slow. That is because the starter code is using the Stop-and-Wait algorithm, transmitting one packet at a time! You can do much better by using a window of outstanding packets to send on the network. Extend the implementation to: 1) Change the sequence numbers and ACK numbers to represent the number of bytes sent and received (rather than segments) 2) Implement TCP’s sliding window algorithm to send a window of packets and utilize the advertised window to limit the amount of data sent by the sender [2]. You do not need to implement Nagle’s algortihm.</li>

 <li>RTT Estimation – You will notice that loss recovery is very slow! One reason for this is the starter code uses a fixed retransmission timeout (RTO) of 3 seconds. Implement an adaptive RTO by estimating the RTT with Jacobson/Karels Algorithm or using the Karns/Partridge algorithm [3].</li>

 <li>Duplicate ACK Retransmission – Another reason loss recovery is slow is the starter code relies on timeouts to detect packet loss. One way to recover more quickly is to retransmit whenever you see triple duplicate ACKs. Implement retransmission on the receipt of 3 duplicate ACKs.</li>

</ol>

<h1>5        Checkpoint 2</h1>

Once you have implemented the basics, you can add a Congestion Control Algorithm (CCA) to handle overloading in the network. You will implement TCP Reno, as discussed in class. Hence, the number of outstanding (unACKed) packets will now be be min(window size, congestion window size). You will have to demonstrate to us using traces of real connections that your TCP Reno implementation uses Additive Increase under normal operation, and Multiplicative Decrease under loss.

<h2>5.1        Learning Objectives</h2>

With this section, you will learn how TCP manages to maintain a view of the network state, as well as how the TCP state machine for congestion control works (shown below). Implementing TCP Reno will allow you to transfer data faster as you utilize more available bandwidth for data transmission. Additionally, fast recovery will allow you efficiently recover when packets have been lost.

<h2>5.2        Checkpoint 2 Tasks</h2>

<ol>

 <li>Basics – Review and understand the TCP congestion control state machine in depth! [12]</li>

 <li>Update the your code from Checkpoint 1 to add a new parameter, the congestion window: cwnd. The size of your sending window should now be the minimum of cwnd and the advertised window. You should additionally maintain that the total amount of data buffered for the application (unread data, both ordered and unordered bytes) should be less than MAX NETWORK BUFFER.</li>

 <li>Congestion Control – Implement the slow start and congestion avoidance features of the state machine, this should make your implementation like TCP Tahoe.</li>

 <li>Fast Recovery – Implement fast recovery, this should move your implementation from Tahoe to Reno. Note, you only need to implement TCP Reno, not TCP NewReno.</li>

 <li>Make sure to test your new features with many different network settings using tcconfig. You should transmit a large file again using your TCP implementation, like you did in Checkpoint 1, set the bandwidth small in relation to the size of your file (ex: transferring 100Mb file, 1Mbps bandwidth) and add packet loss (ex: 5%) in order to see the TCP sawtooth pattern.</li>

</ol>

Here is a copy of the full state machine

Here are the values from grading.h you must use in your code for this checkpoint. We will test your code (and you should too!) by changing these values. All of these values are in bytes.

<ol>

 <li>WINDOW INITIAL WINDOW SIZE: Initial window size for slow start. In slow start, you should initially set cwnd = WINDOW INITIAL WINDOW SIZE.</li>

 <li>WINDOW INITIAL SSTHRESH: ssthresh value for congestion control. In slow, start, you should initially set ssthresh = WINDOW INITIAL SSTHRESH.</li>

 <li>MAX LEN: Max packet length of any packet – including header. This value will not change and will always remain fixed.</li>

 <li>MAX NETWORK BUFFER: Maximum number of bytes that the TCP implementation can hold/buffer for the application. (This includes unread, ordered and unordered bytes received on the network, and received by the application). Thus, the size of your sending buf should be set to MAX NETWORK BUFFER and the size of received buf should be set to MAX NETWORK BUFFER.</li>

</ol>

<h1>6        Checkpoint 3</h1>

In checkpoint 3, you will improve your TCP implementation for one of three scenarios:

<ol>

 <li>Moon – For the moon, set loss rate to 30%, 2.5 seconds of delay, and a max bandwith of 10 megabits per second.</li>

 <li>Data Center – For the data center, set loss rate to 0.05%, delay 0.1 seconds, and a max bandwith of 5</li>

</ol>

gigabits per second.

<ol start="3">

 <li>AWS Server to Client – For the server, set loss rate to 1%, delay to 0.2 seconds, and a max bandwith of 50 megabits per second.</li>

</ol>

<ul>

 <li>Your first step is to <em>profile </em>your Reno implementation. Using your Reno implementation, transfer a 20MB file in your chosen scenario. How long does it take for the file transfer to complete? Consider Reno’s design: what makes it slower than it could be? If the file transfer does not complete in a reasonable amount of town, why is this?</li>

</ul>

<strong>Deliverable: </strong>In a file called <strong>designdiscussion.pdf</strong>, turn in a graph of your Reno implementation running in your chosen scenario. Write at least two paragraphs describing what aspects of Reno’s design make it perform less-well than it could in your chose scenario.

<ul>

 <li>Your second step is to design a new congestion control algorithm. There is only one requirement: thatyour new implementation finishes at least 10% faster than your original implementation (or, if your first implementation did not complete within 20 minutes, that your new implementation does complete in that amount of time). One way to pass this step is to implement TCP Cubic or TCP BBR, but you may also design your own algorithm.</li>

</ul>

<strong>Deliverable: </strong>In the same file called <strong>designdiscussion.pdf</strong>, provide a graph of your new algorithm running in your chosen scenario. Answer the following questions:

<ol>

 <li>Describe your new algorithm to someone else who needs to implement it too. How does it work? What is the ‘state machine’ to update your new algorithm?</li>

 <li>How long does it take for your new CCA to transfer a 20MB file? How long does it take your Reno implementation?</li>

 <li>What is different about your new CCA that makes it perform better than Reno in tranferring 20MB files?</li>

 <li>How long does it take for your new CCA to transfer a 3MB file? How long does it take your Reno implementation?</li>

 <li>Explain why one algorithm performs better than the other, or why they perform equally.</li>

</ol>

<ul>

 <li>(Optional) In this step you will tune your algorithm for competition! The TAs will test the top-5 fastestalgorithms by running them in a real datacenter (AWS or Google CLoud), over the real wide area (from CMU to AWS or Google Cloud), or to the (simulated, sorry) Moon.</li>

</ul>

We will run two tests. First, we will transfer one 20MB file. We will measure the <em>average goodput </em>(the number of useful bytes transferred over time) from when we start sending to when the file transfer completes. Second, we will transfer four 20MB files simultaneously. We will measure each connection’s average goodput and compute <em>Jain’s Fairness Index. </em>Your total score will be the goodput for the single connection test, multiplied by Jain’s Fairness Index for the four-connection test.

You can run these tests yourself and tune your code. There will be one prize for each of the three categories. We have some credits for Google Cloud and AWS if you would like to test over the real scenarios, please email <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="087b7c696e6e253c3c39486b7b266b657d266d6c7d">[email protected]</a> to request credits.

<h1>7        Testing Your Code</h1>

<h2>7.1        Virtual Machine</h2>

For this project, you will do all of your development on your own machine using VMs. You should install VirutalBox [10] and Vagrant [9] on your own machine. We have provided a Vagrantfile for two VM’s, client and server. If you keep the Vagrantfile in the same directory as the 15-441-project-2, folder then you can edit code on your machine, or on either VM, and Vagrant will automatically sync it to both VMs /vagrant directory. Refer to references for more information on how to install and use VirtualBox and Vagrant.

The server and client are connected via a private network with IP addresses 10.0.0.1 and 10.0.0.2 respectively. (Note: your code should work even if these IP addresses were changed). The interface name for these addresses is eth1 on both machines.

The following sections we describe the tools that are already installed on the VMs to help you (and us) test your code. You can also install any additional tools you need. Please document any additional tools you install to run tests in the test.txt file.

<h2>7.2        Control the network characteristics with tcconfig</h2>

tcconfig [4] is installed on the VMs to enable you to control the network characteristics for traffic between the VMs. The initial default settings on the VMs are a 20ms delay on both machines (so the total RTT is 40ms), and 100Mbps bidirectional bandwidth. Running tcshow eth1 on the VMs will show you these settings. Refer to the references for more information on how to use tcconfig to simulate different network characteristics including packet loss, reordering, and corruption which will be useful for testing your code.

<h2>7.3        Capture and analyze packets with tcpdump and tshark</h2>

tcpdump [7] and Wireshark (terminal program: tshark [5]) are installed on the VMs to enable you to capture packets sent between the VMs and analyze them. We provide the following files in the directory 15-441-project-2/utils/ to help with packet analysis (feel free to modify these if you want):

<ul>

 <li>utils/capture packets.sh: A simple program showing how you can start and stop packet captures, as well as analyze packets using tshark. The start function starts a packet capture in the background.</li>

</ul>

The stop function stops a packet capture. Lastly, the analyze function will use tshark to output a CSV file with header information from your TCP packets.

<ul>

 <li>utils/tcp.lua: A Lua plugin so Wireshark can dissect our custom cmu packet format [6]. capture packets.sh shows how you can pass this file to tshark to parse packets. To use the plugin with the Wireshark GUI on your machine, you add this file to Wireshark’s plugin folder [8].</li>

</ul>

<h2>7.4        Running tests with pytest</h2>

There are many ways you can write tests for your code for this project. To help get you started, pytest [11] is installed on the VMs and we provide example basic tests in test/test cp1.py. Running make test will run these tests automatically. You should expand these tests or use a different tool to test your code (but make test should still run your tests). As in Project 1, you should also use standard C debugging tools including gdb and Valgrind which are also installed on the VMs.

<h2>7.5        Running a large file transfer</h2>

The starter code client.c and server.c will transmit a small file, cmu tcp.c between the client and server. You should also test your code by transmitting a larger file (ex: 100MB file), capturing the packets, and plotting the number of unacked packets vs. time. You will turn in a PDF of this graph and this PCAP file.

You can use the utilities described in 7.3 to create submit.pcap by running the following commands:

Start tcpdump and the server:

<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="1066717762717e6450637562667562">[email protected]</a>:/vagrant/15-441-project-2$ make <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="0c7a6d6b7e6d62784c7f697e7a697e">[email protected]</a>:/vagrant/15-441-project-2$ utils/capture_packets.sh start submit.pcap <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="93e5f2f4e1f2fde7d3e0f6e1e5f6e1">[email protected]</a>:/vagrant/15-441-project-2$ ./server Start the client:

<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="7402151306151a003417181d111a00">[email protected]</a>:/vagrant/15-441-project-2$ ./client

When the client and server code finishes running, stop the packet capture on the server:

<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="6b1d0a0c190a051f2b180e191d0e19">[email protected]</a>:/vagrant/15-441-project-2$ utils/capture_packets.sh stop submit.pcap

<h2>7.6        Autolab</h2>

<strong>There will be no tests in Autolab. Autolab will only be used to submit your code. We will pull everyones code and run our tests in the VMs. We are supplying these VMs to you to aid in your development and so that you can also test your own code.</strong>

Make sure that you cover the basics such as no seg-faults, reliable data transfer, and error handling. Then ensure that your code manages flow control and congestion control according to the given checkpoint.

<ul>

 <li></li>

</ul>