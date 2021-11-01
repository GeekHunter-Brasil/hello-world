<p align="center">
  <img alt="logo" src="/docs/logo.png" width="200">
</p>

# 🎨 Backend Style Guide

Welcome to our backend style guide! Grab your cup of coffee (or tea) and read this with attention. ☕🍵

The goal of this guide is to help our team to understand and follow our code style and best practices, maintaining a pattern in all repositories.

`#letscode 😎👩‍💻👨‍💻`

# :pushpin: Summary

* [Introduction](#introduction)
* [Organization](#organization)
* [Application Layers](#application-layers)
* [Code Style](#code-style)
* [Tests](#tests)

## Introduction

Internally, we mostly use [Ruby on Rails](https://rubyonrails.org/).

Our applications usually provide a [GraphQL](https://graphql.org/) API with [GraphQL Ruby](https://graphql-ruby.org/w) in order to allow communication with other applications.

To test our applications we implement TDD, using [RSpec](https://rspec.info/).

It's a great idea to read & follow these guides before continuing:

- [Rails Getting Started](https://guides.rubyonrails.org/getting_started.html)
- [Learn Ruby on Rails - Full Course, by FreeCodeCamp](https://www.youtube.com/watch?v=fmyvWz5TUWg)
- [Getting started with RSpec](https://rspec.info/)

There are also some really important principles that should be followed when developing. These are:

- [SOLID](https://medium.com/backticks-tildes/the-s-o-l-i-d-principles-in-pictures-b34ce2f1e898)
- [DRY](https://medium.com/@nrk25693/dry-or-wet-and-why-867ac3096483)
- [YAGNI](https://martinfowler.com/bliki/Yagni.html)
- [KISS](https://www.interaction-design.org/literature/article/kiss-keep-it-simple-stupid-a-design-principle)

[Back to top ⬆️](#pushpin-summary)

## Organization

We follow the standard Rails file organization pattern with a few tweaks, to organize our service and domains.

It's a great idea to read these posts to get started with the basics:

- [Ruby on Rails - Directory Structure](https://www.tutorialspoint.com/ruby-on-rails/rails-directory-structure.htm)
- [Folder structure in Ruby on Rails](https://dev-yakuza.posstree.com/en/ruby-on-rails/folder-structure/)

[Back to top ⬆️](#pushpin-summary)

## Application Layers

We usually break down our application between multiple layers. Here are some great reads that are recommended before continuing:

- [Clean Architecture - A Little Introduction](https://medium.com/swlh/clean-architecture-a-little-introduction-be3eac94c5d1)
- [DDD for Rails Developers. Part 1: Layered Architecture](https://www.sitepoint.com/ddd-for-rails-developers-part-1-layered-architecture/)

[Back to top ⬆️](#pushpin-summary)

## Code Style

Code style is maintained by [Rubocop](https://docs.rubocop.org/).

However, there are some important patterns to be followed.

### Functional code, imperative shell

<img alt="functional-core-imperative-shell" src="/docs/functional-core-imperative-shell.png" width="200">

We should always strive to write an application consisting of a functional core, with an imperative shell.

- The functional core should hold most of the complexity while ideally being stateless, meaning it generates state values and never calls the shell, which is stateful.
- The imperative shell should be reactive and hold little complexity, only serving as a glue/plumbing for the application layers

Here are some good reads on the topic:

- [Functional Core, Imperative Shell by Destroy All Software](https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell)
- [Functional Core/Imperative Shell by Brian Sung](https://medium.com/@dev.junehoe/functional-core-imperative-shell-a5d1696a4ccb)

### Do not put logic into controllers

Controllers should only care how to parse & format requests and responses. They should not hold any application logic, instead calling services or other resources to do the job.

❌ Bad
```ruby
def sell_book
  @book = Book.find(params[:id])

  if book.sold?
    book.errors.add :base, "The book is already sold"
  else
    book.sell
  end
end
```

✅ Good
```ruby
def sell_book
  @book = Book.find(params[:id])
  BookSellingService.sell_book(@book)
end
```

### Break down complex services into a manager service

We use services to extract and isolate business rules within our problem domain. This is a great idea since it becomes much easier to test each service and rule.

However this can get too complex and hard to test when dealing with complex rules - we usually end up creating a big ball of mud, full of business rules.

It's a great idea to break it down the flow between multiple services (or methods inside the same service), and then have one manager service to handle how the flow should be executed.

For example, suppose we are creating a new hiring between a candidate and a company. There are many rules and validations here, and many other things need to happen.

❌ Bad
```ruby
module Services::Hirings::CreateHiringService
  def call(params)
    prepared_params = # Code that prepares the params here

    hiring = # Code that creates the hiring here

    # Code that updates the company here

    # Code that sends emails here

    # Code that tracks conversion here
  end
end
```

✅ Good
```ruby
module Services::Hirings::HiringManagerCreator
  def flow(params)
    prepared_params = prepare_params(params)
    hiring = hiring_creator.create(prepared_params)

    update_company(params)

    send_email_to_candidate(params)
    send_email_to_company(params)
    send_email_to_admin(params)
    track_conversion(params)

    hiring
  end

  private

  def prepare_params(params)
    # Implementation goes here
  end

  def update_company(params)
    # Implementation goes here
  end

  def send_email_to_candidate(params)
    # Implementation goes here
  end

  def send_email_to_company(params)
    # Implementation goes here
  end

  def send_email_to_admin(params)
    # Implementation goes here
  end

  def track_conversion(params)
    # Implementation goes here
  end

  def hiring_creator
    # Notice that this is another service that deals only with creating the record,
    # since there are more rules & validations to be done there.
    @hiring_creator ||= ::Services::Hirings::Create::HiringCreator.new
  end
end
```

### Do not use ActiveRecord callbacks

In order to keep our application organized and easy to test, we decided not to use callbacks. The logic here is simple: callbacks are heavily dependant on the application, and forbid us from taking other paths to solve our problems (e.g. updating data in batch, isolating domains in another application, updating our database).

Instead, one should create manager services to define exactly how the flow for a business rule should happen, and test it.

For example, if we assume that we need to check a book title after creating it:

❌ Bad
```ruby
# In our model:
class Book < ApplicationRecord
  def after_create
    check_title
  end

  def check_title
    # Implementation goes here
  end
end

# Somewhere else, we create the record and hope that everything works 🙏💣
Book.create!(params)
```

✅ Good
```ruby
module Services::Books::BookManagerCreator
  def flow(params)
    prepared_params = prepare_params(params)
    book = create_book(params)

    check_title(book)

    book
  end

  private

  def prepare_params(params)
    # Implementation goes here
  end

  def create_book(params)
    # Implementation goes here
  end

  def check_title(book)
    # Implementation goes here
  end
end
```

### Use strings to define enumerations

We usually use [Enumerize](https://github.com/brainspec/enumerize) to create enumerations that integrate with our repository layer.

However, it is a great idea to define the values as strings instead of integers. This allows the data to be saved as strings in our database, which makes it much easier to be analyzed later by our data science team (removing the need to check the code to see what each number means).

❌ Bad
```ruby
enumerize :gender, in: {
  feminine: 0,
  masculine: 1,
  other: 2,
  non_binary: 3,
  prefer_not_answer: 4,
}, scope: true
```

✅ Good
```ruby
enumerize :gender, in: %w[
  feminine
  masculine
  other
  non_binary
  prefer_not_answer
], scope: true
```

[Back to top ⬆️](#pushpin-summary)

## Tests

<img alt="logo" src="/docs/no-test-no-beer.jpg" width="200">

Tests are a **really, really important** part of our development. Internally, **we use TDD** and **tests are an implicit requirement for every single task**.

There are also some good practices when it comes to testing to keep in mind.

### Do not implement logic in tests

Test should only strive to replicate the scenario you are testing, and expect the results you already know you should have. We should absolutely not implement (or even worse, re-implement) logic into a test.

Using this function as an example:

```ruby
def calculate_score(a, b)
  (a / b) + 5
end
```

❌ Bad
```ruby
it 'correctly calculates the score' do
  expected_value = (10 / 2) + 5 # <- look how we're re-implementing the function here 🤦

  expect(calculate_score(10, 2)).to eq(expected_value)
end
```

✅ Good
```ruby
it 'correctly calculates the score' do
  expect(calculate_score(10, 2)).to eq(10) # <- we already know the value, so just compare it 😆
end
```

### Cover all code paths

Taking [Cyclomatic Complexity](https://docs.codeclimate.com/docs/cyclomatic-complexity) into account, it's really important to cover all possible code paths.

Using this function as an example:

```ruby
def age_description(age)
  return 'Senior' if age > 50

  'Young'
end
```

❌ Bad
```ruby
# WRONG! does not cover the Young result
it 'correctly formats Senior age' do
  expect(age_description(61)).to eq('Senior')
end
```

✅ Good
```ruby
it 'correctly formats ages' do
  expect(age_description(61)).to eq('Senior')
  expect(age_description(19)).to eq('Young')
end
```


[Back to top ⬆️](#pushpin-summary)
