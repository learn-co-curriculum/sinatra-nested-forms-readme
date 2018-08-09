# Nested Forms Readme

## Overview

In this lesson, we'll cover nested forms that can create multiple objects.

## Objectives

1.  Understand how to create models for each class of objects
2.  See how to structure data for a controller action to receive in order to
    handle multiple objects
3.  See how to structure the HTML in `.erb` files that handle nesting
4.  Understand how to create a view file that displays the objects back to the user
5.  Understand how to create two controller actions that serve up the form and
    process the data from the form

## Forms That Create Multiple Objects

In web apps, we use forms to create objects. When you fill out a form for a
dinner reservation on Open Table, you're creating a reservation object. When you
upload a photo to Instagram, you're creating an image object.

Those are examples of using forms to create a single object, but what if you
wanted to use a form to create more than one object? This is where nested forms
comes in.

Let's say we're the registrar's office at a school and it's the start of the
school year. We need to create each student and their course schedule. It would
be tedious to go through the steps to first create the student and then go
through the same steps again and again to create each of that student's courses.
Wouldn't it be nice to create the student **and** their courses in one go?

## The Models

To create these two different classes of objects, we need to create two models,
`Student` and `Course`.

#### `Student` class

Our `Student` class, with `name` and `grade` attributes, will look something
like this:

```ruby
class Student
  attr_reader :name, :grade

 STUDENTS = []

  def initialize(params)
    @name = params[:name]
    @grade = params[:grade]
    STUDENTS << self
  end

  def self.all
    STUDENTS
  end

end
```

In this model, we have an `attr_reader` for `name` and `grade`, and we set the
value of those attributes on initialization. We also set up the class method
`self.all`, which returns an array containing all of the students.

#### `Course` class

Now let's set up the model for the courses each student is taking.

```ruby
class Course
  attr_reader :name, :topic

  COURSES = []

  def initialize(args)
    @name = args[:name]
    @topic = args[:topic]
    COURSES << self
  end

  def self.all
    COURSES
  end
end
```

Here, exactly like with our `Student` class, we have an `attr_reader` for `name`
and `topic`, and we set the value of those attributes on initialization. We
also have a `self.all` class method to return all of the courses.

## Creating the Form

The first thing we need is to create the form. For later use in the controller,
we'll call this file `new.erb`.

Before we dive into the HTML, let's think about how we want to structure the
data our controller action will receive. Typically, if we were just doing student
information, we would expect the `params` hash to look something like this:

```ruby
params = {
  "name" => "Joe",
  "grade" => "9"
}
```

But how do we handle a student **and** a course? Both course and student have a
`name` attribute. If keys in hashes have to be unique, we can't have `name`
twice. We could call our keys `student_name` and `course_name`, but that really
isn't best practice. And how would the hash look with two courses?
`course_one_name` and `course_two_name`? Suddenly our keys are getting messy.

Instead, we need to think about restructuring our `params` hash to have nested
hashes. We can have one hash for all of the student information:

```ruby
params = {
  "student" => {
    "name" => "Joe",
    "grade" => "9",
  }
}
```

Now we have a `student` key that stores a hash containing a given student's
`name` and `grade`.

How would we create a hash like this in Ruby? Like so:

```ruby
my_hash = {}
my_hash["student"] = {}
my_hash["student"]["name"] = "Joe"
```

Thankfully, ERB provides a similar syntax. It handles that first level of
nesting, so instead of having to do `my_hash["student"]={}` we can just go
straight into the `student` hash. ERB assumes that the name of your top-level
hash is the first key, so the code to call the value associated with the
nested `"name"` key would be `student["name"]`.

This makes it easy for us to insert a second nested hash to hold the student's
course(s). Let's go ahead and build out the HTML for this form:

```html
<form action="/student" method="post">
  Student Name: <input type="text" name="student[name]">
  Student Grade: <input type="text" name="student[grade]">
  <input type="submit">
</form>
```

We know this form is going to get submitted via a POST request and processed by
a controller action. In this case, we've named the action `/student`. You'll
notice the `name` attribute of the `input` tag is set up as `student[name]`.
This way, when the form gets submitted, the `params` sent to the `/student`
controller action will look exactly as we planned.

Now, let's think about how we want a course to fit in a student's `params` hash:

```ruby
params = {
  "student" => {
    "name" => "Joe",
    "grade" => "9",
    "course" => {
      "name" => "US History",
      "topic" => "History"
    }
  }
}
```

In this hash, both `student` and `course` can have the key `name` because
they're in different namespaces.

Let's think about how we'd build this hash using Ruby:

```ruby
my_hash = {}
my_hash["student"] = {}
my_hash["student"]["name"] = "Joe"
my_hash["student"]["course"] = {}
my_hash["student"]["course"]["name"] = "US History"
my_hash["student"]["course"]["topic"] = "History"

my_hash
  => {"student"=>{"name"=>"Joe", "course"=>{"name"=>"US History", "topic"=>"History"}}}
```

Again, we can use the ERB syntax to set up our form. We can ignore the first
level of nesting, the `my_hash` portion, and just dive straight into `student`
and `course`, turning `my_hash["student"]["course"]["name"]` into
`student[course][name]`.

Let's go ahead and build out the corresponding HTML for the form:

```html
<form action="/student" method="post">
  Student Name: <input type="text" name="student[name]">
  Student Grade: <input type="text" name="student[grade]">
  Course Name: <input type="text" name="student[course][name]">
  Course Topic: <input type="text" name="student[course][topic]">
  <input type="submit">
</form>
```

In this form, a given `course`'s `name` value is stored in
`student[course][name]`, conforming to the nested design we outlined above. But
this leaves us with a much bigger problem. How do we handle **two** (or more!)
courses?

We need to once again restructure how we want to store data in the `params`
hash. To allow for multiple courses, the `courses` key should store an array of
nested hashes:

```ruby
params = {
  "student" => {
    "name" => "Vic",
    "grade" => "12",
    "courses" => [
      {
        "name" => "AP US History",
        "topic" => "History"
      },
      {
        "name" => "AP Human Geography",
        "topic" => "History"
      }
    ]
  }
}
```

This simple, nested pattern is easy to mimic no matter what type of object
you're creating. It's much simpler than creating a new key for each course,
e.g., `first_course`, `second_course`, `third_course`, etc.

The HTML for the form looks like this:

```html
<form action="/student" method="post">
  Student Name: <input type="text" name="student[name]">
  Student Grade: <input type="text" name="student[grade]">
  Course Name: <input type="text" name="student[courses][][name]">
  Course Topic: <input type="text" name="student[courses][][topic]">
  Course Name: <input type="text" name="student[courses][][name]">
  Course Topic: <input type="text" name="student[courses][][topic]">
  <input type="submit">
</form>
```

We removed the singular `student[course][name]` and `student[course][topic]`
inputs and replaced them with pairs of inputs that allow for the creation of TWO
courses. Again, because a `Student` can have multiple courses in their
schedule, we've nested each student's courses within their primary hash. This
creates a key called `courses` inside of the `student` hash in `params`. The
`courses` key will store an array of hashes, each containing course details.

This is where ERB syntax differs from Ruby. In Ruby, if you wanted a hash to
store an array, you would do something like this:

```ruby
my_hash = {}
my_hash["student"] = {}
my_hash["student"]["name"] = "Joe"
my_hash["student"]["courses"] = []
my_hash["student"]["courses"][0] = { "name" => "AP US History", "topic" => "History"}
my_hash["student"]["courses"][1] = { "name" => "AP Human Geography", "topic" => "History"}
```

To access the name of the first course, you would do something like:

```ruby
my_hash["student"]["courses"][0]["name"]
  => "AP US History"
```

ERB makes it even easier on us. Instead of manually indexing each entry, we can
use an empty array (`[]`) in our form view, and ERB will automagically index the
array for us, turning `my_hash["student"]["courses"][0]["name"]` into `student[courses][][name]`. The `[]` is some ERB magic that we just need to
accept and use. It saves us time and simplifies our code!

## The Display View

We need a way to display the objects back to the user (in this case the
registrar) once the student and their courses have been created. For later use
in the controller, we'll call this file `student.erb`.

```html
<h1>Student</h1>

<div class="student">
  <h3>Name: <%= @student.name %></h3><br>
  <h4>Grade: <%= @student.grade %></h4>
</div><br>

<h1>Courses</h1>
<% @courses.each do |course| %>
  <div class="course">
    <p>Name: <%= course.name %></p><br>
    <p>Topic: <%= course.topic %></p><br>
  </div><br>
<% end %>
```

In this view, we use the instance variable `@student` and the reader methods
`.name` and `.grade` to display the student's information.

We then iterate over `@courses` to display the name and topic of each course.

## The Controller

Now, we need two controller actions – one to serve up the form, and one to
process the data from the form.

In order to serve the form in the browser, we need a `GET` request:

```ruby
get '/' do
  erb :new
end
```

And now we need a way to process the input from the user and to display the
student and their courses. We process a form with a `POST` request:

```ruby
post '/student' do
  @student = Student.new(params[:student])

  params[:student][:courses].each do |details|
    Course.new(details)
  end

  @courses = Course.all

  erb :student
end
```

In this controller action, we first create a new `Student` using the info stored
in `params[:student]`, which contains the student's `name`, `grade`, and
`courses`.

Then we iterate over `params[:student][:courses]`, which is an array containing
a series of hashes that each store individual course information:

```ruby
[
  0 => {
    "name" => "AP US History",
    "topic" => "History"
  },
  1 => {
    "name" => "AP Human Geography",
    "topic" => "History"
  }
]
```

During the iterative process, we use the course values passed into the `.each`
block to create instances of our `Course` class. We store the instantiated
courses in the instance variable `@courses`, making the course information
available within our view, `student.erb`.

Finally, the controller action loads the erb file `student.erb`, and we can see
all of the newly-created student and course information in the browser.

<p class='util--hide'>View <a href='https://learn.co/lessons/sinatra-nested-forms-readme'>Sinatra Nested Forms</a> on Learn.co and start learning to code for free.</p>
