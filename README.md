Download Link: https://assignmentchef.com/product/solved-cpsc3600-assignment-2-a-reliable-data-transfer-protocol-v2
<br>
In this project you will be implementing a simple reliable data transfer (RDT) protocol. This is <strong>NOT </strong>a real protocol used on the internet, but it does illustrate some ideas that show up in real protocols (like TCP).

All network communication in this project will be simulated, so as to allow us complete control over when packets are corrupted, lost, or misordered. This is important for testing and proving that your solution works, but isn’t something we can do with real network traffic.

<h1>1        Assignment Instructions</h1>

You will be implementing a full-duplex GBN host. Full-duplex means that the program is capable of both sending and receiving data. The autograder will create two instances of your GBN host and have them talk

with each other through a simulated network.

The project template includes three files. You will only be editing one of these files; the others will be used to test your projects.

<ol>

 <li><strong>network simulator.py</strong>: <strong>You do not need to implement any code in <em>network simulator.py</em></strong>. You’ll be using functions defined in it to manage sending and receiving through the simulated network, as well as starting and stopping timers.

  <ul>

   <li><strong><em>EventEntity enumeration </em></strong>network simulator.py defines an enumeration used to identify which of the two hosts are sending or receiving a given packet. The entity assigned to a given instance of gbn host is passed in to the constructor and is stored in self.entity.</li>

   <li><strong><em>pass to network layer(entity, packet, is ACK) </em></strong>Passes packed messages to the simulated network layer, where it is communicated to the host on the other end of the simulated socket. Your code should call this once it has packed a message intended for the client on the other side of the simulated socket. The three expected parameters are 1) the entity that is sending the message (stored in self.entity), 2) the packed bytes of the packet, and 3) a boolean indicating if this packet is an ACK messge, or a data message. This final parameter is used by the network simulator to track the flow of information through the network.</li>

   <li><strong><em>pass to application layer(entity, data) </em></strong>Passes decoded data up to the application layer. Your code should call this once it has received a packet containing data. The two expected parameters are 1) the entity that is passing a packet’s payload to an application, and 2) the unpacked data to be delivered to an application.</li>

   <li><strong><em>start timer(entity) </em></strong>Starts a timer event for a given host. A single parameter is expected, the entity for which a timer should be started.</li>

   <li><strong><em>stop timer(entity) </em></strong>Stops any current timer events associated with a given host. A single parameter is expected, the entity for which a timer should be stopped.</li>

  </ul></li>

 <li><strong>gbn host.py</strong>:

  <ul>

   <li><strong><em>receive from application layer(payload) </em></strong>The autograder will call this function when a given entity receives a piece of data from a simulated application to transmit across the simulated network. The behavior of this function is specified in the Sender FSM shown in Figure 2.</li>

   <li><strong><em>receive from network layer(bytes) </em></strong>The autograder will call this function when a packet addressed to a given entity is received from the simulated network. The behavior of this function is specified in the Sender and Receiver FSMs shown in Figures 2 and 3. Note that this function needs to handle behavior that occurs in both the sender and the receiver FSM. When you receive data from the network layer, you’ll need to determine which functionality is appropriate.</li>

   <li><strong><em>timer </em></strong><strong><em>interrupt() </em></strong>The autograder will call this function when a timer set by your program expires.</li>

   <li><strong><em>is corrupt(bytes) </em></strong>should determine whether the received data is corrupted, based on the checksum that is included with the packet. This will be used by your code, and will also be tested by the autograder.</li>

  </ul></li>

 <li><strong>gbn tester.py</strong>: This is the file you should run to test your code on your local machine. You can control which test cases are run by uncommenting the different test files listed in the <em>tests </em>list in the main method. Only the first test is uncommented in the file you will download with your template. Once you have this one working, begin uncommenting the other tests.</li>

</ol>

You do not have to use this file at all, but it will be helpful for debugging. Alternatively, you can just submit your files to Gradescope once you are ready to test them and base your development on the output log shown there.

<h1>2         Reading a Finite State Machine</h1>

We will use finite state machines to describe the logical functionality of this RDT protocol. Finite state machines show how a system changes state in response to specific events. An example FSM is shown below in Figure 1. This particular example illustrates the behavior of a simple vending machine.

Figure 1: This finite state machine illustrates the behavior of a simple vending machine.

Circles represent different states that the system can occupy. Transitions between states are show using solid arrows. Each transition is annotated with information about what event caused the transition and what is done in response to it. Events are indicated above the line in the annotation, and responses are indicated below the line in the annotation. A symbol Λ below the line indicates that nothing happens in response to this event. Transitions can go back to the same state; this indicates that an event has occurred but that logical state of the system has not changed. A dashed arrow pointing at a state indicates that the program starts in that state. Initial values of variables can be indicated via a transition annotation linked to this dashed line.

<ul>

 <li><strong>Sender Functionality</strong></li>

</ul>

The sender’s functionality is specified in the FSM shown in Figure 2.

<ul>

 <li><strong>Receiver Functionality</strong></li>

</ul>

The receiver’s functionality is specified in the FSM shown in Figure 3.

<h1>5       Packet Format</h1>

Your packets will contain the following header fields:

<ol>

 <li>Packet Type (unsigned half) – a bitwise flag indicating if this is a data packet or an acknowledgement packet. We’ll use the eighth bit as the flag, meaning that you should use the number 128 for data packets and 0 for ack packets.</li>

</ol>

We’re using a half for this to make the checksum process simpler (this way the checksum will be aligned at the byte boundaries).

Figure 2: GBN Sender: All code shown here is pseudocode intended to describe the logical functionality of this RDT protocol. Your code will be similar to this, however it’s implementation may differ at certain points.

Figure 3: GBN Receiver: All code shown here is pseudocode intended to describe the logical functionality of this RDT protocol. Your code will be similar to this, however it’s implementation may differ at certain points.

<ol start="2">

 <li>Packet Number (signed int) – the relevant tracking number associated with this packet. For data packets, this is the sequence number. For acknowledgement packets, this is the sequence number of the last successfully received data packet.</li>

</ol>

We’re using a <em>signed int </em>so we can use a -1 for our default ACK packet created when the host starts. In a real implementation, we’d use an unsigned int and implement a wrap-around check to determine if an ACK is for an old packet, rather than the simple less than used in the provided FSM.

<ol start="3">

 <li>Checksum (unsigned half) – this should contain the the Internet checksum computed for your packet, as discussed in Section 6.</li>

 <li>Payload Length (unsigned int) – the length of the included *payload*. Does not include the length of the header.</li>

</ol>

A variable-length payload can be included after the header. Acknowledgment messages do not contain any data, so nothing should be included here. The payload length should be set to 0 for acknowledgment messages.

<h1>6         Computing an Internet Checksum</h1>

The algorithm to compute an Internet checksum is fairly straightforward:

<ol>

 <li>Convert your packet into a byte array using pack. One complication here is that you don’t yet know the checksum, but the checksum should be part of the packed data. We can get around this by substituting 0 for where the checksum should go when packing the data, and then repacking the data once we have computed the checksum.</li>

</ol>

<table width="555">

 <tbody>

  <tr>

   <td width="555"># We’re packing an arbitrary set of data in this example. The checksum will eventually # be stored in the H value. We’re going to put a 0 there for now. message = pack(<em>“!IH2s”</em>, 378, 0, <em>“ab”</em>.encode())# Get the checksum using a function we’ve written checksum = compute_checksum(message)# Now re-pack the data with the checksum message = pack(<em>“!IH2s”</em>, 378, checksum, <em>“ab”</em>.encode())</td>

  </tr>

 </tbody>

</table>

1

2

3

4

5

6

7

8

9

<ol start="2">

 <li>Divide your packet up into 16-bit words. If your packet contains an odd number of bytes, you’ll need to pad the end of the packet with a 0-byte (0x0000) in order to get the final 16-bit word.</li>

</ol>

<table width="555">

 <tbody>

  <tr>

   <td width="555"># Check to see if the packet contains an odd number of bits. If so, append a 0-byte to the # end of the packet data. bytes(1) creates a byte array of length 1, initialized with 0’s.if len(packet) % 2 == 1:packet = packet + bytes(1)# Next, divide your byte array into a series of 16-bit words. Every entry in a bytearray# is a single byte, which means it only uses the lowest 8 bits. To convert individual bytes# into a 16-bit word we need to perform a bitwise shift operation, which can move the lowest # 8 bits into the position of the next highest 8 bits. We do this with the bitwise left shift # operator &lt;&lt;.# Once this has been done, we can OR the result with the next byte. This results in a # combination of the two bytes, where the first byte is placed in the 8-15 bit positions, # and the second byte is placed in the 0-7 bit positions.for i in range(0, len(pkt), 2): word = pkt[i] &lt;&lt; 8 | pkt[i+1]</td>

  </tr>

 </tbody>

</table>

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

<ol start="3">

 <li>Sum each of the words together, carrying any overflow bits.</li>

</ol>

<table width="555">

 <tbody>

  <tr>

   <td width="555"># Compute the two words you want to sum, as discussed above word1 = … word2 = …# Add the words together like you would add any other numbers summed_words = word1 + word2# Carry any overflow bits. When adding 2 16-bit words, it’s possible that a 1 will result in # the 17th bit position. If this occurs, we want to remove this bit and add it back to the # lowest 16 bits of the computed number. The line of code below does both of these.# (summed_wrods &amp; 0xffff) ensures that the upper 16 bits will always be 0, and# (summed_words &gt;&gt; 16) right shifts the sum by 16 bits, ensuring that it will equal either # a 0 or a 1, depending on what value was in the 17th position.result = (summed_words &amp; 0xffff) + (summed_words &gt;&gt; 16)</td>

  </tr>

 </tbody>

</table>

1

2

3

4

5

6

7

8

9

10

11

12

13

14

<ol start="4">

 <li>Perform the 1’s complement on the result.</li>

</ol>

<table width="555">

 <tbody>

  <tr>

   <td width="555"># The one’s complement is computed using the ~ operator. We specifically want a 16-bit value, # and Python is internally representing all of the numbers we’re working with as 32-bit # integers, so we need to zero out the upper 16 bits by ANDing the result with 0xffff.checksum = ~result &amp; 0xffff</td>

  </tr>

 </tbody>

</table>

1

2

3

4

You now have the checksum and can repack the data with the checksum as shown in step 1.

<h2>6.1         Handling When the Packet Length is Corrupted</h2>

Any part of your packet can be corrupted. The above algorithm can be executed without issue when any part of a packet is corrupted <strong>except </strong>for when the packet length is corrupted. You can only check for corruption once the entire packet has been received, but you can’t receive the entire packet unless you first know the packet length. If the packet length is corrupted then you won’t be able to fetch the entire packet.

In the test cases provided, corruption of the packet length results in a value much larger than the actual packet. This will cause an exception to be thrown when you attempt to fetch the packet payload when using a corrupted packet length. This exception will contain a message like “<em>unpack requires a buffer of 134217728 bytes</em>”. <strong>This is expected behavior</strong>.

When you receive this exception (once you have your pack and unpack functions working correctly), you should treat the packet you’ve received as corrupt and handle it appropriately. This will require you to wrap calls to unpack in a <em>try…except… </em>block in order to catch the exception. This will allow you to catch this particular instance of corruption prior to calling the <em>is corrupt() </em>method you will be implementing. All other instances of corruption should be detected using that function.

<h1>7         Reading the program’s debug output</h1>

Below is a sample of the debug output you’ll get when you run your program. Each entry contains the following information: [the entity where this event occurred] @ [the simulated time it occurred at]: [the specific event].

<table width="589">

 <tbody>

  <tr>

   <td width="589">B @ 36.4374: Rcvd from Application Layer: aaaaB @ 36.4374: Passing to Network Layer: [TYPE: Data, NUM: 0, CKSUM: 15545, LEN: 4, PAYLOAD: aaaa] B @ 36.4374: Starting TimerA @ 36.8282: Rcvd from Network Layer: [TYPE: Data, NUM: 0, CKSUM: 15545, LEN: 4, PAYLOAD: aaaa]A @ 36.8282: Passing to Application Layer: aaaaA    @ 36.8282: Passing to Network Layer: [TYPE: ACK, NUM: 0, CKSUM: 65535, LEN: 0]B    @ 37.4192: Rcvd from Network Layer: [TYPE: ACK, NUM: 0, CKSUM: 65535, LEN: 0]B @ 37.4192: Stopping TimerA @ 62.8736: Rcvd from Application Layer: bbA @ 62.8736: Passing to Network Layer: [TYPE: Data, NUM: 0, CKSUM: 40219, LEN: 2, PAYLOAD: bb] A @ 62.8736: CORRUPTING PACKET!A    @ 62.8736: Starting TimerB    @ 63.0393: Rcvd from Network LayerB @ 63.0393: Passing to Network Layer: [TYPE: ACK, NUM: -1, CKSUM: 0, LEN: 0]A @ 63.5983: Rcvd from Network Layer: [TYPE: ACK, NUM: -1, CKSUM: 0, LEN: 0]A @ 65.8736: Timer InterruptA                      @ 65.8736: Passing to Network Layer: [TYPE: Data, NUM: 0, CKSUM: 40219, LEN: 2, PAYLOAD: bb]A @ 65.8736: Starting TimerB                      @ 66.7249: Rcvd from Network Layer: [TYPE: Data, NUM: 0, CKSUM: 40219, LEN: 2, PAYLOAD: bb]B @ 66.7249: Passing to Application Layer: bbB @ 66.7249: Passing to Network Layer: [TYPE: ACK, NUM: 0, CKSUM: 65535, LEN: 0]A @ 67.6581: Rcvd from Network Layer: [TYPE: ACK, NUM: 0, CKSUM: 65535, LEN: 0]A @ 67.6581: Stopping TimerA @ 87.7515: Rcvd from Application Layer: cccA                      @ 87.7515: Passing to Network Layer: [TYPE: Data, NUM: 1, CKSUM: 14616, LEN: 3, PAYLOAD: ccc]A @ 87.7515: Starting TimerB                      @ 88.5380: Rcvd from Network Layer: [TYPE: Data, NUM: 1, CKSUM: 14616, LEN: 3, PAYLOAD: ccc]B @ 88.5380: Passing to Application Layer: cccB @ 88.5380: Passing to Network Layer: [TYPE: ACK, NUM: 1, CKSUM: 65534, LEN: 0] B @ 88.5380: CORRUPTING PACKET!A @ 88.8423: Rcvd from Network LayerA    @ 88.8423: Passing to Network Layer: [TYPE: ACK, NUM: 0, CKSUM: 65535, LEN: 0]B    @ 88.9450: Rcvd from Network Layer: [TYPE: ACK, NUM: 0, CKSUM: 65535, LEN: 0]A @ 90.7515: Timer InterruptA @ 90.7515: Passing to Network Layer: [TYPE: Data, NUM: 0, CKSUM: 40219, LEN: 2, PAYLOAD: bb]A                      @ 90.7515: Passing to Network Layer: [TYPE: Data, NUM: 1, CKSUM: 14616, LEN: 3, PAYLOAD: ccc]A @ 90.7515: Starting TimerB                      @ 90.9320: Rcvd from Network Layer: [TYPE: Data, NUM: 1, CKSUM: 14616, LEN: 3, PAYLOAD: ccc]B @ 90.9320: Passing to Network Layer: [TYPE: ACK, NUM: 1, CKSUM: 65534, LEN: 0]B @ 91.1572: Rcvd from Network Layer: [TYPE: Data, NUM: 0, CKSUM: 40219, LEN: 2, PAYLOAD: bb] B @ 91.1572: Passing to Network Layer: [TYPE: ACK, NUM: 1, CKSUM: 65534, LEN: 0] B @ 91.1572: CORRUPTING PACKET!A @ 91.8871: Rcvd from Network Layer: [TYPE: ACK, NUM: 1, CKSUM: 65534, LEN: 0]A @ 91.8871: Stopping TimerA @ 91.9883: Rcvd from Network Layer: [TYPE: ACK, NUM: 257, CKSUM: 65534, LEN: 0] A @ 91.9883: Passing to Network Layer: [TYPE: ACK, NUM: 0, CKSUM: 65535, LEN: 0] A @ 91.9883: LOSING PACKET!</td>

  </tr>

 </tbody>

</table>

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

Each of the possible messages that may appear in the logs are explained below.

<ol>

 <li><strong>Rcvd from Application Layer </strong>indicates that the simulated application has called send() and given your transport layer protocol data to send across the network. This maps on to the simulator calling your program’s <em>receive from application layer() </em></li>

 <li><strong>Rcvd from Network Layer </strong>indicates the simulated network layer has received a packet from the other end of the connection, which could be a data message or an acknowledgment message. This maps on to the simulator calling your program’s <em>receive from network layer() </em></li>

 <li><strong>Passing to Network Layer </strong>indicates that your transport layer protocol is passing a prepared packet to the simulated network layer for transmission to the other end of the connection. This maps on to your program calling <em>pass </em><em>to network layer()</em>. The contents of the packet sent are displayed in the log, assuming you’ve packed your packet correctly.</li>

 <li><strong>Passing to Application Layer: </strong>indicates that your transport layer protocol has received a valid piece of data in the correct order and has given that to the application it was addressed to. This maps on to your program calling <em>pass to application layer()</em>. The data you give to the application layer is displayed in the log.</li>

 <li><strong>Starting Timer </strong>indicates that a timer has been started. This maps on to your program calling <em>start timer()</em>.</li>

 <li><strong>Stopping Timer </strong>indicates that a timer has been stopped. This maps on to your program calling <em>stop timer()</em>.</li>

 <li><strong>Timer Interrupt </strong>indicates that a timer your application had previously set has expired. This maps on to the simulator calling your program’s <em>timer interrupt() </em></li>

 <li><strong>LOSING PACKET </strong>indicates that a packet loss event has occurred in the simulator. The packet that was lost will never be received by the other end of the connection. The packet that was lost is visible in the line above the LOSING PACKET message.</li>

 <li><strong>CORRUPTING PACKET </strong>indicates that a packet corruption event has occurred in the simulator. The packet will arrive, but the checksum should indicate that the packet has been corrupted. The packet that</li>

</ol>

was corrupted is visible in the line above the LOSING PACKET message.

<ol start="10">

 <li><strong>ERROR: ATTEMPTED TO START TIMER WHILE ONE IS ALREADY RUNNING </strong>(not shown) indicates that you have attempted to start a timer when another timer was already running. The GBN protocol never has more than one timer running at a time on a specific host.</li>

 <li><strong>ERROR: ATTEMPTED TO STOP A TIMER BUT NONE WERE RUNNING </strong>(not shown) indicates that you have attempted to stop a timer but no timer was actually running.</li>

</ol>

<h1>8       Testing your project</h1>

Your program will be graded against 14 test cases, including 2 focused on your checksum and 12 simulating a range of different network conditions. The 2 test cases checking your checksum can only be run on Gradescope. The others can be run locally using <em>gbn tester.py</em>.

The remaining 12 test conditions are based on three factors: packet corruption rate, packet loss rate, and data arrival rate. Logs from my reference implementation for each of the final 12 test cases are included in the template files. The autograder checks the validity of your code by comparing your program’s behavior against the behavior you will see in the logs. In other words, a correct implementation will produce the same behavior as my reference implementation.

<h2>1. Testing your is corrupt() function</h2>

<ul>

 <li>Uncorrupt packet</li>

 <li>Corrupt packet</li>

</ul>

<h2>2. Slow arrival rate (1 packet every 20 seconds)</h2>

<ul>

 <li>No loss, no corruption</li>

 <li>25% loss, no corruption</li>

 <li>No loss, 25% corruption</li>

 <li>25% loss, 25% corruption</li>

</ul>

<h2>3. Medium arrival rate (3 packets every second)</h2>

<ul>

 <li>No loss, no corruption</li>

 <li>10% loss, no corruption</li>

 <li>No loss, 10% corruption</li>

 <li>10% loss, 10% corruption</li>

</ul>

<h2>4. Fast arrival rate (100 packets every second)</h2>

<ul>

 <li>No loss, no corruption</li>

 <li>10% loss, no corruption</li>

 <li>No loss, 10% corruption (d) 10% loss, 10% corruption</li>

</ul>


