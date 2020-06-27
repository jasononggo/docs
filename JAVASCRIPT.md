### For Loop

```
// if you did not add the update param directly, it will only loop once.
for (first = 0; first < 0; ) {
    for (second = 0; second < 0; second++) {
        // update param for the `first` for statement here, if you intend to leverage the loop (first -> second -> second -> ... -> first -> second ...)
        first++;
    }
    console.log(first) // it will only loop once.
}

// console.log(first) -> console.log(second) -> console.log(second) -> repeat
for (first = 10; first < 20; first++) {
    console.log(first)
    for (second = 0; second < 2; second++) {
        console.log(second)
    }
}
```
