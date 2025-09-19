# Chapter 12: Blocks, Procs, and Lambdas - Ruby's Secret Weapons

Blocks, procs, and lambdas are what make Ruby feel like magic. They're chunks of code you can pass around, store in variables, and execute later. Think of them as portable pieces of behavior—like takeout, but for code.

## Blocks: The Gateway Drug

You've been using blocks all along without realizing it. Every time you use `each`, `map`, or `times`, you're using a block:

```ruby
# Block with do...end (multiline)
5.times do
  puts "Hello!"
end

# Block with curly braces (single line)
5.times { puts "Hello!" }

# Blocks can take parameters
[1, 2, 3].each do |number|
  puts "Number: #{number}"
end

# Multiple parameters
hash = { a: 1, b: 2, c: 3 }
hash.each do |key, value|
  puts "#{key}: #{value}"
end

# Blocks can do complex things
result = [1, 2, 3, 4, 5].select do |n|
  n.even? && n > 2
end
# result is [4]
```

## Yield: Letting Blocks Into Your Methods

You can make your own methods accept blocks using `yield`:

```ruby
def greet
  puts "Hello!"
  yield  # Calls the block
  puts "Goodbye!"
end

greet { puts "Nice to meet you!" }
# Hello!
# Nice to meet you!
# Goodbye!

# Yield with parameters
def repeat(times)
  times.times do |i|
    yield(i)  # Pass i to the block
  end
end

repeat(3) { |n| puts "Iteration #{n}" }

# Check if block was given
def maybe_yield
  if block_given?
    yield
  else
    puts "No block provided"
  end
end

maybe_yield  # "No block provided"
maybe_yield { puts "Block!" }  # "Block!"

# Multiple yields
def sandwich
  puts "Top bread"
  yield
  puts "Lettuce"
  yield
  puts "Bottom bread"
end

sandwich { puts "Turkey" }
# Top bread
# Turkey
# Lettuce
# Turkey
# Bottom bread
```

## Capturing Blocks: The & Operator

You can capture a block as a proc object:

```ruby
def capture_block(&block)
  puts "Block class: #{block.class}"  # Proc
  block.call  # Execute the block
end

capture_block { puts "I'm a block!" }

# Store and reuse blocks
def make_repeater(&block)
  return block  # Return the proc
end

say_hi = make_repeater { puts "Hi!" }
say_hi.call  # "Hi!"
say_hi.call  # "Hi!"

# Pass blocks to other methods
def method_a(&block)
  puts "In method A"
  method_b(&block)
end

def method_b
  puts "In method B"
  yield if block_given?
end

method_a { puts "Block executed!" }
```

## Procs: Blocks You Can Save

A Proc is a saved block of code:

```ruby
# Creating procs
my_proc = Proc.new { puts "I'm a proc!" }
my_proc = proc { puts "Another way!" }

# Procs with parameters
greeter = Proc.new { |name| puts "Hello, #{name}!" }
greeter.call("Alice")  # "Hello, Alice!"
greeter.("Bob")        # Shorthand syntax
greeter["Charlie"]     # Another way

# Procs in variables
operations = {
  add: Proc.new { |a, b| a + b },
  subtract: Proc.new { |a, b| a - b },
  multiply: Proc.new { |a, b| a * b },
  divide: Proc.new { |a, b| a.to_f / b }
}

puts operations[:add].call(5, 3)      # 8
puts operations[:multiply].call(4, 7)  # 28

# Procs as method arguments
def apply_to_array(arr, &processor)
  arr.map(&processor)
end

doubler = Proc.new { |n| n * 2 }
puts apply_to_array([1, 2, 3], &doubler)  # [2, 4, 6]
```

## Lambdas: Procs with Stricter Rules

Lambdas are like procs but with two key differences:
1. They check the number of arguments
2. They return to the lambda, not the enclosing method

```ruby
# Creating lambdas
my_lambda = lambda { puts "I'm a lambda!" }
my_lambda = ->(x) { x * 2 }  # Stabby lambda syntax

# Lambdas check argument count
proc_greeter = Proc.new { |name| puts "Hello, #{name}!" }
lambda_greeter = lambda { |name| puts "Hello, #{name}!" }

proc_greeter.call  # "Hello, !" (no error, name is nil)
# lambda_greeter.call  # ArgumentError! (wrong number of arguments)

# Return behavior difference
def proc_return
  my_proc = Proc.new { return "Proc return" }
  my_proc.call
  "Method end"  # Never reached!
end

def lambda_return
  my_lambda = lambda { return "Lambda return" }
  my_lambda.call
  "Method end"  # This IS reached
end

puts proc_return    # "Proc return"
puts lambda_return  # "Method end"

# Lambdas with multiple arguments
calculator = ->(operation, a, b) do
  case operation
  when :add then a + b
  when :multiply then a * b
  when :power then a ** b
  else "Unknown operation"
  end
end

puts calculator.call(:add, 5, 3)       # 8
puts calculator.call(:power, 2, 10)    # 1024
```

## Closures: Remembering Context

Blocks, procs, and lambdas are closures—they remember the context where they were created:

```ruby
def make_counter
  count = 0
  
  lambda do
    count += 1  # Remembers count from outer scope
    puts "Count: #{count}"
  end
end

counter1 = make_counter
counter2 = make_counter

counter1.call  # Count: 1
counter1.call  # Count: 2
counter2.call  # Count: 1 (different closure)
counter1.call  # Count: 3

# More complex closure
def make_multiplier(factor)
  lambda { |n| n * factor }  # Remembers factor
end

times_2 = make_multiplier(2)
times_10 = make_multiplier(10)

puts times_2.call(5)   # 10
puts times_10.call(5)  # 50

# Closures can modify outer variables
total = 0
adder = lambda { |n| total += n }

adder.call(5)
adder.call(10)
puts total  # 15
```

## Blocks vs Procs vs Lambdas: When to Use Which?

```ruby
# Blocks: When you need simple, one-time behavior
[1, 2, 3].each { |n| puts n }

# Procs: When you need reusable blocks with flexible arguments
logger = Proc.new { |msg| puts "[LOG] #{msg}" }
logger.call("Starting")
logger.call("Processing", "extra args ignored")

# Lambdas: When you need strict argument checking and predictable returns
validator = ->(email) { email.include?("@") }
raise "Invalid email" unless validator.call(user_email)

# Real-world example: Callbacks
class EventEmitter
  def initialize
    @callbacks = Hash.new { |h, k| h[k] = [] }
  end
  
  def on(event, &block)
    @callbacks[event] << block
  end
  
  def emit(event, *args)
    @callbacks[event].each { |callback| callback.call(*args) }
  end
end

emitter = EventEmitter.new
emitter.on(:click) { puts "Clicked!" }
emitter.on(:click) { |x, y| puts "Clicked at #{x}, #{y}" }

emitter.emit(:click, 100, 200)
# Clicked!
# Clicked at 100, 200
```

## Method Objects: Methods as Procs

You can convert methods to procs:

```ruby
# Convert instance method to proc
class Greeter
  def say_hello(name)
    "Hello, #{name}!"
  end
end

greeter = Greeter.new
method_obj = greeter.method(:say_hello)
proc_version = method_obj.to_proc

puts proc_version.call("Alice")  # "Hello, Alice!"

# Symbol to_proc (the & shorthand)
numbers = [1, 2, 3, 4, 5]

# Long way
numbers.map { |n| n.to_s }

# Short way using symbol to_proc
numbers.map(&:to_s)  # Same result!

# How it works:
# &:to_s converts the symbol :to_s to a proc that calls to_s on its argument

# More examples
["hello", "world"].map(&:upcase)     # ["HELLO", "WORLD"]
[1.5, 2.7, 3.9].map(&:round)        # [2, 3, 4]
["  trim  ", " spaces "].map(&:strip) # ["trim", "spaces"]
```

## Curry: Partial Application

Currying lets you create new functions by partially applying arguments:

```ruby
# Basic currying
add = ->(a, b, c) { a + b + c }
curried_add = add.curry

add_5 = curried_add[5]        # Partially applied
add_5_and_10 = add_5[10]      # More partial application
result = add_5_and_10[2]      # 17

# Or chain it
result = add.curry[5][10][2]  # 17

# Practical example: Configuration
configure = ->(env, db, port, app_name) do
  puts "Starting #{app_name} in #{env} mode"
  puts "Database: #{db}, Port: #{port}"
end

# Partial application for different environments
dev_config = configure.curry["development"]["localhost:5432"]
prod_config = configure.curry["production"]["prod.db.com"]

dev_config[3000]["MyApp"]
prod_config[80]["MyApp"]

# Building specialized functions
multiply = ->(a, b) { a * b }.curry
double = multiply[2]
triple = multiply[3]

puts double[5]  # 10
puts triple[5]  # 15
```

## Practical Example: Building a DSL (Domain Specific Language)

Let's use blocks to create a simple HTML DSL:

```ruby
class HTMLBuilder
  def initialize
    @html = ""
    @indent_level = 0
  end
  
  def method_missing(tag_name, attributes = {}, &block)
    indent
    @html << "<#{tag_name}#{attributes_string(attributes)}>"
    
    if block_given?
      @html << "\n"
      @indent_level += 1
      instance_eval(&block)
      @indent_level -= 1
      indent
    end
    
    @html << "</#{tag_name}>\n"
  end
  
  def text(content)
    indent
    @html << "#{content}\n"
  end
  
  def to_s
    @html
  end
  
  private
  
  def indent
    @html << "  " * @indent_level
  end
  
  def attributes_string(attrs)
    return "" if attrs.empty?
    " " + attrs.map { |k, v| "#{k}=\"#{v}\"" }.join(" ")
  end
end

def html(&block)
  builder = HTMLBuilder.new
  builder.instance_eval(&block)
  builder.to_s
end

# Using our DSL
page = html do
  html do
    head do
      title { text "My Ruby Page" }
    end
    body do
      h1(class: "header") { text "Welcome!" }
      div(id: "content") do
        p { text "This is a paragraph." }
        ul do
          li { text "Item 1" }
          li { text "Item 2" }
          li { text "Item 3" }
        end
      end
    end
  end
end

puts page
```

## Advanced Example: Functional Programming Helpers

```ruby
module FunctionalHelpers
  # Compose functions
  def compose(*functions)
    ->(x) do
      functions.reverse.reduce(x) { |result, fn| fn.call(result) }
    end
  end
  
  # Pipe functions (opposite of compose)
  def pipe(*functions)
    ->(x) do
      functions.reduce(x) { |result, fn| fn.call(result) }
    end
  end
  
  # Memoization
  def memoize(fn)
    cache = {}
    
    ->(*args) do
      cache[args] ||= fn.call(*args)
    end
  end
  
  # Throttle function calls
  def throttle(fn, delay)
    last_call = nil
    
    ->(*args) do
      now = Time.now
      if last_call.nil? || (now - last_call) >= delay
        last_call = now
        fn.call(*args)
      end
    end
  end
  
  # Debounce function calls
  def debounce(fn, delay)
    timer = nil

    ->(*args) do
      timer&.exit
      timer = Thread.new do
        sleep delay
        fn.call(*args)
      end
    end
  end
end

include FunctionalHelpers

# Composition example
add_one = ->(x) { x + 1 }
double = ->(x) { x * 2 }
square = ->(x) { x ** 2 }

transform = compose(square, double, add_one)
puts transform.call(3)  # ((3 + 1) * 2)² = 64

# Pipe example (more readable order)
process = pipe(add_one, double, square)
puts process.call(3)  # Same result, clearer flow

# Memoization example
expensive = ->(n) do
  puts "Computing for #{n}..."
  sleep 1  # Simulate expensive operation
  n ** 2
end

fast = memoize(expensive)
puts fast.call(5)  # Computing for 5... (waits 1 second)
puts fast.call(5)  # Returns immediately (cached)
puts fast.call(6)  # Computing for 6... (new value)
```

## Block/Proc/Lambda Patterns

### The Builder Pattern
```ruby
class DatabaseConfig
  attr_accessor :name, :pool
end

class Configuration
  attr_accessor :host, :port, :timeout
  attr_reader :db_config

  def self.build(&block)
    config = new
    config.instance_eval(&block)
    config
  end

  def database(&block)
    @db_config = DatabaseConfig.new
    @db_config.instance_eval(&block)
  end
end

config = Configuration.build do
  self.host = "localhost"
  self.port = 3000
  self.timeout = 30

  database do
    self.name = "myapp"
    self.pool = 5
  end
end
```

### The Retry Pattern
```ruby
def with_retry(max_attempts = 3, &block)
  attempts = 0
  
  begin
    attempts += 1
    block.call
  rescue => e
    if attempts < max_attempts
      puts "Attempt #{attempts} failed: #{e.message}. Retrying..."
      sleep(attempts)  # Exponential backoff
      retry
    else
      raise e
    end
  end
end

with_retry(3) do
  # Code that might fail
  api_call
end
```

## Your Turn: Block Challenges

1. **Custom Iterator**: Build a method that yields every nth element
2. **Transaction Block**: Create a method that rolls back changes on error
3. **Benchmark Block**: Write a method that times how long a block takes
4. **Pipeline Builder**: Create a DSL for building data pipelines
5. **Event System**: Build a pub/sub system using blocks

## What You've Learned

You now understand:
- How blocks work and how to use yield
- The differences between blocks, procs, and lambdas
- Closures and variable scope
- Method to proc conversion and symbol to_proc
- Currying and partial application
- Building DSLs with blocks

## What's Next?

Next, we'll explore the Enumerable module in depth—Ruby's Swiss Army knife for working with collections. You'll see how blocks make Enumerable so powerful and learn techniques that will level up your Ruby code instantly.

Remember: Blocks, procs, and lambdas are what make Ruby feel like Ruby. They enable the elegant, expressive APIs that Ruby is famous for. Master them, and you'll write code that's not just functional, but beautiful.