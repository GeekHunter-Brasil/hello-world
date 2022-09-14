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

- [Introduction](#introduction)
- [Concepts](#concepts)
- [Strategies](#strategies)
- [In practice: Rails](#in-practice-rails)
- [In practice: React](#in-practice-react)

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

We can even say Test Driven Development is more than just a requirement: it is a way of thinking our software. On the more granular level of black-box testing, we first write our specification file that describes how a piece of code should behave, i.e the functionality we expect from it. Then, we move to the next phase of writing the code to implement it. It requires a good understanding of our design patterns, our dependency chain, the programming paradigm we are using (OOP for most part), and specificities of the frameworks we adopt. All of that beforehand. Besides, we always have choices to make when it comes to whether or not we mock dependencies. On the practical sections, we will see examples of how to do it in our stack and what kind of choices made sense for us. Those choices translated to a guideline that we at Geek follow when writing our tests.

Also, it's worth noting that doing TDD helps building the basis of our test pyramid: if for every piece of code we write we have an underlying test to support it, the basis of the pyramid will increase effortlessly.

Next we will list some practices that we believe that are good to follow while writing tests using TDD:

- Test Isolation: one test should never affect other test. If I have one test broken, I have one problem. If I have two tests broken, I have two problems. This also implies that the tests are order independent.
- Assert First: start writing the tests with assertions that will pass when it is done. Always try to answer these two questions: "What is the right answer?" and "How am I going to check for the right answer?"
- Evident Data: How do you represent the intent of the data? Include expected and actual results in the text itself, and try to make their relationship apparent. You are writing tests for a reader, not just for the computer.
- Break: What do you do when you feel tired or stuck? Take a break. And if you still feel this way after the break, than raise HelpException, or, in other words, call for help 😉.

If you want to learn more about TDD, we strongly recommend you to take a look in these two books:

- [Test Driven Development: By Example](https://www.amazon.com.br/dp/B095SQ9WP4/?coliid=I3VU9HPFYLWNXJ&colid=U42P2DJ549XI&psc=0&ref_=lv_ov_lig_dp_it), from Kent Beck (aka the Father of TDD) where he builds from sketch a Money API showing all the steps and paths that you take by doing TDD the right way.
- [Effective Testing with Rspec 3: Build Ruby Apps with Confidence](https://www.amazon.com.br/Effective-Testing-RSpec-Myron-Marston/dp/1680501984/ref=sr_1_1?__mk_pt_BR=%C3%85M%C3%85%C5%BD%C3%95%C3%91&crid=3QGCORSGDQ4OV&keywords=effective+testing+with+rspec+3&qid=1662209082&sprefix=effective+testing+with+rspec+3%2Caps%2C182&sr=8-1&ufe=app_do%3Aamzn1.fos.6d798eae-cadf-45de-946a-f477d47705b9) from Myron Marston and Ian Dees. In this book you will learn how to use the power of rspec combined with TDD to create tests that truly reflects your app behavior.

### BDD

Behavior Driven Design fills the void we sometimes have between the acceptance criteria of a given task and the expectations we write for our code. In BDD, when building the specifications, we use a user-driven language (usually supported by a DSL) that connects the test scenarios with the product need. Therefore, doing BDD requires we think about tests from the very beggining, i.e from the refinement phase of a user story.

On the practical sections, we will see examples of implementing BDD for a sample task. Check out [this post](https://blog.geekhunter.com.br/a-juncao-do-behavior-driven-development-e-metodologia-agil/#:~:text=BDD%20%C3%A9%20uma%20metodologia%20de,e%20%E2%80%9Ccrit%C3%A9rios%20de%20aceita%C3%A7%C3%A3o%E2%80%9D) in our blog for a very good article on the role of BDD in Agile Culture.

### Do not reimplement logic

When writing black-box testing, make sure not to reimplement logic. If we are doing classic TDD, the chances of reimplementing logic are dramatically reduced since we start first with the specification and then we move to the implementation.

However, we often have to cope with legacy code that is not tested. In those cases, pay attention to write your test without mimicking the logic itself. Another situation where we can be tempted to reimplement logic is when we decide to mock our dependencies. While writing our test doubles, we are necessarily thinking on the implementation already. In those cases, make sure not to test the doubles themselves.

Take a look [at this video](https://www.youtube.com/watch?v=W40mpZP9xQQ&t=918s) that addresses this and other common mistakes when doing TDD.

### Non-decreasing Code Coverage

In order to increase code quality overtime, it's very useful to enforce a non-decreasing policy in our CI/CD pipelines. With that in mind, we don't allow code to be merged in the main branches if the overall code coverage decreases. Additionally, we look at the branches coverage as the criteria taken into account as it helps us focus on non trivial code we want to test instead of enforcing a 100% line coverage. Therefore, make sure you cover all possible code paths in your specification file.

It's worth noting that too many code paths to cover may indicate a method that can be simplified and broken down. Take a look at [this definition](https://docs.codeclimate.com/docs/cyclomatic-complexity) to help guide on that regard and at [this introduction](https://www.atlassian.com/continuous-delivery/software-testing/code-coverage) to code coverage as well.

### Do not refactor code unless tested

By refactoring, we mean the act of changing the implementation of a subject without changing its external behavior/functionality. Therefore, by definition, any black-box testing that is written for that entity should continue to pass. That means we can, and should, see tests as a guardrail that will prevent us from making mistakes during refactoring.

Therefore, do not refactor code unless tested; otherwise you risk introducing bugs to the codebase that may not be caught by someone else in a Code Review.

[Back to top ⬆️](#pushpin-summary)

## In practice: Rails

Sometimes we find ourselves dealing with some complex domain logic (such as hiring a candidate). To make our life easier, in these scenarios, we tend to split our problem in some layers:

1. Entrypoint: controllers, mutations, workers, etc
2. Manager: the point of truth, where we handle all the business rule
3. Service: code that we think that is a good idea to concentrate in one place
4. Validator: where we validate the input data
5. Repository: where we communicate with the database

With this approach, we make our application more manageable with well defined responsibilities to each abstraction. Besides, and not by chance, this simplifies the task of writing tests in different granularity.

> **A very important note here: we don't always structure our code in these five layers, because, when we are doing TDD the right way, we don't mind how we are going to implement some feature, but how we are going to TEST it. We end up, for example, writing and implementing the feature directly in the Controller.**
>
> **Finished the tests, we are able to refactor it and decide how we want to do it. Sometimes we want to split the feature in all these five layers, sometimes we don't feel the need to create a manager, and so on. There is not right or wrong, the most important thing is to write code that are covered by tests and easy to refactor it**

With that said, we can go with the guideline for how we usually write the tests in each one of those layers.

### 📨 Entrypoint Layer

Here is where our Controllers, Mutations, Workers, Rakes and so on, are defined. For this layer, we want an integration test going all the way to the database and external APIs. In each integration test, we find useful to test side effect and the response itself. With that in mind, two main things are asserted in an entrypoint test:

- That the input is correctly handled (e.g. that an ActiveRecord attribute is correctly updated at the database level)
- That the response is the expected (e.g. response.body, response.status, etc)

Here we want to hit a testing database with a set of transactions, and assert changes in the ActiveRecord itself. In addition to that, external APIs can be mocked with their expect behavior for a given scenario (we use [VCR](https://github.com/vcr/vcr) to handle this cases)

Let's use a Controller that handles a Hiring creation as example and go with the happy path, expecting that the hiring was created and the response status is 201.

```ruby
describe HiringsController, type: :request do
  context 'when hiring a candidate' do
    # Use FactoryBot.create to create an instance of your data object and persist it to the testing database
    let(:candidate_to_hire) { create(:candidate) }
    let(:job_to_hire_for) { create(:job) }
    let(:params) do
      {
        candidate_id: candidate_to_hire.id,
        job_id: job_to_hire_for.id
      }
    end
    let(:hire_a_candidate) do
      post 'hiring', params: params
    end

    it 'creates a new hiring' do
      # Assert if the hiring was created
      expect { hire_a_candidate }.to change(Hiring, :count).from(0).to(1)
    end

    it 'returns status code 201' do
      hire_a_candidate

      # Assert response code is 201
      expect(response).to have_http_status(201)
    end
  end
end
```

#### 🎛️ Manager Layer

This layer is the source of truth when talking about business rules. The manager is responsible to call all services that are required to complete a specific task.

Since we only create our managers in the REFACTOR step of TDD, the business rule is already tested, so we DO NOT ALWAYS create tests for managers itself. Usually, if we feel the need to create tests specific for a manager (for example, the manager has some `prepared_params` method that has a high cyclomatic complexity) we tend to use unit tests to help us quickly and reliably test more scenarios and edge cases than we do at an entrypoint layer.

So we don't necessarily test everything, and assuming other parts of the application are also tested (they should be!), it becomes much easier to simply spy the dependencies and assert how the manager is communicating with them (mock roles, not objects).

Following our example, let's say the manager needs to do two operations: create the hiring, as a side effect, send an email to the candidate hired, ie. So we know it has to call at least two methods in two different services (ie, dependencies), named `HiringCreator.create` and `CandidateMailer.send_hiring_mail`.

`HiringCreator.create` receives a hash with the attributes to create and returns the created hiring;

`CandidateMailer.send_hiring_mail` receives the `email` related to the candidate that is been hired.

```ruby
# Use Rspec.instance_double to be able to mock the dependencies
let(:hiring_updater) { instance_double ::Services::Hiring::Update::HiringUpdater }
let(:job_updater) { instance_double ::Services::Job::Update::JobUpdater }

# Use FactoryBot.build_stubbed to generate your data
let(:hiring) { build_stubbed(:hiring) }

context 'when manager is called with valid params' do
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
describe Validators::Candidate::CandidateUpdateValidator, type: :validator do
  let(:errors) { subject.errors }

  subject { Validators::Candidate::CandidateUpdateValidator.new params, :update }

  describe 'validates presence' do
    before do
      subject.valid?
    end

    context 'when params are invalid' do
      let(:params) { {} }

      it 'adds :blank error to all missing required fields' do
        expect(errors).to be_added :name, :blank
        expect(errors).to be_added :email, :blank
      end

      it 'adds :inclusion error to available field' do
        expect(errors).to be_added :available, :inclusion
      end
    end

    context 'when params are valid' do
      let(:params) do
        {
          name: 'John Doe',
          email: 'john@doe.com',
          available: available
        }
      end

      context 'and available has "true" value' do
        let(:available) { true }

        it 'has no errors' do
          expect(errors).to be_empty
        end
      end

      context 'and available has "false" value' do
        let(:available) { false }

        it 'has no errors' do
          expect(errors).to be_empty
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
describe Repositories::Hirings::Finder, '#find_all_by_job_id', type: %i[repository hiring] do
  subject { described_class.new Hiring }

  let(:job_to_find_hirings) { create(:job) }
  let(:other_job) { create(:job) }

  context 'when exist hirings for given job' do
    let!(:hiring_to_return_1) { create(:hiring, job: job_to_find_hirings) }
    let!(:hiring_to_return_2) { create(:hiring, job: job_to_find_hirings) }
    let!(:hiring_not_to_return) { create(:hiring, job: other_job) }

    it 'returns an array containing all the hirings for the given job' do
      result = subject.find_all_by_job_id(job_to_find_hirings.id)

      expect(result).to contain_exactly(hiring_to_return_1, hiring_to_return_2)
    end
  end

  context 'when does not exist hirings for given job' do
    let!(:hiring_not_to_return) { create(:hiring, job: other_job) }

    it 'returns empty' do
      result = subject.find_all_by_job_id(job_to_find_hirings.id)

      expect(result).to be_empty
    end
  end
end
```

[Back to top ⬆️](#pushpin-summary)
