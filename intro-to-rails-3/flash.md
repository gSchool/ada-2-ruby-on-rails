# Flash

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?pid=7035c2f6-8428-473a-ba8a-ac68018a92f5&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals
- Learn about and utilize `flash` in Rails
- Understand when `flash.now` is appropriate to use

## `flash`
`flash` is a special type of hash that we utilize to send one-time messages from controllers to our views. It is similar to `params` in that it is something special that Rails creates for us. It is different because it persists from one one request to another.

`flash` is often used to denote overall success or failure when submitting forms. `flash` messages are used a single time, displayed once then disappear.

The scenario that we are going to use to utilize `flash` is when we are adding a new book. Previously, we added the new book to the database and then used `redirect_to` to go back to the book list page. We want to add some sort of notice to the book list page to inform the user that their new book has been added successfully. Let's take a look at our `create` method as it exists right now.
```ruby
# app/controllers/books_controller.rb
def create
  @book = Book.new(book_params)
  if @book.save
    redirect_to root_path
    return
  else
    render :new, status: :bad_request
    return
  end
end
```

Let's update this controller action to add a key-value pair in the `flash` hash to contain a message that we want to display on the book list page when a book has been successfully added.

```ruby
# app/controllers/books_controller.rb
def create
  @book = Book.new(book_params)
  if @book.save
    flash[:success] = "Book added successfully"
    redirect_to root_path
    return
  else
    flash.now[:error] = "Something happened. Book not added."
    render :new, status: :bad_request
    return
  end
end
```

Now let's test it out to see if this will be displayed somewhere on our book list page after successful creation. Any luck? No!

As with all variables that we set in our controller actions, the way we get this data to display to our users is to put it in a view! Since we can use `flash` in many different contexts, we will add this to our `application.html.erb` view so that any controller action will be able to use this in the future (though you can add this to any view you'd like).

```erb
# app/views/layouts/application.html.erb
...
<body>
  <section class="flash-msg__section">
    <% flash.each do |name, message| %>
      <strong class="<%= name %>">
        <%= message %>
      </strong>
    <% end %>
  </section>

  <%= yield %>
</body>
```

This view code will also dynamically add a `class` to the message `div` so that we can add any distinguishing styles to these flash messages.

Now let's see what we end up with!

### `flash.now`

In the above example, the data in the `flash` persisted through a `redirect`, which involves a complete new request-response cycle. This is a new phenomenon - previously, controller actions have been completely separate, with no way to provide continuity from one to the next. However, data added to `flash` is only persisted through the end of the next request-response cycle, and after that it's gone.

This is great for `redirect`, but for `render` it's a little overkill. Because `render` sends back HTML immediately instead of initiating a new request-response cycle, putting information in the `flash` will make it visible for two responses in a row. If you want to store data in the `flash` but only have it available for this request, use `flash.now` instead.

Data stored in `flash.now` will be visible in `flash` for the remainder of this request, but will not be visible for the next request. You do not ever need to read the contents of `flash.now`.

## Exercise

Return to either `TaskList` or `RideShareRails`. 
1. Utilize `Flash` in at least 3 places in your controller(s).
  * Consider whether `flash` or `flash.now` is appropriate.
2. Update the appropriate views to display the `flash` messages.
3. Share your work with a neighbor.

## Review

<!--BEGIN CHALLENGE-->

### !challenge

* type: multiple-choice
* id: c0ad8357-edf5-4c33-9ebf-8d36f902a162
* title: Flash 
<!--Other optional fields (checkpoints only) -->
<!--`points: 1`: the number of points for scoring as a checkpoint-->
<!--`topics: python, pandas`: the topics for analyzing points-->

##### !question

How long do Flash messages persist?

##### !end-question

##### !options

* For one request
* Multiple request-response cycles
* Both

##### !end-options

##### !answer

* For one request

##### !end-answer


<!--optional-->
##### !explanation

* Flash messages and their data persist through one response-request cycle, then disappears. 

##### !end-explanation

### !end-challenge

<!--END CHALLENGE-->

<!--BEGIN CHALLENGE-->

### !challenge

* type: multiple-choice
* id: 61a75672-cac9-4367-8f19-8785febe4d08
* title: Flash.now 
<!--Other optional fields (checkpoints only) -->
<!--`points: 1`: the number of points for scoring as a checkpoint-->
<!--`topics: python, pandas`: the topics for analyzing points-->

##### !question

Which scenario should use `flash.now`?

##### !end-question

##### !options

* Redirecting user to a view with a success message.
* Rendering a view with a failure message.
* Displaying the same flash message for two responses.

##### !end-options

##### !answer

Rendering a view with a failure message.

##### !end-answer

<!--optional-->
##### !explanation

The `flash.now` method is best paired with `render`. Because `render` does not create another response-request cycle, the `flash.now` data will only display for the view it is 'rendered' in and disapear for the next view.

##### !end-explanation

### !end-challenge

<!--END CHALLENGE-->

## Key Takeaways
Rails provides a few special hash-like objects that allow us to go above and beyond local and instance variables in our Rails applications.

We will utilize `flash` and `flash.now` to provide additional feedback to our users as they use our applications.

### Table of Rails Hash-like Objects

| Name        | Data Comes From                    | Available |
|:------------|:-----------------------------------|:----------|
| `flash`     | This or the last controller action | The rest of this request cycle and the next complete request cycle |
| `flash.now` | This controller action. Adds to the `flash` from the last cycle, but will not be carried over to the next one. | The rest of this request cycle (in `flash`) |
| `params`    | The request (URL or body)          | The corresponding request cycle   |


## Additional Resources
- [Rails Guides: The Flash](https://guides.rubyonrails.org/action_controller_overview.html#the-flash)
