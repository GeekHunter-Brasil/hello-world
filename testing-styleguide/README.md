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

We can even say Test Driven Development is more than just a requirement: it is a way of thinking our software. On the more granular level of black-box testing, we first write our specification file that describes how a piece of code should behave, i.e the functionality we expect from it. Then, we move to the next phase of writing the code to implement it. It requires a good understanding of our architecture, our dependency chain, the programming paradigm we are using (OOP for most part), and specificities of the frameworks we adopt. All of that beforehand. Besides, we always have choices to make when it comes to whether or not we mock dependencies. On the pratical sections, we will see examples of how to do it in our stack and what kind of choices made sense for us. Those choices translated to a guideline that we at Geek follow when writing our tests.

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

It's worth noting that too many code paths to cover may indicate a method that can be simplified and broken down. Take a look at [this definition](https://docs.codeclimate.com/docs/cyclomatic-complexity) to help guide on that regard and [this introduction](https://www.atlassian.com/continuous-delivery/software-testing/code-coverage) to code coverage.

### Do not refactor code unless tested

By refactoring, we mean the act of changing the implementation of an entity without changing its external behavior/functionality. Therefore, by definition, any black-box testing that is written for that entity should continue to pass. That means we can, and should, see tests as a guardrail that will prevent us from making mistakes during refactoring.

Therefore, do not refactor code unless tested; otherwise you risk introducing bugs to the codebase that may not be caught by someone else in a Code Review.

[Back to top ⬆️](#pushpin-summary)

## In practice: Rails

### Design Principles and Patterns

Our Rails application is structured in a layered architecture: we start at the entrypoints, passing by managers, services, validators and up a respository layer. This approach makes our application more manageable with well defined responsabilities to each abstraction. Besides, and not by chance, this makes the task of writing tests in different granularities much easier as we rely on a Dependency Rule approach.

Image below show what kind of tests we target on each layer.

IMAGE

### Guideline

With that in mind, we show here our guidelines to help you to write tests for each one of those layers.

#### 📨 Entrypoint Layer

Here is where our Controllers, Mutations, Workers, Rakes and so on, are defined. For this layer, we want an integration test, going all the way to the database and APIs, asserting 3 main things:

- If the inner layer (manager) was called correctly, with the expected parameters and the expected number of times
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
