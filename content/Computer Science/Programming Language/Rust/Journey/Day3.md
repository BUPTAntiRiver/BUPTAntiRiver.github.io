# Control Flow

Keep reading **the book**. So actually in rust we do have simple and basic control flow keywords like `if`, `loop`, `while` and `for`. So why in the guess game we have to use a redundant enum? Maybe just to terrify you. So let's try to rewrite the guess game with the common way first.

# Ownership

It says that ownership is the most unique feature of rust, let's see what is its difference comparing ownership in C++.

So it is said that there are different methods to manage memory: like garbage collector that regularly looks for no-longer-used memory as the program runs; or like in C, we explicitly allocate and free the memory. Rust decides to solve such problem in compile time, **if the ownership system rules is violated**, the program **won't compile**.
