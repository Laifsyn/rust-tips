
# Annotating #[track_caller] to functions

<!-- Hellow to my first attempt to tagging. They should be on singular form, no `panics`, `errors`, `attributes` etc. unless the name ends in `s` for whatever reason.-->
#error #panic #attribute #debug #date:2025-04
<!-- The "quantum" for date is until months because it felt like a nice time frame. Not so big, nor too small -->
## Situation

You have a `methods/functions` which calls some other piece of code that could possibly panic. Either because the said function is 

- Using `unwrap` or `expect` on a `Result` or `Option`, 
- The function itself uses a panic-related macro.
- The function calls a function that panics.

## Possible Symptom(s)

- You have a function that panics, however it isn't returning a useful source-code location.



## Suggested Solution
Use the `#[track_caller]` attribute to annotate the function that panics.

For library authors and any Public-API function, a solution/example could look like this:

```rust
use std::borrow::Cow;

// We annotate #[track_caller] because either:
// 1. We are unwrapping a result in place of the user
// 2. We are calling a function that's annotated with #[track_caller].
// #[track_caller] we can interpret as a hint to a potentially panicking function.
#[track_caller]
pub fn public_fn(a: usize, b: usize) -> usize {
    match protected_fn(a, b) {
        Ok(result) => result,
        Err(e) => panic!("{e}"),
    }
}

// Mistake 1: Not to annotate #[track_caller].
// Why? because if `incorrect_private_fn()` panics, panic message
// won't show the actual wrong source line.
pub(crate) fn incorrect_protected_fn_1(a: usize, b: usize) -> usize {
    incorrect_private_fn(a, b)
}

// Mistake 2:
// - Not to return a Result: User won't be able to handle the error
// - Not annotating #[track_caller]: User likely won't know where's the
//   actual offending line when panic occurs.
pub(crate) fn incorrect_protected_fn_2(a: usize, b: usize) -> usize {
    private_fn(a, b).expect("No division by zero")
}

pub(crate) fn protected_fn(
    a: usize,
    b: usize,
) -> Result<usize, Cow<'static, str>> {
    private_fn(a, b)
}

#[track_caller] // We always annotate any function that panics
fn incorrect_private_fn(a: usize, b: usize) -> usize {
    if b == 0 {
        panic!("Division by zero");
    }
    a / b
}

// No Annotation needed because this function never panics.
fn private_fn(a: usize, b: usize) -> Result<usize, Cow<'static, str>> {
    if b == 0 {
        return Err(Cow::Borrowed("Division by zero"));
    }
    Ok(a / b)
}

```

## Afterwords

The main advantage of being smarter with #[track_caller] is
1. You can get a more accurate/useful source code location when a panic occurs.
2. You don't taint the protected/private API with #[track_caller], potentially returning unintended source code location when debugging, or slightly downgrading performance. But I'm just speculating on this last one.

You don't need to unwrap inside the public function in place of the user, however the public API might not be as user-friendly depending on how fast you want panic when the user mis-call the function.





