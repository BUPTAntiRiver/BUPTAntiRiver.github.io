# The `match` Control Flow

In my opinion `match` acts like `switch` in C. Actually, the key different between `match` and other traditional control flow methods are those methods handles a boolean condition value, but `match` can be performed on any type.

The use case on previous `Option<T>` is like this:

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
	match x {
		None => None,
		Some(i) => Some(i + 1),
	}
}
let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

The good point of `match` is that it ensures _exhaustive_. Which means all the cases of a enum must be covered inside `match`, or we will have compile error.

But if we have to be exhaustive, how to handle cases other than enum? Like using `match` on integers? Don't worry, we have `other` to represent a catch-all pattern, which is similar to `default`.

```rust
let dice_roll = 9;
match dice_roll {
	3 => add_fancy_hat(),
	7 => remove_fancy_hat(),
	other => move_player(other),
}
```

Also there might be cases we don't want to use the `other` value, we can use `_` instead. It is also a catch-all pattern but it is more like a placeholder, so that compiler won't warn us about unused variable.

# Some more concise usage

Use the most popular `Option<T>` as example, if we only wants to deal with a `Option` when it is `Some`, we will need to write the full `match` arms, and that will be redundant:

```rust
let config_max = Some(3u8);
match config_max {
	Some(max) => println!("The maximum is configured to be {max}"),
	_ => (),
}
```

We can rewrite in a more concise way:

```rust
let config_max = Some(3u8);
if let Some(max) = config_max {
	println!("The maximum is configured to be {max}");
}
```

I have to say, this is just like the modern `optional` syntax in C++:

```cpp
optional<int> getValue() {
	return 42;
}

if (auto var = getValue()) {
	cout << *var << endl;
}
```

And in my work experience, we might write code like:

```cpp
if (auto var = other_var.As(type_a)) {
	// do stuff with var
} else if (auto var = other_var.As(type_b)) {
	// do stuff with var
}
```

To behave differently when `other_var` has different type. But when the code to execute inside the `if let` statement becomes too long or the code would become awful. So we use harness the feature that expressions produces a value to extract the value we want and operate with it.

```rust
impl UsState {
    fn existed_in(&self, year: u16) -> bool {
        match self {
            UsState::Alabama => year >= 1819,
            UsState::Alaska => year >= 1959,
            // -- snip --
        }
    }
}

// code inside if let body
fn describe_state_quarter(coin: Coin) -> Option<String> {
    if let Coin::Quarter(state) = coin {
        if state.existed_in(1900) {
            Some(format!("{state:?} is pretty old, for America!"))
        } else {
            Some(format!("{state:?} is relatively new."))
        }
    } else {
        None
    }
}

//use state to represent the value
fn describe_state_quarter(coin: Coin) -> Option<String> {
    let state = if let Coin::Quarter(state) = coin {
        state
    } else {
        return None;
    };

    if state.existed_in(1900) {
        Some(format!("{state:?} is pretty old, for America!"))
    } else {
        Some(format!("{state:?} is relatively new."))
    }
}
```
