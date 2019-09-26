---
layout: post
title:  "Symbolic regression with Jenetics"
date:   2019-09-26 21:00:20 +0200
categories: jenetics genetic-programming machine-learning
---

Jenetics is an advanced Genetic Algorithm, Evolutionary Algorithm and Genetic Programming library, written in modern day Java. It is designed with a clear separation of the several algorithm concepts, e.g. `Gene`, `Chromosome`, `Genotype`, `Phenotype`, population and fitness `Function`. Jenetics allows you to minimize or maximize a given fitness function without tweaking it.

The following post describes how to solve a symbolic regression problem with the genetic programming capability of Jenetics.

```java
static final class Polynomial {
    private final double[] a;
    
    Polynomial(final double... a) {
        this.a = a.clone();
    }
    
    // Uses Horner's Method to evaluate the polynomial.
    double apply(final double x) {
        double result = a[a.length - 1];
        for (int i = a.length - 2; i >= 0; i--) {
            result = x*result + a[i];
        }
        return result;
    }
}

static Codec<Polynomial, DoubleGene> codec(final int degree) {
    return Codec.of(
        Genotype.of(DoubleChromosome.of(
            DoubleRange.of(-100, 100),
            degree + 1)
        ),
        gt -> new Polynomial(
            gt.getChromosome()
                .as(DoubleChromosome.class)
                .toArray()
        )
    );
}
```
