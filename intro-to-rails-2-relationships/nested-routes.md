# Nested Routes

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?pid=04243bb2-ea79-4c0a-8d0d-ac5e0145d0c0&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals

At the end of this lesson we will be able to:

- Use **nested routes** to make our webapp reflect the structure of our data
- Modify our controllers to take advantage of nested routes

## Introduction

Currently our library webapp does not reflect the new relation between `Author` and `Book` very well. Selecting from a drop-down menu when you create or edit a book is fine, but we could imagine a much more fluid user experience.

Here are some user stories to consider:

- As a librarian, I want to view the list of books for a specific author
- As a librarian, I want to see a link to add a book for a specific author on the details page for that author

Both these user stories break up the collection of books. They require us to consider the books written by _Octavia Butler_ separately from those written by _Ursula K. Le Guin_. One way to address this problem is by using **nested routes**.

## Nested Routes

The big idea behind nested routes is that we ought to be able to access our collection of books two ways: all the books, and only the books for one author.

Verb | Path                   | Description
---  | ---                    | ---
GET  | `/books`               | Show a list of all books
GET  | `/authors/7/books`     | Show books for author 7
GET  | `/books/new`           | Form to add a new book (needs author dropdown)
GET  | `/authors/7/books/new` | Form to add a new book for author 7 (no dropdown)

To nest routes in Rails, add a block to the `resources` in the route file. Note that we're only nesting the `index` and `new` actions.

```ruby
# config/routes.rb
resources :authors do
  resources :books, only: [:index, :new]
end

# We still want to be able to access the full collection,
# so books needs resources too
resources :books
```



<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: f22a0da9-2858-435d-8418-acc6f2015bb2
* title: What does `rails routes` do?
* points: 1
* topics: rails, rails-routes

##### !question

**Activity:** Use `rails routes` to look at the route table. What do you notice about the new nested routes?

##### !end-question

##### !placeholder

What do you see with rails routes?

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation


When we inspect our route table, we can see two new routes have been added.

```
$ rails routes
         Prefix Verb   URI Pattern                             Controller#Action
   author_books GET    /authors/:author_id/books(.:format)     books#index
new_author_book GET    /authors/:author_id/books/new(.:format) books#new
[... author routes ...]
          books GET    /books(.:format)                        books#index
                POST   /books(.:format)                        books#create
       new_book GET    /books/new(.:format)                    books#new
[... other book routes ...]
```

We can make a few observations about these new routes:
- The URI pattern matches what we had in the table above
    - The parameter is called `:author_id`, not `:id`
- Since these routes have prefixes, we can use path helpers (`author_books_path`, `new_author_book_path`)
- The original routes (`/books` and `/books/new`) are still there
- These routes point to the same controller actions we were using before. This will help keep things DRY.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: cc338abf-0ddc-4e2f-b73d-2b66a398c282
* title: Why only new and index actions?
* points: 1
* topics: rails, rails-routes

##### !question

**Question:** So far we have only nested the `index` and `new` actions. Should we nest the other 5 RESTful routes? Why or why not?

##### !end-question

##### !placeholder

Other RESTful routes?

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

You only need to use the nested route if the related model is important to the request.  In the example of `index` we were listing only books by a specific author.  In the `new` action we were creating a new book belonging to a specific author.  

With `update`, `create`, `show`, `delete`, or `edit` etc all these methods do not need to know the author, as that field is already stored in the model or body of the request so they do not need to be nested.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

## Controllers and Views

### Index

Because our routes are more complex, we now need to make our controllers and views a little more intelligent. A request for the book list may now come in with or without an author ID; we need to handle both cases.

```ruby
# app/controllers/books_controller.rb
class BooksController < ApplicationController
  # ...
  def index
    if params[:author_id]
      # This is the nested route, /author/:author_id/books
      author = Author.find_by(id: params[:author_id])
      @books = author.books

    else
      # This is the 'regular' route, /books
      @books = Book.all
    end
  end
  # ...
end
```

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: multiple-choice
* id: 09f3c6d2-78c2-4e91-b1de-ca422a7538a0
* title: What about if the author isn't found?
* points: 1
* topics: rails, rails-routes

##### !question

<strong>Question:</strong> What should our code do if the author is not found, that is, if the user goes to <code style="white-space: nowrap;">/authors/789012/books</code> or <code>/authors/toaster/books</code>?

##### !end-question

##### !options

* Respond with 200 ok
* Respond with 404 not found
* Respond with 500 server error
* Respond with 400 bad request

##### !end-options

##### !answer

* Respond with 404 not found

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

Though the above code does not handle this case, if we were doing this for a project, we would want to make sure that we respond with a not_found (404) response code in this case.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

#### Adding a link

Now that we've added a new route and updated the controller to handle this route, we need to add at least one link to this route. (Just because we understand restful routes doesn't mean the user will know how to find this route!)

We could add the following line to our authors show view:
```erb
  <%= link_to "Books by #{@author.name}", author_books_path(@author.id) %>
```

### New

The `new` action will be very similar. In the past we've seen that the `form_with` view helper will automatically insert any attributes on the model into the form. We will take advantage of this functionality by filling in the `author` on the new model.

```ruby
# app/controllers/books_controller.rb
class BooksController < ApplicationController
  # ...
  def new
    if params[:author_id]
      # This is the nested route, /author/:author_id/books/new
      author = Author.find_by(id: params[:author_id])
      @book = author.books.new

    else
      # This is the 'regular' route, /books/new
      @book = Book.new
    end
  end
  # ...
end
```

Now if we go to `/authors/2/books/new`, we should see that the dropdown menu starts with the second author selected.

<details style="max-width: 700px; margin: auto;">
  <summary>**Question:** Can we omit the dropdown entirely when the author is already filled in? How will this affect the view for the new action?</summary>

  We can omit the drop-down in the form, but then when the user submits the form the post request will not include the author because it's not in the form.

  Omitting the author from the form also prevents the user from changing the author, if they made a mistake.

  An alternative solution is to have a hidden input with the author id.  This lets the author id be submitted with the form, but hides it from the user.
</details>


#### Adding a link

Again, since we've added a new route, we need to add at least one link to this route so the user can actually get to it!

We could add the following line somewhere in our authors show view:
```erb
  <%= link_to "New book by #{@author.name}", new_author_book_path(@author.id) %>
```


## Over-nesting

A common mistake is to use nested routes when they aren't required. For an example of why this is problematic, consider a nested version of the route to show a book, `/authors/:author_id/books/:id`.

What should our `BooksController` do with the `author_id` parameter? Looking up a book requires the book ID, so we don't need it for that. Moreover, what if the `author_id` we loaded from the database doesn't match what was typed in the URL? Is this an error? The same questions come up with the `edit`, `update` and `destroy` paths.

The central issue here is that the `author_id` parameter is _redundant_. We could already figure out the author given the book. Asking the user to send it to us again only creates an opportunity for confusion.

**In general, if your route specifies a resource by ID (e.g. `/books/:id`), it probably should not be nested.**

### Create

The decision on whether to nest the `create` route is a little more nuanced.

In our library example we have the author ID in the form data, so including the author ID in the `create` URL would introduce redundancy. This is similar to what we say above for the individual routes. What if we get a `POST /authors/7/books`, but the form data indicates the `author_id` should be 13? Is this an error? If not, which one should we trust?

However, sometimes you need to do a `POST` without using a form, or the form doesn't have the information you need. In this case, nesting the `create` route might be the right way to go.

## Summary

- Nested routes are a tool to reflect model relations in the user's experience
- Nested routes are created by adding a block to the call to `resources` of the parent route
- Our controller actions need to be aware of nested routes, but we can usually re-use view code
- Be careful not to over-nest routes
    - Only `index`, `new` and sometimes `create` should be nested

## Additional Resources

- [Ruby on Rails: Nested Routes](http://guides.rubyonrails.org/routing.html#nested-resources)