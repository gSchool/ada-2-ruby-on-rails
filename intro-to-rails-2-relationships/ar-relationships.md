# Active Record Relationships

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?pid=4414f6c2-214d-4af9-9f41-ac5d01726c45&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals

- Move beyond single-model, single-table applications
- Learn the Rails vernacular for describing _model relationships_
- Explore the __awesome__ functionality provided by *belongs_to* and *has_many*

## Intro

It's 9:00 AM on Monday, and our CTO has just called an all-hands meeting. Our customers love the library application we built last week, and are delighted to be able to keep track of their books. In fact, they're so impressed that they're asking for more functionality. The customer wants to be able to:

- See a list of authors
- Keep track of details for a given author
- See the list of books written by a particular author

With the tools we've seen so far, keeping track of authors by themselves would be straightforward. However, we need to keep track of not just a list of authors but of how they relate to books. Managing these relations calls for a new set of techniques.

## *has_many* & *belongs_to*

Two Rails models can be related to each other through an identifier field (what we call a _foreign key_ in SQL). Take a look at the these two tables:

**authors**

| id   | name                 |
| :--- | :------------------- |
| 1    | Sandi Metz           |
| 2    | Margot Lee Shetterly |

**books**

|  id   |                  title                   |                     description                      | price | author_id |
| :---: | :--------------------------------------: | :--------------------------------------------------: | :---: | :-------: |
|   1   | Practical Object-Oriented Design in Ruby |        A great book on object-oriented design        | 19.99 |     1     |
|   2   |            99 Bottles of OOP             | An even more in-depth book on object-oriented design | 24.99 |     1     |
|   3   |              Hidden Figures              |         Good book that came before the movie         | 14.99 |     2     |

We would call this a __one-to-many__ association. We would say that an Author *has_many* Books, and each Book *belongs_to* an Author. The `author_id` column for a Book corresponds to the `id` of an Author record.


### Defining an Association

ActiveRecord provides lovely methods to quickly create an association between two (or more!) models. We can use class methods within models to make the definition.

First we can create a new Model for our books to have a relationship with:

```bash
rails generate model Author name:string 
rails db:migrate
```

We can specify this relationship in the Model classes.

```ruby
class Author < ApplicationRecord
  # plural because many books could be associated with this single author
  has_many :books
end
```

```ruby
class Book < ApplicationRecord
  # singular because it belongs to only a single author
  belongs_to :author
end
```

To make this relationship work, the underlying `books` table needs to have an `author_id` column to store the ID of the associated `Author`.

So we can update the `Book` model to remove the `.author` field and replace it with an `author_id` field which references the author to which this book belongs.

```bash
rails generate migration relateBooksToAuthors
```

Then we can remove the author field in the `Book` model and add a reference to the `Authors` table.

```ruby
class RelateBooksToAuthors < ActiveRecord::Migration[6.0]
  def change
    remove_column :books, :author
    add_reference :books, :author, index: true
  end
end 
```

And run `rails db:migrate` to run the migrations.

__Note:__ ActiveRecord does _not_ require a formal `foreign_key` relationship defined at the database level in order to leverage these associations, but it can be a really good idea to create them in your migrations.

__Note:__ We will also need to update all our views and controllers to reference the fact that our Author is now a related model.  So we will need to change `book.author` to `book.author.name`

<details style="max-width: 700px; margin: auto;">
  <summary>Books Controller</summary>

  ```ruby
  # Only changes to the book_params method
  def book_params
    return params.require(:book).permit(:title, :author_id, :description)
  end
  ```
</details>

<details style="max-width: 700px; margin: auto;">
  <summary>Books index view</summary>

  ```erb
  <h1>Book List</h1>
<ul>
  <% @books.each do |book|  %>
    <li>
      <%= link_to book.title, book_path(book) %>
      By: <%= book.author.name %> <%= link_to "Edit Book", edit_book_path(book.id) %>
      <%= link_to "Delete", book_path(book.id), method: :delete, class: 'book---delete-link' %>
    </li>
  <% end %>
</ul>
<%= link_to "Add Book", new_book_path %>
  ```
</details>

<details  style="max-width: 700px; margin: auto;">
  <summary>Books show view</summary>

  ```erb
  <div>
  <h1><%= @book.title %></h1>
  <h2>By: <%= @book.author.name %></h2> 
  <%= link_to "Edit", edit_book_path(@book) %>
  <%= link_to "Delete", book_path(@book), method: :delete, data: {confirm: "Are you sure?"}%>
</div>


<p><%= link_to "Return to Book List", books_path %></p> 
  ```
</details>

<details  style="max-width: 700px; margin: auto;">
  <summary>_form.html.erb partial view</summary>

  Note that we have a `select` as a drop-down to select the author, since we now have a table of authors.
  ```erb
  <%= form_with model: @book, class: 'create-book' do |f| %>
  <p>Please provide the following information to edit your book in our database:</p>

  <%= f.label :title %>
  <%= f.text_field :title %>

  <%= f.label :author %>
  <%= f.label :author %>
    <%= f.select :author_id, Author.all.map{ |auth| [auth.name, auth.id] } %>

  <%= f.label :description %>
  <%= f.text_field :description %>

  <%= f.submit "#{action_name} Book", class: "book-button" %>
<% end %>
  ```
</details>



### So what do these associations give us?

A whole slew of nice lookup methods that help us build queries for the associated model. Here'a sampling, and the [Rails Guides on Active Record Associations](http://guides.rubyonrails.org/association_basics.html) has complete details:

### belongs_to :author

- `book.author`: get the Author associated with this Book
- `book.author = author_object`: reassign this Book to an Author

### has_many :books

- `author.books`: returns an array of all the Books associated with this Author
- `author.books << book_object`: associate a Book with this Author by adding it to the array of Books
- `author.books = book_collection`: remove all prior Book associations and replace them with a new set of associations
- `author.books.find(id)`: find a specific Book, scoped to just those associated with this Author (not the most useful)
- `author.books.where(conditions_hash)`: get the Books associated with this Author that also satisfy the conditions in the `where` hash (much more useful)


And, for the table-oriented among you:

|                    Call                     |       Returns       |     Touches DB     |               Note                |
| :-----------------------------------------: | :-----------------: | :----------------: | :-------------------------------: |
|                `book.author`                |    Author object    |      Memoized      |                                   |
|        `book.author = author_object`        |    Author object    | No (requires save) |                                   |
| `book.author.update(author: author_object)` |    Author object    |        Yes         |                                   |
|      `book.build_author(author_hash)`       |    Author object    |         No         |  Does **not** set book.author_id  |
|      `book.create_author(author_hash)`      |    Author object    |        Yes         | Does **not** set `book.author_id` |
|               `author.books`                | Collection of books |      Memoized      |                                   |
|        `author.books << book_object`        | Collection of books |        Yes         |   **Does** set `book.author_id`   |
|      `author.books = book_collection`       | Collection of books |        Yes         |                                   |
|           `author.books.find(id)`           |     Book object     |        Yes         |                                   |
|       `author.books.where(condition)`       | Collection of books |        Yes         |                                   |
|       `author.books.build(book_hash)`       |     Book object     |         No         |   **Does** set `book.author_id`   |
|      `author.books.create(book_hash)`       |     Book object     |        Yes         |   **Does** set `book.author_id`   |

### Step by Step Guide

Now let's try it out for ourselves. We're going to run through the steps to update our book app to have this relationship using our step-by-step buide.

[AR Relationships Guide](active-record-relationships-exercise.resource.md).

## Relationships Comprehension Questions

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: multiple-choice
* id: 80abc098-30f1-4bdd-ba3b-7d9667f62f66
* title: Has-many & Belongs To
* points: 1
* topics: rails, rails-relationships

##### !question

For Tasklist if we created a model called, `Person` who is assigned to a `Task`.  So each `Task` is assigned to a `Person` and each `Person` can have many tasks assigned to them.

What line would we need to put in the `Task` class.

##### !end-question

##### !options

* has_many :tasks
* belongs_to :task
* has_many :persons
* belongs_to :person

##### !end-options

##### !answer

* belongs_to :person

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

Since each task is assigned to a single person you put `belongs_to :person` in the `app/models/Task.rb` file.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: multiple-choice
* id: acfeeb28-e47e-4026-8cf9-641b17710054
* title: Has-many & Belongs To
* points: 1
* topics: rails, rails-relationships

##### !question

For Tasklist if we created a model called, `Person` who is assigned to a `Task`.  So each `Task` is assigned to a `Person` and each `Person` can have many tasks assigned to them.

What line would we need to put in the `Person` class.

##### !end-question

##### !options

* has_many :tasks
* belongs_to :task
* has_many :persons
* belongs_to :person

##### !end-options

##### !answer

* has_many :tasks

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

Since each task is assigned to a single person you put `has_many :tasks` in the `app/models/Person.rb` file.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: multiple-choice
* id: af16a596-f5b5-4746-9aef-49808d1f6f65
* title: Relationship Methods
* points: 1
* topics: rails, rails-relationships

##### !question

How can we get a collection of the _titles_ of all the books the author below has written?

```ruby
author = Author.find_by(name: "Sandi Metz")
```

##### !end-question

##### !options

* `author.books.name`
* `author.books.map { |book| book.title }`
* `book.authors.title`
* It's impossible

##### !end-options

##### !answer

* `author.books.map { |book| book.title }`

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

`author.books` gets a collection of books by that author.

We then can use `.map` to convert the collection of `Book` instances into a list of strings containing the titles of all the books by that author.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

## Summary

In this lesson, we looked at how we can implemenent has-many and belongs-to relationships in our Active Record Models.  First we added the line `has_many :books` in the `Author` model which has books and we added `belongs_to :author` in the `Book` model to indicate that each book belongs to an author.  That adds a `.books` method to `Author` and `.author` method to each `Book`.  

Then we added a database migration to change the structure of the database.  In this migration we added an `author_id` field to the `books` table.  Rails will use this field to connect each book to the author for which it belongs.  Each of these methods (listed above) can help us in our controllers and views to do things like get a list of all the books by a specific author, or find the author for a specific book.

## Resources

- [Rails Guides on Active Record Associations](http://guides.rubyonrails.org/association_basics.html)