<!--
 Copyright (c) 2022 Joao Mello (joao.mello@geekhunter.com.br)
 
 This software is released under the MIT License.
 https://opensource.org/licenses/MIT
-->

<p align="center">
  <img alt="logo" src="/docs/logo.png" width="200">
</p>

# ♾️ The Geek Testing Style Guide

Welcome to our testing style guide!

The goal of this guide is to help our team to think about tests, write them, and review someone else's code taking them into account. In order to to that, we introduce some definitions and best practices on testing, along with pratical tips.

`#letscode 😎👩‍💻👨‍💻`

# :pushpin: Summary

* [Introduction](#introduction)
* [Concepts](#concepts)
* [Strategies](#strategies)
* [In practice: Rails](#in-practice-rails)
* [In practice: React](#in-practice-react)
* [In practice: Beyond](#in-practice-beyond)

## Introduction

Software testing is a very important aspect of our daily job at Geek: it's fundamental to our goal of delivering high-value products through high-quality software.

There're many arguments on why to test software. We highlight a few:

- good test strategies are KEY to increase codebase quality overtime; and a high-quality codebase is KEY to a high-quality product
- a good set of practises DECREASES cognitive effort in write/review phases; and less effort in write/review BUYS us time and energy to focus on delivering value to customers
- test is cheaper than bugfix. MUCH cheaper

[Back to top ⬆️](#pushpin-summary)

## Concepts

We have a few concepts to keep in mind when it comes to testing.

- Test Pyramid: a pragmatical approach that entails we should write tests with different granularities and the more high-level we get the fewer tests we should have. Check this [introductory article](https://martinfowler.com/articles/practical-test-pyramid.html) and then a more [advanced discussion](https://www.thoughtworks.com/insights/blog/transitioning-conventional-shift-left-testing) from Thoughworks.
- Black-box testing: a method of testing in which we assert the behavior/functionality of an entity without taking into account one's internal details, i.e implementation. 
- White-box testing: a method of testing in which we test the internal structures of an entity as opposed to its funcionality.

## Strategies

We lay down here some strategies that we adopt at Geek to which we pay great attention.  

### TDD

Quoting the Geek Manifest:

`"TDD is an implicit requirement of any code intervention"`

We can even say Test Driven Development is more than just a requirement, it is a way of thinking our software. On the more granular level of black-box testing, we first write our specification file that describes how a piece of code should behave, i.e what the functionality we expect from it. Then, we move to the next phase of writing the code to implement it. This is a simple concept, but not trivial to implement. It requires a good understand of our architecture, our dependency paths and the programming paradigm we are using (OOP for most part) beforehand. On the pratical sections, we will see examples of how to do it in Rails and React using the libs we have.

Also, it's worth noting that doing TDD will help build the basis of out test pyramid: if for every piece of code we write we have a underlying test to support it, the basis of the pyramid will increase effortlessly.

### BDD

### Do not reimplement logic

### Non-decreasing Code Coverage

### Do not refactor code unless tested

[Back to top ⬆️](#pushpin-summary)

## In practice: Rails

TBD

[Back to top ⬆️](#pushpin-summary)

## In practice: React

TBD

[Back to top ⬆️](#pushpin-summary)

## In practice: Beyond

TBD

[Back to top ⬆️](#pushpin-summary)
