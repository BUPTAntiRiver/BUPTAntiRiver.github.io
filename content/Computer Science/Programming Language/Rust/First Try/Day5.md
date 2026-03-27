# Methods

Methods are similar to functions, we declare them with the `fn` keyword and a name, they can have parameters and a return value. But unlike functions, methods are defined within the context of a struct (or an enum or a trait object, which will be covered in the future), and their first parameter is always `self` (Fuck! I know you are Python! I know it!), which represents the instance of the struct the method is being called on.

The syntax looks like this:

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```
