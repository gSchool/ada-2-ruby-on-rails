# Testing OAuth

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?pid=9573b0c7-cb5d-44b5-9fda-ac6f000a16c5&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals:

- Know how to simulate a multi-request browser session in your tests
- Write some useful tests for login and logout functionality
- Set up OmniAuth for testing
- Understand what test mocking is

## Integration Tests and OAuth

Since OAuth relies on `session`, every test we write will need to send not one but two requests to the server:

1. A request that logs in the user
1. The request we're interested in testing

This approach worked great for MediaRanker, but now that we've added OAuth it's got some problems:

- Logging in with GitHub OAuth is complicated
- We'd need real GitHub usernames and passwords for our tests
- This introduces GitHub as a **dependency** to our tests. If talking to GitHub is slow then our tests will be slow, and if GitHub is down then our tests will fail even though our code is correct.

To resolve these issues, we'll use a strategy known as **mocking**. The basic idea is when we're running our test, instead of going all the way to GitHub for user data we'll short-circuit the process and use some made-up data instead.

As before, we will work through this process in the context of a single test first, then extract it to a helper method.

### Mocking GitHub OAuth

Here is how we are going to approach this problem:

1. Tell OmniAuth to use mock data instead of connecting to GitHub
1. Define some test fixtures for the `User` model
1. Define a `perform_login` function for our tests that sends the login request using fixture data
1. Test controller actions that require login!

#### Turn on Mocking

To be able to mock the interaction with Github, add this function to the `ActiveSupport::TestCase` class in `test/test_helper.rb`:

```ruby
# test/test_helper.rb
class ActiveSupport::TestCase
  # ...
  # Whatever stuff was here before
  # ...

  # OK this is what you're adding
  def setup
    # Once you have enabled test mode, all requests
    # to OmniAuth will be short circuited to use the mock authentication hash.
    # A request to /auth/provider will redirect immediately to /auth/provider/callback.
    OmniAuth.config.test_mode = true
  end
end
```

The `setup` method defined here will be run once, before all tests. All of our `describe` blocks will "inherit" from this class, so the setup code will be applied to them too.

#### Define Test Fixtures

These test fixtures are very similar to those we've made in the past. The only difference is in how we'll use them, not in how they are defined. Make sure the field names match the schema for your User model.

```yml
ada:
  provider: github
  uid: 12345
  email: ada@adadevelopersacademy.org
  name: countess_ada

grace:
  provider: github
  uid: 13371337
  email: grace@hooper.net
  name: graceful_hopps
```

#### Logging In

Next, we need to use this information to log in. We'll start by testing the auth callback itself, which will allow us to exercise this functionality in isolation.


<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: c3db3e0e-0400-4a35-aecd-cf5b26ead4ce
* title: Test cases
* points: 1
* topics: rails, testing, oauth

##### !question

What interesting test cases can you think of for the auth callback? Hint: there are at least 3.

##### !end-question

##### !placeholder

Test cases

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

Some test cases could be:

1. A normal login (nominal success)
1. Someone making a `get` request to the callback route without coming from the Auth provider (no auth_hash).
1. A request with an invalid auth provider `get auth_callback_path(:bogus)
1. A request with an invalid auth_hash, like missing a uid.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

##### Building the Auth Hash

In order to log in a user, we'll need to provide data to our controller in the exact same format as GitHub does. To make this process easy, we'll create a helper method that turns an instance of the `User` model into a mocked auth hash. This method will be almost the opposite of the `User#build_from_github` method we defined when we set up OAuth in the first place.

To make the method available to all our tests, we'll put it in `test/test_helper.rb`. Open up `test/test_helper.rb` again and add the following at the bottom of `class ActiveSupport::TestCase`, after the `setup` method we added earlier:

```ruby
# test/test_helper.rb
class ActiveSupport::TestCase
  # ...
  # Whatever was here before, including the setup function
  # ...

  # Test helper method to generate a mock auth hash
  # for fixture data
  def mock_auth_hash(user)
    return {
      provider: user.provider,
      uid: user.uid,
      info: {
        email: user.email,
        nickname: user.name
      }
    }
  end
end
```

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: 9eefd33c-bc65-4dc5-be2b-5da24807f6f1
* title: Why not an instance method?
* points: 1
* topics: rails, testing, oauth

##### !question

Why didn't we add this as an instance method on the User model?

##### !end-question

##### !placeholder

Why not make this  User model instance method?

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

Because we are not planning to use this method outside of testing, so it makes sense as a helper method for testing.  By putting the method in `test/test_helper.rb` we can use it in all of our test cases.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

##### Test: Returning User

Now the test itself:

```ruby
# test/controllers/users_controller_test.rb
require "test_helper"

describe UsersController do
  describe "auth_callback" do
    it "logs in an existing user and redirects to the root route" do
      # Count the users, to make sure we're not (for example) creating
      # a new user every time we get a login request
      start_count = User.count

      # Get a user from the fixtures
      user = users(:grace)

      # Tell OmniAuth to use this user's info when it sees
      # an auth callback from github
      OmniAuth.config.mock_auth[:github] = OmniAuth::AuthHash.new(mock_auth_hash(user))

      # Send a login request for that user
      # Note that we're using the named path for the callback, as defined
      # in the `as:` clause in `config/routes.rb`
      get auth_callback_path(:github)

      must_redirect_to root_path

      # Since we can read the session, check that the user ID was set as expected
      session[:user_id].must_equal user.id

      # Should *not* have created a new user
      User.count.must_equal start_count
    end

    it "creates an account for a new user and redirects to the root route" do
    end

    it "redirects to the login route if given invalid user data" do
    end
  end
end
```

Note that we do check `session[:user_id]` here. Rails controller tests do let us see what the `session` looks like after each request, and there's essentially no other way to verify the user is logged in. This may feel a little dirty, but for such a specialized case as this it's OK.

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: a3ac676e-1a0f-43e0-9736-705a85dc57a9
* title: What do the other tests look like?
* points: 1
* topics: rails, testing, oauth

##### !question

What do the other two tests for this controller action look like?

##### !end-question

##### !placeholder

Describe the other two tests

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

In the 1st test we fill the mock_auth_hash with a new user and then do a get request to the callback path and test the results.  In the 2nd test we can set the `uid` to `nil` making the new user invalid and do the same thing, testing the result.

```ruby
    it "creates an account for a new user and redirects to the root route" do
            # Count the users, to make sure we're not (for example) creating
      # a new user every time we get a login request
      start_count = User.count

      # Create a new user (but don't save it to the DB)
      user = User.new(provider: 'github',  uid: 554433, email: 'potus@whitehouse.gov' name: 'Joe Biden')

      # Tell OmniAuth to use this user's info when it sees
      # an auth callback from github
      OmniAuth.config.mock_auth[:github] = OmniAuth::AuthHash.new(mock_auth_hash(user))

      # Send a login request for that user
      # Note that we're using the named path for the callback, as defined
      # in the `as:` clause in `config/routes.rb`
      get auth_callback_path(:github)

      must_redirect_to root_path

      # find the new user in the DB
      user = User.find_by(uid: user.uid, provider: user.provider)
      # Since we can read the session, check that the user ID was set as expected
      session[:user_id].must_equal user.id

      # Should have created a new user
      User.count.must_equal start_count + 1
    end

    it "redirects to the login route if given invalid user data" do
      # Count the users, to make sure we're not (for example) creating
      # a new user every time we get a login request
      start_count = User.count

      # Create a new user without a uid
      user = User.new(provider: 'github',  uid: nil, email: 'don@trump_tower.com' name: 'Bunker Boy')

      # Tell OmniAuth to use this user's info when it sees
      # an auth callback from github
      OmniAuth.config.mock_auth[:github] = OmniAuth::AuthHash.new(mock_auth_hash(user))

      # Send a login request for that user
      # Note that we're using the named path for the callback, as defined
      # in the `as:` clause in `config/routes.rb`
      get auth_callback_path(:github)

      must_redirect_to root_path

      # find the new user in the DB
      user = User.find_by(uid: user.uid, provider: user.provider)
      
      # Make sure the user is nil
      expect(user).must_equal nil

      # Since we can read the session, check that the user ID was set as expected
      session[:user_id].must_equal nil

      # Should *not* have created a new user
      User.count.must_equal start_count
    end
```

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

##### Test: New User

```ruby
it "creates a new user" do
  start_count = User.count
  user = User.new(provider: "github", uid: 99999, name: "test_user", email: "test@user.com")

  OmniAuth.config.mock_auth[:github] = OmniAuth::AuthHash.new(mock_auth_hash(user))
  get auth_callback_path(:github)

  must_redirect_to root_path

  # Should have created a new user
  User.count.must_equal start_count + 1

  # The new user's ID should be set in the session
  session[:user_id].must_equal User.last.id
end
```

**Question:** Why did we create a new `User` here? Do you notice any repeated code?

#### Login Helper

Now that we've sorted out our new testing procedure for logging in, we should update the existing `perform_login` method so that it's available to all our tests. Open up `test/test_helper.rb` again and add the following at the bottom of `class ActiveSupport::TestCase`:

```ruby
# test/test_helper.rb
def perform_login(user = nil)
  user ||= User.first

  OmniAuth.config.mock_auth[:github] = OmniAuth::AuthHash.new(mock_auth_hash(user))
  get auth_callback_path(:github)

  return user
end
```

This method takes a `User`, tells OmniAuth to use that user's data as it's mock auth hash, and then sends the login request, exactly like we did in our tests above.

Refactoring our returning user test above to use this method, it would look like:

```ruby
# test/controllers/users_controller_test.rb
it "logs in an existing user" do
  start_count = User.count
  user = users(:grace)

  perform_login(user)
  must_redirect_to root_path
  session[:user_id].must_equal  user.id

  # Should *not* have created a new user
  User.count.must_equal start_count
end
```

Much neater!

## Refactor the other tests to use perform_login

Now refactor the other tests to use your `perform_login` method.

## What Did We Accomplish?

- Discussed the difference between _unit testing_ and _integration testing_
- Identified GitHub as a _dependency_ for our tests
  - Decided this is a problem we should do something about
- Discussed _mocking_ as a strategy for short-circuiting external dependencies
- Turned on OmniAuth mocking using the `setup` method in `test/test_helper.rb`
- Did some ground-work on our models
  - Defined fixtures
  - Added a method to turn a model back into a (mocked) auth hash
- Wrote tests for our login controller using fixture data
- Moved the login functionality to it's own test helper method, again in `test/test_helper.rb`

## Additional Resources

- [OmniAuth Integration Testing](https://github.com/omniauth/omniauth/wiki/Integration-Testing)
- [Integration Testing Docs](http://api.rubyonrails.org/classes/ActionDispatch/IntegrationTest.html)
- [DHH on mutating `session` in Rails 5](https://github.com/rails/rails/issues/23386)