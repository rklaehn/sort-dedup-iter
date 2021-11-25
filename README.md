# Problem

You have an iterator, and you want to sort and deduplicate its elements into a [Vec](https://doc.rust-lang.org/std/vec/struct.Vec.html) or [SmallVec](https://docs.rs/smallvec/1.7.0/smallvec/).

## Naive approach 1

```rust
fn sort_and_dedup<I: Iterator>(iter: I) -> Vec<I::Item>
    where I::Item: Ord
{
    let mut res = iter.collect::<Vec<_>>();
    res.sort();
    res.dedup();
    res
}
```

The downside of this approach is that it will create a very large intermediate collection, before 
removing duplicates. This is usually not a big issue, but for e.g. an iterator with a lot of 
duplicates, this might even run out of memory.

```ignore
sort_and_dedup((0..1_000_000_000u64).map(|i| i % 10)) // maybe OOM
```

## Naive approach 2

```rust
# use std::collections::BTreeSet;
fn sort_and_dedup<I: Iterator>(iter: I) -> Vec<I::Item>
    where I::Item: Ord
{
    let res = iter.collect::<BTreeSet<_>>();    
    res.into_iter().collect()
}
```

The downside of this approach is that it creates an expensive intermediate collection. E.g. for an
iterator that contains already sorted elements, this is very expensive as well as unnecessary

```ignore
sort_and_dedup((0..1_000_000_000u64)) // unnecessarily creates a large BTreeSet
```

## This crate

This crate uses heuristics to sort and deduplicate so that the intermediate vec is never more than 2 
times as large as the result. It also has some optimizations to make sure already sorted collections
are handled efficiently.

Nevertheless, in most cases you are probably best off using one of the 2 naive approaches described 
above.
