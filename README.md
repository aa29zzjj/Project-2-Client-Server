# 95-702 Distributed Systems for ISM

## Project 2 Client-Server Computing

### Assigned: Friday, February 19, 2021

### Due: Friday March 5, 2021,11:59pm

### Learning Objectives:

Our **first objective** is for you to be able to work with client side and server side UDP and TCP sockets.
Our **second objective** is to understand the differences (at the programming level) between the TCP and UDP protocols.

Our **third objective** is for you to understand the abstraction provided by Remote Procedure Calls (RPC). We do this by asking that you use a proxy design and hide communication code and keep it separate from your application code. And, our **fourth objective** is to expose you to some of the mechanics behind RSA digital signatures.

There are five separate and distinct tasks in Project 2.

For each project task, software documentation is required. The software that you write (Java files and so on) must contain comments that describe what each significant piece of code is intended to accomplish. Points will be deducted if code is not well documented. Each significant block of code will contain a comment describing what the block of code is being used for.

The General Course Rubric (on Canvas) will be applied to a subset of these five tasks.

In all of what follows, we are concerned with designing servers to handle one client at a time. We are not exploring the important issues surrounding multiple, simultaneous visitors. If you write a multi-threaded server to handle several visitors at once, that is great but is not required. It gains no additional credit.

In addition, for all of what follows, we are assuming that the server is run before the client is run. If you want to handle the case where the client is run first, without a running server, that is great but will receive no additional credit.

In this assignment, you need not be concerned with data validation. You may assume that the data entered by users is correctly formatted.

In general, if these requirements do not explicitly ask for a certain feature, then you are not required to provide that feature. No additional points are awarded for extra features.

If you use any code that is not yours, you are required to clearly cite the source - a full URL where appropriate.

## Task 1 Use the IntelliJ Project Name "Project2Task1".

In Task 1, you will make several modifications to EchoServerUDP.java and EchoClientUDP.java. Note that
these two programs are standard Java and we do not need to construct a web application in IntelliJ. Both of these programs may be placed in the same IntelliJ project.

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

Note how the server uses a DatagramPacket to receive data from a client (in the request object) and how it uses a DatagramPacket to send data back to the client (in the reply object). A DatagramPacket is always based on a byte array. So, to send a message in a DatagramPacket, we must first convert the message to a byte array. To receive a message from a DatagramPacket, we must convert the byte array to a message.

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
  		System.out.println("Reply: " + new String(reply.getData()));
    }

	}catch (SocketException e) {System.out.println("Socket: " + e.getMessage());
	}catch (IOException e){System.out.println("IO: " + e.getMessage());
	}finally {if(aSocket != null) aSocket.close();}
    }
}
```
0. Get these programs running in IntelliJ.
1. Change the client&#39;s &quot;arg[0]&quot; to a hardcoded &quot;localhost&quot;.
2. Document the client and the server. Describe what each line of code does.
3. Add a line at the top of the client so that it announces, by printing a message, &quot;The client is running.&quot; at start up.
4. Add a line at the top of the server so that it announces &quot;The server is running.&quot; at start up.
5. On the server, examine the length of the requestString and note that it is too large. Make modifications to the server code so that the request data is copied to an array with the correct number of bytes. Use this array of bytes to build a requestString of the correct size. Without these modifications, incorrect data may be displayed on the server. Upon each visit, your server will display the request arriving from the client.
6. If the client enters the command &quot;stop!&quot;, both the client and the server will halt execution. When the client program receives &quot;stop!&quot; from the user, it sends &quot;stop!&quot; to the sever and exits and does not wait for any reply.
7. Add a line in the client so that it announces when it is quitting. It will write "Client side quitting" to the client side console.
8. Add a line in the server so that it makes an announcement when it is quitting. The server only quits when it is told to do so by the client. It will write "Server side quitting" to the server side console.
9. Note, in the remaining tasks (Tasks 2 through 5), we do not provide the client with the ability to stop the server. In those tasks, the server is left running - forever.

Produce a screen shot illustrating a successful execution and submit the screenshot in the description folder as described at the end of this document. In the screenshot, use your name for the data that the client is reading from the keyboard and sending to the server. Also, show the client using the &quot;stop!&quot; option and show how the client and server respond.

Alternatively, you can create a screencast video of your working client and server.

- The video cannot be more than 3 minutes long.
- You may use an audio voiceover, but you do not need to.
- You should publish the video as &#39;Unlisted&#39; to YouTube. (See more discussion on this in the Submission section below.)
- Include the URL of the YouTube video in a document in the Project2Task1 Description folder that you submit.
- In the video, use your name for the data that the client is reading from the keyboard and sending to the server.

## Task 2 Use the IntelliJ Project Name "Project2Task2"

Make the following modifications to the original "EchoServerUDP.java" and "EchoClientUDP.java":

0. Name the client "AddingClientUDP.java". Name the server "AddingServerUDP.java".
1. The server will hold an integer value sum, initialized to 0, and will receive requests from the client - each of which includes an integer value (positive or negative or 0) to be added to the sum. Upon each request, the server will return the new sum as a response to the client. On the server side console, upon each visit by the client, the client's request and the new sum will be displayed.
2. Separate concerns on the client. On the client, all of the communication code will be placed in a method named &quot;add&quot;. In other words, the main method of the client will have no code related to interacting with the server. Instead, the main routine will simply call a local method named &quot;add&quot;. The &quot;add&quot; method will not perform any addition, instead, it will request that the server perform the addition. The &quot;add&quot; method will encapsulate or hide all communication with the server. It is within the &quot;add&quot; method where we actually work with sockets. This is a variation of what is called a &quot;proxy design&quot;. The &quot;add&quot; method is serving as a proxy for the server. When your code makes a call on the local "add" method, you are actually making a remote procedure call (RPC). The client side &quot;add&quot; method has the following signature:

```
public static int add(int i)

```
3. Separate concerns on the server. Your code that listens for a socket connection should be separate from the code that performs the add operation.

4. Write a client and server that has the following client side interaction with a user:

```
The client is running.
3
The server returned 3.
2
The server returned 5.
-1
The server returned 4.
6
The server returned 10.
stop!
Client side quitting.

If the client is restarted (note that the server is still running) we have:
The client is running.
1
The server returned 11.
stop!
Client side quitting.

```
Note: UDP messages are made up of byte arrays. You will need to take an int and place it into a four byte byte array before sending. When receiving, you will need to extract an int from the byte array. You may use code from external sources to help you do this. But be careful to site your sources with a clear URL.


Produce a screen shot illustrating a successful execution (client and server) and submit the screenshot in the description folder as described at the end of this document.

Alternatively, you can create a screencast video of your working client and server.

- The video cannot be more than 3 minutes long.
- You may use an audio voiceover, but you do not need to.
- You should publish the video as &#39;Unlisted&#39; to YouTube. (See more discussion on this in the Submission section below.)
- Include the URL of the YouTube video in a document in the Project2Task2 Description folder that you submit.

## Task 3 Use the IntelliJ Project Name "Project2Task3"

0. Name the client "RemoteVariableClientUDP.java". Name the server "RemoteVariableServerUDP.java".

1. Modify your work in Task 2 so that the client may request either an &quot;add&quot; or &quot;subtract&quot; or &quot;view&quot; operation be performed by the server. In addition, each request will pass along an integer ID. This ID is used to uniquely identify the user. Thus, the client will form a packet with the following values: ID, operation (add or subtract or view), and value (if the operation is other than view). The server will carry out the correct computation (add or subtract or view) using the sum associated with the ID found in each request. The client will be menu driven and will repeatedly ask the user for the user ID, operation, and value (if not a view request). When the operation is &quot;view&quot;, the value held on the server is simply returned. When the operation is &quot;add&quot; or &quot;subtract&quot; the server performs the operation and returns the sum. During execution, the client will display each returned value from the server to the user. If the server receives an ID that it has not seen before, that ID will initially be associated with a sum of 0.

2. On the server, you will need to map each ID to the value of a sum. Different ID&#39;s may be presented and each will have its own sum. The server is given no prior knowledge of what ID&#39;s will be transmitted to it by the client. You may only assume that ID&#39;s are positive integers. You are required to store the pair, the ID and its associated sum, in a Java TreeMap.

The client side menu will provide an option to exit the client. Exiting the client has no impact on the server. Here is an example client side interaction:

```
1. Add a value to your sum.
2. Subtract a value from your sum.
3. View your sum.
4. Exit client
1
Enter value to add:
5
Enter your ID:
33
The result is 5.

1. Add a value to your sum.
2. Subtract a value from your sum.
3. View your sum.
4. Exit client.
1
Enter value to add:
14
Enter your ID:
33
The result is 19.

1. Add a value to your sum.
2. Subtract a value from your sum.
3. View your sum.
4. Exit client.
1
Enter value to add:
10
Enter your ID:
32
The result is 10.

1. Add a value to your sum.
2. Subtract a value from your sum.
3. View your sum.
4. Exit client.
3
Enter your ID:
33
The result is 19.

1. Add a value to your sum.
2. Subtract a value from your sum.
3. View your sum.
4. Exit client.
4
Client side quitting. The remote variable server is still running.

```

3. As you did in Task 2, use a proxy design to encapsulate the communication code.

Produce a screen shot illustrating a successful execution and submit the screenshot in the description folder as described at the end of this document. Show three different clients interacting with the server using three distinct ID&#39;s.

Alternatively, you can create a screencast video of your working client and server.

- The video cannot be more than 3 minutes long.
- You may use an audio voiceover, but you do not need to.
- You should publish the video as &#39;Unlisted&#39; to YouTube. (See more discussion on this in the Submission section below.)
- Include the URL of the YouTube video in a document in the Project2Task3 Description folder that you submit.
- The video will show three different clients interacting with the server using three distinct ID&#39;s.

## Task 4 Use the IntelliJ Project Name "Project2Task4"


0. This is almost the same task as Task 3. The only difference is you will use TCP rather than UDP. Make the necessary modifications to EchoServerTCP.java and EchoClientTCP.java so that they behave the same way as does your solution to Task 3. Rename these files "RemoteVariableClientTCP.java" and "RemoteVariableServerTCP.java".

EchoServerTCP.java from Coulouris text

```
import java.net.*;
import java.io.*;
import java.util.Scanner;

public class EchoServerTCP {

    public static void main(String args[]) {
        Socket clientSocket = null;
        try {
            int serverPort = 7777; // the server port we are using

            // Create a new server socket
            ServerSocket listenSocket = new ServerSocket(serverPort);

            /*
             * Block waiting for a new connection request from a client.
             * When the request is received, "accept" it, and the rest
             * the tcp protocol handshake will then take place, making
             * the socket ready for reading and writing.
             */
            clientSocket = listenSocket.accept();
            // If we get here, then we are now connected to a client.

            // Set up "in" to read from the client socket
            Scanner in;
            in = new Scanner(clientSocket.getInputStream());

            // Set up "out" to write to the client socket
            PrintWriter out;
            out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(clientSocket.getOutputStream())));

            /*
             * Forever,
             *   read a line from the socket
             *   print it to the console
             *   echo it (i.e. write it) back to the client
             */
            while (true) {
                String data = in.nextLine();
                System.out.println("Echoing: " + data);
                out.println(data);
                out.flush();
            }

        // Handle exceptions
        } catch (IOException e) {
            System.out.println("IO Exception:" + e.getMessage());

        // If quitting (typically by you sending quit signal) clean up sockets
        } finally {
            try {
                if (clientSocket != null) {
                    clientSocket.close();
                }
            } catch (IOException e) {
                // ignore exception on close
            }
        }
    }
}
```

EchoClientTCP.java from Coulouris text

```
import java.net.*;
import java.io.*;

public class EchoClientTCP {

    public static void main(String args[]) {
        // arguments supply hostname
        Socket clientSocket = null;
        try {
            int serverPort = 7777;
            clientSocket = new Socket(args[0], serverPort);

            BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
            PrintWriter out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(clientSocket.getOutputStream())));


            BufferedReader typed = new BufferedReader(new InputStreamReader(System.in));
            String m;
            while ((m = typed.readLine()) != null) {
                out.println(m);
                out.flush();
                String data = in.readLine(); // read a line of data from the stream
                System.out.println("Received: " + data);
            }
        } catch (IOException e) {
            System.out.println("IO Exception:" + e.getMessage());
        } finally {
            try {
                if (clientSocket != null) {
                    clientSocket.close();
                }
            } catch (IOException e) {
                // ignore exception on close
            }
        }
    }
}
```

1. As in Task 3, be sure to use a **proxy design** to encapsulate the communication code. This requires a re-organization of the code but it is important to separate concerns.

Produce a screen shot illustrating a successful execution and submit the screenshot in the description folder as described at the end of this document. The screenshot will show three different clients interacting with the server using three distinct ID&#39;s.

Alternatively, you can create a screencast video of your working client and server.

- The video cannot be more than 3 minutes long.
- You may use an audio voiceover, but you do not need to.
- You should publish the video as &#39;Unlisted&#39; to YouTube. (See more discussion on this in the Submission section below.)
- Include the URL of the YouTube video in a document in the Project2Task4 Description folder that you submit.
- The video will show three different clients interacting with the server using three distinct ID&#39;s.

## Task 5 Use the IntelliJ Project Name "Project2Task5"

Before starting this task, study the three programs below. RSAExample.java shows how you can generate RSA keys in Java. ShortMessageSign.java and ShortMessageVerify.java shows you how you can sign and check the signature on very small messages.   

This Task is modeled after the way an Ethereum blockchain client signs requests.

Make the following modifications to your work in Task 4.

0. Rename these files "SigningClientTCP.java" and "VerifyingServerTCP.java".

1. As before, the client will be interactive and menu driven. It will transmit add or subtract or view requests to the server, along with the ID computed in 3 below, and provide an option to exit.

2. We want to send signed request from the client. Each time the client program runs, it will create new RSA public and private keys and **display** these keys to the user. See the RSAExample.java program below for guidance on how to build these keys. It is fine to use the code that you find in RSAExample.java (with citations, of course). After the client program creates and displays these keys, it interacts with the user and the server.
3. The client&#39;s ID will be formed by taking the least significant 20 bytes of the hash of the client&#39;s public key. Note: an RSA public key is the pair e and n. Prior to hashing, you will combine these two integers with concatenation. Unlike in Task 4, we are no longer prompting the user to enter the ID – the ID is computed in the client code. As in Bitcoin or Ethereum, the user's ID is derived from the public key.

4. The client will also transmit its public key with each request. Again, note that this key is a combination of e and n. These values will be transmitted in the clear and will be used by the server to verify the signature.

5. Finally, the client will sign each request. So, by using its private key (d and n), the client will encrypt the hash of the message it sends to the server. The signature will be added to each request. It is very important that the big integer created with the hash (before signing) is positive. RSA does not work with negative integers. See details in the code of ShortMessageSign.java and ShortMessageVerify.java below. You may use this code if cited.

6. The server will make two checks before servicing any client request. First, does the public key (included with each request) hash to the ID (also provided with each request)? Second, is the request properly signed? If both of these are true, the request is carried out on behalf of the client. The server will add, subtract or view. Otherwise, the server returns the message &quot;Error in request&quot;.
7. By studying ShortMessageVerify.java and ShortMessageSign.java you will know how to compute a signature. Your solution, however, will not use the short message approach as exemplified there. Note that we are not using any Java crypto API&#39;s that abstract away the details of signing.
8. We will use SHA-256 for our hash function h(). To clarify further:

The client will send the id: last20BytesOf(h(e+n)), the public key: e and n in the clear, the operation (add, view, or subtract), the operand, and the signature E(h(all prior tokens),d). The signature is thus an encrypted hash. It is encrypted
using d and n - the client&#39;s private key. E represents standard RSA encryption. The function h(e+n) is the SHA-256 hash of e concatenated with n.

During one client session, the ID will always be the same. If the client quits and restarts, it will have a new ID and operate on a new sum. The server is left running and survives client restarts.

As before, use a **proxy design** to encapsulate the communication code.

Produce a screen shot illustrating a successful execution and submit the screenshot in the description folder as described at the end of this document. The screen shot will show three different clients interacting with the server using three distinct ID&#39;s.

Alternatively, you can create a screencast video of your working client and server.

- The video cannot be more than 3 minutes long.
- You may use an audio voiceover, but you do not need to.
- You should publish the video as &#39;Unlisted&#39; to YouTube. (See more discussion on this in the Submission section below.)
- Include the URL of the YouTube video in a document in the Project2Task5 Description folder that you submit.
- The video will show three different clients interacting with the server using three distinct ID&#39;s.

RSAExample.java - Key generation and sample encryption and decryption

```
/* Demonstrate RSA in Java using BigIntegers */
​
import java.math.BigInteger;
import java.util.Random;
​
/**
 *  RSA Algorithm from CLR Algorithms text
 *
 * 1. Select at random two large prime numbers p and q.
 * 2. Compute n by the equation n = p * q.
 * 3. Compute phi(n)=  (p - 1) * ( q - 1)
 * 4. Select a small odd integer e that is relatively prime to phi(n).
 * 5. Compute d as the multiplicative inverse of e modulo phi(n). A theorem in
 *    number theory asserts that d exists and is uniquely defined.
 * 6. Publish the pair P = (e,n) as the RSA public key.
 * 7. Keep secret the pair S = (d,n) as the RSA secret key.
 * 8. To encrypt a message M compute C = M^e (mod n)
 * 9. To decrypt a message C compute M = C^d (mod n)
 */
​
public class RSAExample {
​
    public static void main(String[] args) {
        // Each public and private key consists of an exponent and a modulus
        BigInteger n; // n is the modulus for both the private and public keys
        BigInteger e; // e is the exponent of the public key
        BigInteger d; // d is the exponent of the private key
​
        Random rnd = new Random();
​
        // Step 1: Generate two large random primes.
        // We use 400 bits here, but best practice for security is 2048 bits.
        // Change 400 to 2048, recompile, and run the program again and you will
        // notice it takes much longer to do the math with that many bits.
        BigInteger p = new BigInteger(400,100,rnd);
        BigInteger q = new BigInteger(400,100,rnd);
​
        // Step 2: Compute n by the equation n = p * q.
        n = p.multiply(q);
​
        // Step 3: Compute phi(n) = (p-1) * (q-1)
        BigInteger phi = (p.subtract(BigInteger.ONE)).multiply(q.subtract(BigInteger.ONE));
​
        // Step 4: Select a small odd integer e that is relatively prime to phi(n).
        // By convention the prime 65537 is used as the public exponent.
        e = new BigInteger ("65537");
​
        // Step 5: Compute d as the multiplicative inverse of e modulo phi(n).
        d = e.modInverse(phi);
​
        System.out.println(" e = " + e);  // Step 6: (e,n) is the RSA public key
        System.out.println(" d = " + d);  // Step 7: (d,n) is the RSA private key
        System.out.println(" n = " + n);  // Modulus for both keys
​
        // Encode a simple message. For example the letter 'A' in UTF-8 is 65
        BigInteger m = new BigInteger("65");
​
        // Step 8: To encrypt a message M compute C = M^e (mod n)
        BigInteger c = m.modPow(e, n);
​
        // Step 9: To decrypt a message C compute M = C^d (mod n)
        BigInteger clear = c.modPow(d, n);
        System.out.println("Cypher text = " + c);
        System.out.println("Clear text = " + clear); // Should be "65"
​
        // Step 8 (reprise) Encrypt the string 'RSA is way cool.'
        String s = "RSA is way cool.";
        m = new BigInteger(s.getBytes()); // m is the original clear text
        c = m.modPow(e, n);     // Do the encryption, c is the cypher text
​
        // Step 9 (reprise) Decrypt...
        clear = c.modPow(d, n); // Decrypt, clear is the resulting clear text
        String clearStr = new String(clear.toByteArray());  // Decode to a string
​
        System.out.println("Cypher text = " + c);
        System.out.println("Clear text = " + clearStr);
    }
}

```

ShortMessageSign.java - Signing

```
import java.math.BigInteger;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Scanner;

/** ShortMessageSign.java provides capabilities to sign
 *  very short messages. These messages are 4 hex digits.
 *  ShortMessageSign has three private members: RSA e,d and n.
 *  These are all very small java BigIntegers. ShortMessageSign is
 *  only used for instructional purposes.
 *
 *  For signing: the ShortMessageSign object is constructed with RSA
 *  keys (e,d,n). These keys are not created here but are passed in by the caller.
 *  Then, a caller can sign a message - the string returned by the sign
 *  method is evidence that the signer has the associated private key.
 *  After a message is signed, the message and the string may be transmitted
 *  or stored.
 *  The signature is represented by a base 10 integer.
 */


public class ShortMessageSign {

    private BigInteger e,d,n;

    /** A ShortMessageSign object may be constructed with RSA's e, d, and n.
     *  The holder of the private key (the signer) would call this
     *  constructor. Only d and n are used for signing.
     */
    public ShortMessageSign(BigInteger e, BigInteger d, BigInteger n) {
        this.e = e;
        this.d = d;
        this.n = n;
    }

    /**
     * Signing proceeds as follows:
     * 1) Get the bytes from the string to be signed.
     * 2) Compute a SHA-1 digest of these bytes.
     * 3) Copy these bytes into a byte array that is one byte longer than needed.
     *    The resulting byte array has its extra byte set to zero. This is because
     *    RSA works only on positive numbers. The most significant byte (in the
     *    new byte array) is the 0'th byte. It must be set to zero.
     * 4) Create a BigInteger from the byte array.
     * 5) Encrypt the BigInteger with RSA d and n.
     * 6) Return to the caller a String representation of this BigInteger.
     * @param message a sting to be signed
     * @return a string representing a big integer - the encrypted hash.
     * @throws Exception
     */
    public String sign(String message) throws Exception {

        // compute the digest with SHA-256
        byte[] bytesOfMessage = message.getBytes("UTF-8");
        MessageDigest md = MessageDigest.getInstance("SHA-256");
        byte[] bigDigest = md.digest(bytesOfMessage);

        // we only want two bytes of the hash for ShortMessageSign
        // we add a 0 byte as the most significant byte to keep
        // the value to be signed non-negative.
        byte[] messageDigest = new byte[3];
        messageDigest[0] = 0;   // most significant set to 0
        messageDigest[1] = bigDigest[0]; // take a byte from SHA-256
        messageDigest[2] = bigDigest[1]; // take a byte from SHA-256

        // The message digest now has three bytes. Two from SHA-256
        // and one is 0.

        // From the digest, create a BigInteger
        BigInteger m = new BigInteger(messageDigest);

        // encrypt the digest with the private key
        BigInteger c = m.modPow(d, n);

        // return this as a big integer string
        return c.toString();
    }

    public static void main(String args[]) throws Exception {

        // Test driver for ShortMessageSign

        // Since we are signing only very short messages, we can generate some really small keys.
        // The keys were generated by the RSA algorithm
        // p and q were 20 bits each.
        // In practice, the keys would be larger.
        BigInteger e = new BigInteger("65537");
        BigInteger d = new BigInteger("5420920152787448033");
        BigInteger n = new BigInteger("9013594933187057813");

        ShortMessageSign sov = new ShortMessageSign(e,d,n);

        Scanner sc = new Scanner(System.in);

        // Get some data to sign
        System.out.println("Enter data to be signed (at most 4 hex digits)");
        System.out.println("All data will be converted to lower case");
        String inStr = sc.nextLine();
        if(inStr.length() != 4) {
            System.out.println("Error in input " + inStr);
            return;
        }
        inStr = inStr.toLowerCase();

        // add a 0 byte to keep it positive
        byte b[] = hexStringToByteArray("00"+inStr);

        // This messsage is the unsigned integer value of the
        // two bytes entered
        BigInteger m = new BigInteger(b);

        String signedVal = sov.sign(inStr);
        System.out.println("Signed Value");
        System.out.println(signedVal);


    }
    // From Stack overflow
    public static byte[] hexStringToByteArray(String s) {
        int len = s.length();
        byte[] data = new byte[len / 2];
        for (int i = 0; i < len; i += 2) {
            data[i / 2] = (byte) ((Character.digit(s.charAt(i), 16) << 4)
                    + Character.digit(s.charAt(i+1), 16));
        }
        return data;
    }
}

```
ShortMessageVerify - Signature verification

```
import java.math.BigInteger;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Scanner;

/** ShortMessageVerify.java provides capabilities to verify
 *  very short messages. These messages are 4 hex digits.
 *  ShortMessageVerify has two private members: RSA e and n.
 *  These are all very small java BigIntegers. ShortMessageVerify is
 *  only used for instructional purposes.
 *
 *
 *  For verification: the object is constructed with keys (e and n). The verify
 *  method is called with two parameters - the string to be checked and the
 *  evidence that this string was indeed manipulated by code with access to the
 *  private key d.
 *  The message that is signed or verified is 4 hex digits.
 *  The signature is represented by a base 10 integer.
 */


public class ShortMessageVerify {

    private BigInteger e,n;

    /** For verifying, a SignOrVerify object may be constructed
     *   with a RSA's e and n. Only e and n are used for signature verification.
     */
    public ShortMessageVerify(BigInteger e, BigInteger n) {
        this.e = e;
        this.n = n;
    }

    /**
     * Verifying proceeds as follows:
     * 1) Decrypt the encryptedHash to compute a decryptedHash
     * 2) Hash the messageToCheck using SHA-256 (be sure to handle
     *    the extra byte as described in the signing method.)
     * 3) If this new hash is equal to the decryptedHash, return true else false.
     *
     * @param messageToCheck  a normal string (4 hex digits) that needs to be verified.
     * @param encryptedHashStr integer string - possible evidence attesting to its origin.
     * @return true or false depending on whether the verification was a success
     * @throws Exception
     */
    public boolean verify(String messageToCheck, String encryptedHashStr)throws Exception  {

        // Take the encrypted string and make it a big integer
        BigInteger encryptedHash = new BigInteger(encryptedHashStr);
        // Decrypt it
        BigInteger decryptedHash = encryptedHash.modPow(e, n);

        // Get the bytes from messageToCheck
        byte[] bytesOfMessageToCheck = messageToCheck.getBytes("UTF-8");

        // compute the digest of the message with SHA-256
        MessageDigest md = MessageDigest.getInstance("SHA-256");

        byte[] messageToCheckDigest = md.digest(bytesOfMessageToCheck);

        // messageToCheckDigest is a full SHA-256 digest
        // take two bytes from SHA-256 and add a zero byte
        byte[] extraByte = new byte[3];
        extraByte[0] = 0;
        extraByte[1] = messageToCheckDigest[0];
        extraByte[2] = messageToCheckDigest[1];

        // Make it a big int
        BigInteger bigIntegerToCheck = new BigInteger(extraByte);

        // inform the client on how the two compare
        if(bigIntegerToCheck.compareTo(decryptedHash) == 0) {

            return true;
        }
        else {
            return false;
        }
    }


    public static void main(String args[]) throws Exception {

        // Test driver for ShortMessageVerify

        // ShortMessageVerify may use some really small keys
        // The keys were generated by the RSA algorithm
        // p and q were 20 bits each
        BigInteger e = new BigInteger("65537");

        BigInteger n = new BigInteger("9013594933187057813");

        ShortMessageVerify verifySig = new ShortMessageVerify(e,n);

        Scanner sc = new Scanner(System.in);

        // Check an existing signature

        System.out.println("Enter hash that was signed (4 hex digits)");
        System.out.println("All data will be converted to lower case");
        String data = sc.nextLine();
        if(data.length() != 4) {
            System.out.println("Invalid input");
            System.exit(0);
        }
        data = data.toLowerCase();
        System.out.println("Enter an integer representing the signature");
        String sig = sc.nextLine();
        if(verifySig.verify(data, sig)) {
            System.out.println("Valid signature");
        }
        else {
            System.out.println("invalid signature");
        }

    }
    // from Stack overflow
    public static byte[] hexStringToByteArray(String s) {
        int len = s.length();
        byte[] data = new byte[len / 2];
        for (int i = 0; i < len; i += 2) {
            data[i / 2] = (byte) ((Character.digit(s.charAt(i), 16) << 4)
                    + Character.digit(s.charAt(i+1), 16));
        }
        return data;
    }
}
```

## Summary &amp; Submission:

Be sure to review the General Grading Rubric on Canvas.

**Video sharing rights:** If you are creating screencast videos, then you should set the YouTube sharing rights &#39;Unlisted&#39; when publishing to YouTube. There are three types of sharing rights on YouTube: Public, Private and Unlisted. You do not want other students to be able to see your video (that would be cheating), and &#39;Unlisted&#39; restricts viewing to only those who have your URL.

Be sure you have named your IntelliJ project folders correctly.

For each IntelliJ project, File/Export Project to zip. You must export in this way and NOT just zip the IntelliJ project folders.

You should also have five description folders:

Project2Task1 Description

Project2Task2 Description

Project2Task3 Description

Project2Task4 Description

Project2Task5 Description

Each description folder contains either a single document (you may choose an appropriate name) with screenshots or a link to a video showing your work. (If you upload your video to YouTube, make sure your video is selected as &#39;unlisted&#39;.)

You should have five description folders:

Project2Task1 Description

Project2Task2 Description

Project2Task3 Description

Project2Task4 Description

Project2Task5 Description

You should also have five zip files:

Project2Task1.zip

Project2Task2.zip

Project2Task3.zip

Project2Task4.zip

Project2Task5.zip

Create a new empty folder named with your Andrew id - [your andrew id]Project2.zip. Mine would be mm6Project2.zip.  Put all files mentioned above into this new folder.

Zip that folder, and submit it to Canvas. The submission should be a single zip file.

Now you should have only one zip file named with your Andrew id.

Submission File Structure:

The file named **YourAndrewIDProject2.zip** contains:

##### Project2Task1.zip
##### Project2Task2.zip
##### Project2Task3.zip
##### Project2Task4.zip
##### Project2Task5.zip
##### Project2Task1 Description (folder)
##### Project2Task2 Description (folder)
##### Project2Task3 Description (folder)
##### Project2Task4 Description (folder)
##### Project2Task5 Description (folder)
