# Database Seeds

<iframe src="https://adaacademy.hosted.panopto.com/Panopto/Pages/Embed.aspx?id=e5a3a8f4-bad1-41cf-a2d2-ac5d0187b39f&autoplay=false&offerviewer=true&showtitle=true&showbrand=false&start=0&interactivity=all" height="405" width="720" style="border: 1px solid #464646;" allowfullscreen allow="autoplay"></iframe>

## Learning Goals

By the end of this lesson we will be able to answer:

- What are database seeds?
- Why would we need seeds?
- How do we create database seeds?
- What does the `rails db:seed` command do?

## What and Why

> **Database seeding is the initial seeding of a database** with data. **Seeding a database** is a process in which an initial set of data is provided to a database when it is being installed . It is especially useful **when we want to populate the database** with data we want to develop in future.

- [_Wikipedia article on Database Seeding_](https://en.wikipedia.org/wiki/Database_seeding)

_Seeds_ are pieces of data that you configure to use as starter data for your web application. Seeds might be a few sample blog posts, or a few possible tasks for your task list.

Specifically in a Rails application, because "data" is typically accessed through Model objects, we'll use seeds to create model objects that save to the database. It's good to set up some seeds to start with so that you don't have to manually add Model objects through the `rails console` or wait until your object creation form works.

## Creating Database Seeds in Rails

Rails applications come pre-installed with an empty seed file located in each app at `db/seeds.rb`. This file, `db/seeds.rb`, is usually a Ruby script configured to populate the database with specific pieces of data. The script run with the command `$ rails db:seed`

The seed file is a "normal" ruby file, and it has access to the Model objects that we set up in our application. If we want to create a number of seeded objects, we can create a list with the properties for each object, then iterate over it, creating objects as we go.

### Reading Through an Example

This is a simplified example which assumes that we have already created an `Author` model object which has a `name` and a `bio_url` field in the schema.

```ruby
# db/seeds.rb

authors = [
  {
    name: "Margot Lee Shetterly",
    bio_url: "https://en.wikipedia.org/wiki/Roxane_Gay"
  },
  {
    name: "Sandi Metz",
    bio_url: "https://en.wikipedia.org/wiki/Sandi_Metz"
  },
  {
    name: "Octavia E. Butler",
    bio_url: "https://en.wikipedia.org/wiki/Octavia_E._Butler"
  },
  {
    name: "Jim Butcher",
    bio_url: "https://en.wikipedia.org/wiki/Jim_Butcher"
  }
]

authors.each do |author|
  Author.create(author)
end


books = [
  {
    title: "Practical Object Oriented Programming in Ruby",
    description: "This book explores Object-Oriented concepts in Ruby.",
    author_id: (Author.find_by name: "Sandi Metz").id,
    publication_date: 2020
  },
  {
    title: "An Untamed State",
    description: "The novel explores the interconnected themes of race, privilege, sexual violence, family, and the immigrant experience. An Untamed State is often referred to as a fairy tale because of its structure and style, especially in reference to the opening sentence, \"Once upon a time, in a far-off land, I was kidnapped by a gang of fearless yet terrified young men with so much impossible hope beating inside their bodies it burned their very skin and strengthened their will right through their bones,\" and the author's exploration of the American dream and courtship of Mireille's parents",
    author_id: (Author.find_by name: "Margot Lee Shetterly").id,
    publication_date: 2014
  },
  {
    title: "Storm Front",
    description: "Dirty Harry Potter",
    author_id: (Author.find_by name: "Jim Butcher").id,
    publication_date: 2000
  },    
]

books.each do |book|
  Book.create(book)
end
```

Once this seed file is ready to go, we run the Rails seed command in the terminal to run the seed script, which populates the database.

```bash
$ rails db:seed
```

If this command runs successfully, we can go into the `rails console` to verify that the data was set up successfully.

```bash
$ rails console
```

```bash
Author.all
```

## Our More Complex Example

Seeds files are just Ruby scripts! Feel free to configure them however you'd like, with as much specificity, output (with `puts`), and ancillary files/gems as you need.

[Here is an example seed file that we can use at this moment](db-seeds-sample.resource.md) that assumes the following things:

- There exists a model named `Book` with the following attributes:
  - `title` of type `string`
  - `author_id` as a foreign key to `Author`
  - `description` of type `string`
  - `publication_date` of type `integer`
- There exists a model named `Author` with the following attributes:
  - `name` of type `string`

<details style="max-width: 700px; margin: auto;">

<summary>
  If you don't yet have the `publication_date` column on the books table, run and commit this migration
</summary>

```bash
$ rails g migration
add_column :books, :publication_date, :integer
```

</details>

<br/>

This assumes that there are no validations on presence on any field.

<!-- >>>>>>>>>>>>>>>>>>>>>> BEGIN CHALLENGE >>>>>>>>>>>>>>>>>>>>>> -->
<!-- Replace everything in square brackets [] and remove brackets  -->

### !challenge

* type: short-answer
* id: d5a240a3-5bc9-48cd-8976-3d6caeb2ef99
* title: Why `Author`s before `Book`s?
* points: 1
* topics: rails, rails-seeds

##### !question

**Question:** This specific script requires that `Author`s be seeded before `Book`s. Why does order matter for this script? How could we have avoided this dependency in our seed script?

##### !end-question

##### !placeholder

Why is order important?

##### !end-placeholder

##### !answer

/.+/

##### !end-answer

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, users can see after a failed attempt) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
##### !explanation

Our source data (which is a hard-coded array of hashes) says that the books data references authors by name. The script assigns the relationship between a `Book` and an `Author` using the `book.author = Author.find_by(name: ...)` line. Therefore, for `Author.find_by` to not return `nil`, we'd need to create `Author`s first.

If we didn't want this dependency, we could do any of these options or more:

- restructure our hard-coded data
- update the script to assign `Author`s to `Book`s after all data was created.

##### !end-explanation

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->

## More about Databases: Clearing the Database

Most seed files focus on adding new records of Models. If you want to delete all the data in the database (be very very careful) and reseed ,you can use reset command:

```bash
$ rails db:reset
```

### Stay Up to Date

As a project evolves over time, don't forget that any change to models may mean an update is needed for the seeds file.

## Summary

In this lesson we looked at the concept of _seeding a database_.  In this the development database is given a starting or _seeded with_ an initial set of data.  This means we do not have to manually enter data to experiment with.  

In Rails seed data is created through the `seeds.rb` file in the `db` folder.  This is a normal Ruby file which can execute the same commands we put earlier in to the Rails console.  

We can run the seeds file with the command `rails db:seed`.  If we get into a point where we need to wipe our development database and revert to the seed data we can run `rails db:reset`.  

## Resources

Seeds can work with any number of Model objects that you have configured, not just one!

- [RailsZilla Seed Example](http://www.railszilla.com/rails-seed-data-example/rails)