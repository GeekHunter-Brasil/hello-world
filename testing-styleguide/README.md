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

## Introduction

<img alt="no-test-no-beer" src="/docs/no-test-no-beer.jpg" width="200">
Software testing is a very important aspect of our daily job at Geek: it's fundamental to our goal of delivering high-value products through high-quality software.

There're many arguments on why to test software. We highlight a few:

- good test strategies are KEY to increase codebase quality overtime; and a high-quality codebase is KEY to a high-quality product
- a good set of practises DECREASES cognitive effort in write/review phases; and less effort in write/review BUYS us time and energy to focus on delivering value to customers
- test is cheaper than bugfix; and SCALES better
- meaningful testing ENCOURAGES us to think about useful design patterns 

[Back to top ⬆️](#pushpin-summary)

## Concepts

We have a few concepts with which we should be familiar. Things like test pyramid, test doubles, black-box vs white-box testing, and so on. We highly recommend you carefully read this [introductory article](https://martinfowler.com/articles/practical-test-pyramid.html) before moving on. It talks about different types of tests and how they fit in the analogy of the test pyramid.

[Back to top ⬆️](#pushpin-summary)

## Strategies

We lay down here some strategies that we adopt at Geek and to which we pay great attention. Fundamentally, they guide us while we write tests.

### TDD

Quoting the Geek Manifest:

`"TDD is an implicit requirement of any code intervention"`

We can even say Test Driven Development is more than just a requirement: it is a way of thinking our software. On the more granular level of black-box testing, we first write our specification file that describes how a piece of code should behave, i.e the functionality we expect from it. Then, we move to the next phase of writing the code to implement it. It requires a good understanding of our design patterns, our dependency chain, the programming paradigm we are using (OOP for most part), and specificities of the frameworks we adopt. All of that beforehand. Besides, we always have choices to make when it comes to whether or not we mock dependencies. On the pratical sections, we will see examples of how to do it in our stack and what kind of choices made sense for us. Those choices translated to a guideline that we at Geek follow when writing our tests.

Also, it's worth noting that doing TDD helps building the basis of our test pyramid: if for every piece of code we write we have an underlying test to support it, the basis of the pyramid will increase effortlessly.

### BDD

Behavior Driven Design fills the void we sometimes have between the acceptance criteria of a given task and the expectations we write for our code. In BDD, when building the specifications, we use a user-driven language (usually supported by a DSL) that connects the test scenarios with the product need. Therefore, doing BDD requires we think about tests from the very beggining, i.e from the refinement phase of a user story.

On the practical sections, we will see examples of implementing BDD for a sample task. Check out [this post](https://blog.geekhunter.com.br/a-juncao-do-behavior-driven-development-e-metodologia-agil/#:~:text=BDD%20%C3%A9%20uma%20metodologia%20de,e%20%E2%80%9Ccrit%C3%A9rios%20de%20aceita%C3%A7%C3%A3o%E2%80%9D) in our blog for a very good article on the role of BDD in Agile Culture.

### Do not reimplement logic

When writting black-box testing, make sure not to reimplement logic. If we are doing classic TDD, the chances of reimplementing logic are dramatically reduced since we start first with the specification and then we move to the implementation.

However, we often have to cope with legacy code that is not tested. In those cases, pay attention to write your test without mimicking the logic itself. Another situation where we can be tempted to reimplement logic is when we decide to mock our dependencies. While writing our test doubles, we are necessarily thinking on the implementation already. In those cases, make sure not to test the doubles themselves.

Take a look [at this video](https://www.youtube.com/watch?v=W40mpZP9xQQ&t=918s) that addresses this and other common mistakes when doing TDD.

### Non-decreasing Code Coverage

In order to increase code quality overtime, it's very useful to enforce a non-decreasing policy in our CI/CD pipelines. With that in mind, we don't allow code to be merged in the main branches if the overall code coverage decreases. Additionaly, we look at the branches coverage as the criteria taken into account as it helps us focus on non trivial code we want to test instead of enforcing a 100% line coverage. Therefore, make sure you cover all possible code paths in your specification file.

It's worth noting that too many code paths to cover may indicate a method that can be simplified and broken down. Take a look at [this definition](https://docs.codeclimate.com/docs/cyclomatic-complexity) to help guide on that regard and at [this introduction](https://www.atlassian.com/continuous-delivery/software-testing/code-coverage) to code coverage as well.

### Do not refactor code unless tested

By refactoring, we mean the act of changing the implementation of a subject without changing its external behavior/functionality. Therefore, by definition, any black-box testing that is written for that entity should continue to pass. That means we can, and should, see tests as a guardrail that will prevent us from making mistakes during refactoring.

Therefore, do not refactor code unless tested; otherwise you risk introducing bugs to the codebase that may not be caught by someone else in a Code Review.

[Back to top ⬆️](#pushpin-summary)

## In practice: Rails

Our Rails application is structured in a layered architecture: we start at the entrypoints (controllers, mutations, etc), passing by managers (where business rules are defined), then services, validators and up to a respository layer. This approach makes our application more manageable with well defined responsabilities to each abstraction. Besides, and not by chance, this simplifies the task of writing tests in different granularities.

We show here a guideline for writing tests in each one of those layers.

### 📨 Entrypoint Layer

Here is where our Controllers, Mutations, Workers, Rakes and so on, are defined. For this layer, we want an integration test going all the way to the database and external APIs. In each integration test, we find useful to test side effects, internal transformations, and the response itself. With that in mind, three main things are asserted in an entrypoint test:

- That the inner layer (manager) is called correctly, with the expected set of parameters
- That the input is correctly handled (e.g. that an ActiveRecord attribute is correctly updated at the database level)
- That the response is the expected (e.g. response.body, response.status, etc)

Note here we usually hit a testing database with a set of transactions, and assert changes in the ActiveRecord itself. In addition to that, external APIs can be mocked with their expect behavior for a given scenario.

Let's use a Controller that handles a Hiring update as example. It gets a hash with the updated hiring values and let's imagine we are interested in the response status code being 200 if all goes well with the updated hiring hash in a json format. We also expect our controller to call a manager in order to perform the task in hand: here we name it `HiringManagerUpdater.update`.

❌ Bad

```ruby
# !! Do not replace the manager with a fake implementation
let(:hiring_manager_updater) { instance_double ::Services::Companies::Hirings::HiringManagerUpdater }

context 'when controller is called with valid params' do
  before do
    put 'companies/hiring', params: params
  end

  it "should call the hiring manager updater with appropriate arguments" do
    # !! Do not assert interactions with the stubbed manager
    expect(hiring_manager_updater).to have_received(:update).with(expected_params)
  end
end
```

✅ Good

```ruby
context 'when controller is called with valid params' do
  # Use FactoryBot.create to create an instance of your data object and persist it to the testing database
  let(:hiring) { create(:hiring, status: Hiring.status.waiting) }
  let(:params) do
    {
      hiring: {
        id: hiring.id,
        status: Hiring.status.approved
      }
    }
  end
  # Describe the expected set of (transformed) parameters your manager needs to properly work
  let(:manager_params) do
    ActionController::Parameters.new( params.map { |k,v| [k, v.to_s] }.to_h )
      .require(:hiring).permit(:id, :status)
  end
  let(:update_hiring) do
    put 'companies/hiring', params: params
  end

  before do
    # Only allow calls to HiringManagerUpdater.create that happen with the expected set of parameters
    allow_any_instance_of(::Services::Companies::Hirings::HiringManagerUpdater).to receive(:update).with(manager_params).and_call_original
    # If you don't expect exceptions to be rescued in this context, add this line
    allow(::Services::Logger::ExceptionLoggerService).to receive(:call).and_raise('Oups. No exception should have been seen!')
    update_hiring
    # Reload your ActiveRecord object to reflect changes performed by the subject under test
    hiring.reload
  end

  it 'should update the hiring status' do
    # Assert status has changed 
    expect(hiring.status).to eq Hiring.status.approved.value
  end
  it 'should return a response with status code 200 and content_type application/json' do
    # Assert response code is 200
    expect(response).to have_http_status(200)
    # Assert response content type is 'application/json'
    expect(response.content_type).to be == 'application/json'
  end
end
```

Note we apply a whitebox approach to this test as we opt to verify the inner integration with the manager. This means we should know what the manager needs in order to perform its job, so slighly compromising in the classic TDD approach of not taking the controller implementation into account. In the following layers, we will be conducting Unit Tests and mocking inner calls in order to isolate our subjects under tests. In all these tests, keep in mind we are using a more mocking approach to TDD where we take into account implementation details (i.e. we know who is going to be called and how) instead of a more classic black-box testing. Both are good, but we feel using a mocking approach help us to faster implement and test our Domain, which historically is the place we tend to apply more changes given the nature of our product.

A point about `Rspec.allow_any_instance_of`: when dealing with legacy code, the instance of the manager might not be injected during the test. So `allow_any_instance_of` comes in handy to allow us to verify that any instance of a manager was properly called. However, if you do have access to an instance of your manager (by injection) then you use `Rspec.allow` instead.

Finally, a point about exception handling: in the 'happy path' context, your code might not be supposed to rescue any exception. In those cases, it's very useful to mock your Exception Handling Service, using `Rspec.and_raise`, to raise a test exception in case it's ever called. This will make sure you surface exceptions caught in the process and also have the whole error stack on your test output.

### 🎛️ Manager Layer

This layer is the source of truth when talking about business rules. The manager is responsible to call all services that are required to complete a specific task.

Here we want unit tests to help us quickly and reliably test more scenarios and edge cases than we do at an entrypoint layer. So we don't necessarily test everything, and assuming other parts of the application are also tested (they should be!), it becomes much easier to simply mock the dependencies and assert their calls.

Following our example, let's say the manager needs to do two operations: update the Hiring status and, as a side effect, update the underlying Job. So we know it has to call at least two methods in two different services, named `HiringUpdater.update` and `JobUpdater.update`. `HiringUpdater.update` receives a hash with a hiring key, the attributes to update and returns the updated Hiring; `JobUpdater.update` receives the `job_id` related to the Hiring. Let's also not forget that we expect our manager to return the updated Hiring, as we see in the Controller layer.

❌ Bad

```ruby
context 'when manager is called valid params' do
  it 'should update the hiring' do
    # ... code asserting hiring is updated
    # !! The service that updates hirings already tests this
  end

  it 'should update the job' do
    # ... code asserting job properties have changed
    # !! The service that updates jobs already tests this
  end
end
```

✅ Good

```ruby
# Use Rspec.instance_double to be able to mock the dependencies
let(:hiring_updater) { instance_double ::Services::Hiring::Update::HiringUpdater }
let(:job_updater) { instance_double ::Services::Job::Update::JobUpdater }

# Use FactoryBot.build_stubbed to generate your data
let(:hiring) { build_stubbed(:hiring) }

context 'when manager is called valid params' do
  let(:params) do
    {
      id: hiring.id,
      status: Hiring.status.approved
    }
  end
  let(:updated_hiring) do
    hiring.update(status: Hiring.status.approved)
    hiring
  end

  before do
    # Replace HiringUpdater and JobUpdater in the subject under test
    allow(subject).to receive(:hiring_updater).and_return hiring_updater
    allow(subject).to receive(:job_updater).and_return job_updater

    # Only allow calls to update that happen with the expected set of parameters
    allow(hiring_updater).to receive(:update).with(params).and_return updated_hiring
    allow(job_updater).to receive(:update).with(updated_hiring.job_id)

    @result = subject.update(params)
  end

  it "should call the hiring updater with appropriate arguments the right number of times" do
    # Assert that HiringUpdater.update has been called once
    expect(hiring_updater).to have_received(:update).once
  end

  it "should call the job updater with appropriate arguments the right number of times" do
    # Assert that JobUpdater.update has been called once
    expect(job_updater).to have_received(:update).once
  end

  it "should return the updated Hiring" do
    # Assert the return is the expected
    expect(@result.status).to eq(Hiring.status.approved)
  end

  # ... remaining test scenarios
end
```

A point about performance: sometimes we want to use Factories to faster, and more reliably, build our data. Whenever possible, use stubbed methods such as `FactoryBot.build_stubbed`. Methods like `FactoryBot.create` update the test database, which is very useful in integration tests but is also a very expensive operation; in scale, hitting the database in manager layer considerably slows down the test pipeline. If the manager is not supposed to directly access an ActiveRecord, avoid using create methods.

### 🛠️ Service layer

Using the same logic as in the Manager layer, we mock the dependencies and test if they are called appropriately.

Let's follow the example of the `HiringUpdater`. In order to update the ActiveRecord records, it needs to first validate the parameters received from the manager via a `HiringUpdateValidator`. Then it either raises an Exception if parameters are invalid or proceeds to call a `DelegateRepository` in order to effectively persist the updates to the database. After updating, it returns the updated Hiring.

❌ Bad

```ruby
context 'when service is called with valid params' do
  it 'updates the hiring' do
    # ... code asserting if hiring is updated
    # !! The repository that updates jobs already tests this
  end
end
```

✅ Good

```ruby
let(:provide_hiring_repo) { instance_double Repositories::Hirings::DelegateRepository }
let(:hiring_update_validator) { instance_double Validators::Hiring::HiringUpdateValidator }

let(:hiring) { build_stubbed(:hiring) }

context 'when service is called with valid params' do
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

Here, the same performance consideration applies: prefer `FactoryBot.build_stubbed` over `FactoryBot.create` whenever possible.
### 🚫 Validator layer

The validator layer is responsible to check the data that is been passed to the database.
Here we want to assert whether the validator is setting the errors properly according to the parameters validation rules.

Always cover all edge cases for possible types as this can avoid a lot of validation problems, which won't be necessarily tested in outer layers.

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
### 💾 Repository Layer

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

[Back to top ⬆️](#pushpin-summary)
