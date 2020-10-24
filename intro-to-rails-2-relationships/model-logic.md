# Model Logic

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?id=300b1ded-dc3c-4177-b236-ac5e0011a75d&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals

By the end of this lesson we will be able to...

- Define a method in an ActiveRecord model
- Identify when to use model methods

## Business Logic

In Rails, we frequently say that your _business logic_ should go in the model. But what is business logic?

Generally speaking, business logic is anything that isn't part of the mechanics of running a web server. Here are some examples:

- Calculating the score of a Scrabble word
- Selecting an available driver to assign to a trip
- Determining which rooms in your hotel are available on a given date

One key observation is that these tasks are all independent of context. You could imagine a method to do any of these in a terminal program, a webapp, a mobile app, or many other settings. Because they're concerned with the problem at hand rather than with the specifics of the platform or the process of presenting content to the user, they all qualify as business logic.

As an example of business logic for our library app, we'll be writing a method to answer the following question: in what year was an author first published?



<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: 27077e17-e679-446a-9944-f24104ab3358
* title: How to compute 1st published?
* points: 1
* topics: rails, rails-models

##### !question

**Question:** How would you compute when the author was 1st published? Don't worry about _where_ to put this logic yet, only how it might work.

##### !end-question

##### !placeholder

How to figure out when an Author was 1st published

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

Well you could:

```ruby
# Get the author's books
books = author.books

# Then get a list of the publication years with
years_published = author.books.map { |book| book.publication_date }

# Then return the min date
return years_published.min
```

We could also use some Active Record methods like `where` to eliminate books with no publication date.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

## Model Methods

In Rails, methods that implement business logic are defined in the model. Since our method answers a question about one author, we will write an instance method in the `Author` model.

```ruby
# app/models/author.rb
class Author < ApplicationRecord
  # ... relations and validations ...

  def first_published
    books_with_year = self.books.where.not(publication_date: nil)
    first_book = books_with_year.order(publication_date: :asc).first
    return first_book.publication_date
  end
end
```

There are a few things to observe here:

- This is a regular old Ruby method! All of your previous Ruby skills still apply.
- We can access attributes of this model like database fields and relations via `self.thing`
- We use local variables to store intermediate values just like any other ruby method

**Question:** How would we call this method? Where should we put this call?

## Updating Ada Books

We can then update Ada books to use this method and have an Authors controller.

<details style="max-width: 700px; margin: auto;">
  <summary>routes.rb</summary>

  ```ruby
  # config/routes.rb
  Rails.application.routes.draw do
  # verb 'path', to: 'controller#action'

    root to: 'books#index'

    resources :books
    resources :authors, only: [:index, :show]
  end
  ```
</details>

<details style="max-width: 700px; margin: auto;">
  <summary>app/controllers/authors_controller.rb</summary>

  ```ruby
    class AuthorsController < ApplicationController
    def index
      @authors = Author.all.order(:name)
    end

    def show
      @author = Author.find_by(id: params[:id])

      if @author.nil?
        head :not_found
        return
      end
    end
  end
  ```

</details>

<details style="max-width: 700px; margin: auto;">
  <summary>app/views/index.html.erb</summary>

  ```erb
    <h1>Authors</h1>

    <ul>
    <% @authors.each do |author| %>
      <li><%= link_to author.name, author_path(author.id) %>
    <% end %>
    </ul>
  ```

</details>

<details style="max-width: 700px; margin: auto;">
  <summary>app/views/show.html.erb</summary>

  ```erb
    <h1><%= @author.name %></h1>
    <h2>First Published <%= @author.first_published.nil? ? "Never": @author.first_published %>

    <h2>Books</h2>
    <ul>
      <% @author.books.each do |book| %>
        <li><%= link_to book.title, book_path(book.id) %></li>
      <% end %>
    </ul>
  ```

</details>

## Summary

In this lesson we learned

- Business logic is anything concerned with the problem domain
    - Does not include logic to run the platform or present information to the user
- In Rails, business logic goes in the model
- Models are regular Ruby classes, so you can define methods in the same way we've seen before
- Calling `self.something` inside a model method gives access to ActiveRecord fields and relations

## Additional Resources

- [Where do I put my code?](http://codefol.io/posts/Where-Do-I-Put-My-Code)
- [Rails Guide to Scopes](http://guides.rubyonrails.org/active_record_querying.html#scopes)
    - Scopes are methods that you can call on a collection of that model, similar to `.order` or `.where`. They're an advanced technique we won't cover formally, but they can be super useful.