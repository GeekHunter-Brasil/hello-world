<p align="center">
  <img alt="logo" src="/docs/logo.png" width="200">
</p>

# 💻 Backend Style Guide

Welcome to our backend style guide!

The goal of this guide is to help our team to understand and follow our code style and best practices, maintaining a pattern in all repositories.

`#letscode 😎👩‍💻👨‍💻`

# :pushpin: Summary

* [Introduction](#introduction)
* [Organization](#organization)
* [Application Layers](#application-layers)
* [Code Style](#code-style)
* [Tests](#tests)
* [Translate](#translate)

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

[Back to top ⬆️](#pushpin-summary)

## Tests

<img alt="no-test-no-beer" src="/docs/no-test-no-beer.jpg" width="200">

Tests are a **really, really important** part of our development. Internally, **we use TDD** and **tests are an implicit requirement for every single task**.

Here's some good resources to understand tests / TDD:

- [The 3 Types of Unit Test in TDD](https://www.youtube.com/watch?v=W40mpZP9xQQ)

There are also some good practices when it comes to testing to keep in mind. These are:

### 👉 Do not implement logic in tests

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

### 👉 Cover all code paths

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

### 👉 Each application layer has its own way of testing

Let's assume that we want to test all the layers responsible to update a hiring. We have 5 layers in this process.

#### 📨 Entrypoint Layer

Here is where our Controllers, Mutations, Workers, Rakes and so on, are defined. For this layer, we want an integration test, going all the way to the database and returning the result, asserting 3 main things:

- If the manager was called correctly, with the expected parameters and the expected number of times
- If the input was correctly handled (e.g. if the hiring status was correctly updated)
- If the response is what we expected (e.g. response.body, response.status)

Using an example of a Controller that handles the hiring update

```ruby
module Companies
  class HiringsController

  def update
    hiring_manager_updater.update params do |callback|
      callback.on_success do |updated_hiring|
        render json: updated_hiring
      end

      callback.on_failed do |exception|
        respond_to do |format|
          @errors = exception&.errors
          format.js { render json: @errors.full_messages.join('<br>'), status: 400 }
        end
      end
    end
  end

  private

  def hiring_manager_updater
    @hiring_manager_updater ||= ::Services::Companies::Hirings::HiringManagerUpdater.new
  end
end
```

❌ Bad

```ruby
let(:hiring_manager_updater) { instance_double ::Services::Companies::Hirings::HiringManagerUpdater }

context 'with valid params' do
  before do
    put 'companies/hiring', params
  end

  it "should call the hiring manager updater with appropriate arguments the right number of times" do
    expect(hiring_manager_updater).to have_received(:update).with(expected_params).exactly(1).times
  end
end
```

✅ Good

```ruby
context 'with valid params' do
  let(:hiring) { create(:hiring, status: Hiring.status.waiting) }
  let(:params) do
    {
      id: hiring.id,
      status: Hiring.status.approved
    }
  end

  let(:update_hiring) do
    put 'companies/hiring', params: params
  end

  describe 'validate behavior' do
    after do
      update_hiring
    end

    it "should call the hiring manager updater with appropriate arguments the right number of times" do
      expect_any_instance_of(::Services::Companies::Hirings::HiringManagerUpdate).to receive(:update).with(expected_params).exactly(1).times
    end
  end

  describe 'validate logic' do
    before do
      update_hiring
      hiring.reload
    end

    it 'should return 200 as status code' do
      expect(response).to have_http_status(200)
    end

    it 'should update the hiring status' do
      expect(hiring.status).to eq Hiring.status.approved.value
    end
  end
end
```

#### 🎛️ Manager Layer

This layer it's our source of truth when talking about business rules. The manager is responsible to call all the services required (included other managers) to complete the a specific task.

Here we want unit tests so we don't necessarily need to test everything. Assuming other parts of the application are also tested (they should be!), it becomes much easier to simply mock the dependency or assume it has been called when its result is not important to the test.

Following our example, we have a manager responsible to update the hiring

```ruby
module Services
  module Hiring
    class HiringManagerUpdater

    def update(params)
      hiring = hiring_updater.update(params)

      job_updater.update(hiring.job_id, :new_hiring)

      hiring
    end

    private

    def hiring_updater
      @hiring_updater ||= ::Services::Hiring::Update::HiringUpdater.new
    end

    def job_updater
      @job_updater ||= ::Services::Job::Update::JobUpdater.new
    end
  end
end
```

❌ Bad

```ruby
context 'with valid params' do
  it 'updates the hiring' do
    # Code asserting if hiring is updated
  end

  it 'updates the job' do
    # Code asserting if job properties have changed
    # Wrong! the service that updates jobs already tests this
  end
end
```

✅ Good

```ruby
let(:hiring_updater) { instance_double ::Services::Hiring::Update::HiringUpdater }
let(:job_updater) { instance_double ::Services::Job::Update::JobUpdater }

let(:hiring) { double('hiring', id: 1) }

context 'with valid params' do
  let(:params) do
    {
      id: hiring.id,
      status: Hiring.status.approved
    }
  end

  before do
    allow(subject).to receive(:hiring_updater).and_return hiring_updater
    allow(subject).to receive(:job_updater).and_return job_updater

    allow(hiring_updater).to receive(:update).with(kind_of(Hash))
    allow(job_updater).to receive(:update).with(kind_of(Hash), kind_of(Symbol))

    subject.update(params)
  end

  it "should call the hiring updater with appropriate arguments the right number of times" do
    expect(hiring_updater).to have_received(:update).with(expected_params).exactly(1).times
  end

  it "should call the job updater with appropriate arguments" do
    expect(job_updater).to have_received(:update).with(expected_params).exactly(1).times
  end
end
```

#### 🛠️ Service layer

Using the same logic in the Manager layer, here will mock the dependencies and test if
they were called appropriately.

```ruby
  module Services
    module Hiring
      module Update
        class HiringUpdater
          include RepositoriesInjection

          def update(params)
            validate_params(params)
            provide_hiring_repo.update(params)
          end

          private

          def validate_params(params)
            validator = Validators::Hiring::HiringUpdateValidator.new params, :update
            raise Exceptions::RecordInvalid.new(errors: validator.errors) unless validator.valid?
          end
        end
      end
    end
  end
```

❌ Bad

```ruby
context 'with valid params' do
  it 'updates the hiring' do
    # Code asserting if hiring is updated
  end
end
```

✅ Good

```ruby
let(:provide_hiring_repo) { instance_double Repositories::Hirings::DelegateRepository }
let(:hiring_update_validator) { instance_double Validators::Hiring::HiringUpdateValidator }

let(:hiring) { double('hiring', id: 1) }

context 'with valid params' do
  let(:params) do
    {
      id: hiring.id,
      status: Hiring.status.approved
    }
  end

  before do
    allow(subject).to receive(:provide_hiring_repo).and_return provide_hiring_repo
    allow(Validators::Hiring::HiringUpdateValidator).to receive(:new).with(kind_of(Hash), kind_of(Symbol)).and_return hiring_update_validator

    allow(hiring_update_validator).to receive(:valid?).and_return true
    allow(provide_hiring_repo).to receive(:update).with(kind_of(Hash))

    subject.update(params)
  end

  it "should call the hiring repository with appropriate arguments the right number of times" do
    expect(provide_hiring_repo).to have_received(:update).with(expected_params).exactly(1).times
  end
end
```

#### 🚫 Validator layer

The validator layer is responsible to check the data that is been passed to the database.
Here we want to assert if the validator is setting the errors properly according to the parameters validation rules.

Always cover all edge cases for possible types, this can avoid a lot of validation problems which can occurs in others layers.

```ruby
module Validators
  module Candidate
    class CandidateUpdateValidator < Validator
      def initialize(attrs = {}, scope = :update)
        super attrs, scope
      end

      private

      attr_reader :available, :name

      validates :available, inclusion: [true, false], if: -> { scope == :update }
      validates :name, presence: true, allow_blank: false, if: -> { scope == :update }
    end
  end
end
```
❌ Bad

```ruby
# frozen_string_literal: true

require 'rails_helper'

describe Validators::Candidate::CandidateUpdateValidator, type: :validator do
  let(:errors) { subject.errors }

  subject { Validators::Candidate::CandidateUpdateValidator.new params, :update }

  describe 'validates presence' do
    before do
      subject.valid?
    end

    context 'when params are invalid' do
      let(:params) { {} }

      it 'should add :inclusion to available' do
        expect(errors.added?(:available, :inclusion)).to be_truthy
      end
    end

    context 'when params are valid' do
      let(:params) { { available: true } }

      it 'should not add :inclusion to available' do
        expect(errors.added?(:available, :inclusion)).to be_falsey
      end
    end
  end
end
```

✅ Good

```ruby
# frozen_string_literal: true

require 'rails_helper'

describe Validators::Candidate::CandidateUpdateValidator, type: :validator do
  let(:errors) { subject.errors }

  subject { Validators::Candidate::CandidateUpdateValidator.new params, :update }

  describe 'validates presence' do
    before do
      subject.valid?
    end

    context 'when params are invalid' do
      let(:params) { {} }

      it 'should add :blank to name' do
        expect(errors.added?(:name, :blank)).to be_truthy
      end

      it 'should add :inclusion to available' do
        expect(errors.added?(:available, :inclusion)).to be_truthy
      end
    end

    context 'when params are valid' do
      context 'name' do
        let(:params) { { name: 'Test Name' } }

        it 'should not add :blank to name' do
          expect(errors.added?(:name, :blank)).to be_falsey
        end
      end

      context 'available' do
        context "when has 'true' value" do
          let(:params) { { available: true } }

          it 'should not add :inclusion to available' do
            expect(errors.added?(:available, :inclusion)).to be_falsey
          end
        end
        context "when has 'false' value" do
          let(:params) { { available: false } }

          it 'should not add :inclusion to available' do
            expect(errors.added?(:available, :inclusion)).to be_falsey
          end
        end
      end
    end
  end
end
```

#### 💾 Repository Layer

Last but not least we have the Repository Layer, responsible for communicating with the database.

In this layer we are looking for integration tests such as in the Entrypoint layer, because we want to truly communicate with the database.

Let's assume that we want to update the status of all the hirings that came from a specify job. To do that first we need to find all the hirings that we want to update.

```ruby
module Repositories
  module Hirings
    class Finder
      include Searchable

      def find_all_by_job_id(job_id)
        @repository.where(
          job_id: job_id,
        )
      end
    end
  end
end
```

❌ Bad

```ruby
subject { Repositories::Hirings::Finder.new Bid }

context 'when exists hirings with given job_id' do
  let(:job) { double(:job, id: 1) }
  let(:other_job) { double(:job, id: 1) }

  let(:first_hiring) { double(:hiring, job_id: job.id) }
  let(:second_hiring) { double(:hiring, job_id: job.id) }
  let(:other_hiring) { double(:hiring), job_id: other_job.id }

  before do
    allow(subject).to receive(:find_all_by_job_id).and_return [first_hiring, second_hiring]
  end

  it 'should return the expected hirings' do
    result = subject.find_all_by_job_id(job.id)
    expect(result).to [first_hiring, second_hiring]
  end
end
```

✅ Good

```ruby
subject { Repositories::Hirings::Finder.new Bid }

context 'when exists hirings with given job_id' do
  let(:job) { create(:job) }
  let(:other_job) { create(:job) }

  let(:first_hiring) { create(:hiring, job: job) }
  let(:second_hiring) { create(:hiring, job: job) }
  let(:other_hiring) { create(:hiring), job: other_job }

  it 'should return the expected hirings' do
    result = subject.find_all_by_job_id(job.id)
    expect(result).to [first_hiring, second_hiring]
  end
end
```

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
        message: 'Massum Ipsum Lorem'
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
      title: 'Boas-vindas'
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
      title: 'Boas-vindas'
```
If necessary to use a param in view, use:

```
<%= t('.title', username: 'People') %>
```

yml file:
```
title: 'Boas-vindas %{username}'
```

[Back to top ⬆️](#pushpin-summary)
