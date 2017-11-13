# README file

This is **Wei Le** Readme file.

## Compile and Run

- Compile

	// Go to test directory and compile files
	
		make

	// Go to src directory and compile files
	
		make

- Run

	// run the store server waiting for requests from client
	
	// an example for port number is 8803
	
	./src/store vendor_addresses.txt $PORTNUMBER $NUMBEROFTHREADS
	
	// run the vendor server waiting for requests from store
	
	./test/run_vendors ../src/vendor_addresses.txt
	
	// run the client side to send requests to store server
	
	// server_addr should include both FQDN and port like 0.0.0.0:8803
	
	./test/run_tests $SERVER_ADDR $NUMBEROFTHREADS

## Project Brief Description

This project is to implement store server using GRPC. The store server should be responsible for the following works:


1. Establish asynchronous GRPC communication to clients

2. Establish asynchronous GRPC communication to vendors

3. Create a thread pool and use it. When receiving a client request, the store server will assign a 

   thread from the thread pool to the incoming request for processing 

4. When responses from vendors come back, the store will collate the results and reply to the store client with the results of the call

## Source Code Description

- __store.cc__

	- In the main function, get the number of thread, port number of store server, and address file from the comment line arguments, and get the address list of vendors from the address file (vendor_addresses.txt).
	- Instantiate a ThreadPool object, so when receiving a client request, the store will assign a thread from the thread pool to process the incoming request.
	- In this file, I also define a class StoreService, which has a primary method ThreadTask() processing the requests from multiple clients asynchronously. 
	- The ThreadTask() method is to register service for store server, create completion queue for asynchronous processing, and spawn a new CallData instance to serve new client. The new CallData instance doesn't start to process the request until it reads the next event from the completion queue.
	- The created CallData instance processes the request by invoking Proceed(), which firstly requests the system to start processing the request, during the processing, spawning another new CallData instance to serve new clients to achieve asynchronous processing, awaits for results from vendor to come back, collates the results, and then replies to the client.
	- After the class StoreService is defined, the main function initiates a StoreService object and invokes the ThreadTask().


- __threadpool.h__

	- This file defines a class ThreadPool, in which the constructor is to assign a certain number of threads, each of which executes the task in the tasks queue.
	- The only public method provided to store server to get responses from vendors is processVendorQuery(), which adds task (queryBid()) to the tasks queue and waits for the results for each request to come back from running thread using get_future() API provided by the library.
	- The queryBid() method, which is private, does two things: one is to send a client query to all vendors in an asynchronous manner, the other is to listen for completed responses from vendors.

- __vendorclient.h__

	- This file defines a class VendorClient for class ThreadPool to make query to vendors for the client request by the store server.
	- Two methods are included in this class: one is to assemble the client's payload and send it to the vendor servers, the other is to listen for completed responses and return results.
	
## Additional observations that you have about what you've done

- When trying to compile codes for grpc and protobuf in grpc folder (step 13 for getting the dependencies in project README), even though both grpc and protobuf have been installed successfully, there is an error: fail to create symbolic link
  Reason: the downloaded codes for grpc and protobuf are saved in a folder in Ubuntu which shares with the Windows host.
  Solution: download the codes to a directory which doesn't shared with the Windows host.

## References

__https://github.com/grpc/grpc/blob/master/examples/cpp/helloworld/greeter_async_server.cc__
__https://github.com/grpc/grpc/blob/master/examples/cpp/helloworld/greeter_async_client2.cc__
__https://github.com/progschj/ThreadPool/blob/master/ThreadPool.h__
