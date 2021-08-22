Rust supports cooperative multitasking in the form of async/await.
Before i expanin what  async/await is and how it works, lets see how futures and asynchronous programming works in Rust.

Futures:
A future represents a value that might not be available yet. 
This could be for example an integer that is computed by another task or a file that is downloaded from the network. 
Instead of waiting until the value is available, futures enable the program routine for execution until the value is needed.

Lets say we have a main function that reads a file from the file system and then calls a function foo. 
With the synchronous call, the main function needs to wait until the file is loaded from the file system. 
Only then it can call the foo function, which requires it to again wait for the result.

Using the asynchronous async_read_file routine though, the file system directly returns a future and loads the file
asynchronously in the background. This allows the main function to call foo much earlier, which then runs in parallel
with the file load. 

In conclusion, the file load even finishes before foo returns, so main can directly work with the file without further waiting
after foo returns.


Code Example for concurrent processing with threading:
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for _ in 1..10 {
            println!("Hello after 1 second from the spawned thread");
            thread::sleep(Duration::from_millis(1000));
        }
    });

    for _ in 1..5 {
        println!("Hello after 1 second from the main thread");
        thread::sleep(Duration::from_millis(1000));
    }
}

We spawn a new thread using the standard library’s spawn function.
The spawn function take a closure as its argument and executes it in parallel.
As can be observed, by running the program, it only takes five seconds to print all ten statements because
each thread is sleeping independently.



Async Code Example:


[dependencies]
futures = "0.3.5"
async-std = "1.5.0"
use std::time::Duration;
use futures::executor::block_on;
use async_std::task;

fn main() {
    let future = async_main();
    block_on(future);
}

async fn async_main() {
    print_for_five("await 1").await;

    let async_one = print_for_five("async 1");
    let async_two = print_for_five("async 2");

    futures::join!(async_one, async_two);
}

async fn print_for_five(msg: &str) {
    for _ in 0..5 {
        task::sleep(Duration::from_secs(1)).await;
        println!("one second has passed: {}", msg)
    }
}


The routine starts by creating a new async function, async_main, then use the block_on executor to block and execute async_main. Because async_main is an asynchronous function, we are able to await other async functions inside of it, as well as execute them concurrently.
The first call in async_main is an await on our async print_for_five function which prints the message “one second has passed: await 1” once each second for 5 seconds. Because we used the await keyword, async_main will block and wait for print_for_five.
Next, we create two futures (very similar to JavaScript promises) by calling print_for_five anew with new messages. The futures do not begin execution until the next line where we use the join macro. Join executes the futures concurrently and blocks until all the futures have completed. Join is very similar to JavaScript’s PromiseAll.
The program will print the following:
one second has passed: await 1
one second has passed: await 1
one second has passed: await 1
one second has passed: await 1
one second has passed: await 1
one second has passed: async 1
one second has passed: async 2
one second has passed: async 1
one second has passed: async 2
one second has passed: async 1
one second has passed: async 2
one second has passed: async 1
one second has passed: async 2
one second has passed: async 1
one second has passed: async 2

Where the last ten lines are all printed within five seconds of each other because print_for_five("async 1") 
and print_for_five("async 2") were executed concurrently.

The block_on executor that we used blocks the main thread, which means that all the concurrency happened on a single thread.


