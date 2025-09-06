# Chapter 20: Debugging - The Art of Finding That One Stupid Typo

Debugging is detective work where you're simultaneously the detective, the criminal, and the crime scene. The bug is there, hiding in your code, laughing at you. But with the right tools and techniques, you'll track it down, corner it, and fix it. Then you'll wonder how you ever made such an obvious mistake. Welcome to debugging!

## The Universal Debugging Tool: Puts

Let's start with the most basic debugging tool—the humble `puts` statement:

```ruby
def calculate_total(items)
  puts "Starting calculate_total with #{items.inspect}"
  
  subtotal = 0
  items.each do |item|
    puts "Processing item: #{item}"
    price = item[:price] * item[:quantity]
    puts "Price for this item: #{price}"
    subtotal += price
  end
  
  puts "Subtotal before tax: #{subtotal}"
  tax = subtotal * 0.08
  puts "Tax: #{tax}"
  
  total = subtotal + tax
  puts "Final total: #{total}"
  
  total
end

# When something goes wrong, the puts statements tell the story
items = [
  {name: "Book", price: 10, quantity: 2},
  {name: "DVD", price: 15, quantity: 1}
]

calculate_total(items)
```

## P and PP: Puts' Smarter Siblings

```ruby
# puts converts to string
puts [1, 2, 3]  # 1
                 # 2
                 # 3

# p shows inspect output
p [1, 2, 3]     # [1, 2, 3]

# pp pretty prints complex objects
require 'pp'
complex_hash = {
  user: {
    name: "Alice",
    address: {
      street: "123 Ruby Lane",
      city: "Codetown"
    },
    hobbies: ["coding", "reading", "debugging"]
  }
}

pp complex_hash
# {
#   :user=>{
#     :name=>"Alice",
#     :address=>{
#       :street=>"123 Ruby Lane",
#       :city=>"Codetown"
#     },
#     :hobbies=>["coding", "reading", "debugging"]
#   }
# }

# Awesome Print gem for even better output
require 'awesome_print'
ap complex_hash  # Colorized, formatted output
```

## The Debugger: Byebug

Byebug lets you pause execution and explore:

```ruby
require 'byebug'

def problematic_method(users)
  total = 0
  
  users.each do |user|
    byebug  # Execution stops here
    
    score = calculate_score(user)
    total += score
  end
  
  total
end

# When byebug stops:
# (byebug) user                    # Show current user
# (byebug) local_variables         # List all local variables
# (byebug) total                   # Show value of total
# (byebug) step                    # Step into calculate_score
# (byebug) next                    # Go to next line
# (byebug) continue                # Continue execution
# (byebug) backtrace               # Show call stack
# (byebug) up                      # Move up the call stack
# (byebug) down                    # Move down the call stack
# (byebug) list                    # Show surrounding code
```

## Pry: The Better REPL

Pry is like IRB on steroids:

```ruby
require 'pry'

def complex_calculation(data)
  result = data.map { |x| x * 2 }
  
  binding.pry  # Drops into Pry session here
  
  final = result.sum
  final
end

# In Pry session:
# [1] pry(main)> result
# => [2, 4, 6]
# [2] pry(main)> result.class
# => Array
# [3] pry(main)> ls         # List methods and variables
# [4] pry(main)> cd result  # Change context to result object
# [5] pry(Array)> self
# => [2, 4, 6]
# [6] pry(Array)> exit
```

## Better Errors for Web Apps

For web applications, the better_errors gem provides amazing error pages:

```ruby
# Gemfile
group :development do
  gem 'better_errors'
  gem 'binding_of_caller'  # Enables REPL in error page
end

# When an error occurs in development, you get:
# - Full stack trace
# - Source code preview
# - Local and instance variables
# - REPL console at the point of error
```

## Stack Traces: Reading the Map

Stack traces tell you where the error happened and how you got there:

```ruby
def method_a
  method_b
end

def method_b
  method_c
end

def method_c
  1 / 0  # Error happens here
end

method_a

# Stack trace:
# ZeroDivisionError: divided by 0
#   from debug.rb:10:in `/'
#   from debug.rb:10:in `method_c'
#   from debug.rb:6:in `method_b'
#   from debug.rb:2:in `method_a'
#   from debug.rb:13:in `<main>'

# Read from top to bottom:
# 1. Error type and message
# 2. Where it happened (line 10 in method_c)
# 3. How we got there (the call chain)
```

## Common Ruby Errors and What They Mean

```ruby
# NoMethodError: undefined method 'upcase' for nil:NilClass
# You're calling a method on nil
name = nil
name.upcase  # Error!

# Fix:
name&.upcase  # Safe navigation
name.upcase if name  # Guard clause

# NameError: undefined local variable or method 'foo'
# Variable or method doesn't exist
puts foo  # Error if foo isn't defined

# Fix:
foo = "bar"
puts foo

# ArgumentError: wrong number of arguments
def greet(name)
  "Hello, #{name}"
end
greet  # Error! Missing argument
greet("Alice", "Bob")  # Error! Too many arguments

# TypeError: no implicit conversion
"5" + 5  # Error! Can't add string and number

# Fix:
"5".to_i + 5  # Convert to integer
"5" + 5.to_s  # Convert to string

# SyntaxError: unexpected token
if true
  puts "Hello"
# Missing 'end'

# LoadError: cannot load such file
require 'non_existent_gem'  # Error!

# Fix:
# Install the gem or check the path
```

## Debugging Techniques

### Binary Search Debugging

When you don't know where the bug is:

```ruby
def complex_process(data)
  step1 = transform_data(data)
  puts "After step 1: #{step1.class}"  # Add checkpoint
  
  step2 = validate_data(step1)
  puts "After step 2: #{step2.class}"  # Add checkpoint
  
  step3 = process_data(step2)
  puts "After step 3: #{step3.class}"  # Add checkpoint
  
  step4 = format_output(step3)
  step4
end

# If error happens after step 2, you know it's in process_data
# Remove checkpoints before step 2, add more after
```

### Rubber Duck Debugging

Explain your code to a rubber duck (or anyone):

```ruby
# "Okay duck, this method takes an array of users"
# "It filters out inactive users"
# "Then it sorts them by... wait, I'm not sorting them!"
# "That's why the order is wrong!"

def get_active_users(users)
  users.select(&:active?)
  # Missing: .sort_by(&:name)
end
```

### Git Bisect for Regression Bugs

When something used to work:

```bash
git bisect start
git bisect bad                  # Current version is bad
git bisect good v1.0            # v1.0 was good

# Git checks out a commit in the middle
# Test if the bug exists
git bisect good  # or git bisect bad

# Git narrows down until it finds the commit that introduced the bug
```

## Logging for Production Debugging

```ruby
require 'logger'

class Application
  def initialize
    @logger = Logger.new('app.log')
    @logger.level = Logger::DEBUG
  end
  
  def process_order(order)
    @logger.info "Processing order #{order.id}"
    @logger.debug "Order details: #{order.inspect}"
    
    begin
      validate_order(order)
      @logger.debug "Order validated successfully"
      
      charge_payment(order)
      @logger.info "Payment charged: #{order.total}"
      
      ship_order(order)
      @logger.info "Order shipped"
      
    rescue ValidationError => e
      @logger.error "Validation failed: #{e.message}"
      raise
    rescue PaymentError => e
      @logger.error "Payment failed: #{e.message}"
      @logger.debug "Payment gateway response: #{e.response}"
      raise
    rescue => e
      @logger.fatal "Unexpected error: #{e.class} - #{e.message}"
      @logger.debug e.backtrace.join("\n")
      raise
    end
  end
end

# Log levels:
# DEBUG: Detailed information for diagnosing problems
# INFO: General informational messages
# WARN: Warning messages
# ERROR: Error messages
# FATAL: Fatal errors that cause program to abort
```

## Performance Debugging

```ruby
require 'benchmark'

# Measure execution time
time = Benchmark.realtime do
  expensive_operation
end
puts "Operation took #{time} seconds"

# Compare different approaches
Benchmark.bm do |x|
  x.report("approach 1:") { 1000.times { approach_1 } }
  x.report("approach 2:") { 1000.times { approach_2 } }
  x.report("approach 3:") { 1000.times { approach_3 } }
end

# Memory profiling
require 'memory_profiler'

report = MemoryProfiler.report do
  # Code to profile
end

report.pretty_print

# CPU profiling with ruby-prof
require 'ruby-prof'

RubyProf.start
# Code to profile
result = RubyProf.stop

printer = RubyProf::FlatPrinter.new(result)
printer.print(STDOUT)
```

## Debugging Async Code

```ruby
# Debugging threads
threads = []

5.times do |i|
  threads << Thread.new do
    puts "Thread #{i} starting"
    sleep(rand(3))
    puts "Thread #{i} done"
    
    # Debug thread issues
    Thread.current[:id] = i
    Thread.current[:result] = i * 2
  end
end

threads.each do |thread|
  thread.join
  puts "Thread #{thread[:id]} result: #{thread[:result]}"
end

# Debugging race conditions
require 'thread'

counter = 0
mutex = Mutex.new

threads = 10.times.map do
  Thread.new do
    1000.times do
      mutex.synchronize do
        temp = counter
        # Uncomment to see race condition:
        # sleep 0.0001
        counter = temp + 1
      end
    end
  end
end

threads.each(&:join)
puts "Counter: #{counter}"  # Should be 10000
```

## Debugging Tools and Gems

```ruby
# Debug gems to know:
gem 'pry-byebug'      # Combines pry and byebug
gem 'pry-rails'       # Pry for Rails console
gem 'better_errors'   # Better error pages
gem 'binding_of_caller' # REPL in error pages
gem 'awesome_print'   # Pretty printing
gem 'hirb'           # Formatted database output
gem 'bullet'         # N+1 query detection
gem 'rack-mini-profiler' # Performance debugging
gem 'flamegraph'     # Flame graphs for profiling

# System tools:
# - strace (Linux): System call tracing
# - dtrace (macOS): Dynamic tracing
# - tcpdump: Network debugging
# - top/htop: Process monitoring
# - lsof: Open files and sockets
```

## Debugging Checklist

When facing a bug, ask yourself:

1. **What did I expect to happen?**
2. **What actually happened?**
3. **What changed recently?**
4. **Can I reproduce it consistently?**
5. **What's the simplest test case?**
6. **Have I checked the obvious things?**
   - Typos
   - nil values
   - Off-by-one errors
   - Wrong variable names
   - Missing `end` statements
7. **Have I read the error message carefully?**
8. **Have I googled the exact error message?**

## Real-World Debugging Session

```ruby
# The bug: Users report getting wrong discount

class ShoppingCart
  def initialize
    @items = []
    @discount_codes = []
  end
  
  def add_item(item)
    @items << item
  end
  
  def add_discount(code)
    @discount_codes << code
  end
  
  def total
    # Step 1: Add debugging output
    puts "Calculating total for #{@items.length} items"
    
    subtotal = @items.sum(&:price)
    puts "Subtotal: #{subtotal}"
    
    # Step 2: Found the bug - using each instead of sum
    discount = 0
    @discount_codes.each do |code|
      puts "Applying discount code: #{code}"
      discount = code.discount_amount  # Bug! Should be +=
    end
    puts "Total discount: #{discount}"
    
    total = subtotal - discount
    puts "Final total: #{total}"
    
    total
  end
end

# The fix:
def total
  subtotal = @items.sum(&:price)
  discount = @discount_codes.sum(&:discount_amount)  # Fixed!
  subtotal - discount
end
```

## Debugging Philosophy

1. **Stay Calm**: Bugs are normal, not personal attacks
2. **Be Systematic**: Random changes rarely fix bugs
3. **Question Assumptions**: The bug is often where you're sure it isn't
4. **Take Breaks**: Fresh eyes see obvious problems
5. **Learn from Bugs**: Each bug teaches you something
6. **Document Solutions**: Future you will thank present you

## Your Turn: Debugging Challenges

1. **Mystery Bug**: Debug a program with a subtle logic error
2. **Performance Issue**: Find and fix a performance bottleneck
3. **Race Condition**: Debug a threading issue
4. **Memory Leak**: Track down a memory leak
5. **Integration Bug**: Debug an issue between multiple systems

## What You've Learned

You now know how to:
- Use puts, p, and pp for basic debugging
- Debug with byebug and pry
- Read and understand stack traces
- Recognize common Ruby errors
- Use logging effectively
- Profile performance issues
- Debug async and concurrent code
- Apply systematic debugging techniques

## What's Next?

You've learned to test your code and debug it when tests fail. In upcoming chapters, we'll explore more advanced topics and web development with Sinatra. The journey continues!

Remember: Debugging is a skill that improves with practice. Every bug you fix makes you better at finding the next one. Embrace debugging as a learning opportunity, not a frustrating chore. And always remember the programmer's prayer: "Please let it be a typo. Please let it be a typo. Please let it be a typo."