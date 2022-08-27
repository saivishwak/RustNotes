# The Async and Await paradime in Rust

Lets consider the below code and go thrugh each step

```rust
use ::std::future::Future;

fn foo1() -> impl Future<Output = ()> {
    async {
        println!("Foo1");
        tokio::time::sleep(tokio::time::Duration::from_millis(10)).await;
    }
}

async fn foo2() {
    println!("foo2");
    foo1().await;
    println!("Foo2");
}

async fn foo3(a: i32) {
    println!("Foo3");
}

#[tokio::main]
async fn main() {
    let b = foo2();
    let a = tokio::task::spawn(b);
    //foo2().await;
    foo3(10).await;
    println!("Hello, world!");
    let _ = tokio::join!(a);
}
```

When we run this code the foo2() gets sent to a ```rust tokio::task::spawn()``` method.

The foo2() runs concurrently or even can be run on a different thread, its based on the tokio runtime configuration provided.

Now the main method run's the foo3() and awaits for the result. Once the result is retrieved then the foo2() can also be run or the print
statement can also be run. It depends on the runtime execution.

But the catch is if the foo2() is not completed before the print ends the main function goes out of scope and the foo2() Future gets
dropped so the second "foo2" print does not happen.

This is the case if we don't add the ```rust tokio::join!``` macro and pass the joinHandler that handles the foo2().

One interesting point here is, I cant directly use the variable a in the join! macr0, because the ownership is moved in the spawn and we cannot
borrow it.

Now, when the join! runs on the spawn handler, it awaits for the future to completion, hence we can see the last "foo2" print as well.

You can comment out the join! macro line and run the code, you won't see that extra "foo2" print I am talking about.

In this example you can notice that we use ```rust tokio::time::sleep``` method. If instead we used the std::thread::sleep. it would just be
a normal operation. We need to use the implementaions that are asynchronous for IO tasks, these are provided by tokio.

The reason for this is, when we use the tokio implementaion then we are actually telling the executor that this Future is not completed and is yeilded.
This is important as the executor will then awaits the other Futures in the queue.


## Resources
- https://rust-lang.github.io/async-book/01_getting_started/01_chapter.html
- https://www.youtube.com/watch?v=ThjvMReOXYM&t=1139s
- https://blog.logrocket.com/a-minimal-web-service-in-rust-using-hyper/
- https://www.viget.com/articles/understanding-futures-in-rust-part-1/
