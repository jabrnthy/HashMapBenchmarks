### Hashmap investigation

Came across something that was a bit surprising, and I'm going to spend a bit of time running it down to see where it goes.  I had a C version of a chained hashmap lying around.  From the bad, old, pre-C++ days.  And I thought that I should finally junk this thing and replace it with standard C++14 constructs.  It'll be standard, and it's probably faster because it'll be better performance tuned.  And, additionally, a particulary application of the hashmap is basically equivalent to unordered_set, which makes me think it ought to be faster because it'll use less memory and be more cache friendly, eh?

So I ran tests with the [nonius microbenchmarking suite](https://github.com/libnonius/nonius) and...um...not so much.  At least not under Visual Studio 2017 (more results to come soon).  In these cases, I'm careful to ensure that the same hash function and equal functions are used.  For the purposes of this experiment, I'm not particularly interested in optimizing the hash function, and I'm assuming the hash function doesn't advantage one data structure over the other (given that both hashmap implementations use chaining to resolve collisions, this seems likely).

Results were garnered by compiling the release mode binaries and then doing:

````
Release/HashMapBenchmarks --reporter=html --output=../my_file_name.html
````

Explanation of what I'm benchmarking and why.  Basically, the I have a `CHashMap` and an `std::unordered_set` whose purpose is to hold unique strings.  So the string for the `CHashMap` is both the key and the value.  If the string, represented as a `const char*`, is determined not to be in the hashmap, then a copy is made (`malloc`, `strcpy`), and then inserted.  The details are exactly the same in both cases, except for the hashmap data structures and the finding/inserting methods.  Nonius wants to rerun its benchmarks if the function your benchmarking doesn't take enough time to be accurately timed by the computer.  This is no good for me as I expect the behavior to change as I add things to the hashmap, and I want reproducibility.  So each benchmark is adding 5,000 randomly generated strings.  nonius, by default, then runs the benchmark 100 times.  The hashmaps are never discarded, so each run continues to grow the hashmap 5,000 elements at a time.

One optimization in the `CHashMap` side which is certainly was contributing to better insertion performance is the fact that I can determine whether or not the item exists in the hashmap, and if so, make a copy of it and add it unconditionally with no further checks.  I can't think of a way to do this (at least not for this kind of copying) without doing a *second* find in the `unordered_set` case because `unordered_set::insert()` cannot be performed unconditionally...it will always do equality checks.  So, to try to make a more apples-to-apples comparison, I've set up the benchmark to not do the `find` operation when the argument passed in is expected to be new.  This doesn't reflect my need, but it makes reasoning about the differeing timings a bit easier.

To examine the nonius output, I don't recommend spending too much time on the summary view.  The hashmap growth causes too many outliers for that statistical analysis to be useful.  Instead, I recommend switching the dropdown to "samples" and use click-drag to drag out a rectangle to zoom in on to the data (double-click to restore to a full view of the data).

Easy links to open the benchmark data via [htmlpreview.github.com](https://htmlpreview.github.com):

* [VisualStudio2017Release.html](https://htmlpreview.github.io/?https://github.com/jfultz/HashMapBenchmarks/blob/master/VisualStudio2017Release.html)
* [XcodeClang8.1Release.html](https://htmlpreview.github.io/?https://github.com/jfultz/HashMapBenchmarks/blob/master/XcodeClang8.1Release.html)
