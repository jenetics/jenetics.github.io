---
layout: post
title:  "Creating random seeds with Java"
description: "How to create better seeds for PRNGs"
date:   2017-07-10 21:00:20 +0200
categories: java prng random-seed
---


> Random number generators should not be chosen at random. - _Donald Knuth_

![Dandelions](/assets/dandelions.jpg)

While implementing the _Jenetics_ library, I faced the problem of creating proper seed values for the `Random` engines I used. The usual way for doing this, is to take the current time stamp.

```java
public static long seed() {
    return System.nanoTime();
}
```

To get a feeling about the quality for this kind of seed values, I treated it as special random engine and applied the dieharder test suite to it. As you might expect, the quality of this random engine was disastrous. It didn't pass a single [dieharder](http://www.phy.duke.edu/%7Ergb/General/dieharder.php) test. All of the 114 tests failed.

Since this result was not really satisfying, I was searching for a different entropy source. For Linux system you probably want to choose `/dev/random` or `/dev/urandom` as source for your seeds. But this approach is not portable, which was a prerequisite for the _Jenetics_ library. In my second approach I was playing around with the value of the `Object.hashCode()` function. To get a 64 bit seed value, I concatenated the hashes of two newly created Java objects:

```java
public static long seed() { 
    return ((long)(new Object().hashCode()) << 32) | 
        new Object().hashCode(); 
}
```

This time, the _dieharder_ returned 28 passed and 86 failed tests. Looks better than the timestamp seed, but 86 failing tests are still not very satisfying. After additional experimentation, I tried to combine the timestamp and the object seeding. The rational behind this was, that the seed shouldn’t rely on a single entropy source.

```java
public static long seed(final long base) {
    return mix(base, objectHashSeed());
}

private static long mix(final long a, final long b) {
    long c = a^b;
    c ^= c << 17;
    c ^= c >>> 31;
    c ^= c << 8;
    return c;
}

private static long objectHashSeed() {
    return (long)new Object().hashCode() << 32 | 
        new Object().hashCode();
}
```

The code above shows how the timestamp is mixed with the object seed. The `mix` method was inspired by the mixing step of the [lcg64_shift](https://github.com/rabauke/trng4/blob/master/trng/lcg64_shift.hpp) random engine, which has been re-implemented in the [LCG64ShiftRandom](https://github.com/jenetics/prngine/blob/master/prngine/src/main/java/io/jenetics/prngine/LCG64ShiftRandom.java) class. Testing this `seed` method leads to the following result: 112 passed, 2 weak and 0 failed tests.

**Open questions**

* How does this method perform on operating systems other than Linux and
* how does this method perform on JMVs other then Java 8.

**Conclusion**

The statistical performance of the combined seed is better, according to the dieharder test suite, than most of the real random engines I tested, including the default Java [Random](https://docs.oracle.com/javase/8/docs/api/java/util/Random.html) engine. Using the proposed `seed()` method is in any case preferable to the simple `System.nanoTime()` call.

**References**

* I used the [DieHarder](https://github.com/jenetics/prngine/blob/master/prngine/src/main/java/io/jenetics/prngine/internal/DieHarder.java) wrapper class for executing the dieharder tests and for creating the test reports.
* The following links contain the full dieharder test result: [timestamp seed](https://gist.github.com/jenetics/53b4fc805407eed6db54745ec0d0803c), [object hash seed](https://gist.github.com/jenetics/97dcd2536c18527b810c1088d7f89607) and [combined seed](https://gist.github.com/jenetics/b33d8cc0d77d17e84e7636c8c554086b).
