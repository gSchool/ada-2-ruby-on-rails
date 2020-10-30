# Active Record Validations

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?pid=331d23d5-762b-4f2c-9554-ac5e0133a56d&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals

By the end of this lesson we will be able to:

- Explore the _validations library_ provided by Active Record
- Discuss the role _validations_ play in data management
- Guide users towards providing better data by using validations and feedback messages

## Validating User Input

Rails gives us a nice way to validate data before we try to put it into the database.

### Why would we want to validate data?

To prevent harmful side effects. A common example is a system trying to send an email to something that isn't an email address. Validating user input--and providing users an opportunity to correct invalid input--increases the reliability and quality of our applications and data.

## Using Active Record Validations

Using an example from our Book application, if our application depends on each book having a title, we will want to _validate_ that a title is provided.

In an Active Record model we can use the `validates` method to describe what valid data looks like. The `validates` method takes two arguments. First is the field name to
validate (`:title`). The second is a hash describing how we want to validate it (`presence: true`)

```ruby
class Book < ApplicationRecord
  validates :title, presence: true
end
```

## Validation Helpers

There are many validation helpers (like `presence`) the most common are:

- `:presence`: ensures that the data field is not null *or empty*
- `:uniqueness`: ensures the value doesn't already exist for that field in the db
- `:format`: uses a regular expression to ensure data conforms to a pattern
- `:length`: ensures text is within a given length
- `:numericality`: ensures the value is a number

Each of the helpers takes additional options that provide further refinement.


Let's assume we've extended our Author object to have an `email`, `username` and `age` fields in addition to their current fields. Here are a few a more potential examples:

```ruby
class Author < ApplicationRecord
  # must provide an email address and it must include an @ sign
  validates :email, presence: true, format: {with: /@/}

  # usernames must be unique and between 5 and 25 characters in length
  validates :username, uniqueness: true, length: { in: 5..25 }

  # age must be a number. An positive integer > 12, to be more precise.
  validates :age, numericality: { only_integer: true, greater_than: 12 }
end
```

## When is an object validated?

Stolen directly from the [Active Record Validations Rails Guide](http://guides.rubyonrails.org/active_record_validations.html) (but tidied and tweaked) The following methods trigger validations, and will save the object to the database __only if the object is valid__:

- `create`
- `create!`
- `save`
- `save!`
- `update`
- `update!`

The bang versions (e.g. save!) raise an exception if the record is invalid. The non-bang versions don't. `save` and `update` return false, while `create` just returns the object.

Active Record models also have a `valid?` method that will perform validations on demand without attempting to persist information to the database. This is a helpful debugging tool and we use it often when we're interacting with objects in the _Rails console_.

## Error Messages: When Validations Fail

When a validation fails, Active Record keeps track of which value(s)
caused the failure. Active Record collects these failures into the object's `errors` hash. By default it is empty, but when a record fails validation, `errors` is updated with information about each failure:

```ruby
book = Book.new(title: nil)
book.errors # => {} empty because the validations haven't run yet.
book.save   # validations run when a save is attempted.
book.errors # => contains :title=>["can't be blank"]
```

Since each column could have multiple validations (for example, email must be present and match a regex) the value associated with a column is an Array of error messages.

### Exposing Error Messages

Because errors are collected directly into the model instance, we can expose error information in views. This is particularly helpful when we follow the pattern of re-rendering a form when validation/saving fails.

```erb
<h2>New Book</h2>

<% if @book.errors.any? %>
  <ul class="errors">
    <% @book.errors.each do |column, message| %>
      <li>
        <strong><%= column.capitalize %></strong> <%= message %>
      </li>
    <% end %>
  </ul>
<% end %>

<div class="new-book">
  <%= form_with model: @book do |f| %>
    <%= f.label :title %>
    <%= f.text_field :title %>

    <%= f.label :description %>
    <%= f.text_area :description %>

    <%= f.submit %>
  <% end %>
</div>
```

## Comprehension Questions

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: d65a66ae-ade8-4203-a814-67277aaac014
* title: Title a required field
* points: 1
* topics: rails, active record, rails-validations

##### !question

What line would you need to add to `app/models/book.rb` to make the title field required?

##### !end-question

##### !placeholder

How to make title required?

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

You can make the title required with:

```ruby
validates :title, presence: true
```

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: multiple-choice
* id: e94ed372-085a-4729-9b6b-27d3e2906eaa
* title: Getting Error Messages
* points: 1
* topics: rails, active record, rails-validations

##### !question

How can we find out **why** a model instance, `book` failed validations?

##### !end-question

##### !options

* `book.valid?`  Will return an array of error messages
* `book.errors` will contain a hash like object with the fields as keys and an array of error messages for the values.
* `book.errors` will contain an array of error messages
* `book.save` will return a list of error messages

##### !end-options

##### !answer

* `book.errors` will contain a hash like object with the fields as keys and an array of error messages for the values.

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

`[model-name].errors` is a way to get a hash-like object which gives the fields as keys and the values as an array of error messages for that field.  We can then use them in our views like this:

```erb
<% if @book.errors.any? %>
  <ul class="errors">
    <% @book.errors.each do |column, message| %>
      <li>
        <strong><%= column.capitalize %></strong> <%= message %>
      </li>
    <% end %>
  </ul>
<% end %>
```

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: 7ed3e225-b251-49d5-af2e-1d1ed88791ce
* title: Validations in Ride-Share-Rails
<!-- * points: [1] (optional, the number of points for scoring as a checkpoint) -->
<!-- * topics: [python, pandas] (optional the topics for analyzing points) -->

##### !question

What is one validation you think you should perform in Ride Share Rails?

##### !end-question

##### !placeholder

RSR Validations?

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

Some possibilities include:

- Required fields like names, VIN numbers, etc
- Requiring the proper length of a VIN number
- Requiring a rating to be between 1 and 5
- Requiring the cost of a trip to be positive

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

## Summary

In this lesson we learned that validations are methods that run and check to see if our data is valid prior to saving.  Validations are important because they help prevent users from entering invalid data into our system.  

We create validations by adding the method name `validates` to our models and list the symbol name of the field to validate and options for how to validate.  Such options include `presence: true` for required fields (can't be `nil`), or `format:` for a regular expression.

Validations are triggered when the model attempts to save to the database.  So they run when the user runs methods like `.create`, `.save`, `.update`, including the bang versions.  You can also trigger validations with the `.valid?` method.
