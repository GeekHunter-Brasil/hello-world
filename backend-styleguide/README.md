<p align="center">
  <img alt="logo" src="/docs/logo.png" width="200">
</p>

# 💻 Backend Style Guide

Welcome to our backend style guide!

The goal of this guide is to help our team to understand and follow our code style and best practices, maintaining a pattern in all repositories.

`#letscode 😎👩‍💻👨‍💻`

# :pushpin: Summary

- [Introduction](#introduction)
- [Organization](#organization)
- [Application Layers](#application-layers)
- [Code Style](#code-style)
- [Translate](#translate)

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

To keep our code organized and also domain-driven, we have created a folder structure based on artifacts and domains.

We have today four main domains `Company`, `Candidate`, `Bid`, `Hiring` and some artifacts such as `Services`, `Validators`, `Repositories`.

```
📁 project/
└──📁 app/
   └──📁 services/ -> artifact
      └──📁 companies/ -> domain
         └──📁 candidates/ -> object
            └──📁 search/ -> behavior/action
                └──📝 company_candidates_search_manager.rb
      └──📁 candidates/ -> domain
         └──📁 jobs/ -> object
            └──📁 search/ -> behavior/action
                └──📝 candidates_jobs_search_manager.rb

```

[Back to top ⬆️](#pushpin-summary)

## Code Style

Code style is maintained by [Rubocop](https://docs.rubocop.org/).

However, there are some important patterns to be followed.

### 👉 Functional core, imperative shell

<img alt="functional-core-imperative-shell" src="/docs/functional-core-imperative-shell.png" width="500">

We should always strive to write an application consisting of a functional core, with an imperative shell.

- The functional core should hold most of the complexity while ideally being stateless, meaning it generates state values and never calls the shell, which is stateful.
- The imperative shell should be reactive and hold little complexity, only serving as a glue/plumbing for the application layers

Here are some good reads on the topic:

- [Functional Core, Imperative Shell by Destroy All Software](https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell)
- [Functional Core/Imperative Shell by Brian Sung](https://medium.com/@dev.junehoe/functional-core-imperative-shell-a5d1696a4ccb)

### 👉 Do not put logic into controllers

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

### 👉 Break down complex services into a manager service

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

### 👉 Do not use ActiveRecord callbacks

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

### 👉 Do not query data outside of the repository layer

Often when retrieving data from the database, we do that whenever it's needed. This is considered to be a bad practice here, since any query should be extracted to the repository layer of the respective model.

By doing it this way, we can write better, less repeated code that is also much easier to test (we only need to test the repository layer and call it later).

For example, suppose we have a service to notify all active users of some new action or feature:

❌ Bad

```ruby
module Services::NotifyAllActiveUsers
  def call
    users = User.find_by(status: :active)

    # Implementation goes here
  end
end
```

✅ Good

```ruby
module Repositories::Users::Finder
  # Note: This should be tested!
  def active_users
    User.find_by(status: :active)
  end
end

module Services::NotifyAllActiveUsers
  def call
    users = Repositories::Users::Finder.active_users

    # Implementation goes here
  end
end
```

### 👉 Use strings to define enumerations

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

An exception to this rule is if the data we are enumerating have some sort of order or level, for example:

```ruby
  enumerize :experience_level, in: {
    level_0: 0, # 0-1 ano
    level_1: 1, # 1-2 anos
    level_2: 2, # 2-3 anos
    level_3: 3, # 3-4 anos
    level_4: 4, # 4-5 anos
    level_5: 5, # 5-6 anos
    level_6: 6, # 6-7 anos
    level_7: 7, # 7-8 anos
    level_8: 8, # 8-9 anos
    level_9: 9, # 9-10 anos
    level_10: 10, # 10-11 anos
    level_11: 11, # 11-12 anos
    level_12: 12 # 12 ou mais anos
  }, predicates: true, scope: true
```

### 👉 Declaring GraphQL enums with enumerize

We use the [enumerize](https://github.com/brainspec/enumerize) gem to handle creating enums in our models and databases.

They should be used to specify GraphQL enums.

❌ Bad

```ruby
module Types
  module Enums
    module Candidate
      class SituationEnumType < Types::BaseEnum
        value 'registered', 'When the user has just registered on the platform.',
              value: 0
        value 'register_complete', 'When the user is registered and with his profile is complete.',
              value: 1
        value 'waiting_approval', 'When the user has passed a required test and is waiting to be approved and available to the companies.',
              value: 2
      end
    end
  end
end
```

✅ Good

```ruby
module Types
  module Enums
    module Candidate
      class SituationEnumType < Types::BaseEnum
        ::Candidate.situation.values.each { |situation| value(situation, value: situation.to_s) }
      end
    end
  end
end
```

### 👉 Use comments to define tables and fields

We use comments to define tables and fields. This is a good approach to help newcomers and other Geeks.

Don't put sensitive information, comments don't have any security mechanism and they are stored globally so any user connected to any database in the cluster can see all comments for shared objects.

Avoid adding obvious information. Good practices on code commenting also apply here!

❌ Bad

```ruby
# frozen_string_literal: true

class Person < ActiveRecord::Migration[5.0]
  def change
    create_table :person, comment: 'A person' do |t|
      t.references :address, index: false, foreign_key: true, comment: 'The address'

      t.string :phone_number, comment: 'The phone'

      t.string :name, comment: 'The name, like Joao Da Silva Mendes'

      t.timestamps null: false
    end
  end
end
```

✅ Good

```ruby
# frozen_string_literal: true

class Person < ActiveRecord::Migration[5.0]
  def change
    create_table :person, comment: 'Stores primary person data' do |t|
      t.references :address, index: false, foreign_key: true, comment: 'A reference for the address of this person'

      t.string :phone_number, comment: 'The phone number of this person including country and area code'

      t.string :name, comment: 'The complete name of this person'

      t.timestamps null: false
    end
  end
end
```

### 👉 Always use symbols in hashs

Ruby is type sensitive when dealing with hash keys, in a way that `hash[:key]` is different than `hash["key"]`.
But for Rails model validator, it doesn't matters, which can lead to some unexpected behavior.
Make sure to always use symbols as hash keys.

❌ Bad

```ruby
person = {
  'name' => 'John',
  'age' => 30,
}
```

✅ Good

```ruby
person = {
  name: 'John',
  age: 30,
}
```

### 👉 Symbolize hashs comming from redis

We use [Redis](https://redis.io/) to handle async tasks and processing.
When a execution is scheduled in Redis, it's stored as a JSON and deserialized when it's retrieved.

Due that, the hashs will always come with strings as keys.
When handling the data in a method that will be executed in the future, make sure to symbolize the hash keys.

❌ Bad

```ruby
def call_async(params)
  id = params[:id]
end
```

✅ Good

```ruby
def call_async(params)
  params = params.deep_symbolize_keys # params was stored in redis as a JSON, so we need to symbolize the keys
  id = params[:id]
end
```

[Back to top ⬆️](#pushpin-summary)

## Translate

### 👉 Use I18n to translate

When writing your programs avoid using hard coded strings that will be displayed to users, this allows support for multiple languages. The path with translated keys is `<REP>/config/locales/<locale>`

❌ Bad

```ruby
  render json: {
    message: 'Lorem Ipsum at message'
  }, status: :ok
```

✅ Good

```ruby
  render json: {
    message: I18n.t('<path_key_message>')
  }, status: :ok
```

### 👉 Directory Locales structure

For a better organization of the yml files the ideal is to `mirror the folders`, thus not creating too big files, with this structure we obtain a better organization.

to declare the yml, you must use the same name as the parent directory 'e.g. directory/directory.yml',

```
📁 project/
└──📁 config/
   └──📁 locales/
      └──📁 br/
         └──📁 views/
            └──📁 pages/
               └──📁 directory/
                  └──📝 directory.yml
               └──📝 pages.yml
      └──📝 pt-BR.yml
      └──📝 en.yml
```

file `pt-BR.yml` contains constants and common terms application.

```yml
pt-BR:
  cancel: Cancelar
  confirm: Confirmar
```

file `directory.yml` respect folder structure.

```yml
pt-BR:
  views:
    pages:
      directory:
        message: "Mussum Ipsum Lorem"
```

### 👉 Translate view

We use definition "Lazy" Lookup to translate views. For more information [Lazy Lookup](https://guides.rubyonrails.org/i18n.html#lazy-lookup)

example the `pages.index.title` values inside `app/views/pages/index.html.erb`

```
<%= t('.title') %>
```

Structe in yml file:

```yml
pt-BR:
  pages:
    index:
      title: "Boas-vindas"
```

In some cases, we may have phrases that repeat themselves. Thus, a standard structure can be created

```
<%= t('pages.default.title') %>
```

Structe in yml file:

```yml
pt-BR:
  pages:
    default:
      title: "Boas-vindas"
```

If necessary to use a param in view, use:

```
<%= t('.title', username: 'People') %>
```

yml file:

```
title: 'Boas-vindas %{username}'
```

### 👉 Translate e-mails

We use SendGrid for sending e-mails. SendGrid works with e-mail templates that need to be created in their platform, then we make an API call passing the template name and its parameters.

For a given e-mail we will have a different template for each language.

We use the following template nomenclature:

- `template_name` for `pt_br` template
- `en__template_name` for `en` template

Example: for the password recovery e-mail templates:

- `empresa-reset-password`
- `en__empresa-reset-password`

Then in the `monolith` we have two helper functions to generate the template name based on the user language or a given language:

- `MailersHelper.build_prefix_for_record(record, context_name)` - this function gets the `language` from the `record` and concatenates the `language` with the `context_name`, where `record` is the logged-in user object and `context_name` is the base template name
- `MailersHelper.build_prefix_for_language(language, context_name)` - this function concatenates the `language` with the `context_name`

Finally we send the e-mail, example:

```ruby
template = CustomDeviseMailer.build_prefix_for_record(record, template)
notification_params = {
  model_id: record.id,
  context: {
    name: template
  },
  messages: [{
    recipient: {
      user_id: record.id,
      user_type: DarwinHelper.get_user_type_by_instance(record),
      phone: record.try(:phone_number) || record.try(:phone),
      email: record.try(:email)
    },
    variables: {
      USER_NAME: record.is_a?(AdminUser) ? record.email : record.name,
      RESET_LINK: reset_link
    }
  }]
}
Workers::Darwin::Api.perform_async(notification_params)
```

[Back to top ⬆️](#pushpin-summary)
