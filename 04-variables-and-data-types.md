# Chapter 4: Variables and the Art of Naming Things

Welcome to the chapter where we teach Ruby to remember things! Variables are like labeled boxes where you store data. Except unlike that box in your closet labeled "misc," you'll actually remember what's in these.

## What's a Variable?

A variable is just a name that points to a value. Think of it like a nametag at a party, except instead of "Hello, my name is Bob," it's "Hello, my name is user_age and I'm holding the number 25."

```ruby
age = 25
name = "Alice"
is_hungry = true
bank_balance = 0.42
hopes_and_dreams = nil
```

See? We just taught Ruby to remember five different things. Ruby's memory is now better than mine after a long debugging session.

## Ruby's Naming Convention: snake_case

Ruby uses snake_case for variables. Not camelCase, not PascalCase, not SCREAMING_SNAKE_CASE. Just gentle, lowercase snake_case.

```ruby
# Good Ruby style
user_name = "Bob"
account_balance = 1000
is_logged_in = true

# Bad Ruby style (but technically works)
userName = "Bob"        # This isn't JavaScript
AccountBalance = 1000   # This looks like a constant
ISLOGGEDIN = true      # WHY ARE WE YELLING
```

If you use camelCase in Ruby, somewhere in Japan, Matz sheds a single tear.

## The Rules of Variable Naming

Ruby has some rules about variable names. Break them, and Ruby will complain:

```ruby
# Valid variable names
name = "Alice"
age2 = 25
_private = "secret"
user_name = "bob"
こんにちは = "Hello"  # Ruby supports Unicode!

# Invalid variable names
2pac = "rapper"       # Can't start with a number
user-name = "bob"     # Hyphens aren't allowed (that's subtraction)
def = "something"     # Can't use reserved words
```

## Ruby's Type System: "What Type System?"

Ruby is dynamically typed, which means variables can hold any type of data and change their minds whenever they want:

```ruby
x = 42           # x is holding a number
x = "forty-two"  # now it's holding a string
x = [:a, :b, :c] # now it's holding an array
x = nil          # now it's holding nothing
```

This flexibility is either liberating or terrifying, depending on your background. If you're coming from Java, you might need a moment to breathe into a paper bag.

## The Basic Data Types

### Numbers: Integers and Floats

Ruby has two types of numbers: integers (whole numbers) and floats (decimal numbers):

```ruby
# Integers
age = 30
temperature = -5
population = 7_900_000_000  # Underscores for readability!
distance_to_moon = 384_400

# Floats
pi = 3.14159
bank_balance = 0.42
temperature = 98.6
chance_of_rain = 0.8

# Ruby can do math
sum = 10 + 5        # 15
difference = 10 - 5  # 5
product = 10 * 5     # 50
quotient = 10 / 5    # 2
remainder = 10 % 3   # 1
power = 2 ** 8       # 256

# Integer division gotcha!
puts 5 / 2    # 2 (not 2.5!)
puts 5.0 / 2  # 2.5 (there we go)
puts 5 / 2.0  # 2.5 (this works too)
```

**Pro tip**: That integer division thing trips up everyone. When in doubt, add `.0` to make it a float.

### Strings: Text and More Text

Strings are sequences of characters. Ruby loves strings almost as much as it loves making developers happy:

```ruby
# Different ways to create strings
name = "Alice"
name = 'Alice'
name = %q(Alice)
name = %Q(Alice)
name = <<~HEREDOC
  Alice
HEREDOC

# String interpolation (only works with double quotes)
name = "Alice"
age = 30
message = "#{name} is #{age} years old"  # "Alice is 30 years old"
message = '#{name} is #{age} years old'  # '#{name} is #{age} years old' (oops!)

# String methods (so many!)
text = "hello world"
text.upcase      # "HELLO WORLD"
text.capitalize  # "Hello world"
text.reverse     # "dlrow olleh"
text.length      # 11
text.include?("world")  # true
text.gsub("world", "Ruby")  # "hello Ruby"

# String concatenation
greeting = "Hello" + " " + "World"  # "Hello World"
greeting = "Hello" " " "World"      # "Hello World" (Ruby auto-concatenates)
greeting = "Hello" << " " << "World" # "Hello World" (more efficient)

# Multiline strings
poem = "Roses are red
Violets are blue
Ruby is awesome
And so are you"

# Better multiline strings
poem = <<~POEM
  Roses are red
  Violets are blue
  Ruby is awesome
  And so are you
POEM
```

### Symbols: The Weird Ones

Symbols are like strings, but immutable and more memory-efficient. They start with a colon:

```ruby
status = :active
direction = :north
mood = :happy

# Symbols are often used as hash keys
user = {
  :name => "Alice",
  :age => 30,
  :status => :active
}

# Or with the modern syntax
user = {
  name: "Alice",
  age: 30,
  status: :active
}

# Symbols vs Strings
"hello".object_id  # 123456 (different every time)
"hello".object_id  # 789012 (different object!)
:hello.object_id   # 234567 (same every time)
:hello.object_id   # 234567 (same object!)
```

Think of symbols as the name of something rather than the thing itself. Like the difference between the word "cat" and an actual cat.

### Booleans: True or False

Ruby has true and false, and that's it. No 0 and 1 nonsense:

```ruby
is_awesome = true
is_boring = false

# Boolean methods (convention: end with ?)
5.even?      # false
5.odd?       # true
[].empty?    # true
nil.nil?     # true
"hello".start_with?("h")  # true
```

### nil: The Absence of Value

`nil` represents nothing. It's not zero, it's not an empty string, it's just... nothing:

```ruby
result = nil
puts result  # (prints nothing)

# Checking for nil
if result.nil?
  puts "Got nothing"
end

# The safe navigation operator (Ruby 2.3+)
user = nil
user&.name  # nil (doesn't blow up!)
# Without &, this would error:
# user.name  # NoMethodError: undefined method `name' for nil:NilClass
```

## Truth and Falsehood in Ruby

Here's something crucial: in Ruby, everything is "truthy" except `false` and `nil`:

```ruby
# These are all truthy
if true then puts "yes" end     # prints "yes"
if 1 then puts "yes" end        # prints "yes"
if 0 then puts "yes" end        # prints "yes" (surprise!)
if "" then puts "yes" end       # prints "yes" (another surprise!)
if [] then puts "yes" end       # prints "yes"

# Only these are falsy
if false then puts "yes" end    # doesn't print
if nil then puts "yes" end      # doesn't print
```

This is different from many languages where 0 and empty strings are false. In Ruby, they're truthy! This will either delight or haunt you.

## Type Conversion: Shape-shifting Data

Ruby makes it easy to convert between types:

```ruby
# To string
42.to_s         # "42"
3.14.to_s       # "3.14"
true.to_s       # "true"
nil.to_s        # ""
[1, 2, 3].to_s  # "[1, 2, 3]"

# To integer
"42".to_i       # 42
"42.5".to_i     # 42 (truncates!)
"hello".to_i    # 0 (interesting choice, Ruby)
3.14.to_i       # 3
true.to_i       # NoMethodError (can't do this one)

# To float
"3.14".to_f     # 3.14
"42".to_f       # 42.0
42.to_f         # 42.0

# To array
"hello".chars   # ["h", "e", "l", "l", "o"]
(1..5).to_a     # [1, 2, 3, 4, 5]

# To symbol
"active".to_sym # :active
"user_name".intern # :user_name (same thing)
```

## Constants: Variables That Don't Vary

Constants in Ruby start with a capital letter. They're supposed to never change, but Ruby will let you change them (with a warning):

```ruby
PI = 3.14159
SPEED_OF_LIGHT = 299_792_458  # m/s
COMPANY_NAME = "RubyCorp"

# Ruby will warn but allow this
PI = 3.14  # warning: already initialized constant PI

# Class names are constants too
class User
  DEFAULT_ROLE = "member"
  MAX_LOGIN_ATTEMPTS = 3
end
```

## Variable Scope: Who Can See What

Ruby has different types of variables based on their prefix:

```ruby
# Local variables (no prefix)
name = "Local"

# Instance variables (@ prefix)
@name = "Instance"

# Class variables (@@ prefix)
@@name = "Class"

# Global variables ($ prefix - avoid these!)
$name = "Global"

# Constants (UPPERCASE)
NAME = "Constant"
```

We'll dive deeper into scope later, but for now, just use local variables (no prefix) and you'll be fine.

## Parallel Assignment: Ruby Magic

Ruby lets you assign multiple variables at once:

```ruby
# Basic parallel assignment
x, y = 10, 20
# x is now 10, y is now 20

# Swapping values (no temp variable needed!)
x, y = y, x
# x is now 20, y is now 10

# Ignoring values
first, _, third = [1, 2, 3]
# first is 1, third is 3, middle value ignored

# Splat operator
first, *rest = [1, 2, 3, 4, 5]
# first is 1, rest is [2, 3, 4, 5]

*most, last = [1, 2, 3, 4, 5]
# most is [1, 2, 3, 4], last is 5

first, *middle, last = [1, 2, 3, 4, 5]
# first is 1, middle is [2, 3, 4], last is 5
```

## Practical Example: A Variable Playground

Let's put it all together with a fun program:

```ruby
# variable_playground.rb

# User information
user_name = "Alex"
user_age = 25
user_balance = 1337.42
is_premium = true
favorite_colors = [:blue, :green, :purple]
last_login = nil

puts "="*50
puts " Welcome to the Variable Playground! "
puts "="*50

# String interpolation and methods
puts "\nUser Profile:"
puts "Name: #{user_name.upcase}"
puts "Age: #{user_age} (that's #{user_age * 365} days old!)"
puts "Account type: #{is_premium ? 'Premium' : 'Basic'}"
puts "Balance: $#{'%.2f' % user_balance}"

# Working with nil
if last_login.nil?
  puts "First time user - Welcome!"
  last_login = Time.now
else
  puts "Welcome back! Last login: #{last_login}"
end

# Type conversion fun
age_string = user_age.to_s
puts "\nYour age as a string: '#{age_string}' (type: #{age_string.class})"
puts "Your age in binary: #{user_age.to_s(2)}"
puts "Your age in hex: #{user_age.to_s(16)}"

# Playing with symbols and strings
status = :active
puts "\nAccount status: #{status}"
puts "Status uppercase: #{status.to_s.upcase}"
puts "Status class: #{status.class}"

# Parallel assignment magic
x, y, z = 100, 200, 300
puts "\nBefore swap: x=#{x}, y=#{y}, z=#{z}"

# Circular rotation!
x, y, z = y, z, x
puts "After rotation: x=#{x}, y=#{y}, z=#{z}"

# Constants (sort of)
MAX_ATTEMPTS = 3
attempts = 0

puts "\nYou have #{MAX_ATTEMPTS - attempts} login attempts remaining"

# Boolean logic
can_purchase = is_premium && user_balance > 0
puts "Can make purchase: #{can_purchase}"

# Fun with numbers
lucky_number = 7
puts "\nYour lucky number is #{lucky_number}"
puts "Is it even? #{lucky_number.even?}"
puts "Is it odd? #{lucky_number.odd?}"
puts "Next prime after #{lucky_number}: #{lucky_number.next}"

# The truth about Ruby's truthiness
things_to_test = [true, false, nil, 0, "", [], {}]
puts "\nTruthiness test:"
things_to_test.each do |thing|
  if thing
    puts "#{thing.inspect} is truthy"
  else
    puts "#{thing.inspect} is falsy"
  end
end

puts "\n" + "="*50
puts " Thanks for playing! "
puts "="*50
```

## Common Gotchas and How to Avoid Them

### 1. Integer Division
```ruby
# Wrong
average = sum / count  # Might lose decimals!

# Right
average = sum.to_f / count
```

### 2. String Interpolation Only Works with Double Quotes
```ruby
# Wrong
name = 'Bob'
message = 'Hello, #{name}'  # Outputs: Hello, #{name}

# Right
message = "Hello, #{name}"  # Outputs: Hello, Bob
```

### 3. nil vs empty
```ruby
# They're different!
name = nil   # No value at all
name = ""    # Empty string, but still a string

# Check appropriately
if name.nil?
  puts "No name provided"
elsif name.empty?
  puts "Name is empty"
end
```

### 4. Mutating Strings
```ruby
greeting = "Hello"
loud_greeting = greeting.upcase!  # ! means it modifies in place
puts greeting  # "HELLO" (original changed!)

# vs

greeting = "Hello"
loud_greeting = greeting.upcase  # No !, creates new string
puts greeting  # "Hello" (original unchanged)
```

## Your Turn: Variable Challenges

Try these exercises to solidify your understanding:

1. **Temperature Converter**: Create variables for Celsius and convert to Fahrenheit
2. **Name Formatter**: Take a full name and format it different ways
3. **Age Calculator**: Calculate someone's age in days, hours, and minutes
4. **Truth Detector**: Create a method that reports if something is truthy or falsy
5. **Type Juggler**: Take a number, convert it to various types and back

## What You've Learned

You now know:
- How to create and name variables (snake_case forever!)
- Ruby's basic data types (numbers, strings, symbols, booleans, nil)
- Type conversion (making data shape-shift)
- Truth and falsehood in Ruby (only nil and false are falsy)
- Parallel assignment (because why assign one at a time?)
- The difference between constants and variables (one complains when changed)

## What's Next?

In the next chapter, we'll learn about control flow—teaching Ruby to make decisions. We'll cover if statements, loops, and how to make your programs actually do different things based on conditions. It's where your code stops being a straight line and starts being interesting!

Remember: Variables are just labels for values. They're not scary, they're not complicated, they're just Ruby's way of remembering things so you don't have to. And unlike that password you forgot, Ruby never forgets what you put in a variable (until the program ends or you change it).

Now go practice! Create some variables with ridiculous names, store weird values in them, and see what happens. The worst that can happen is an error message, and those are just Ruby's way of teaching you something new.