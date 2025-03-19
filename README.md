# Advanced Programming Module 6 Reflections

## Module 06 - Concurrency

### Commit 1 Reflection

##### What is inside the handle_connection method?

The code `main.rs` sets up a basic single-threaded TCP web server. This web server uses a `TcpListener` to listen for browser requests at the address it's bound to `127.0.0.1` and port `7878`.

The following modules for networking `net::{TcpListener, TcpStream}` and `io::{prelude::*, BufReader}` for handling inputs and outputs are imported.

The `main` function first creates a TCP listener and `unwrap()` will stop the program from executing if there's an error.
Next, a loop that can handle and iterates through incoming connection requests is entered. At each iteration `unwrap()` is called in case of any errors.
If the connection is successful, `unwrap()` will extract data of the TCP stream from the `Result<T, E>`, a TCP stream is created and the handle_connection function is called to process it.

The `handle_connection` function uses `BufReader` to read each line of the stream data received. 
The data is then processed as it uses a map and `unwrap()` to collect the data which is put into a vector, moreover `take_while` stops the processing if an empty line is encountered.
Finally, using `println!`, the function prints out details of the HTTP request.


### Commit 2 Reflection

Besides reading HTTP requests from the client, the new `handle_connection` function can now format and send HTTP responses, create HTTP headers and read HTML files. 
That's why the contents of `hello.html` can be displayed on `http://127.0.0.1:7878/`.

The module `fs` is imported to enable reading of HTML files.
The function still takes `TcpStreams` as a parameter. The stream gets wrapped in a BufReader so its contents can be read line by line.
If there are no errors then the function sets the status of the HTTP request as `HTTP/1.1 200 OK` and reads the contents of `hello.html` with `unwrap()` for error handling.

`Content-Length` defines the length of the file content which is the HTTP response header.
This line creates and formats the HTTP response `let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");`.
Resulting in a HTTP response with the status line from above, header as content length and body as contents of `hello.html`.

Finally, the HTTP response is written and sent to the client. 
![Reflection 2 Image](hello/images/Reflection2.jpg)

### Commit 3 Reflection

I had to change the handle_connection function so that it can display different html files based on the request line from the client.
I created a new HTML file called `404.html` which will be displayed when `http://127.0.0.1:7878/bad` is accessed.

#### How to split the responses:
The `handle_connection` function now finds and directly reads the request line in the TCP stream which is assigned to `request_line`.
I used an IF statement that checks what is the the request line from the client. If the request line is `"GET / HTTP/1.1"` then `hello.html` page is displayed and the status line `"HTTP/1.1 200 OK"` is sent in the response. Else, then `404.html` page is displayed and the status line `"HTTP/1.1 404 NOT FOUND"` is sent in the response to the client.  

The rest of the code follows the same process as the previous commit.

#### Why do refactoring?
There are a lot of lines in the code that can be simplified for better readability and maintainability. For example, instead of assigning the file to be read in a different line, it can be associated with its respective status line and request line.
As shown in the IF statement, `"GET / HTTP/1.1"` is associated with `hello.html` and `"HTTP/1.1 200 OK"`. While `404.html` is associated with other request lines and the status `"HTTP/1.1 404 NOT FOUND"`.
![Reflection 3 Image](hello/images/Reflection3.jpg)


### Commit 4 Reflection
The modules `thread` and `time::Duration` are imported for creating this line: `thread::sleep(Duration::from_secs(10));`.
`thread` is for creating a single thread where the web server function executions are paused or on sleep for 10 seconds.
`time::Duration` determines the time duration for a certain method, in this case it's sleep.
When `http://127.0.0.1:7878/sleep` is accessed the web server will stop serving other requests during the 10 second delay before displaying `hello.html` and `"HTTP/1.1 200 OK"`.
Since there's only a single thread when sleep is in the path, the web server can only serve one user at a time.
The rest of the code functions the same as before in the previous previous commit.


### Commit 5 Reflection
A thread pool is a group of more than one spawned threads that wait for a task to process on its own.
Implementing a thread pool lets the web server be able to handle more than one request simultaneously, increasing latency and throughput.
I gave the web server fixed finite number of threads so that the web server resources won't be overwhelmed when it receives many requests, additionally DoS attacks can't happen.

To implement a thread pool, I first edited `main.rs` and added a FOR loop for spawning a threads if there are more than one incoming requests. I created a new file `lib.rs` that contains the thread pool. There's also a worker struct that will delegate code that has to be run and then executes it in its thread.


### Bonus Commit Reflection
The new build feature for creating new thread pools in `main.rs` has error handling regarding the thread pools.
In order to do that, I had to make changes to `main.rs` and `lib.rs`.
In the main function in `main.rs`, I used `build()` instead of `new` for creating new thread pools.
Now if the thread count is invalid then it will output the error message: `" "Unable to create ThreadPool, make sure thread count are positive nonzero integers!".`. Moreover, the new `PoolCreationError` type contains information on what caused the error. Debugging the program is now faster because of this.
Before implementing error handling, the program will panic and stop executing. Now errors are handled more gracefully and are logged.