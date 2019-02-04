## Aggregated data in ActiveRecord

London-Madrid, February, 2019

[Carlos I. Peña](http://linkedin.com/in/carlosipe) / [@carlosipe](https://github.com/carlosipe)

<a href="https://bridge-u.com" style="color:white">
<img src="img/bridge-u.svg" width="140" style="margin-top:3rem">
</a>

---

# The problem

---

[ Image removed due privacy concerns ]

---

```rb
class School < ActiveRecord::Base
  has_many :students
end

class Student < ActiveRecord::Base
  belongs_to :school
end


```

---

## The Rails Way

```rb
# Controller
@schools = School.all
# View
- @schools.each do |school|
  = "Name: #{school.name}"
  = "Students count: #{school.students.count}"
```

---

## Presenters

```rb
# Controller
@schools = School.all.map do |school|
  SchoolPresenter.new(school) 
end
# View
- @schools.each do |school|
  = "Name: #{school.name}"
  = "Students count: #{school.students_count}"
```

---

## Presenters

```rb
class SchoolPresenter
  def initialize(school)
    @school = school
  end

  def name
    school.name
  end

  def students_count
    school.students.count
  end
end
```

---

### but...

---

# N+1 queries 😱

---

## The Rails Way (without N+1)

```rb
# Controller
@schools = School.includes(:students)
# View
- @schools.each do |school|
  = "Name: #{school.name}"
  = "Students count: #{school.students.count}"
```

---

## Presenters (without N+1)

```rb
# Controller
@schools = School.includes(:students).map do |school|
  SchoolPresenter.new(school) 
end
# View
- @schools.each do |school|
  = "Name: #{school.name}"
  = "Students count: #{school.students_count}"
```

---

## is this better?

---

## nope

---

<h2> ¯\_(ツ)_/¯ </h2>

---

<ul>
  <li >N+1 queries are inefficient </li>
  <li class="fragment">But `includes` is worse 😱  </li>
</ul>

---

###  the numbers
<h3 class="fragment"> But let's see another option first </h3>

---

## Aggregated queries

```rb
# Controller
@schools = SchoolsListing.listed_schools # data objects
# Views
- @schools.each do |school|
  = "Name: #{school.name}"
  = "Students count: #{school.students_count}"
```

---

## Aggregated queries

```rb
class SchoolListing
  ListedSchool = Struct.new(:name, :students_count)

  def self.listed_schools
    School.
      joins(:students).
      group("schools.id").
      pluck("name",
            "count(students.id) as students_count").
      map {|data| ListedSchool.new(*data) }
  end
end
```

---

## Demo

https://github.com/carlosipe/activerecord-agg-queries

---

## Some notes

<ul>
  <li class="fragment">Don't "fix" N+1 with `includes` unless you're sure it's better </li>
  <li class="fragment">Both the N+1 or the `includes` increase time geometrically</li>
  <li class="fragment">That means you'll notice the problems in production</li>
  <li class="fragment">The database is always faster</li> 
</ul>

---

## More complex scenarios

- Nested counts
- Multiple counts (see page 1)

---

## Architecture concerns

- Queries and data requirements should come together
- Exposing a data object makes sure that there's only one place touching the db
  - Easy to improve performance
  - Easy to refactor (clear interfaces)

---

## Rails `counter cache` 
#### (thanks @MariaCheca)

<ul>
  <li class="fragment"> If it has the word `cache` , it's not a fix. It's a patch. </li>
  <li class="fragment"> It's a source of many problems (stale caches, callback hell)</li>
  <li class="fragment"> It's not enough: see match_students in first page</li>

---

## Thanks


