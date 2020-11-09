# Session

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?pid=c4e0e334-f6f5-4201-b243-ac6a00194aaf&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals
- Learn about and utilize `session` in Rails
- See how `session` & `flash` are different than other variables in Rails

## `session`
`session` is a special type of hash-like-object that we utilize to keep track of data throughout a users "session", which normally ends when they close their browser. `session` is similar to `flash`, except that data stored there will not go away after the next request-response cycle. Instead it will be stored indefinitely. `session` is most often used to store information about a user when they log in.

Rails will automatically delete the information from the `session` when the user closes the browser. We can also manually remove the data if we not longer want to track it (like when the user logs out).

Let's see it! We are going to create a special form which is going to allow our users to "log in". This is not a secure way to simulate logging in, but we are using this as our first stepping stone to creating "real" authentication, which we'll do later on. We'll allow the user to type a username which will "log in" as that user.

Before working through the next steps, checkout the branch for the previous lesson `$ git checkout 30-flash`


- Create a new `User` model with one column, `username`
    ```sh
    $ rails generate model User username:string
    $ rails db:migrate
    ```
- Create a `UsersController`
    ```sh
    $ rails generate controller Users
    ```
- Add the following routes to your routefile:
    ```ruby
    get "/login", to: "users#login_form", as: "login"
    post "/login", to: "users#login"
    post "/logout", to: "users#logout", as: "logout"
    get "/users/current", to: "users#current", as: "current_user"
    ```
    ### !challenge

    * type: short-answer
    * id: <!--unique-id (command-option-u)-->
    * title: RESTful Review
    <!--Other optional fields (checkpoints only) -->
    <!--`points: 1`: the number of points for scoring as a checkpoint-->
    <!--`topics: python, pandas`: the topics for analyzing points-->

    ##### !question

    Are these routes RESTful? If not, why not?

    ##### !end-question

    ##### !answer

    /\bno\b|\bNo\b/

    ##### !end-answer

    ##### !placeholder

    Are these routes RESTful?

    ##### !end-placeholder


    <!--optional-->
    ##### !explanation

    No, these routes do not map to CRUD actions and are not RESTful.  Many applications have custom routes that connect to specific actions, in this case, handling user login/logout actions.

    ##### !end-explanation

    ### !end-challenge



- Create the `users#login_form` action and view. The view for this action should use `form_with` to create a form for the `User` model, accepting just the `username`.
    
    **Question:** Do you need to do anything in the controller action to enable this?

    **Question:** Given the routes above, do we need to do anything special with `form_with`?

    <details>
    <summary>Click here to see our implementation</summary>

    ```ruby
    # app/controllers/users_controller.rb
    def login_form
      @user = User.new
    end
    ```

    ```html+erb
    <!-- app/views/users/login_form.html.erb -->
    <%= form_with model: @user, class: 'login__form', url: login_path do |f| %>
      <%= f.label :username %>
      <%= f.text_field :username %>

      <%= f.submit "Log In" %>
    <% end %>
    ```
    </details>
- Create the `users#login` action. This action should read the `username` field sent by the form, and do one of two things:
    - If the `username` corresponds to a `User` in the database, that user is returning to our site. Save their ID in `session[:user_id]`
    - If the `username` does not correspond to an existing user, this is a new user logging in for the first time. Create a `User` model, and use its ID to populate `session[:user_id]`.

    **Question:** How will the data sent by the form be organized? If you're not sure, how could you find out?

    <details>
    <summary>Click here to see our implementation</summary>

    ```ruby
    # app/controllers/users_controller.rb
    def login
      username = params[:user][:username]
      user = User.find_by(username: username)
      if user
        session[:user_id] = user.id
        flash[:success] = "Successfully logged in as returning user #{username}"
      else
        user = User.create(username: username)
        session[:user_id] = user.id
        flash[:success] = "Successfully logged in as new user #{username}"
      end

      redirect_to root_path
      return
    end
    ```
    </details>
- Create the `users#current` action. This page should display information about the currently logged-in user.
    <div>

### !challenge

* type: checkbox
* id: caa3b13b-2596-4e3b-88c8-1ff8265a181b
* title: Error Handling
<!--Other optional fields (checkpoints only) -->
<!--`points: 1`: the number of points for scoring as a checkpoint-->
<!--`topics: python, pandas`: the topics for analyzing points-->

##### !question

What should the `current` method do if no one is currently logged in?

##### !end-question

##### !options

* Respond with 404 not found
* Redirect to the login (login_path)
* Use flash to show an error and redirect to the login (login_path)
* Redirect to the home page (root_path)
* Use flash to show an error and redirect to the home page (root_path)

##### !end-options

##### !answer

* 

##### !end-answer
    
<!--optional-->
##### !explanation

All of these are valid options.  Different sites handle situations like this in a variety of ways.  In our implementation below, we use flash to show the user an error and then redirect to root_path.  Generally from a user's standpoint, it is better to be redirected to something useful with instructions rather than getting a 404 message, which may lead a user to thinking that your site is broken.

##### !end-explanation

### !end-challenge

<details>
<summary>Click here to see our implementation</summary>
    
```ruby
# app/controllers/users_controller.rb
def current
  @current_user = User.find_by(id: session[:user_id])
  unless @current_user
    flash[:error] = "You must be logged in to see this page"
    redirect_to root_path
    return
  end
end
```

```html+erb
<!-- app/views/users/current.html.erb -->
<p>You are logged in as user <%= @current_user.username %></p>
```
</details>
    </div>

- Create the `users#logout` action. This should set `session[:user_id]` to `nil` and redirect the user back to the `root_path`.

- Now that you have your routes and controller actions, update your view(s) so that a user can perform the actions of logging in and logging out.
- Think about other things you could add to improve the user experience. What has worked well on other sites you've visited?
- Think about how you could restrict actions based on who is (or isn't) logged in. You don't need to write code for these, just think about how you _might_ do it.
    - Only logged in users may change data on the site (the C, U and D of CRUD)
    - Users may mark books as their "favorites", and the current user page includes a list of that user's favorite books
    - Only the user that added a book to the site may edit or delete that book
    
    We will discuss these _authorization_ workflows further in the coming weeks.
<!--BEGIN CHALLENGE-->

### !challenge

* type: short-answer
* id: 71005a9d-849a-439d-a1df-4746b2d2cf60
* title: Brainstorm
<!--Other optional fields (checkpoints only) -->
<!--`points: 1`: the number of points for scoring as a checkpoint-->
<!--`topics: python, pandas`: the topics for analyzing points-->

##### !question

What is a feature that you could imagine adding to Ada-Books now that we can track users and/or is only possible now that users can log in?

##### !end-question

##### !answer

/.+/

##### !end-answer

##### !placeholder

Brainstorm

##### !end-placeholder

<!--optional-->
##### !explanation

For a great example of what other people have done with this idea, check out the site http://www.goodreads.com.  So many possibilities!

##### !end-explanation

### !end-challenge

<!--END CHALLENGE-->
## Key Takeaways
Rails provides a few special hash-like objects that allow us to go above and beyond local and instance variables in our Rails applications.

We will utilize `session` to keep track of logged in user information.

### Table of Rails Hash-like Objects
See this updated table that now includes `session`.

| Name        | Data Comes From                    | Available |
|:------------|:-----------------------------------|:----------|
| `flash`     | This or the last controller action | The rest of this request cycle and the next complete request cycle |
| `flash.now` | This controller action. Adds to the `flash` from the last cycle, but will not be carried over to the next one. | The rest of this request cycle (in `flash`) |
| `session`   | Some controller action             | Until the user closes the browser |
| `params`    | The request (URL or body)          | The corresponding request cycle   |


## Additional Resources
- [Sessions, Cookies and Authentication ](http://www.theodinproject.com/courses/ruby-on-rails/lessons/sessions-cookies-and-authentication)(not including 'Rolling Your Own Auth')
- [Rails Guide on Session](http://guides.rubyonrails.org/action_controller_overview.html#session)