## Fast tests in Ruby

London-Madrid, November, 2018

[Carlos I. Peña](http://linkedin.com/in/carlosipe) / [@carlosipe](https://github.com/carlosipe)

<a href="https://bridge-u.com" style="color:white">
<img src="img/bridge-u.svg" width="140" style="margin-top:3rem">
</a>

---

## Problem

- Slow CI wastes developer time.
- Slow local tests make TDD impossible

---

# Slow CI

- `git commit && git push`
- <div style="color: red"> Wait 10 minutes </div>
- ..or switch context and start a new thing
- (10 minutes later): <span style="color:red"> red tests </span> => switch context again 

---

## Slow local tests and TDD

Standard setup: `guard` or tests run after saving file
```pre
- Writes test. Save. Waits 3/4 seconds
- `uninitialized constant UserRegistrationService (NameError)`
- creates the file and the class. Hits save. Waits 3/4 seconds
- ``initialize': wrong number of arguments (given 1, expected 0) (ArgumentError)`
- Fix the error. Hits save. Waits 3/4 seconds
...
```

---

## Why is this important?

Our intuitions about time are wrong.
Becoming more efficient in our everyday work is our best move

- 15 minutes/day per developer
<br>
(being conservative)

---

10 developers: 2.5hs per day

---

1 week off per month (!)
<iframe src="https://giphy.com/embed/Kg22My7WG9vMc" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/summer-beach-vacation-Kg22My7WG9vMc">via GIPHY</a></p>

---

Being more realistic:

---

A few more projects per year
<iframe src="https://giphy.com/embed/YQitE4YNQNahy" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/YQitE4YNQNahy">via GIPHY</a></p>

---

The bigger the team the more we need to invest in efficiency.

---

Our test suite is going to be even slower. 

---

Red team has 10 people working full-time to make it <em style="font-size:48px">slower</em>

---

## This isn't the biggest problem:

<div class="fragment"> We are missing the benefits of unit testing </div>
<div class="fragment"> Unit tests ensure less coupled code</div>

---

## What is a unit test?

---

## Integrated test

I use the term integrated test to mean any test whose result (pass or fail) depends on the correctness of the implementation of more than one piece of non-trivial behavior.
(J.B. Rainsberger dixit)

---

## What is a unit?

- Anything that can change independently should be isolated
- Anything that is slow should be isolated.

---

## Slow dependencies

- <div class="fragment">Third party services</div>
- <div class="fragment">Rails</div>

---

## Subject under test vs collaborators

Testing state vs testing behaviour (message passing)

---

## Testing pyramid

<img src="img/the-testing-pyramid.png" />

---

## Our case:

- Browser tests (integration)
- Database test (Models/Queries) (unit)
- Services tests (unit)

---

## Gradual move

- Two test suites
- Different setup strategies

---

## Two test suites

- Two tests suites: fast and slow
- `rake fast_test`
- It doesn't make sense to have `norails_test` if we run it through `rails test`

---

## Different setup strategies. Consistency

---

## Services

- DB calls and third-party calls are replaced by test doubles
- It can use real "service" objects but it must avoid touching the db
or doing an HTTP request

---

## Models/Queries

- Setup: Probably factories
```ruby
# Given
user  = create(:user)
admin = create(:user, :admin)
# When
admins = User.admin
# Then
assert_includes(admins, admin)
refute_includes(admins, user)
```

---

## Integration

- Fixtures?
- Sql dumps?
- There's room for improvement here

---

# Thanks
