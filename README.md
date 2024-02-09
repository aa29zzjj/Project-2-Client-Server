# 95-702 Distributed Systems for ISM

## Project 2 Client-Server Computing

### Assigned: Friday, February 9, 2024

### Due: Friday, February 23, 2024,11:59pm

#### Six Tasks

:checkered_flag: Submit to Canvas a ***single PDF file*** named Your_Last_Name_First_Name_Project2.pdf along with a single zip file containing each of the six IntelliJ projects below (Task 0 through 5).

The single PDF will contain your responses to the questions marked with a checkered flag. It is important that you ***clearly label*** each answer with the labels provided below. It is also important to ***be prepared*** to demonstrate your working code if we need to verify your submission. Be sure to provide your name and email address at the top of the PDF submission.

The six IntelliJ projects will be submitted as six zip files, each one will be the zip of your WHOLE IntelliJ project for tasks 0, 1, 2, 3, 4 and 5. Each IntelliJ project, except Task 1, will contain one client and one server. Task 1 will contain one client, one server, and one malicious player in the middle. For each project, zip the whole project, you need to use "File->Export Project->To Zip" in IntelliJ.

When all of your work is complete, zip the one PDF and the six project zip files into one big zip file for submission. Name this final file your_andrew_id.zip.

### Learning Objectives:

Our **first objective** is for you to be able to work with the User Datagram Protocol (UDP). UDP is used in many internet applications. The Domain Name Service (DNS) and the Dynamic Host Configuration Protocol (DHCP) both use UDP. Most video and audio traffic uses UDP. We use UDP when we need high performance and do not mind an occasional dropped packet.

TCP, on the other hand, is also widely used. It works hard to make sure that not a single bit of information is lost in transit. The Hyper Text Transfer Protocol (HTTP) uses TCP. We are not using TCP in this project.

Our **second objective** is to understand the implications of a malicious player in the middle.

Our **third objective** is for you to understand the abstraction provided by Remote Procedure Calls (RPC's). We do this by asking that you use a proxy design and hide communication code and keep it separate from your application code. RPC has been used for four decades and is at the foundation of many distributed systems.

Our **fourth objective** is the learn how to distribute a stand alone application. We use a simple neural network as our application. Our intent is not to study neural networks in a class on distributed systems. But some of you might decide to dig further into neural networks and use this application as a starting point.

Optionally, you may use a large language model (based on neural networks), such as ChatGPT or Copilot, to create some of your code. Task 0, Task 1, and Task 5 must be done without the help of a large language model. There will be exam questions that ask specifically about the code in these three tasks. While you are allowed to use AI tools for these three tasks, it is totally optional. There will be questions about these tasks, too, but these questions will be more generic (since different students may code these tasks using different techniques).

### Submission notes:

When you are asked to submit Java code (on the single pdf) it should be documented. Points will be deducted if code is not well documented. Each significant block of code will contain a comment describing what the block of code is being used for. See Canvas/Home/Documentation for an example of good and bad documentation.

### Rubric
See the General Course Rubric (on Canvas). We will use a specific, unpublished rubric for this assignment but the general rubric provides rough guidance on how this assignment will be evaluated.

### Some simplifications:

In all of what follows, we are concerned with designing servers to handle one client at a time. We are not exploring the important issues surrounding multiple, simultaneous visitors. If you write a multi-threaded server to handle several visitors at once, that is great but is not required. It gains no additional credit.

In addition, for all of what follows, we are assuming that the server is run before the client is run. If you want to handle the case where the client is run first, without a running server, that is great but will receive no additional credit.

In Task 1, we are assuming that the server is run before the malicious player and the malicious player is run before the client.

In this assignment, you need not be concerned with data validation. You may assume that the data entered by users is correctly formatted.

In general, if these requirements do not explicitly ask for a certain feature, then you are not required to provide that feature. No additional points are awarded for extra features.

### Cite your sources

If you use any code that is not yours (including code from a large language model), you are required to clearly cite the source - include a full URL in a comment and place it just above the code that is copied. If you use a large language model to generate code, be sure to say so. Be careful to cite your sources. If you submit code that you did not create on your own and you fail to include proper citations then that will be reported as an academic violation.

## Task 0 introduces UDP. Name the IntelliJ project "Project2Task0".

In Task 0, you will make several modifications to EchoServerUDP.java and EchoClientUDP.java. Note that
these two programs are standard Java and we do not need to construct a web application in IntelliJ. Both of these programs will be placed in the same IntelliJ project.

EchoServerUDP.java from Coulouris text

```
import java.net.*;
import java.io.*;
public class EchoServerUDP{
	public static void main(String args[]){
	DatagramSocket aSocket = null;
	byte[] buffer = new byte[1000];
	try{
		aSocket = new DatagramSocket(6789);
		DatagramPacket request = new DatagramPacket(buffer, buffer.length);
 		while(true){
			aSocket.receive(request);     
			DatagramPacket reply = new DatagramPacket(request.getData(),
      	   	request.getLength(), request.getAddress(), request.getPort());
			String requestString = new String(request.getData());
			System.out.println("Echoing: "+requestString);
			aSocket.send(reply);
		}
	}catch (SocketException e){System.out.println("Socket: " + e.getMessage());
	}catch (IOException e) {System.out.println("IO: " + e.getMessage());
	}finally {if(aSocket != null) aSocket.close();}
	}
}

```

Note the difference between a DatagramSocket and a DatagramPacket. The server uses a DatagramPacket to receive data from a client (in the request object). And it uses a DatagramPacket to send data back to the client (in the reply object). A DatagramPacket is always based on a byte array. So, to send a message in a DatagramPacket, we must first convert the message to a byte array. To receive a message from a DatagramPacket, we must convert the byte array to a String message (if we are expecting a String message).

Note below how the client does the same thing. The client wants to send a String message. So, it extracts a byte array from the String (the variable m). And we then use m to build the DatagramPacket.

When the client receives a reply, the method reply.getData() returns a byte array - which we use to build a String object.

EchoClientUDP.java from Coulouris text

```
import java.net.*;
import java.io.*;
public class EchoClientUDP{
    public static void main(String args[]){
	// args give message contents and server hostname
	DatagramSocket aSocket = null;
	try {
		InetAddress aHost = InetAddress.getByName(args[0]);
		int serverPort = 6789;
		aSocket = new DatagramSocket();
		String nextLine;
		BufferedReader typed = new BufferedReader(new InputStreamReader(System.in));
		while ((nextLine = typed.readLine()) != null) {
		  byte [] m = nextLine.getBytes();
		  DatagramPacket request = new DatagramPacket(m,  m.length, aHost, serverPort);
  		aSocket.send(request);
  		byte[] buffer = new byte[1000];
  		DatagramPacket reply = new DatagramPacket(buffer, buffer.length);
  		aSocket.receive(reply);
  		System.out.println("Reply from server: " + new String(reply.getData()));
    }

	}catch (SocketException e) {System.out.println("Socket Exception: " + e.getMessage());
	}catch (IOException e){System.out.println("IO Exception: " + e.getMessage());
	}finally {if(aSocket != null) aSocket.close();}
    }
}
```
0. Get these programs running in IntelliJ. The two programs are placed in the same Intellij project and you are provided with two windows to interact with the two programs. Make the following modifications to the client and the server.
1. Change the client&#39;s &quot;arg[0]&quot; to a hardcoded &quot;localhost&quot;.
2. Document the client and the server. Describe what each line of code does.
3. Add a line at the top of the client so that it announces, by printing a message on the console, &quot;The UDP client is running.&quot; at start up.
4. After the announcement that the client is running, have the client prompt the user for the server side port number. It will then use that port number to contact the server. For now, enter 6789.
5. Add a line at the top of the server so that it announces &quot;The UDP server is running.&quot; at start up.
6. After the announcement that the server is running, have the server prompt the user for the port number that the server is supposed to listen on. Enter 6789 when prompted.
7. On the server, examine the length of the requestString and note that it is too large. Make modifications to the server code so that the request data is copied to an array with the correct number of bytes. Use this array of bytes to build a requestString of the correct size. Without these modifications, incorrect data may be displayed on the server. Upon each visit, your server will display the request arriving from the client.
8. Do the same on the client side to properly handle the size of the response.
9. If the client enters the command &quot;halt!&quot;, both the client and the server will halt execution. When the client program receives &quot;halt!&quot; from the user, it sends &quot;halt!&quot; to the sever and after hearing the &quot;halt!&quot; message from the server, the client exits. When the server receives &quot;halt!&quot; from the client, it will respond to the client with &quot;halt!&quot; and then exit.
10. Add a line in the client so that it announces when it is quitting. It will write "UDP Client side quitting" to the client side console.
11. Add a line in the server so that it makes an announcement when it is quitting. The server only quits when it is told to do so by the client (and has just responded to the client with the &quot;halt!&quot; message). It will write "UDP Server side quitting" to the server side console.

:checkered_flag:**On your single pdf, make a copy of your client and label it clearly as "Project2Task0Client".**

:checkered_flag:**On your single pdf, make a copy of your server and label it clearly as "Project2Task0Server".**

:checkered_flag:**Make a screenshot of your client console screen. It will include five lines of data sent by the client to the server and the client's response to a request by the user to &quot;halt!&quot;. On your single pdf, label this screenshot as "Project2Task0ClientConsole".**

:checkered_flag:**Make a screenshot of your server console screen. It will include five lines of data sent by the client and the server's response to the &quot;halt!&quot; request by the client. On your single pdf, label this screenshot as "Project2Task0ServerConsole".**


## Task 1 illustrates a malicious player in the middle attack on UDP. Name the IntelliJ Project "Project2Task1".

In Task 1, you will experiment with a malicious player in the middle attack. This malicious player is interested in more than simply eavesdropping on the conversation between the client and the server. It is an active malicious player. You might want to get started by working on a passive malicious player - one that only eavesdrops and passes messages along to the server and then back to the client.

We will have three UDP programs in one project.

Name your malicious player EavesdropperUDP.java. You will need to design and write EavesdropperUDP.java as it is described below.

First, run EchoServerUDP.java as it has been modified in Task 0. EchoServerUDP will prompt you for its port. Enter the port 6789 for EchoServerUDP to listen on.

Second, run EavesdropperUDP.java. EavesdropperUDP will state that it is running and will ask you for two ports. One port will be the port that the EavesdropperUDP.java will listen on and the other port will be the port number of the server that Eavesdropper.java is masquerading as. We want Eavesdropper.java to display (on its console) all messages that go through it. We want it to eavesdrop on the wire. It will be masquerading as the server on port 6789. It will listen on port 6798 - hoping a foolish client will make a transposition error.

Third, when you run EchoClientUDP.java, provide it with either the correct port (of the real server) or the port that Eavesdropper is listening on. That is, it will work with either 6789 or 6798.  

Eavesdropper is an active attacker. If the client sends a string containing the word &quot;like&quot; the eavesdropper will replace the word &quot;like&quot; with the word &quot;dislike&quot;. The eavesdroppr will not bother with the string &quot;like&quot; if it is included as a substring of another word, such as &quot;dislike&quot;.

If the client sends the message &quot;halt!&quot; then the server will respond, as usual, and then halt execution. The client will halt when it hears from the server. Our malicious player runs forever. And it displays everything it sees to its console.

:checkered_flag:**On your single pdf, make a copy of your documented EavesdropperUDP.java program.

:checkered_flag:**Make a screenshot showing your client, server, and eavesdropper consoles. The shot will show a few lines of data sent by the client and the server's response to the &quot;halt!&quot; request by the client. It will also show the eavesdropper console - showing the entire interaction between the client and the server. On your single pdf, label this screenshot as "Project2Task1ThreeConsoles". Be sure to show the client using port 6789 (correct server) and 6798 (malicious player). The idea is to provide screenshots that demonstrate that the client works against both servers. You also need to show the word &quot;like&quot; being replaced with the word &quot;dislike&quot;**

In the remaining tasks (Tasks 2 through 5), we do not provide the client with the ability to stop the server. We are doing that only in Tasks 0 and 1. In the remaining Tasks, the server is left running - forever. In the remaining tasks, we are not using an eavesdropper.

## Task 2 illustrates a proxy design using UDP. Name the IntelliJ Project "Project2Task2".

Make the following modifications to "EchoServerUDP.java" and "EchoClientUDP.java":

0. Name the client "AddingClientUDP.java". Name the server "AddingServerUDP.java".
1. The server will hold an integer value sum, initialized to 0, and will receive requests from the client - each of which includes an integer value (positive or negative or 0) to be added to the sum. Upon each request, the server will return the new sum as a response to the client. On the server side console, upon each visit by the client, the client's request and the new sum will be displayed.
2. Separate concerns on the client. On the client, all of the communication code will be placed in a method named &quot;add&quot;. In other words, the main method of the client will have no code related to client server communications. Instead, the main routine will simply call a local method named &quot;add&quot;. The client side &quot;add&quot; method will not perform any addition, instead, it will request that the server perform the addition. The &quot;add&quot; method will encapsulate or hide all communication with the server. It is within the &quot;add&quot; method where we actually work with sockets. This is a variation of what is called a &quot;proxy design&quot;. The &quot;add&quot; method is serving as a proxy for the server. When your code makes a call on the local "add" method, you are actually making a remote procedure call (RPC). The client side &quot;add&quot; method has the following signature:

```
public static int add(int i)

```
3. Separate concerns on the server. Your code that listens for a socket connection should be separate from the code that performs the add operation. In other words, the actual arithmetic should be done in a separate method. The UDP socket communication code will make calls to this method.

4. Write a client and server that has the following client side interaction with a user:

```
The client is running.
Please enter server port:
6789

3
The server returned 3.
2
The server returned 5.
-1
The server returned 4.
6
The server returned 10.
halt!
Client side quitting.

If the client is restarted (note that the server is still running) we have:
The client is running.
Please enter server port:
6789
1
The server returned 11.
halt!
Client side quitting.

```

5. On the server, the console will show an interaction like the following:

```
Server started
Adding: 3 to 0
Returning sum of 3 to client

Adding: 2 to 3
Returning sum of 5 to client

etc...

```

Note: UDP messages are made up of byte arrays. You will need to take an int and place it into a four byte (32 bit) byte array before sending. When receiving, you will need to extract an int from the byte array. You may use code from external sources to help you do this. But be careful to site your sources with a clear URL.

Another approach would be to only transmit byte arrays containing String data. You may use either approach.

:checkered_flag:**On your single pdf, make a copy of your client and label it clearly as "Project2Task2Client".**

:checkered_flag:**On your single pdf, make a copy of your server and label it clearly as "Project2Task2Server".**

:checkered_flag:**Take a screenshot of your client console screen. It will include five integer inputs (1,2,-3,4, and 5) and show the sums as they arrive back from the server. It will also show the client being stopped, and re-run a second time with the inputs (6,7,-8,9, and 10) and the client's response to a request by the user to &quot;halt!&quot;. On your single pdf, label this screenshot as "Project2Task2ClientConsole".**

:checkered_flag:**Take a screenshot of your server console screen. It will include the 10 lines of data being sent by the client and the server's responses. On your single pdf, label this screenshot as "Project2Task2ServerConsole".**


## Task 3 maintains server state using UDP. Name the IntelliJ project "Project2Task3"

0. Name the client "RemoteVariableClientUDP.java". Name the server "RemoteVariableServerUDP.java".

1. Modify your work in Task 2 so that the client may request either an &quot;add&quot; or &quot;subtract&quot; or &quot;get&quot; operation be performed by the server. The &quot;add&quot; and &quot;subtract&quot; operations are not idempotent but the &quot;get&quot; operation is idempotent. In addition, each request will pass along an integer ID. This ID is used to uniquely identify the user. Thus, the client will form a packet with the following values: ID, operation (add or subtract or get), and value (if the operation is other than get). The server will carry out the correct computation (add or subtract or get) using the sum associated with the ID found in each request. The client will be menu driven and will repeatedly ask the user for the user ID, operation, and value (if not a get request). When the operation is &quot;get&quot;, the value held on the server is simply returned. When the operation is &quot;add&quot; or &quot;subtract&quot; the server performs the operation and returns the sum. During execution, the client will display each returned value from the server to the user. If the server receives an ID that it has not seen before, that ID will initially be associated with a sum of 0. ID's will range between 0 and 999.

2. On the server, you will need to map each ID to the value of a sum. Different ID&#39;s may be presented and each will have its own sum. The server is given no prior knowledge of what ID&#39;s will be transmitted to it by the client. You may only assume that ID&#39;s are positive integers. You are required to store each ID and its associated sum, in a Java TreeMap.

The client side menu will provide an option to exit the client. Exiting the client has no impact on the server. Here is an example client side interaction:

```
The client is running.
Please enter server port:
6789

1. Add a value to your sum.
2. Subtract a value from your sum.
3. Get your sum.
4. Exit client
1
Enter value to add:
5
Enter your ID:
102
The result is 5.

1. Add a value to your sum.
2. Subtract a value from your sum.
3. Get your sum.
4. Exit client.
1
Enter value to add:
14
Enter your ID:
102
The result is 19.

1. Add a value to your sum.
2. Subtract a value from your sum.
3. Get your sum.
4. Exit client.
1
Enter value to add:
10
Enter your ID:
199
The result is 10.

1. Add a value to your sum.
2. Subtract a value from your sum.
3. Get your sum.
4. Exit client.
3
Enter your ID:
102
The result is 19.

1. Add a value to your sum.
2. Subtract a value from your sum.
3. Get your sum.
4. Exit client.
2
Enter value to subtract:
-3
Enter your ID:
199
The result is 13.

1. Add a value to your sum.
2. Subtract a value from your sum.
3. Get your sum.
4. Exit client.
4
Client side quitting. The remote variable server is still running.

```

3. As you did in Task 2, use a proxy design to encapsulate the communication code.

:checkered_flag:**On your single pdf, make a copy of your client and label it clearly as "Project2Task3Client".**

:checkered_flag:**On your single pdf, make a copy of your server and label it clearly as "Project2Task3Server".**

:checkered_flag:**Take a screenshot of your client console screen. Show three different clients interacting with the server using three distinct ID&#39;s. Each client will perform one addition, one subtraction, and finally a get request. It will also show the client being stopped, and re-run a second time with get requests from each of the three clients.  On your single pdf, label this screenshot as "Project2Task3ClientConsole".**

:checkered_flag:**Take a screenshot of your server console. It will show each visitor's ID, the operation requested, and the value of the variable being returned. On your single pdf, label this screenshot as "Project2Task3ServerConsole".**

## Task 4 Distributes a neural network into a client server application. Name the IntelliJ project "Project2Task4"

0. Read over, copy and run the stand alone program NeuralNetwork.java found here:

```
// This program is based upon an excellent blog post from Matt Mazur.
// See: https://mattmazur.com/2015/03/17/a-step-by-step-backpropagation-example/
// Matt's solution is in Python and this program is a translation of the same logic into Java.
// The mathematics underlying this neural network are explained well in the blog post.
// To fully understand this program, you would need to study the math in the blog post.

/*The idea is to train a neural network to learn a truth table.
Here is a truth table for the AND operation:
0   0   0
0   1   0
1   0   0
1   1   1

And here is a truth table for the XOR operation:
0  0  0
0  1  1
1  0  1
1  1  0

The user is asked to provide the rightmost column of the table. For example, if the user wants to
train the table for the XOR operation, the user will provide the following inputs:  0  1  1  0.

10,000 steps are typically used to train the network. If the error is close to 0, for example, 0.053298, the network
will perform well.

If the output of a test is close to 1, for example, .9759876, we will call that a 1.
If the output of a test is close to 0, for example, .0348712, we will call that a 0.

The program is not written to handle input errors. We assume that the user behaves well.
 */

import java.util.*;

// Each Neuron has a bias, a list of weights, and a list of inputs.
// Each neuron will produce a single real number as an output.
class Neuron {
    private double bias;
    public List<Double> weights;
    public List<Double> inputs;
    double output;
    // Construct a neuron with a bias and reserve memory for its weights.
    public Neuron(double bias) {
        this.bias = bias;
        weights = new ArrayList<Double>();
    }
    //Calculate the output by using the inputs and weights already provided.
    //Squash the result so the output is between 0 and 1.
    public double calculateOutput(List<Double> inputs) {

        this.inputs = inputs;

        output = squash(calculateTotalNetInput());
        return output;
    }
    // Compute the total net input from the input, weights, and bias.
    public double calculateTotalNetInput() {

        double total = 0.0;
        for (int i = 0; i < inputs.size(); i++) {
            total += inputs.get(i) * weights.get(i);
        }
        return total + bias;
    }

    // This is the activation function, returning a value between 0 and 1.
    public double squash(double totalNetInput) {
        double v = 1.0 / (1.0 + Math.exp(-1.0 * totalNetInput));
        return v;
    }
    // Compute the partial derivative of the error with respect to the total net input.
    public Double calculatePDErrorWRTTotalNetInput(double targetOutput) {
        return calculatePDErrorWRTOutput(targetOutput) * calculatePDTotalNetInputWRTInput();
    }
    // Calculate error. How different are we from the target?
    public Double calculate_error(Double targetOutput) {
        double theError = 0.5 * Math.pow(targetOutput - output, 2.0);
        return theError;
    }
    // Compute the partial derivative of the error with respect to the output.
    public Double calculatePDErrorWRTOutput(double targetOutput) {
        return (-1) * ( targetOutput - output);
    }
    // Compute the partial derivative of the total net input with respect to the input.
    public Double  calculatePDTotalNetInputWRTInput() {
        return output * ( 1.0 - output);

    }
    // Calculate the partial derivative of the total net input with respect to the weight.
    public Double calculatePDTotalNetInputWRTWeight(int index) {
        return inputs.get(index);
    }
}

// The Neuron layer represents a collection of neurons.
// All neurons in the same layer have the same bias.
// We include in each layer the number of neurons and the list of neurons.
class NeuronLayer {
    private double bias;
    private int numNeurons;

    public List<Neuron> neurons;

    // Construct by specifying the number of neurons and the bias that applies to all the neurons in this layer.
    // If the bias is not provided, choose a random bias.
    // Create neurons for this layer and set the bias in each neuron.
    public NeuronLayer(int numNeurons, Double bias) {
        if(bias == null) {

            this.bias = new Random().nextDouble();
        }
        else {
            this.bias = bias;
        }
        this.numNeurons = numNeurons;
        this.neurons  = new ArrayList<Neuron>();
        for(int i = 0; i < numNeurons; i++) {
            this.neurons.add(new Neuron(this.bias));
        }
    }
    // Display the neuron layer by displaying each neuron.
    public String toString() {
        String s = "";
        s = s + "Neurons: " + neurons.size() + "\n";
        for(int n = 0; n < neurons.size(); n++) {
            s = s + "Neuron " + n + "\n";
            for (int w = 0; w < neurons.get(n).weights.size(); w++) {
                s = s + "\tWeight: " + neurons.get(n).weights.get(w) + "\n";
            }
            s = s + "\tBias " + bias + "\n";
        }

        return s;
    }

    // Feed the input data into the neural network and produce some output in the output layer.
    // Return a list of outputs. There may be a single output in the output list.
    List<Double> feedForward(List<Double> inputs) {

        List<Double> outputs = new ArrayList<Double>();

        for(Neuron neuron : neurons ) {

            outputs.add(neuron.calculateOutput(inputs));
        }

        return outputs;
    }
    // Return a list of outputs from this layer.
    // We do this by gathering the output of each neuron in the layer.
    // This is returned as a list of Doubles.
    // This is not used in this program.
    List<Double> getOutputs() {
        List<Double> outputs = new ArrayList<Double>();
        for(Neuron neuron : neurons ) {
            outputs.add(neuron.output);
        }
        return outputs;
    }
}

// The NeuralNetwork class represents two layers of neurons - a hidden layer and an output layer.
// We also include the number of inputs and the learning rate.
// The learning rate determines the step size by which the network’s weights are
// updated during each iteration of training. This is typically chosen experimentally.

public class NeuralNetwork {

    // The learning rate is chosen experimentally. Typically, it is set between 0 and 1.
    private double LEARNING_RATE = 0.5;
    // This truth table example will have two inputs.
    private int numInputs;

    // This neural network will be built from two layers of neurons.
    private NeuronLayer hiddenLayer;
    private NeuronLayer outputLayer;

    // The neural network is constructed by specifying the number of inputs, the number of neurons in the hidden layer,
    // the number of neurons in the output layer, the hidden layer weights, the hidden layer bias,
    // the output layer weights and output layer bias.
    public NeuralNetwork(int numInputs, int numHidden, int numOutputs, List<Double> hiddenLayerWeights, Double hiddenLayerBias,
                         List<Double> outputLayerWeights, Double outputLayerBias) {
        // How many inputs to this neural network
        this.numInputs = numInputs;

        // Create two layers, one hidden layer and one output layer.
        hiddenLayer = new NeuronLayer(numHidden, hiddenLayerBias);
        outputLayer = new NeuronLayer(numOutputs, outputLayerBias);

        initWeightsFromInputsToHiddenLayerNeurons(hiddenLayerWeights);

        initWeightsFromHiddenLayerNeuronsToOutputLayerNeurons(outputLayerWeights);
    }

    // The hidden layer neurons have weights that are assigned here. If the actual weights are not
    // provided, random weights are generated.
    public void initWeightsFromInputsToHiddenLayerNeurons(List<Double> hiddenLayerWeights) {

        int weightNum = 0;
        for (int h = 0; h < hiddenLayer.neurons.size(); h++) {
            for (int i = 0; i < numInputs; i++) {
                if (hiddenLayerWeights == null) {
                    hiddenLayer.neurons.get(h).weights.add((new Random()).nextDouble());
                } else {
                    hiddenLayer.neurons.get(h).weights.add(hiddenLayerWeights.get(weightNum));
                }
                weightNum = weightNum + 1;
            }
        }
    }

    // The output layer neurons have weights that are assigned here. If the actual weights are not
    // provided, random weights are generated.
    public void initWeightsFromHiddenLayerNeuronsToOutputLayerNeurons(List<Double> outputLayerWeights) {
        int weightNum = 0;
        for (int o = 0; o < outputLayer.neurons.size(); o++) {
            for (int h = 0; h < hiddenLayer.neurons.size(); h++) {
                if (outputLayerWeights == null) {
                    outputLayer.neurons.get(o).weights.add((new Random()).nextDouble());
                } else {
                    outputLayer.neurons.get(o).weights.add(outputLayerWeights.get(weightNum));
                }
                weightNum = weightNum + 1;
            }
        }
    }

    // Display a NeuralNetwork object by calling the toString on each layer.
    public String toString() {
        String s = "";
        s = s + "-----\n";
        s = s + "* Inputs: " + numInputs + "\n";
        s = s + "-----\n";

        s = s + "Hidden Layer\n";
        s = s + hiddenLayer.toString();
        s = s + "----";
        s = s + "* Output layer\n";
        s = s + outputLayer.toString();
        s = s + "-----";
        return s;
    }

    // Feed the inputs provided into the network and get outputs.
    // The inputs are provided to the hidden layer. The hidden layer's outputs
    // are provided as inputs the output layer. The outputs of the output layer
    // are returned to the caller as a list of outputs. That number of outputs may be one.
    // The feedForward method is called on each layer.
    public List<Double> feedForward(List<Double> inputs) {

        List<Double> hiddenLayerOutputs = hiddenLayer.feedForward(inputs);
        return outputLayer.feedForward(hiddenLayerOutputs);
    }

    // Training means to feed the data forward - forward propagation. Compare the result with the target(s), and
    // use backpropagation to update the weights. See the blog post to review the math.
    public void train(List<Double> trainingInputs, List<Double> trainingOutputs) {

        // Update state of neural network and ignore the return value
        feedForward(trainingInputs);
        // Perform backpropagation
        List<Double> pdErrorsWRTOutputNeuronTotalNetInput =
                new ArrayList<Double>(Collections.nCopies(outputLayer.neurons.size(), 0.0));
        for (int o = 0; o < outputLayer.neurons.size(); o++) {
            pdErrorsWRTOutputNeuronTotalNetInput.set(o, outputLayer.neurons.get(o).calculatePDErrorWRTTotalNetInput(trainingOutputs.get(o)));
        }
        List<Double> pdErrorsWRTHiddenNeuronTotalNetInput =
                new ArrayList<Double>(Collections.nCopies(hiddenLayer.neurons.size(), 0.0));
        for (int h = 0; h < hiddenLayer.neurons.size(); h++) {
            double dErrorWRTHiddenNeuronOutput = 0;
            for (int o = 0; o < outputLayer.neurons.size(); o++) {
                dErrorWRTHiddenNeuronOutput +=
                        pdErrorsWRTOutputNeuronTotalNetInput.get(o) * outputLayer.neurons.get(o).weights.get(h);
                pdErrorsWRTHiddenNeuronTotalNetInput.set(h, dErrorWRTHiddenNeuronOutput *
                        hiddenLayer.neurons.get(h).calculatePDTotalNetInputWRTInput());
            }
        }
        for (int o = 0; o < outputLayer.neurons.size(); o++) {
            for (int wHo = 0; wHo < outputLayer.neurons.get(o).weights.size(); wHo++) {
                double pdErrorWRTWeight =
                        pdErrorsWRTOutputNeuronTotalNetInput.get(o) *
                                outputLayer.neurons.get(o).calculatePDTotalNetInputWRTWeight(wHo);
                outputLayer.neurons.get(o).weights.set(wHo, outputLayer.neurons.get(o).weights.get(wHo) - LEARNING_RATE * pdErrorWRTWeight);
            }
        }
        for (int h = 0; h < hiddenLayer.neurons.size(); h++) {
            for (int wIh = 0; wIh < hiddenLayer.neurons.get(h).weights.size(); wIh++) {
                double pdErrorWRTWeight =
                        pdErrorsWRTHiddenNeuronTotalNetInput.get(h) *
                                hiddenLayer.neurons.get(h).calculatePDTotalNetInputWRTWeight(wIh);
                hiddenLayer.neurons.get(h).weights.set(wIh, hiddenLayer.neurons.get(h).weights.get(wIh) - LEARNING_RATE * pdErrorWRTWeight);
            }
        }
    }

    // Perform a feed forward for each training row and total the error.
    public double calculateTotalError(ArrayList<Double[][]> trainingSets) {

        double totalError = 0.0;

        for (int t = 0; t < trainingSets.size(); t++) {
            List<Double> trainingInputs = Arrays.asList(trainingSets.get(t)[0]);
            List<Double> trainingOutputs = Arrays.asList(trainingSets.get(t)[1]);
            feedForward(trainingInputs);
            for (int o = 0; o < trainingOutputs.size(); o++) {
                totalError += outputLayer.neurons.get(o).calculate_error(trainingOutputs.get(o));
            }
        }
        return totalError;
    }

    // Read the user input with a Scanner.
    static Scanner scanner = new Scanner(System.in);

    public static void main(String args[]) {

        // Create an initial truth table with all 0's in the range.
        ArrayList<Double[][]> user_training_sets = new ArrayList<Double[][]>(Arrays.asList(
                new Double[][]{{0.0, 0.0}, {0.0}},
                new Double[][]{{0.0, 1.0}, {0.0}},
                new Double[][]{{1.0, 0.0}, {0.0}},
                new Double[][]{{1.0, 1.0}, {0.0}}
        ));
        //     Create a neural network suitable for working with truth tables.
        //     There will be two inputs, 5 hidden neurons, and 1 output. All weights and biases will be random.
        //     This is the initial neural network on start up.
        NeuralNetwork neuralNetwork = new NeuralNetwork(2, 5, 1, null, null, null, null);

        Random rand = new Random();
        int random_choice;

        // Hold a list of doubles for input for the neural network to train on.
        // In this example, if we want to train the neural network to learn the XOR,
        // the list would have two doubles, say 0 1 or 1 0 or 1 1.
        List<Double> user_training_inputs;

        // Hold a list of double for the output of training. For example, XOR would produce 1 double as output.
        List<Double> user_training_outputs;

        int userSelection = menu();

        while (userSelection != 5) {
            switch (userSelection) {
                case 0: // display the truth table
                    System.out.println("Working with the following truth table");
                    for (int r = 0; r < 4; r++) {
                        System.out.print(user_training_sets.get(r)[0][0] + "  " + user_training_sets.get(r)[0][1] + "  " + user_training_sets.get(r)[1][0] + "  ");
                        System.out.println();
                    }
                    break;
                case 1:  // get the range values of the truth table. These values are from the rightmost column of a
                         // standard truth table.
                    System.out.println("Enter the four results of a 4 by 2 truth table. Each value should be 0 ot 1.");
                    Double a = scanner.nextDouble();
                    Double b = scanner.nextDouble();
                    Double c = scanner.nextDouble();
                    Double d = scanner.nextDouble();
                    // Create the new table with the user input as the rightmost column.
                    user_training_sets = new ArrayList<Double[][]>(Arrays.asList(
                            new Double[][]{{0.0, 0.0}, {a}},
                            new Double[][]{{0.0, 1.0}, {b}},
                            new Double[][]{{1.0, 0.0}, {c}},
                            new Double[][]{{1.0, 1.0}, {d}}
                    ));
                    // Build a new neural network with new random weights.
                    neuralNetwork = new NeuralNetwork(2, 5, 1, null, null, null, null);
                    break;

                case 2: // perform a single trainng step and display total error.
                    random_choice = rand.nextInt(4);
                    // Get the two inputs
                    user_training_inputs = Arrays.asList(user_training_sets.get(random_choice)[0]);
                    // Get the one output (in the case of truth tables).
                    user_training_outputs = Arrays.asList(user_training_sets.get(random_choice)[1]);
                    // Show that row to the neural network
                    neuralNetwork.train(user_training_inputs, user_training_outputs);
                    // Show error as we train
                    System.out.println("After this step the error is : " + neuralNetwork.calculateTotalError(user_training_sets));
                    break;
                case 3: // perform n training steps
                    System.out.println("Enter the number of training sets.");
                    int n = scanner.nextInt();
                    for (int i = 0; i < n; i++) {
                        random_choice = rand.nextInt(4);
                        // Get the two inputs
                        user_training_inputs = Arrays.asList(user_training_sets.get(random_choice)[0]);
                        // Get the one output
                        user_training_outputs = Arrays.asList(user_training_sets.get(random_choice)[1]);
                        // Show that row to the neural network
                        neuralNetwork.train(user_training_inputs, user_training_outputs);
                    }
                    // Show error as we train
                    System.out.println("After " + n + " training steps, our error " + neuralNetwork.calculateTotalError(user_training_sets));
                    break;
                case 4: // test with a pair of inputs.
                    System.out.println("Enter a pair of doubles from a row of the truth table. These are domain values.");
                    double input0 = scanner.nextDouble();
                    double input1 = scanner.nextDouble();

                    List<Double> testUserInputs = new ArrayList<>(Arrays.asList(input0, input1));
                    List<Double> userOutput = neuralNetwork.feedForward(testUserInputs);
                    System.out.println("The range value is approximately " + userOutput.get(0));
                    break;
                case 5:
                    System.exit(0);
                    break;
                default:
                    System.out.println("Error in input. Please choose an integer from the main menu.");
                    break;
            }
            userSelection = menu();
        }
    }
    public static int menu() {
        System.out.println("Using a neural network to learn a truth table.\nMain Menu");
        System.out.println("0. Display the current truth table.");
        System.out.println("1. Provide four inputs for the range of the two input truth table and build a new neural network. To test XOR, enter 0  1  1  0.");
        System.out.println("2. Perform a single training step.");
        System.out.println("3. Perform n training steps. 10000 is a typical value for n.");
        System.out.println("4. Test with a pair of inputs.");
        System.out.println("5. Exit program.");
        int selection = scanner.nextInt();
        return selection;
    }
}

```

 1. An example exectution of the program appears next:

 ```
 Using a neural network to learn a truth table.
 Main Menu
 0. Display the current truth table.
 1. Provide four inputs for the range of the two input truth table and build a new neural network. To test XOR, enter 0  1  1  0.
 2. Perform a single training step.
 3. Perform n training steps. 10000 is a typical value for n.
 4. Test with a pair of inputs.
 5. Exit program.
 0
 Working with the following truth table
 0.0  0.0  0.0  
 0.0  1.0  0.0  
 1.0  0.0  0.0  
 1.0  1.0  0.0  
 Using a neural network to learn a truth table.
 Main Menu
 0. Display the current truth table.
 1. Provide four inputs for the range of the two input truth table and build a new neural network. To test XOR, enter 0  1  1  0.
 2. Perform a single training step.
 3. Perform n training steps. 10000 is a typical value for n.
 4. Test with a pair of inputs.
 5. Exit program.
 1
 Enter the four results of a 4 by 2 truth table. Each value should be 0 ot 1.
 0 1 1 0
 Using a neural network to learn a truth table.
 Main Menu
 0. Display the current truth table.
 1. Provide four inputs for the range of the two input truth table and build a new neural network. To test XOR, enter 0  1  1  0.
 2. Perform a single training step.
 3. Perform n training steps. 10000 is a typical value for n.
 4. Test with a pair of inputs.
 5. Exit program.
 0
 Working with the following truth table
 0.0  0.0  0.0  
 0.0  1.0  1.0  
 1.0  0.0  1.0  
 1.0  1.0  0.0  
 Using a neural network to learn a truth table.
 Main Menu
 0. Display the current truth table.
 1. Provide four inputs for the range of the two input truth table and build a new neural network. To test XOR, enter 0  1  1  0.
 2. Perform a single training step.
 3. Perform n training steps. 10000 is a typical value for n.
 4. Test with a pair of inputs.
 5. Exit program.
 2
 After this step the error is : 0.8037225095271462
 Using a neural network to learn a truth table.
 Main Menu
 0. Display the current truth table.
 1. Provide four inputs for the range of the two input truth table and build a new neural network. To test XOR, enter 0  1  1  0.
 2. Perform a single training step.
 3. Perform n training steps. 10000 is a typical value for n.
 4. Test with a pair of inputs.
 5. Exit program.
 3
 Enter the number of training sets.
 10000
 After 10000 training steps, our error 0.0061667496139162885
 Using a neural network to learn a truth table.
 Main Menu
 0. Display the current truth table.
 1. Provide four inputs for the range of the two input truth table and build a new neural network. To test XOR, enter 0  1  1  0.
 2. Perform a single training step.
 3. Perform n training steps. 10000 is a typical value for n.
 4. Test with a pair of inputs.
 5. Exit program.
 4
 Enter a pair of doubles from a row of the truth table. These are domain values.
 0 1
 The range value is approximately 0.9436960462470947
 Using a neural network to learn a truth table.
 Main Menu
 0. Display the current truth table.
 1. Provide four inputs for the range of the two input truth table and build a new neural network. To test XOR, enter 0  1  1  0.
 2. Perform a single training step.
 3. Perform n training steps. 10000 is a typical value for n.
 4. Test with a pair of inputs.
 5. Exit program.
 4
 Enter a pair of doubles from a row of the truth table. These are domain values.
 1 1
 The range value is approximately 0.04579382338123964
 Using a neural network to learn a truth table.
 Main Menu
 0. Display the current truth table.
 1. Provide four inputs for the range of the two input truth table and build a new neural network. To test XOR, enter 0  1  1  0.
 2. Perform a single training step.
 3. Perform n training steps. 10000 is a typical value for n.
 4. Test with a pair of inputs.
 5. Exit program.
 4
 Enter a pair of doubles from a row of the truth table. These are domain values.
 0 0
 The range value is approximately 0.06329027307191817
 Using a neural network to learn a truth table.
 Main Menu
 0. Display the current truth table.
 1. Provide four inputs for the range of the two input truth table and build a new neural network. To test XOR, enter 0  1  1  0.
 2. Perform a single training step.
 3. Perform n training steps. 10000 is a typical value for n.
 4. Test with a pair of inputs.
 5. Exit program.
 4
 Enter a pair of doubles from a row of the truth table. These are domain values.
 1 0
 The range value is approximately 0.9446770292352884
 Using a neural network to learn a truth table.
 Main Menu
 0. Display the current truth table.
 1. Provide four inputs for the range of the two input truth table and build a new neural network. To test XOR, enter 0  1  1  0.
 2. Perform a single training step.
 3. Perform n training steps. 10000 is a typical value for n.
 4. Test with a pair of inputs.
 5. Exit program.

 ```
 2. Your task is to distribute NeuralNetwork.java into two programs. One of these will act as the client. Name your client NeuralNetworkClient.java. The other will act as the server. Name your server NeuralNetworkServer.java.

 3. Your client and server will communicate using JSON messages over UDP.

 4. The neural network will live only on the server. When the server receives a JSON message, it carries out
 the requested task on the neural network and returns a result (also encoded in JSON) to the client.

 5. The client will behave just like the stand alone application shown above. It will provide the same menu of options but, this time, it will communicate with the server to handle requests on the neural network.

 6. You should think about the format (design) of the JSON messages. Some need to go from the client to the server and some need to go from the server back to the client.

 7. As you did in Task 2, use a proxy design to encapsulate the communication code.

 :checkered_flag:**On your single pdf, make a copy of your client and label it clearly as "Project2Task4Client".**

 :checkered_flag:**On your single pdf, make a copy of your server and label it clearly as "Project2Task4Server".**

 :checkered_flag:**Take a screenshot (or copy and paste) your client console screen. The user should show the neural network being trained to learn the logical "OR", "XOR", and "AND" operations.  On your single pdf, label this screenshot as "Project2Task4ClientConsole".**

 :checkered_flag:**Take a screenshot (or copy and paste) of your server console. It will show each JSON string being sent by the client to the server and will show each JSON string being sent back to the client. On your single pdf, label this screenshot as "Project2Task4ServerConsole".**


## Submission Summary:

:checkered_flag: Submit to Canvas the single PDF file named Your_Last_Name_First_Name_Project2.pdf. It is important that you ***clearly label*** each submission. Be sure to provide your name and email address at the top of the .pdf file.

Finally, create five zip files, each one of which is the zip of your WHOLE project for tasks 0, 1, 2, 3, and 4. Each project will contain one client and one server (except for Task 1). For each project, zip the whole project, you need to use "File->Export Project->To Zip" in IntelliJ.

Zip the one PDF and the six project zip files into one big zip file for submission. Name this file your_andrew_id.zip.
