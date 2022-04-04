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

The goal of this guide is to help our team to think about tests, write them, and review someone else's code taking them into account.

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
- test is cheaper than bugfix; and SCALES better

[Back to top ⬆️](#pushpin-summary)

## Concepts

We have a few concepts with which we should be familiar. Things like test pyramid, test doubles, black-box vs white-box testing, and so on. We highly recommend you carefully read this [introductory article](https://martinfowler.com/articles/practical-test-pyramid.html) before moving on. It talks about different types of tests and how they fit in the analogy of the test pyramid.

[Back to top ⬆️](#pushpin-summary)

## Strategies

We lay down here some strategies that we adopt at Geek and to which we pay great attention. Fundamentally, they guide us while we write tests.

### TDD

Quoting the Geek Manifest:

`"TDD is an implicit requirement of any code intervention"`

We can even say Test Driven Development is more than just a requirement: it is a way of thinking our software. On the more granular level of black-box testing, we first write our specification file that describes how a piece of code should behave, i.e the functionality we expect from it. Then, we move to the next phase of writing the code to implement it. It requires a good understanding of our architecture, our dependency paths, the programming paradigm we are using (OOP for most part), and specificities of the frameworks we adopt. All of that beforehand. Besides, we always have choices to make when it comes to whether or not we mock dependencies. On the pratical sections, we will see examples of how to do it in our stack and what kind of choices made sense for us. Those choices translated to a guideline that we at Geek follow when writing our tests.

Also, it's worth noting that doing TDD will help build the basis of our test pyramid: if for every piece of code we write we have a underlying test to support it, the basis of the pyramid will increase effortlessly.

### BDD

Behavior Driven Design fills the void we sometimes have between the acceptance criteria of a given task and the expectations we write for our code. In BDD, when building the specifications, we use a user-driven language (usually supported by a DSL) that connects the test scenarios with the product need. Therefore, doing BDD requires we think about tests from the very beggining, i.e from the refinement phase of a user story.

On the practical sections, we will see examples of implementing BDD for a sample task. Check out [this post](https://blog.geekhunter.com.br/a-juncao-do-behavior-driven-development-e-metodologia-agil/#:~:text=BDD%20%C3%A9%20uma%20metodologia%20de,e%20%E2%80%9Ccrit%C3%A9rios%20de%20aceita%C3%A7%C3%A3o%E2%80%9D) in our blog for a very good article on the role of BDD in Agile culture.

### Do not reimplement logic

When writting black-box testing, make sure not to reimplement logic. If we are doing classic TDD, the chances of reimplementing logic are dramatically reduced since we start first with the specification and then we move to the implementation.

However, we often have to cope with legacy code that is not tested. In those cases, pay attention to write your test without mimicking the logic itself. Another situation where we can be tempted to reimplement logic is when we decide to mock our dependencies. While writing our test doubles, we are necessarily thinking on the implementation already. In those cases, make sure not to test the doubles themselves.

Take a look [at this video](https://www.youtube.com/watch?v=W40mpZP9xQQ&t=918s) that addresses this and other common mistakes when doing TDD.

### Non-decreasing Code Coverage

In order to increase code quality overtime, it's very useful to enforce a non-decreasing policy in our CI/CD pipelines. With that in mind, we don't allow code to be merged in the main branches if the overall code coverage decreases. Here, we enforce a more permissive policy as we look at the number of methods instead of number of lines. It helps us focus on non trivial code we want to test instead of enforcing a 100% line coverage. Therefore, make sure you call all public methods of the subject under test in your specification file.

It's worth noting that you might want to test the non trivial code paths your application can take. But keep in mind that too many code paths to cover may indicate a method that can be simplified and broken down. Take a look at [this definition](https://docs.codeclimate.com/docs/cyclomatic-complexity) to help guide on that regard.

### Do not refactor code unless tested

By refactoring, we mean the act of changing the implementation of an entity without changing its external behavior/functionality. Therefore, by definition, any black-box testing that is written for that entity should continue to pass. That means we can, and should, see tests as a guardrail that will prevent us from making mistakes during refactoring.

Therefore, do not refactor code unless tested; otherwise you risk introducing bugs to the codebase that may not be caught by someone else in a Code Review.

[Back to top ⬆️](#pushpin-summary)

## In practice: Rails

### Prerequisites and Architecture

WIP

### Test Guideline

WIP

[Back to top ⬆️](#pushpin-summary)

## In practice: React

### Prerequisites and Architecture

TBD

### Test Guideline

TBD

[Back to top ⬆️](#pushpin-summary)

## In practice: Beyond

TBD

[Back to top ⬆️](#pushpin-summary)
