# Nested Forms Readme

In web apps, we use forms to create objects. When you fill out a form for a dinner reservation on Open Table, you're creating a reservation object. When you upload a photo to Instagram, you're creating an image object. 

Those are examples of using forms to create a single object, but what if you wanted to use a form to create more than one object? How would you do that? This is where nested forms comes in.

Let's say we're the registrar's office at a school and it's the start of the school year. We need to create each student and their class schedule It would be tedious to go through a process that made go through the steps to first create the student, and then go through the same step again and again to create each of their courses. Wouldn't it be nice to create the student **and** their courses in one go? 

## The Models

To create these two different classes of objects, we need to create two models, `Student` and `Course`.

### Student Class

Our student class, with name and grade attributes will look something like this:

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

In this model, we have an attr_reader for `name` and `grade`, and we set the value of those attributes on initialization. We also set up a class method `self.all` which returns an array of all the students

### Course Class

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

Here, we have a reader for `name` and `topic` and we set the value of those attributes on initialization. We also have the class method `self.all` to return all the courses.

## Creating The Form

The first thing we need is to create the form. For later use in the controller, we'll call this file `new.erb`. We'll start with a basic HTML form:

```html
<form action="/student" method="post">
  <input type="submit">
</form>
```

We know this form is going to get submitted via a POST request and processed by a controller action. In this case, we've named the action `/student`.

Now let's go ahead and create the part of the form to create the student:

```html
<form action="/student" method="post">
  Student Name: <input type="text" name="student[name]">
  Student Grade: <input type="text" name="student[grade]">
  <input type="submit">
</form>
```

You'll notice the `name` attribute of the `input` tag is set up like `student[name]`. This way, when the form gets submitted, the params sent to the `/students` controller action will look like this:

```ruby
params = { 
  "student"=> {
    "name"=> "vic",
    "grade"=>"12"
  }
}
```

`params` is storing a hash called `student` and that hash is storing a nested hash with the student's name and grade.

Next, let's go ahead and finish out the form with their course schedule:

```html
<form action="/student" method="post">
  Student Name: <input type="text" name="student[name]">
  Student Grade: <input type="text" name="student[grade]">
  Course Name: <input type="text" name="student[course][0][name]" />
  Course Topic: <input type="text" name="student[course][0][topic]" />
  Course Name: <input type="text" name="student[course][1][name]" />
  Course Topic: <input type="text" name="student[course][1][topic]" />
  <input type="submit">
</form>
```

Now, we've added four more input fields, which will create TWO courses. Again, because the student has many courses in their schedule, we're nested the courses under the student `student[course][0][name]`. This will create a key called `course` inside the `student` hash in the params. 

The params hash will look like this:

```ruby
params = { 
  "student"=> {
    "name"=>"vic",
    "grade"=>"12",
    "course"=>{
      "0"=>{
        "name"=> "ap us history", 
        "topic"=>"history"
      }, 
      "1"=>{
        "name"=>"ap human geography", 
        "topic"=>"history"
      }
    }
  }
}
```

## The Display View

We need a way to display the objects back to the user (in this case the registar) once the student and their courses have been created. For later use in the controller, we'll call this file `student.erb`.

```html
<h1>Student</h1>

<div class="student">
  <h3>Name: <%= @student.name %></h3><br>
  <h4>Height: <%= @student.grade %></h4>
</div><br>

<h1>Classes</h1>
<% @courses.each do |course| %>
  <div class="class">
    <p>Name: <%= course.name %></p><br>
    <p>Type: <%= course.topic %></p><br>
  </div><br>
<% end %>
```

In this view, we're using the instance variable `@student` and the reader methods `name` and `grade` to display the student's information.

We're then iterating over `@courses` to display the name and topic of each class.

## The Controller

Now, we need two controller actions - one to serve up the form and one to process the data from the form.

In order to serve the form in the browser, we need a GET request:

```ruby
get '/' do
  erb :new
end
```

And now we need a way to process the input from the user, and to display the student and their classes. We process a form with a POST request:

```ruby
post '/student' do
  @student = Student.new(params[:student])

  params[:student][:course].each do |course, details|
    Course.new(details)
  end

  @classes = Course.all

  erb :student
end
```

In this controller action, we're creating a new student using the `params[:student]`, which just pulls the information about `name` and `grade`. 

`params[:student][:course]` gives us a series of hashes that store each individual course information:

```ruby
{ 
  "0"=>{
    "name"=>"AP US HIStory", 
    "topic"=>"history"
  }, 
  "1"=>{
    "name"=>"ap human geography", 
    "topic"=>"history"
  }
}
```

We can iterate over those nested hashes using `.each` and then use the values to create two instances of our `Course` class.

Lastly, this controller action loads the erb file `student.erb`








<a href='https://learn.co/lessons/sinatra-nested-forms-readme' data-visibility='hidden'>View this lesson on Learn.co</a>
