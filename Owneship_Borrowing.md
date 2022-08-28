# Ownership and Borrowing

The core functionality of safe rust is implemented on top of ownership and borrowing model.

### Usecase 1

```rust
fn main(){
    let a = String::from("Hello World!");
    let b = a;
    println!("{} {}",a,b);
}
```
The above code snippit will give you a compiler error saying borrow of moved value. The reason is, the value of a is atually the pointer to the string
heap data, len and capacity. So when we say ```rust let b = a;``` we are copying the pointer data, len, cap of the heap data to the b var;
We can think of this kind of like a shallow_copy. So now when the main function foes out of scope the variables a and b will also
go out of scope and double free error occurs. After a has freed the memory at that pointer space, now b will also try to remove it causing
issues. Hence, rust has a check at this level. We can get by this by using clone() method which actually copies the heap data as well.

In Rust, automatic copy is always shallow_copy by design.

### Usecase 2

```rust
fn take_and_give_onwership(s: String) -> String {
    s
}

fn take_onwership(mut s: String) {
    s.push_str("owner");
    println!("{}", s);
}

fn move_ownership(s: &String) -> String {
    let b = *s;
    b
}

fn main() {
    let a: String = String::from("Hello");
    let b = a;

    let c = take_and_give_onwership(b);

    take_onwership(c);

    println!("{}", c);

    let d: String = String::from("Hello");
}
```
One interesting thing to consider in the above code snippit is that, first we create a heap and store the pointer in a; then we moved a to b.
Now owns the value and a is not valid anymore from here. Then we move the b to the take_and_give_onwership() method which return the onwership
back to the variable c. Now from here b variable won't work and then we just move the onwership of c to the take_onwership() method. So below
this point c also won't be valid. All this is to ensure double free is not happening.

If we see he move_ownership() method. We are borrowing the string, we don't own it. But here we are trying to move the string to variable b;
This is not acceptable as we don't own the string we just borrowed it. To fix this you should remove the move or use copy() trait which does a
depp_copy.

> Note: 

    For the stack memory we don't really need to worry about the ownership and borowing. Say,

    ```rust
    fn main(){
        let a:i32 = 10;
        let b = a;
        println!("{} {}", a, b);
    }
    ```
    This code snippit works fine because the variables are on the stack memory as we know their size at comile time.


Important Topics:

- Mutable References
- Dangling References
