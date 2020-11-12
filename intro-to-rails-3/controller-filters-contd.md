# Controller Filters for User Authorization

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?pid=93b51f36-2778-4bbd-aac0-ac6f0165c7e3&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals
- Use controller filters to require user login

## Getting Started

We explored how to use controller filters in a previous lecture to DRY up our code. Now, we are going to utilize controller filters to implement *User Authorization*.

### User Login
"Before" filters halt the request cycle prior to accessing the expected controller action. A common "before" filter is one which requires that a user is logged in for an action to be run. The code below creates a method to ensure a user is logged in. Additionaly, by using `before_action` in the ApplicationController, we enforce that the filter will be applied before _every_ action in the application :

### Example
In the `app/controllers/application_controller.rb`:
```ruby
...
  before_action :require_login

  def current_user
    @current_user = User.find(session[:user_id])
  end

  def require_login
    if current_user.nil?
      flash[:error] = "You must be logged in to view this section"
      redirect_to login_path
    end
  end
```

This `before_action` filter assumes that the application has set the `session[:user_id]` property in the `UsersController` as part of the *User Authentication* process.

If we add this to our application currently, it will cause issues since every action will require login and based on the logic inside of the `require_login` method, every action will then redirect to the login page again and again and again...

We want to ensure that our users can login without already being logged in! We can accomplish this by combining two things Rails has for controller filters: the `skip_before_action` filter, and the options to define which actions to exclude with `except`.

In the `app/controllers/users_controller.rb`:
```ruby
...
skip_before_action :require_login, except: [:current_user]
...
```

The `skip_before_action` ensures that the `require_login` method is **skipped on every action defined in this controller,** *except* the action listed after `except`: the `current_user` action.

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: 3dbe2363-c73d-4cd5-81fc-83c410cb2cdd
* title: skip_before_action location
<!-- * points: [1] (optional, the number of points for scoring as a checkpoint) -->
<!-- * topics: [python, pandas] (optional the topics for analyzing points) -->

##### !question

Why is the `skip_before_action` located in the `UsersController` when `before_action` is located in the `ApplicationController`?

##### !end-question

##### !placeholder

Your answer here

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

We want the before filter to run at the beginning of every action in the application (in every controller), **except** for the `current_user` action located specifically in the UsersController. 
In other words, we do this because we want to filter to apply universally to the application and we want the exception to apply in a very specific instance.  This works because **every** controller in our application inherits from `ApplicationController`.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

## Summary
And that's how to use controller filters to enforce that a user is logged in before they can access your website!

## Additional Resources
- [Rails Documentation on Controller Filters](https://guides.rubyonrails.org/action_controller_overview.html#filters)
