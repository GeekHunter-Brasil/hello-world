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

The goal of this guide is to help our team to think about tests, write them, and review someone else's code taking them into account. In order to do that, we introduce some definitions and best practices on testing, along with pratical tips.

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

[Back to top ⬆️](#pushpin-summary)

## Strategies

We lay down here some strategies that we adopt at Geek and to which we pay great attention. Fundamentally, they guide us while we write tests.

### TDD

Quoting the Geek Manifest:

`"TDD is an implicit requirement of any code intervention"`

We can even say Test Driven Development is more than just a requirement: it is a way of thinking our software. On the more granular level of black-box testing, we first write our specification file that describes how a piece of code should behave, i.e the functionality we expect from it. Then, we move to the next phase of writing the code to implement it. This is a simple concept, but not trivial to implement. It requires a good understand of our architecture, our dependency paths, the programming paradigm we are using (OOP for most part), and specificities of the frameworks we adopt. All of that beforehand. On the pratical sections, we will see examples of how to do it in Rails and React using the libs we have.

Also, it's worth noting that doing TDD will help build the basis of out test pyramid: if for every piece of code we write we have a underlying test to support it, the basis of the pyramid will increase effortlessly.

### BDD

Behavior Driven Design fills the void we sometimes have between the acceptance criteria of a given task and the expectations we write for our code. In BDD, when building the specifications, the start point is the acceptance criteria of a task. Next, we use a language (usually supported by a DSL) that connects the test case with the product need. Therefore, doing BDD requires we think about test from the very beggining, i.e from the refinement phase of a user story.

On the practical sections, we will see examples of implementing BDD for a given task. Check out [this post](https://blog.geekhunter.com.br/a-juncao-do-behavior-driven-development-e-metodologia-agil/#:~:text=BDD%20%C3%A9%20uma%20metodologia%20de,e%20%E2%80%9Ccrit%C3%A9rios%20de%20aceita%C3%A7%C3%A3o%E2%80%9D) in our blog for a very good discussion on BDD and Agile culture.

### Do not reimplement logic

When writting black-box testing, make sure not to reimplement logic. If we are doing TDD right, the chances of reimplementing logic are dramatically reduced since we start first with the specification and then we move to the implementation. However, we often have to cope with legacy code that is not tested. In those cases, pay attention to write your test without mimicking the logic itself.

Take a look [at this video](https://www.youtube.com/watch?v=W40mpZP9xQQ&t=918s) that addresses this and other common mistakes when doing TDD.

### Non-decreasing Code Coverage

In order to increase code quality overtime, it's very useful to enforce a non-decreasing policy in our CI/CD pipelines. With that in mind, we don't allow code to be merged in the main branches if the overall code coverage decreases. Therefore, make sure you cover all code paths in your specification file. 

It's worth noting that too many code paths to cover may indicate a function that can be simplified and broken down. Take a look at [this definition](https://docs.codeclimate.com/docs/cyclomatic-complexity) to help guide on that regard.

### Do not refactor code unless tested

By refactoring, we mean the act of changing the implementation of an entity without changing its external behavior/functionality. Therefore, by definition, any black-box testing that is written for that entity should continue to pass. That means we can, and should, see tests as a guardrail that will prevent us from making mistakes during refactoring. 

Therefore, do not refactor code unless tested; otherwise you risk introducing bugs to the codebase that may not be caught by someone else in a Code Review.

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
