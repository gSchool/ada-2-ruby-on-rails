# Unit Testing Models

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?pid=ebe7522a-0a61-4987-91e0-ac65013cd170&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals

By the end of this lesson, we will be able to...

- Write unit tests for Rails Models
- Understand how model tests are typically organized
- Decide what parts of a model need to be tested, and how

## Testing Active Record Models

### What Should You Test?

Since models are quite similar to the code we were writing before Rails, our tests will look similar to the tests we've written so far. Here are some guidelines for what to test:

- Write at least one test for each _relation_ on a model
- Write at least one test for each _validation_ on a model
- Write at least one test for each _custom method_ on a model
- Each model also needs one test **where all the validations pass**

It's important to note that we do _not_ need to verify all the behavior that validations and relations give us. These pieces are implemented by Rails itself, and their tests are very comprehensive! Instead, our job is to make sure we've hooked into their behavior correctly. Essentially we're double checking we've spelled things right.

### The Model

Let's apply these guidelines to the `Book` model from our library app:

```ruby
# app/models/book.rb
class Book < ActiveRecord::Base
  belongs_to :author
  validates :title, presence: true, uniqueness: true
end
```

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: checkbox
* id: 2e86ee7e-f996-4f49-a6c8-5a71ca7d9d47
* title: What to test?
* points: 1
* topics: rails, rails-models, tdd

##### !question

Based on the guidelines above, what test cases will we need to write for this model?

##### !end-question

##### !options

* Do validations fail when the `title` field is missing?
* Do validations fail when the book has the same `title` as another book in the database?
* If you create an instance of the model with all fields, do validations pass?
* to ensure that each book must have an author

##### !end-options

##### !answer

* Do validations fail when the `title` field is missing?
* Do validations fail when the book has the same `title` as another book in the database?
* If you create an instance of the model with all fields, do validations pass?
* to ensure that each book must have an author

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

The model has 2 validations, that title must exist, that title must be unique and that each `Book` belongs to an `Author`.  

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

## Test Files

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: multiple-choice
* id: 644d02e1-35b1-431e-8946-f0c487075d76
* title: Where do model tests live?
* points: 1
* topics: rails, rails-models, tdd

##### !question

When we generated our `Book` model, Rails generated some tests as well. Where do they live?

##### !end-question

##### !options

* `test/book_test.rb`
* `test/models/book_test.rb`
* `test/controllers/Book/book_test.rb`
* `app/models/book_test.rb`

##### !end-options

##### !answer

* `test/models/book_test.rb`

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

All rails model tests go into `test/models`

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

If you generated this model after switching to spec-style testing, it should look something like this:

```ruby
# test/models/book_test.rb
require "test_helper"

describe Book do
  let(:book) { Book.new }

  it "must be valid" do
    value(book).must_be :valid?
  end
end
```

If you have something about `class`es and `ActiveSupport::TestCase` instead, just replace it with the above code.

## Testing Validations

### All Passing

From the list we developed above, our first task is to test that, if all fields are present on the `Book`, the validations will pass. We can follow the pattern shown in the generated file to write our first test.

```ruby
# test/models/book_test.rb
require 'test_helper'

describe Book do
  describe 'validations' do
    before do
      # Arrange
      author = Author.new(name: 'test author')
      @book = Book.new(title: 'test book', author: author)
    end

    it 'is valid when all fields are present' do
      # Act
      result = @book.valid?

      # Assert
      expect(result).must_equal true
    end
  end
end
```

Notice that we follow the same _Arrange_, _Act_, _Assert_ pattern we used before Rails.

The _Arrange_ step for this test is all in the `before` block. Here we call `Book.new`, creating an actual book to test against. By storing it in an instance variable, we make it available in our tests.

Like before, the _Act_ step is always to call the method we're testing. Because these tests are for our validations, that means we must call `.valid?`.

Finally, in the _Assert_ step we verify that `.valid?` returned true. You'll have the same set of minitest expectation available to you here as you did before Rails.

#### Running the Test

From the project root, run

```
$ rails test test/models
```

This test should pass without us having to do anything.

### Testing the `presence` Validation

Next, we'll test that our validations fail when the book doesn't have a title. We can build on our previous work by adding another `it` block that references `@book`.

```ruby
# test/models/book_test.rb
# inside the same describe 'validations' block:
it 'is invalid without a title' do
  # Arrange
  @book.title = nil

  # Act
  result = @book.valid?

  # Assert
  expect(result).must_equal false
end
```

In the _Arrange_ step, we take our previously valid book and modify it so that the validation we're interested in will fail. Remember that the `before` block is run before _every_ test, so this won't affect our previous test, even if they're run out of order.

Run the tests again - this one should also pass, because we're doing our testing after the fact.

<details style="max-width: 700px; margin: auto;">
  <summary>**Question:** How could we modify our model to make the test fail?</summary>

  We could remove the validation to require the presence of the model by commenting it out, just to verify that the test is working.
</details>

#### Checking Which Validation Failed

An ActiveRecord model might have many different validations, and if any one of them fails then the whole model is considered invalid. Some completely different validation could be making this test pass, even if our validation isn't doing what we think.

**Question:** Once validations have been run on a model, how can you find out what caused them to fail? How can we use this to make our test more precise?

<details style="max-width: 700px; margin: auto;">
<summary>Example</summary>
  
```ruby
# test/models/book_test.rb
it 'is invalid without a title' do
  # Arrange
  @book.title = nil

  # Act
  result = @book.valid?

  # Assert
  expect(result).must_equal false
  expect(@book.errors.messages).must_include :title
end
```
</details>

### Exercise: Testing the `uniqueness` Validation

Take a few minutes and write a test for the `uniqueness` validation on the `Book` model. Questions to consider:
- How will you make the validation fail?
- How will this test be similar to the `presence` validation?
- How will this test differ from the `presence` validation?

## Testing Relations

We've thoroughly covered our model validations, and can now be reasonably certain there's no way invalid models can make their way into our database. There's another bit of functionality on the `Book` model that we still haven't covered though: it's relation to `Author`.

How should we think about these tests? First, think of what we can say about a `Book` and it's relation to `Author`.

- `Book` has a `author_id` column
- `Book` has a method named `author`, which will find the `Author` corresponding to `author_id`
- Setting either `author_id` or `author` will change the other accordingly

The tests, then, might look like the following:

```ruby
# test/models/book_test.rb
require 'test_helper'

describe Book do
  describe 'relations' do
    it 'can set the author through "author"' do
      # Create two models
      author = Author.create!(name: "test author")
      book = Book.new(title: "test book")

      # Make the models relate to one another
      book.author = author

      # author_id should have changed accordingly
      expect(book.author_id).must_equal author.id
    end

    it 'can set the author through "author_id"' do
      # Create two models
      author = Author.create!(name: "test author")
      book = Book.new(title: "test book")

      # Make the models relate to one another
      book.author_id = author.id

      # author should have changed accordingly
      expect(book.author).must_equal author
    end
  end
end
```

Note that we use `create!` to build our Author, but `new` to build our book. We're not testing Author here, so it makes sense to throw an exception (and invalidate the test) if it can't be created.

In this test, we're just checking the basic functionality of the `author_id` and `author` fields. There's not a whole lot of complex behavior, we just want to make sure we've spelled everything right.

In the next section, we'll examine a slightly more elegant way to test the same thing.

## What We Accomplished

In this lesson we 

- Thought about which parts of an ActiveRecord model are testable
- Remembered how to use minitest-spec
- Wrote tests for model validations
- Wrote tests for model relations

## Additional Resources

- [Ruby on Rails Guide - Model Testing](http://guides.rubyonrails.org/testing.html#model-testing)
- [Minitest Quick Reference](http://www.mattsears.com/articles/2011/12/10/minitest-quick-reference/)
- [Minitest Expectations](http://ruby-doc.org/stdlib-trunk/libdoc/minitest/spec/rdoc/MiniTest/Expectations.html)
- [Minitest Rails Spec Documentation](http://blowmage.com/minitest-rails/)
- [Minitest Model Testing for Beginners](http://buildingrails.com/a/rails_unit_testing_with_minitest_for_beginners)
