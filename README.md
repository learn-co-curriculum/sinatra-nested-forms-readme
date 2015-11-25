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

The first thing we need is to create the form. For later use in the controller, we'll call this file `new.erb`. 

Before we dive into the HTML, let's think about how we want to structure the data our controller action will receive. Typically, if we were just doing student information, we would expect the `params` hash to look something like this:

```ruby
params = {
  "name" => "Joe",
  "grade" => 9
}
```

But how do we handle a student **and** a course? Both course and student have a `name` attribute. If keys in hashes have to be unique, we can't have `name` twice. We could call our keys `student_name` and `course_name`, but that really isn't best practice. And how would it look with two courses? `course_one_name` and `course_two_name`? Suddenly our keys are getting messy.

Instead, we need to think about restructuring our `params` hash to have nested hashes. We can have one hash for all the student information:

```ruby
params = {
  "student" => {
    "name" => "Joe",
    "grade" => 9,
  }
}
```

Now we have a key `student`, which stores a hash of all the student's information, `name` and `grade`. This makes it easy for us to insert a second nested hash for the student's course. Let's go ahead and built out the HTML for this form:

```html
<form action="/student" method="post">
  Student Name: <input type="text" name="student[name]">
  Student Grade: <input type="text" name="student[grade]">
  <input type="submit">
</form>
```

We know this form is going to get submitted via a POST request and processed by a controller action. In this case, we've named the action `/student`. You'll notice the `name` attribute of the `input` tag is set up like `student[name]`. This way, when the form gets submitted, the params sent to the `/students` controller action will exactly like we designed.


Now, let's think about how we want a student's course to fit in the params hash:

```ruby
params = {
  "student" => {
    "name" => "Joe",
    "grade" => 9,
    "course" => {
      "name" => "US History",
      "topic" => "History"
    }
  }
}
```

In this hash, both the student and the course can have the key `name` because they're in different namespaces. Let's go ahead and build out the corresponding HTML for the form:

```html
<form action="/student" method="post">
  Student Name: <input type="text" name="student[name]">
  Student Grade: <input type="text" name="student[grade]">
  Course Name: <input type="text" name="student[course][name]">
  Course Topic: <input type="text" name="student[course][topic]">
  <input type="submit">
</form>
```

In this form, the information for the course name is set up as `student[course][name]`, giving us the nested hashes we designed in the `params` hash. We're first accessing the `student` key.

But this leaves us with a much bigger problem. How do we now handle **two** (or more!) courses?

Again, we need to restructure how we want the data coming in the `params` hash. The `course` key should store an array of nested hashes:

```ruby
params = { 
  "student"=> {
    "name"=>"vic",
    "grade"=>"12",
    "course"=> [
      {
        "name"=> "ap us history", 
        "topic"=>"history"
      }, 
      {
        "name"=>"ap human geography", 
        "topic"=>"history"
      }
    ]
  }
}
```



In this case, each course is an index of the array. This simple pattern is easy to mimic no matter what objects you're creating. It's much simpler than using keys `first_course`, `second_course`, `third_course`, etc.

The HTML for the form looks like this:

```html
<form action="/student" method="post">
  Student Name: <input type="text" name="student[name]">
  Student Grade: <input type="text" name="student[grade]">
  Course Name: <input type="text" name="student[course][][name]" />
  Course Topic: <input type="text" name="student[course][][topic]" />
  Course Name: <input type="text" name="student[course][][name]" />
  Course Topic: <input type="text" name="student[course][][topic]" />
  <input type="submit">
</form>
```

Now, we've added four more input fields, which will create TWO courses. Again, because the student has many courses in their schedule, we're nested the courses under the student `student[course][][name]`. This will create a key called `course` inside the `student` hash in the params. The `course` key will store an array of hashes, each with the details about the course details.



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







