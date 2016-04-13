# Sinatra and ActiveRecord

## Learning Objectives

- Explain the role of ActiveRecord in a web app
- Diagram the request / response lifecycle in a Sinatra app with AR
- Load Active Record in a Sinatra app
- Build RESTful routes to implement CRUD functionality in Sinatra
- Write ERB views to display AR models
- Write forms with attributes for ActiveRecord models
- Write forms that use a 'namespace' for parameters
- Use `_method` param to emulate PUT and DELETE requests

## Rest && Forms

## Outline


* Why AR + Sinatra
* Demo Tunr Solution
* Web app request / response diagram (MVC)

## Why ActiveRecord + Sinatra? (5)

Sinatra helps us build simple web applications quickly + easily. But we have no
way to store data. ActiveRecord lets us store data in a DB easily, but has no
web interface (only CLI).

Put the two together, and we can build an app with a *web interface* that is
backed by *data in a database*.

Demonstrate Tunr as an example of this.

## The MVC Paradigm - How Sinatra + AR Work Together (10/15)

1.  The user makes a request (clicking a link, typing in a URL). **Sinatra** receives the request.
2. **Sinatra** matches the request to the appropriate route (from all the routes we've defined).
3. That route contains a block of code to run in response (**controller** code). This code is written by us, and determines how to handle that request (unique to our app).
4. In most cases, the **controller** will use **ActiveRecord** methods to:
5. create/read/update/destroy data in the **database**.
6. Once the **controller** has done its job (and gotten any data it needs), it renders a **view** or redirects to a new **request**, which is sent back to the user (their browser) to be displayed.

![Sinatra MVC](images/sinatra_mvc2.jpg)


## App Organization

Our apps are starting to have lots of different dependencies. It's very important to start separating our concerns.

Here's a summary of the important folders / files in our app. It's important to note that some of these are expected by sinatra (`views`), and the rest are just convention to match Rails.

* **tunr**
  - **db**
    - seeds (sample data)
    - database configuration and connection files
  - **models**
    - classes that inherit from ActiveRecord
  - **controllers**
    - route definitions (`get "/artists" do`)
  - **views**
    - erb templates
    - *layout.erb*
      - a template for every view
      - calls `yield` (loads another template)
  - **public**
    - static assets
      - JS, CSS, images, fonts
  - *app.rb*
   - the entry point to our application
   - start the application with `ruby app.rb`
  - *console.rb*
    - interact with models and data in the database

For this lesson, we'll be using [Tunr](http://github.com/ga-dc/tunr_sinatra), so go ahead and clone it and change directories into the `tunr` folder.

### Goal - Seed the database.

In this repo are three files `artist_data.rb`, `song_data.rb`, `seeds.rb` and `console.rb`. Ultimately we want to use the `seeds.rb` file to seed our database and the `console.rb` file to be a REPL for our Sinatra app.

T & T (15/25)
With a partner, write down some facts about what these files do and what we need to get these file to work.

Here's some things to start thinking about:
- what does `artist_data.rb` and `song_data.rb` do on their own?
- what are the enumerators doing?(no need to go to specifics about syntax, what do you think is happening at a high level)
- what's `require_relative` and why is it in this file so much?
- what file(s) will we need to get this work?
- what file(s) will we need that aren't being required?
- what is `console.rb` for?


## 1. Data Model

Awesome, so everyone's ready to go, but before we rush in and start making files like crazy.

We often start building an app by describing the data model. We do this by creating an ERD, defining our schema according to that ERD, loading it into a Database, and then building our ActiveRecord classes (aka models).

### You Do: ERD (5/30)

Spend 5 minutes diagramming the ERD for Tunr.

### You Do: Define the Schema(5/35)

Create the schema file in the db directory.

```
$ touch db/schema.sql
```

Using our ERD, fill in `db/schema.sql` with SQL to create the Schema. Look at the readme for a guide to the attributes of each model.

### You Do: Create the DB & Load the Schema (5/40)

Create our PostgreSQL database:

`$ createdb tunr_db`

Load the schema:

`$ psql -d tunr_db < db/schema.sql`

### You Do: Define ActiveRecord Models (10/50)

Create files for your `Artist` and `Song` classes in the `models` folder.

Remember that the file names should be singular, to match the class name: `models/song.rb`, `models/artist.rb`

Is there any old code that we can leverage?

### I Do: Using `console.rb`

We've created a file for you called `console.rb`. Take a minute to read through it and see what it's doing.

This file isn't special, we've just written it so that it loads active record, connects to the database, loads our models, and then gives us a pry console.

We can use `console.rb` to test our AR models without loading up sinatra. It's just for playing around with them.

#### You Do: Play with `console.rb` (10/60)

Use `console.rb` to create 2 artists, and 4 songs, and associate the songs with artists.

`$ bundle exec ruby console.rb`

### Break (10/70)

#### We Do: Load Seed Data

Take a look at `db/seeds.rb`. We've taken some data from the iTunes API, and formatted it / saved it so we can load it into our database using AR.

Go ahead and run the seed script:
`$ bundle exec ruby db/seeds.rb`

Use the `console.rb` to verify that the data loaded correctly.

## 2. Setting up our sinatra app(10/80)

*You can proceed with your code, or checkout the `solution_step_1` branch to continue.*

Now that our database / data model is all setup, let's build a web app to let us interact with that data.

To begin, let's set up the main `app.rb` file, which will connect to the database, load our models, and define the homepage route.

#### We do: Setup `app.rb` to load files

Demonstrate using `require_relative` to load connection file:

```ruby
require 'sinatra'
require 'sinatra/reloader'
require 'active_record'

# Load the file to connect to the DB
require_relative 'db/connection'

# Load models
require_relative 'models/artist'
require_relative 'models/song'
```

# ATTENTION
From this point, I will be coding the I do portion of the new material in a wdi_app the same domain used in the active record class. You will continue to code on tunr. If at any point your code contains the words `wdi`, `instructor`, or `student` in any variant, you are doing something incorrectly.

> Reference the code in the I do's to complete the objectives for TUNR. Additionally there are solution branches for each step of this process, the solution branch will be linked at the end of each corresponding section.

The codebase I'll be working in has all of the same working parts as the one you're currently working with.

Step-by-step, you're going to add the 7 RESTful routes for artists, and the corresponding views. We're also going to use AR to make sure that the data is persisted.

### The Index Route - WDI App (15/95)

Now that we have the basic structure in place, let's start to put everything together. One of the most common features in the web today is the `index`. This is a feature that lists out collections. In your case you might want a feature to list out all of the artists for your app. That's what the index does.

In `app.rb`:

```ruby
get '/instructors' do
  @instructors = Instructor.all
  erb :"instructors/index"
end
```

There's one big problem with this, we don't actually have any views. Let's make a directory for them and the view we need for the index feature

```bash
$ mkdir views
$ mkdir views/instructors
$ touch views/instructors/index.erb
```

In `views/instructors/index.erb`:

```html
<h2>All Instructors</h2>
<% @instructors.each do |instructor|%>
  <div><%= instructor.first_name %></div>
<% end %>
```

If I run my application, `$ ruby app.rb`, and visit `http://localhost:4567/instructors` I can see all the instructors first names

### You Do - Index Route for Tunr (20/115)

### I do: Show Route - WDI app (10/125)

Another common feature in web apps is to "show" a singular object in all it's glory. In my case an instructor, in your case an artist.

```ruby
get '/instructors/:id' do
  @instructor = Instructor.find(params[:id])
  erb :"instructors/show"
end
```

> we are querying our database for an instructor with an id that we get from the url.

It's corresponding view `views/instructors/show`:

```html
<h2><a href="/instructors">All Instructors</a></h2>
<div class="instructor">
  Instructor First Name: <%= @instructor.first_name %>
  Instructor Age: <%= @instructor.age %>
</div>
```

In this show page we're just showing the instructors first name and that person's age.

We also added a link to go back to the index page for a little bit of better user experience here. We should go back and update the index route to link to the various show pages the instructors are associated with.

in `views/instructors/index`:

```html
<h2>All Instructors</h2>
<% @instructors.each do |instructor|%>
  <div class="instructor">
    <a href="/instructors/<%= instructor.id %>"><%= instructor.first_name %></a>
  </div>
<% end %>
```

## You do: Show Route - Tunr (10/135)

## REST
Representational State Transfer. REST is a convention that we, as developers, follow.

REST defines five main methods, each of which corresponds to one of the CRUD functionalities in a database.

| Method | Crud functionality |
|---|---|
|GET| read |
|POST| create |
|PUT| update |
|PATCH| update |
|DELETE| delete |

We've done `GET` requests, the next thing we're going to talk about is `POST` requests. There a little bit different in the way they get initiated.

With `GET` requests, a user types a URL into the browser or clicks on a link. A `POST` request traditionally only happens when the intended behavior of the request is to create something in the database.

### I Do: Creating things - WDI App (Rest of Class)

Another main feature of web applications is the ability to create new instances of our models(ie. adds rows to a database)

In order to do that we need to provide an interface to the user. That interface is HTML forms.

### An Aside - Forms

To show you a bit about forms and how they work, let's take a look at [this code](https://github.com/andrewsunglaekim/sinatra_simple_post)

This is a simple app that keeps track of first names. Let's checkout the relevant code

`app.rb`:

```ruby
get '/' do
  @first_names = first_names
  erb :index
end

post '/add_name' do
  first_names << params[:first_name]
  redirect "/"
end
```

`views/index.erb`:

```html
<h1>All Names</h1>
<% @first_names.each do |name| %>
  <div><%= name %></div>
<% end %>
<h2>Enter New Name Here and Hit Enter</h2>
<form action="add_name" method="post">
  <input name='first_name'>
</form>
```

> When you get some time, clone this repo down and play with the parameter values and see how it works.

When I enter a new name it gets added to the list! Let's take a closer look at the following code:

```html
<form action="add_name" method="post">
  <input name='first_name'>
</form>
```

3 things to take away from this form:
1. the action inside the form tag corresponds to the path of the `post` request in `app.rb`.
2. the method corresponds to the `post` HTTP verb of the request
3. the `input`'s name attribute corresponds to the parameter value used in the `post` request in `app.rb`


Keeping these things in mind, let's see what a post request to create objects looks like in a sinatra application.

In `app.rb`:

```ruby
post '/instructors' do
  @instructor = Instructor.create(params[:instructor])
  redirect "/instructors/#{@instructor.id}"
end
```

> if you're confused by params[:instructor] thats ok, we haven't gone over that yet. One thing we do know, is that params is a hash and `:instructor` is a key in it. Later, drop a `binding.pry` after the request to inspect the parameters once you've built out the form.

So we've defined the post request, but how do we make that request to our server? We can build a form that will submit post requests to that path. In order to do that, we need to create a new view.

### New Route - WDI app
You've seen this sort of thing before in `app.rb`:

```ruby
get '/instructors/new' do
  erb :"instructors/new"
end
```

(ST-WG) Do we need to pass any data into this view?

Here's the corresponding view `views/instructors/new`:

```html
<h2>New Artist</h2>

<form action="/instructors" method="post" >
  <label>Name:</label>
  <input name="instructor[first_name]">

  <label>Last Name:</label>
  <input name="instructor[last_name]">

  <label>Age:</label>
  <input name="instructor[age]">

  <input type="submit" value="Create">
</form>
```

Here are some important parts about this code:
- the parameter values being passed in this form are nested under the namespace of `instructor`
- the path for this request is `/instructors`(how convenient!)
- the method for this request is `post`

Thats it! Now we have the ability to create new instructors!

### Editing Models (Lab Reference)

Another common feature in web applications is the ability to modify data in a database. In order for the user to manipulate data, you(the developer), need to create an interface for them to do that. We need 2 routes for this feature, one that displays the interface to edit, and one that actually updates the database.

####  `edit/update` Routes

In `app.rb`:
```
get "/instructors/:id/edit" do
  @instructor = Instructor.find(params[:id])
  erb(:"instructors/edit")
end

put '/instructors/:id' do
  @instructor = Instructor.find(params[:id])
  @instructor.update(params[:instructor])
  redirect("/instructors/#{@instructor.id}")
end
```

Both of these routes first find an instructor based on parameters from the path. The `get` request renders a view for the user, that view will contain the interface for editing.

A big problem with `put` and `delete` requests is that they aren't native to HTML forms.

To make a `put` request, we need to 'fake it' in our form with a hidden input for the `_method` param:

```html
<form action="/artists/<%= @artist.id %>" method="post">
  <input type="hidden" name="_method" value="put">

  <label for="instructor[first_name]">First Name:</label>
  <input name="instructor[first_name]" value="<%= @instructor.first_name %>">

  <label for="instructorx[last_name]">Last Name:</label>
  <input name="instructor[last_name]" value="<%= @instructor.last_name %>">

  <label for="instructor[age]">Age:</label>
  <input name="instructor[age]" value="<%= @instructor.age %>">

  <input type="submit" value="Update">
</form>
```

> There's a little bit of subtle UI here. If editing a model, we should provide the current values of the model in the input fields.

Let's provide a link to the edit page in the `show` feature of our application. In `views/instructors/show.erb`, add this line of code:

```html
<a href='/instructors/<%= @instructor.id %>/edit'>Edit This Instructor</a>
```

> This is where it is now, but really we can put this anywhere so long as we can provide an instructor id.

### Deleting Models (Lab Reference)

Ever delete a tweet? Thats a feature! In web apps, you want to be able to allow users to delete things(things you think it's ok for them to delete...) from your database.

Heres the request we'll need in `app.rb`:

```ruby
delete '/instructors/:id' do
  @instructor = Instructor.find(params[:id])
  @instructor.destroy
  redirect("/instructors")
end
```

This is part of a view that initiates that request.

```html
<h2>Delete Instructor</h2>
<form action="/instructors/<%= @instructor.id %>" method="post">
  <input type="hidden" name="_method" value="delete">
  <input class="button-delete" type="submit" value="Delete!">
</form>
```

Where does this view go? Basically wherever you want the ability to delete an instructor so long as `@instructor` is the instructor that should be deleted when that request goes off. Sometimes it goes in the `edit` view other times it goes in the `index` view.
