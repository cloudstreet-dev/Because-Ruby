# Chapter 6: Methods - Teaching Ruby New Tricks

Remember when you were a kid and you taught your dog to shake hands? Methods are like that, except instead of teaching a dog to shake, you're teaching Ruby to do whatever you want. And unlike your dog, Ruby will remember the trick forever and never demand treats.

## What's a Method?

A method is a reusable chunk of code that has a name. Instead of writing the same code over and over like some kind of digital Sisyphus, you write it once, give it a name, and call it whenever you need it.

```ruby
# Without methods (the sad way)
puts "Hello, Alice!"
puts "How are you today, Alice?"
puts "Nice weather we're having, Alice!"

puts "Hello, Bob!"
puts "How are you today, Bob?"
puts "Nice weather we're having, Bob!"

# With methods (the happy way)
def greet(name)
  puts "Hello, #{name}!"
  puts "How are you today, #{name}?"
  puts "Nice weather we're having, #{name}!"
end

greet("Alice")
greet("Bob")
greet("Charlie")
```

See? We just saved ourselves from repetitive strain injury.

## Defining Methods

In Ruby, you define a method with `def` and end it with `end` (Ruby really loves that word):

```ruby
# Simple method with no parameters
def say_hello
  puts "Hello!"
end

# Method with one parameter
def greet(name)
  puts "Hello, #{name}!"
end

# Method with multiple parameters
def introduce(first_name, last_name, age)
  puts "Hi, I'm #{first_name} #{last_name}, and I'm #{age} years old."
end

# Method with default parameters
def coffee_order(size = "medium", type = "latte")
  puts "One #{size} #{type}, coming up!"
end

# Call it different ways
coffee_order                        # One medium latte, coming up!
coffee_order("large")               # One large latte, coming up!
coffee_order("small", "cappuccino") # One small cappuccino, coming up!
```

## Return Values: Methods That Give Back

Every method in Ruby returns something. If you don't explicitly return something, Ruby returns the last expression evaluated:

```ruby
# Implicit return (the Ruby way)
def add(a, b)
  a + b  # This is returned automatically
end

result = add(3, 4)  # result is 7

# Explicit return (when you need to exit early)
def divide(a, b)
  return "Error: Can't divide by zero!" if b == 0
  a / b
end

# Multiple return values (Ruby is generous)
def min_max(numbers)
  [numbers.min, numbers.max]
end

min, max = min_max([3, 7, 2, 9, 1])
# min is 1, max is 9

# Returning nil (the polite way to return nothing)
def find_user(id)
  return nil if id < 0
  # ... database lookup code ...
end
```

## Method Arguments: The Flexible Way

Ruby gives you lots of ways to handle method arguments:

```ruby
# Required arguments
def greet(name)
  "Hello, #{name}!"
end

# Default arguments
def power(base, exponent = 2)
  base ** exponent
end

power(3)     # 9 (3²)
power(3, 3)  # 27 (3³)

# Keyword arguments (named parameters)
def create_user(name:, email:, age: 18)
  {
    name: name,
    email: email,
    age: age
  }
end

create_user(name: "Alice", email: "alice@example.com")
create_user(email: "bob@example.com", name: "Bob", age: 25)  # Order doesn't matter!

# Splat operator (variable number of arguments)
def sum(*numbers)
  numbers.reduce(0, :+)
end

sum(1)           # 1
sum(1, 2)        # 3
sum(1, 2, 3, 4)  # 10

# Double splat (keyword arguments capture)
def process_options(**options)
  options.each do |key, value|
    puts "#{key}: #{value}"
  end
end

process_options(color: "red", size: "large", quantity: 3)

# Combining everything (because Ruby)
def ultimate_method(required, optional = "default", *rest, keyword:, **options)
  puts "Required: #{required}"
  puts "Optional: #{optional}"
  puts "Rest: #{rest}"
  puts "Keyword: #{keyword}"
  puts "Options: #{options}"
end

ultimate_method("hello", "world", 1, 2, 3, keyword: "value", extra: "data", more: "stuff")
```

## Method Names: The Ruby Convention

Ruby has conventions for method names that tell you what the method does:

```ruby
# Predicate methods (return true/false) end with ?
def empty?
  @items.length == 0
end

def valid_email?(email)
  email.include?("@")
end

# Dangerous methods (modify in place) end with !
text = "hello"
text.upcase   # Returns "HELLO", text is still "hello"
text.upcase!  # Returns "HELLO", text is now "HELLO"

# Setter methods end with =
def name=(new_name)
  @name = new_name
end

person.name = "Alice"  # Calls the name= method

# Operator methods (yes, operators are methods!)
def +(other)
  @value + other.value
end

def ==(other)
  @value == other.value
end

# Now you can do: object1 + object2
```

## Blocks: Methods' Best Friend

Methods and blocks go together like peanut butter and jelly:

```ruby
# Method that takes a block
def repeat(times)
  times.times do
    yield  # Calls the block
  end
end

repeat(3) do
  puts "Beetlejuice"
end

# Method that yields values to a block
def each_even(limit)
  0.upto(limit) do |num|
    yield(num) if num.even?
  end
end

each_even(10) do |n|
  puts "Even number: #{n}"
end

# Checking if a block was given
def maybe_yield
  if block_given?
    yield
  else
    puts "No block provided"
  end
end

maybe_yield { puts "I'm a block!" }  # Prints: I'm a block!
maybe_yield                           # Prints: No block provided

# Capturing blocks as proc objects
def capture_block(&block)
  puts "Captured a block!"
  block.call if block
end

capture_block { puts "I was captured!" }
```

## Method Visibility: Public, Private, Protected

Ruby lets you control who can call your methods:

```ruby
class BankAccount
  def initialize(balance)
    @balance = balance
  end
  
  # Public methods (anyone can call)
  def deposit(amount)
    @balance += amount
    log_transaction("Deposit", amount)
  end
  
  def withdraw(amount)
    if sufficient_funds?(amount)
      @balance -= amount
      log_transaction("Withdrawal", amount)
    else
      puts "Insufficient funds"
    end
  end
  
  def balance
    @balance
  end
  
  private  # Everything below here is private
  
  def sufficient_funds?(amount)
    @balance >= amount
  end
  
  def log_transaction(type, amount)
    puts "[LOG] #{type}: $#{amount}"
  end
  
  protected  # These can be called by other instances of the same class
  
  def internal_transfer(amount)
    @balance -= amount
  end
end

account = BankAccount.new(100)
account.deposit(50)           # Works
account.balance               # Works
account.sufficient_funds?(25) # Error! Private method
```

## Method Chaining: The Elegant Way

Ruby methods can return `self` to enable chaining:

```ruby
class Pizza
  def initialize
    @toppings = []
    @size = "medium"
  end
  
  def size(size)
    @size = size
    self  # Return self for chaining
  end
  
  def add_topping(topping)
    @toppings << topping
    self
  end
  
  def bake
    puts "Baking a #{@size} pizza with #{@toppings.join(', ')}"
    self
  end
end

Pizza.new
  .size("large")
  .add_topping("pepperoni")
  .add_topping("mushrooms")
  .add_topping("extra cheese")
  .bake
```

## Recursive Methods: Methods That Call Themselves

Sometimes a method needs to call itself. This is called recursion, and it's either elegant or confusing, depending on your mood:

```ruby
# Classic factorial
def factorial(n)
  return 1 if n <= 1
  n * factorial(n - 1)
end

factorial(5)  # 120 (5 * 4 * 3 * 2 * 1)

# Fibonacci sequence
def fibonacci(n)
  return n if n <= 1
  fibonacci(n - 1) + fibonacci(n - 2)
end

# Print a nested structure
def print_nested(data, indent = 0)
  data.each do |key, value|
    print " " * indent
    if value.is_a?(Hash)
      puts "#{key}:"
      print_nested(value, indent + 2)
    else
      puts "#{key}: #{value}"
    end
  end
end

data = {
  name: "Alice",
  address: {
    street: "123 Ruby Lane",
    city: {
      name: "Codeville",
      zip: "12345"
    }
  }
}

print_nested(data)
```

## Method Aliases and Dynamic Methods

Ruby lets you create methods on the fly:

```ruby
# Method aliases
class String
  alias_method :yell, :upcase
end

"hello".yell  # "HELLO"

# Define methods dynamically
class DynamicClass
  ["red", "green", "blue"].each do |color|
    define_method "#{color}?" do
      @color == color
    end
    
    define_method "make_#{color}" do
      @color = color
    end
  end
end

obj = DynamicClass.new
obj.make_red
obj.red?    # true
obj.blue?   # false
```

## Practical Example: A Todo List Manager

Let's build something useful with methods:

```ruby
class TodoList
  def initialize(name)
    @name = name
    @items = []
    @completed = []
    puts "Created todo list: #{@name}"
  end
  
  def add(*tasks)
    tasks.each do |task|
      @items << {
        description: task,
        added_at: Time.now,
        priority: :normal
      }
    end
    puts "Added #{tasks.length} task(s)"
    self
  end
  
  def add_urgent(task)
    @items.unshift({
      description: task,
      added_at: Time.now,
      priority: :urgent
    })
    puts "Added urgent task: #{task}"
    self
  end
  
  def complete(index)
    return puts "Invalid task number" if index < 0 || index >= @items.length
    
    task = @items.delete_at(index)
    task[:completed_at] = Time.now
    @completed << task
    puts "Completed: #{task[:description]}"
    self
  end
  
  def list(show_completed: false)
    puts "\n#{@name} - Todo List"
    puts "=" * 40
    
    if @items.empty?
      puts "No pending tasks! 🎉"
    else
      puts "Pending Tasks:"
      @items.each_with_index do |task, i|
        priority_marker = task[:priority] == :urgent ? "🔴" : "⚪"
        puts "#{priority_marker} #{i}. #{task[:description]}"
      end
    end
    
    if show_completed && !@completed.empty?
      puts "\nCompleted Tasks:"
      @completed.each do |task|
        puts "✅ #{task[:description]}"
      end
    end
    
    stats
    self
  end
  
  def stats
    puts "\n📊 Stats: #{@items.length} pending, #{@completed.length} completed"
    completion_rate = total_tasks > 0 ? (@completed.length.to_f / total_tasks * 100).round : 0
    puts "Completion rate: #{completion_rate}%"
  end
  
  def clear_completed
    count = @completed.length
    @completed.clear
    puts "Cleared #{count} completed task(s)"
    self
  end
  
  private
  
  def total_tasks
    @items.length + @completed.length
  end
  
  def find_task(description)
    @items.find { |task| task[:description].downcase.include?(description.downcase) }
  end
end

# Using our todo list
list = TodoList.new("Weekend Chores")

list.add("Buy groceries", "Clean room", "Do laundry")
    .add_urgent("Pay bills")
    .list
    .complete(0)
    .complete(1)
    .list(show_completed: true)
```

## Method Best Practices

### 1. Single Responsibility
```ruby
# Bad - method does too many things
def process_user_data(user)
  validate_user(user)
  save_to_database(user)
  send_email(user)
  log_activity(user)
end

# Good - each method has one job
def process_user(user)
  return unless valid_user?(user)
  
  save_user(user)
  notify_user(user)
  log_user_activity(user)
end
```

### 2. Short and Sweet
```ruby
# Bad - too long
def calculate_everything
  # 100 lines of code
end

# Good - broken into smaller methods
def calculate_total
  subtotal = calculate_subtotal
  tax = calculate_tax(subtotal)
  shipping = calculate_shipping
  subtotal + tax + shipping
end
```

### 3. Descriptive Names
```ruby
# Bad
def calc(x, y)
  x * y * 0.1
end

# Good
def calculate_discount(price, quantity)
  price * quantity * 0.1
end
```

### 4. Consistent Return Types
```ruby
# Bad - sometimes returns string, sometimes nil
def find_user(id)
  return "User not found" if id < 0
  # ... lookup code
end

# Good - consistent return type
def find_user(id)
  return nil if id < 0
  # ... lookup code that returns user or nil
end
```

## Common Method Patterns

### The Factory Method
```ruby
class User
  def self.create_admin(name, email)
    new(name, email, role: "admin")
  end
  
  def self.create_guest
    new("Guest", "guest@example.com", role: "guest")
  end
end
```

### The Builder Pattern
```ruby
class EmailBuilder
  def initialize
    @email = {}
  end
  
  def to(recipient)
    @email[:to] = recipient
    self
  end
  
  def subject(text)
    @email[:subject] = text
    self
  end
  
  def body(text)
    @email[:body] = text
    self
  end
  
  def send
    puts "Sending email: #{@email}"
  end
end

EmailBuilder.new
  .to("user@example.com")
  .subject("Hello!")
  .body("How are you?")
  .send
```

### The Guard Clause
```ruby
def process(data)
  return if data.nil?
  return if data.empty?
  return unless data.valid?
  
  # Main logic here
end
```

## Your Turn: Method Challenges

1. **Password Strength Checker**: Create methods to validate password requirements
2. **Calculator Class**: Build a calculator with chainable operations
3. **Text Analyzer**: Methods to count words, sentences, and calculate reading time
4. **Game Character**: Create methods for attacks, defense, and special moves
5. **URL Shortener**: Methods to generate and decode short URLs

## What You've Learned

You now know how to:
- Define methods with various parameter types
- Use return values effectively
- Work with blocks and yields
- Control method visibility
- Chain methods for elegant code
- Create recursive methods
- Follow Ruby method naming conventions

## What's Next?

Next chapter, we'll dive into Arrays and Hashes—Ruby's primary data structures. You'll learn how to store collections of data and retrieve them efficiently. It's like learning to organize your closet, but for data!

Remember: Methods are the verbs of your program. They make things happen. Without methods, your code is just a bunch of nouns sitting around doing nothing. With methods, your code comes alive!

Now go write some methods. Make them small, make them focused, and make them do one thing well. Your future self (and anyone else who reads your code) will thank you.