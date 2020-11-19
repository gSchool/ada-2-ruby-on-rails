# Creating an API

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?pid=f7ae91bc-4d0f-4f03-b6cc-ac77013c20be&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals

By the end of this lesson we should be able to...

- Develop a Rails API to provide CRUD functionality
- Use HTTP status codes to communicate outcomes
- Test get requests in a Rails API application

### Introduction

Previously, we learned how Rails can render JSON and how we can test that JSON response. Now, our goal is to use model data in our JSON response.

## Listing Pets

So currently our controller is responding with:

```ruby
def index
  render json: { ready_for_lunch: "yessss" }, status: :ok
end
```

Our tests are passing.  However our *job* was to provide data about pets!  So we can add another test to ensure that our controller index action is providing pet data.

```ruby
  # pets_controller_test.rb
  it "responds with an array of pet hashes" do
    # Act
    get pets_path

    # Get the body of the response
    body = JSON.parse(response.body)

    # Assert
    expect(body).must_be_instance_of Array
    body.each do |pet|
      expect(pet).must_be_instance_of Hash

      required_pet_attrs = ["id", "name", "species", "age", "owner"]

      expect(pet.keys.sort).must_equal required_pet_attrs.sort
    end
  end
```

The test above:

1. Performs a get request to `/pets`
1. Takes the body of the server's response and parses the JSON into regular Ruby arrays and hashes.
1. Ensures that the response is an array
1. Ensures that each response has only the id, name, species, age, and owner fields
    - These fields are strings because JSON doesn't know about symbols and returns all the keys as strings.

The test fails and now we can edit our controller to match.  

### Responding With Model Data

So our test wants a list of pets, luckily Rails knows how to convert a model collection into JSON.  Sweet!  We can do so by passing the list of pets into the render method.

```ruby
  # pets_controller.rb
  def index
    pets = Pet.all
    render json: pets, status: :ok
  end
```

Rails converted all the pets into JSON like this:

```json
[{"id":1054079548,"name":"Alligator","species":"Dog","age":10,"owner":"Sarah","created_at":"2020-05-20T23:49:35.198Z","updated_at":"2020-05-20T23:49:35.198Z"}]
```

Very handy!  Our tests are _almost_ passing.  We just need a way to exclude some fields.

So we can adjust our controller:

```ruby
  # pets_controller.rb
  def index
    pets = Pet.all.as_json(only: [:id, :name, :age, :owner, :species])
    render json: pets, status: :ok
  end
```

All models in Rails have an `.as_json` method to convert the model data into JSON and this method can be used to limit which fields are returned.

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: eb247a43-9eb1-418c-88a8-3b1c8e036841
* title: Why not use @pets?
* points: 1
* topics: Rails, Rails-api

##### !question

Why did we use `pets` instead of `@pets` for our variable name?
##### !end-question

##### !placeholder

Why not use instance variables?

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

The `@` makes a regular variable into an instance variable. Rails (through some clever programming) _sort of_ treats a view as a method call from the controller. Sort of.

In this case, we aren't rendering a view, so there's no need to pass this value along by forcing it to be an instance variable.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

### Covering the Edges

We can also include a test for when there are no pets in the database:

```ruby
  # pets_controller_test.rb
  it "will respond with an empty array when there are no pets" do
    # Arrange
    Pet.destroy_all

    # Act
    get pets_path
    body = JSON.parse(response.body)

    # Assert
    expect(body).must_be_instance_of Array
    expect(body).must_equal []
  end
```

## Showing Pet Details

Working on you rown, and following the same pattern we used for `index`, implement the `show` endpoint.  Then check your answer with your teammate on this week's project.

Questions to consider:

- How will `show` be different than `index`?
- How will this endpoint be accessed?
  - HTTP verb
  - URI
- What fields should be returned?
- What should the API do if the client asks for a pet that doesn't exist?
  - Status code
  - Response body, **Hint**, you can respond with JSON without an available model.
- What test cases might be useful for this endpoint?
  - There should be at least one test already in the project but see if you should add more
- How do the two endpoints we've implemented so far compare to similar functionality in a non-API Rails app?

When you finish, you can check your implementation against the appropriate branch of our implementation.

## Learning Comprehension Questions

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: 94212e71-ad57-4db9-8a91-f1f53f0b4115
* title: Edge-case with `show` action
* points: 1
* topics: Rails, Rails-api

##### !question

What is an edge-case you should test in the `show` action?

##### !end-question

##### !placeholder

Edge-case for the show action?

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

You should test to see if it responds properly if the pet does not exist?

```ruby
  it "should respond with not_found if no pet matches the id" do
    
    # Act
    get pet_path(-1)

    body = JSON.parse(response.body)
    must_respond_with :not_found

    # Then you can test for the json in the body of the response    
    expect(body["ok"]).must_equal false
    expect(body["errors"]).must_include "Not Found"
  end
```
##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: d948c355-f0b7-46d0-92d8-8bc1aa7e92a9
* title: Difference between API Controller tests & Web App Controller tests
* points: 1
* topics: rails, rails-apis

##### !question

How does a test for an API's `show` action differ from a regular web application?

##### !end-question

##### !placeholder

Differences in show test in an API and web app?

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

Some similarities in between tests for `show` in an API and a Web App:

- Both make the same request to the normal verb-path pattern
- Both test the response code
- Both require tests for existing items and items without the given id

Some **differences** include:

- An API tests the **body** of the response.  
  - In a web application the response includes waaay too much HTML/CSS/JS to test adequately.  Plus HTML/CSS change quite regularly, so it's not worth the efford to test them.
  - In an API, the JSON structure does not and should not change regularly (as it might break applications that depend on the format of the JSON).

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

## What Have We Accomplished

- Built an _API_ - a web server that serves JSON for machines rather than HTML for humans
- Responded to requests for the `index` and `show` action with JSON

## Response Codes

Response codes and their corressponding Rails symbols.

- 200 - :ok
- 201 - :created
- 204 - :no_content
- 400 - :bad_request
- 401 - :unauthorized
- 403 - :forbidden
- 404 - :not_found
- 500 - :internal_server_error

## Resources

- [`.as_json` documentation](http://api.rubyonrails.org/classes/ActiveModel/Serializers/JSON.html#method-i-as_json)
- [ActiveModel Serializers](http://railscasts.com/episodes/409-active-model-serializers)
- [blog post by thoughtbot about serialization](http://robots.thoughtbot.com/better-serialization-less-as-json)
- [Rails API Development Guide](http://edgeguides.rubyonrails.org/api_app.html)
- [Testing a Rails API](https://www.learnhowtoprogram.com/rails/building-an-api/testing-a-rails-api)
