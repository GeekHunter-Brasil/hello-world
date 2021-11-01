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

Here's a quick summary of some important points:

### Services

We use services to extract and isolate business rules within our problem domain. This is a great idea since it becomes much easier to test each service and rule.

When dealing with complex rules, it's a great idea to break it down between multiple services and then create one manager service to execute the flow.

Here's a snippet explaining how to break down multiple services, taking the flow of creating a new hiring as an example:

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

[Back to top ⬆️](#pushpin-summary)

## Code Style

Code style is maintained by [Rubocop](https://docs.rubocop.org/).

However, there are some important patterns to be followed.

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


[Back to top ⬆️](#pushpin-summary)

## Tests

TODO

[Back to top ⬆️](#pushpin-summary)
