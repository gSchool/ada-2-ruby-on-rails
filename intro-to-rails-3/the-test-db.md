# The Test Database

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?pid=ab94f933-85fa-4185-ae92-ac6501423789&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals

- Explain how Rails' testing database differs from its development and production databases
- Use _fixtures_ to define test data
- Understand when _fixtures_ are appropriate and when they are not

## The Test Database

When Rails runs tests, it does so in the __test environment__. This means that we can configure Rails to do things like load different gems or use a different database when running tests. Using a different database is important because we don't want to impact the development database by writing potentially invalid or corrupt data.

The test database is meant to be transient. By default, Rails will reset the test database _between every test_. Data saved to the database in one test won't exist in other tests. The exception is fixture data, which is always available in every test. However, any changes to fixture data in a test will not be preserved into the next test.



<!-- available callout types: info, success, warning, danger, secondary  -->
### !callout-warning

## If the tests seem stuck

Use `rails db:test:prepare` if the test database seems to be stuck in a broken state. It will reset the test database, run any pending migrations, and re-seed the fixture data. Very handy!

This happens **very** rarely.

### !end-callout

## Creating Test Data With _Fixtures_
Writing tests for objects that interact with a database often involves test data. In Rails, we define _fixtures_--temporary data used to populate models in tests--for test data. _Fixtures_ are kept in `test/fixtures` and are defined as [YAML](http://yaml.org/) files.

Each YAML file defines default data for one model. So we'd use `test/fixtures/authors.yml` to create some test data for use when testing `Author` models. Here's what YAML looks like:

<!-- XXX: for some reason the yml fenced syntax highlighter never ends. Major bummer. While editing, useful to remove the "yml" here. -->
```yml
metz:
  name: Sandi Metz
butler:
  name: Octavia E. Butler
madeleine_lengle:
  name: Madeleine L'Engle
```

YML is a set of key/value pairs separated by a colon. It's also __white space sensitive__. It knows how to nest key/value pairs based on their indentation, so pay close attention to your formatting. In the example above, we define two sets of data with keys `metz` and `butler`. Each of those keys has a value which is a second set of key/value pairs (like a hash of hashes). Because our `Author` model doesn't track much data, the _fixtures_ are pretty small. Let's take a look at another set of _fixtures_, this time for the `Book` model:

```yml
kindred:
  title: Kindred
  author: butler
  description: A good sci-fi book
  publication_date: 1979
parable:
  title: Parable of the Sower
  author: butler
  description: Fire drugs?
  publication_date: 1993
poodr:
  title: Practical Object Oriented Design in Ruby
  author: metz
  description: A good book on programming
  publication_date: 2012
```

In this example, we define two data sets, each representing one `Book`. Each _fixture_ contains quite a bit of data. Take a look at the values for the `author` key in each _fixture_. Because we're using Rails, and because Rails knows that an `Book` `belongs_to` an `Author`, and because we're defining our test data with _fixtures_, we can __reference other _fixture_ data in related models by key__. So `author: metz` will connect the _fixture_ `poodr` with the `Author` _fixture_ `metz`.

### Accessing Fixtures

At the beginning of every test, Rails will tear down your entire test database and rebuild it from scratch, importing all the fixtures. This means you can get at fixtures the same way you would access any other model instance: with ActiveRecord methods.

```ruby
metz = Author.find_by(name: "Sandi Metz")
```

It's worth noting that Rails will give the fixture data a random ID every time, so you can't count on `Book.first` or `Book.find(3)` do to what you expect.

You can also reference fixtures by the name you gave them in the YAML file:

```ruby
metz = authors(:metz)
```

All the authors can be accessed through `authors`, all the books through `books`, and so on. However, these collections are **not** enumerable - if you need to enumerate all the rows, use the ActiveRecord methods (e.g. `Book.all.each`, not `books.each`).

### Using Fixtures to Test Relationships

Recall from the last lecture the way we tested model relations. It was a little clunky - we had to create our data inline, for every test.

With fixtures, we can do better. Instead of defining our data in the test we can use the fixture data, as follows:

```ruby
# test/models/book_test.rb
require 'test_helper'

describe Book do
  describe 'relations' do
    it "has an author" do
      book = books(:poodr)
      expect(book.author).must_equal authors(:metz)
    end

    it "can set the author" do
      book = Book.new(title: "test book")
      book.author = authors(:metz)
      expect(book.author_id).must_equal authors(:metz).id
    end
  end
end
```

### When **NOT** to use Fixtures

Fixtures are great, and used well they can go a long way toward DRYing up and simplifying your test code. However, they're not always the right choice.

Because fixtures are available for every test, they work best for defining a baseline of "good" data for your tests to work with - for example, an `Author` for your new `Book` to `belong_to`. In contrast, I have found that when you put "bad" data in your fixtures, things tend to get very confusing very quickly. This is compounded by the fact that there are usually many, many more examples of invalid data than of valid.

What's more, for performance reasons, Rails will never run validations against your fixture data - it simply assumes you know what you're doing. This can make it very difficult to even tell what's _supposed_ to be "good" data.

As a general rule of thumb, fixtures are useful for valid data and positive tests, while invalid model instances should be confined to the test or `describe` block where it's interesting.

## Learning Comprehension Questions

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: multiple-choice
* id: 7700a246-ffbf-4cbe-b84a-0c44ee2bb34e
* title: Getting a Fixture Record
* points: 1
* topics: rails, tdd, rails-models

##### !question

```yml
kindred:
  title: Kindred
  author: butler
  description: A good sci-fi book
  publication_date: 1979
parable:
  title: Parable of the Sower
  author: butler
  description: Fire drugs?
  publication_date: 1993
```

Given the fixture file above, how can you load the `parable` Book?

##### !end-question

##### !options

* fixtures(:parable)
* `books('parable')`
* `books(:parable)`
* `Book.parable`

##### !end-options

##### !answer

* `books(:parable)`

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

In your tests you want to use the name of the table (in this case `books`) and pass in a symbol with the same name as you find in the fixture's yaml file.

So `books(:parable)` gets you that book from the test database.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: 1f0dda7b-4918-4cbd-a394-65cd8f7315c1
* title: Why use fixtures?
* points: 1
* topics: rails, rails-models, tdd

##### !question

What advantages do fixtures provide?

##### !end-question

##### !placeholder

Fixture advantages

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

Fixtures help you dry up your tests and provide them an initial set of data to use allowing you to focus on testing features.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: multiple-choice
* id: 495af356-7d9e-4310-8813-306da1e712a9
* title: When shouldn't you use fixtures
* points: 1
* topics: rails, rails-models, tdd

##### !question

When are fixtures **not** a good choice?

##### !end-question

##### !options

* To define a set of **good** valid data.
* To define a set of **invalid** model instances
* To dry up your tests

##### !end-options

##### !answer

* To define a set of **invalid** model instances

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

Because validations are not run against fixtures and because of readability it's best to use fixtures to define good or valid data, and when testing validations, "break" the model explicitly as part of the test.

For example:

```ruby
it "requires a title" do
  book = books(:parable)
  # explicitly break the validation
  book.title = nil

  expect(book.valid?).must_equal false
  # ...
end
```

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

## What We Accomplished

- Learned about the _test environment_, and how it is torn down and re-created for each test
- Used _fixtures_ to define data available for every test
- Used _fixtures_ to clean up our test code
- Considered when _fixtures_ **aren't** the best choice

## Additional Resources

- [Rails Guide on the Test Database](http://guides.rubyonrails.org/testing.html#the-test-database)
- [Rails fixtures documentation](https://api.rubyonrails.org/v3.1/classes/ActiveRecord/Fixtures.html)
