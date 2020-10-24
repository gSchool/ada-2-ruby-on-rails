# Intro to the Destroy Action

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?pid=8d42ed28-6f2b-4413-85d4-ac5e015dc5e1&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals

- Understand the responsibility of the Rails convention `destroy` action
- Review the nominal and edge cases when testing the `destroy` action
- Use a route parameter in a controller action
- Learn to make a `delete` request

## The `destroy` Action

The `destroy` action is the last of the RESTful controller actions we will define. Its responsibility is, given some information (through route params), find a specific item, and then delete it from the database. **It is the _Delete_ in CRUD.** The `destroy` action is a _Rails convention_, so we will continue to use the name `destroy`.

## Predict with Testing

What should our `destroy` action do?  In our controller tests, we should verify that the record is deleted, and that the number of records decreases by one.  

```ruby
describe "destroy" do
  it "can destroy a model" do
    # Arrange
    poodr = Book.new title: "Practical Object Oriented Programming in Ruby", author: "Sandi Metz"

    poodr.save
    id = poodr.id

    # Act
    expect {
      delete book_path(id)

      # Assert
    }.must_change 'Book.count', -1

    poodr = Book.find_by(title: "Practical Object Oriented Programming in Ruby")

    expect(poodr).must_be_nil
    
    must_respond_with :redirect
    must_redirect_to books_path
  end
end
```

In the `destroy` action we will call the method with a `delete` http verb, and a path including the id of a model instance in the database.

- Nominal Case: If the instance is in the database, expect to remove that item from the database and then redirect back to the `index` action.
- Edge Case: If the instance is not in the database, expect to see a response of `404` or `:not_found`

## Implementing the Full Destroy Action

Fully implementing the `destroy` action means we should anticipate changing the following:

1. View: Adding a link to the destroy action's route, so that the user can see the link and click on it
1. Routes: Making a new route for this new destroy action, and practicing route params
1. Controller: Updating the controller so it recognizes the destroy action and does the app logic
1. Tests: We will write two controller tests-- one for nominal case, and one for edge case.

### View: Adding a Link to the `destroy` Action

In our active site, we will need some way for a user to delete a book.  We can do so with a link, but `link_to` by default makes a `GET` request.  We can add a `link_to` with a `method` attribute similar to this, in our `index.html.erb` and `show.html.erb` views.

```erb
<%= link_to "Delete #{book.title}", book_path(book.id), method: :delete %>
```

<details style="max-width: 700px; margin: auto;">
  <summary>What happens if you leave off the `method: :delete` from the `link_to</summary>
  
    the link will instead go to the item's show page
  </details>
### Routes: Destroy Route Using Route Parameters

We've set up a link for our users to click on, which implies sending a request.

Now, we need to add a route that will anticipate and direct that request in the `config/routes.rb` file.

```ruby
delete '/books/:id', to: 'books#destroy'
```

### Controller: Recognizes Destroy and the Destroy Logic

Open up the `BooksController` - we will need to add a new method `destroy` like this:

```ruby
# app/controllers/books_controller.rb
def destroy
  book_id = params[:id]
  @book = Book.find_by(id: book_id)

  if @book
    @book.destroy
    redirect_to books_path
  else
    render :notfound, status: :not_found
  end
end
```

Here we read the book ID from the params and store it in a variable `book_id`, then use that to find a specific book.  If the book is found, we call the **model** `destroy` method.  If we do not find the book, we render a 404 page.

### Verify with... Manual Testing

Because we didn't write the tests first, we should verify that our code works with manual testing.

## Exercise: Tests! Write the Destroy Action Tests

Now that we've seen the implementation for a working controller action, let's reverse-engineer how we would test it.

Follow these directions:

1. First, read through the implemented controller action. Make a list of every line of code that you could test.
1. Go through your list of possible things to test. Revise your list so you only have a few things to test that "prove the controller action is working."
1. Then, write your first draft of two tests for the destroy action. One test should be the "nominal case" and one should be an "edge case," that anticipates something atypical happening. Feel free to use the code samples above.
1. Lastly, compare your tests to a solution for the controller tests [here](https://raw.githubusercontent.com/Ada-Developers-Academy/textbook-curriculum/master/08-rails/code_samples/destroy_controller_tests.rb).

## Summary

- The destroy action is, by convention, the name for the delete part of the CRUD acronym.
- You need a way for the user to "Delete" something. This is probably a link in the view, made with `link_to`, specifying a path and the `method: :delete`.
- You need a way for the router to direct a delete request. The conventional route for delete is `delete '/books/:id', to: 'books#destroy'`, and will use route params in the controller.
- You need to specify the controller, controller action, and logic for delete. We should use ActiveRecord methods to interact with the database. We should use redirection and rendering intentionally with the context of our code.
  - You can render a default 404, not found, page with `render :notfound, status: :not_found`
- A destroy action will need 2 tests, just like the show action.
