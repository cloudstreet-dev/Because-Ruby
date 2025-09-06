# Chapter 16: Ruby Gets Fancy - The Features That Make Other Languages Jealous

Ruby has features that make programmers from other languages stop and say, "Wait, you can DO that?" These aren't just party tricks—they're powerful tools that, when used wisely, can make your code more expressive, flexible, and elegant. This chapter is about the fancy stuff: metaprogramming, DSLs, and Ruby magic that seems like sorcery until you understand how it works.

## Metaprogramming: Code That Writes Code

Metaprogramming is writing code that manipulates code at runtime. It's like being able to rewrite the rules of the game while you're playing it.

### define_method: Creating Methods on the Fly

```ruby
class DynamicClass
  # Create getter methods dynamically
  ['name', 'age', 'email'].each do |attribute|
    define_method(attribute) do
      instance_variable_get("@#{attribute}")
    end
    
    define_method("#{attribute}=") do |value|
      instance_variable_set("@#{attribute}", value)
    end
  end
end

obj = DynamicClass.new
obj.name = "Alice"
puts obj.name  # "Alice"

# More advanced: create methods based on data
class APIClient
  ENDPOINTS = {
    users: '/api/users',
    posts: '/api/posts',
    comments: '/api/comments'
  }
  
  ENDPOINTS.each do |name, path|
    define_method("get_#{name}") do |id = nil|
      url = id ? "#{path}/#{id}" : path
      make_request(:get, url)
    end
    
    define_method("create_#{name.to_s.singularize}") do |data|
      make_request(:post, path, data)
    end
  end
  
  private
  
  def make_request(method, url, data = nil)
    puts "#{method.upcase} #{url} with #{data.inspect}"
    # Actual HTTP request would go here
  end
end

client = APIClient.new
client.get_users        # GET /api/users
client.get_posts(42)    # GET /api/posts/42
```

### method_missing: The Catch-All

When Ruby can't find a method, it calls `method_missing` as a last resort:

```ruby
class FlexibleHash
  def initialize
    @attributes = {}
  end
  
  def method_missing(name, *args)
    # Setter method (ends with =)
    if name.to_s.end_with?('=')
      @attributes[name.to_s.chomp('=')] = args.first
    # Getter method
    elsif @attributes.key?(name.to_s)
      @attributes[name.to_s]
    # Query method (ends with ?)
    elsif name.to_s.end_with?('?')
      key = name.to_s.chomp('?')
      !@attributes[key].nil? && @attributes[key] != false
    else
      super  # Pass to original method_missing
    end
  end
  
  def respond_to_missing?(name, include_private = false)
    name.to_s.end_with?('=', '?') || @attributes.key?(name.to_s) || super
  end
end

obj = FlexibleHash.new
obj.name = "Ruby"
obj.awesome = true
puts obj.name       # "Ruby"
puts obj.awesome?   # true
puts obj.missing?   # false
```

### Class Macros: DSL Building Blocks

```ruby
module Validatable
  def self.included(base)
    base.extend(ClassMethods)
  end
  
  module ClassMethods
    def validates(attribute, **options)
      @validations ||= []
      @validations << {attribute: attribute, options: options}
      
      define_method :valid? do
        self.class.validations.all? do |validation|
          validate_attribute(validation[:attribute], validation[:options])
        end
      end
    end
    
    def validations
      @validations || []
    end
  end
  
  private
  
  def validate_attribute(attribute, options)
    value = send(attribute)
    
    if options[:presence] && value.nil?
      return false
    end
    
    if options[:format] && !value.to_s.match?(options[:format])
      return false
    end
    
    if options[:length]
      min = options[:length][:minimum]
      max = options[:length][:maximum]
      return false if min && value.to_s.length < min
      return false if max && value.to_s.length > max
    end
    
    true
  end
end

class User
  include Validatable
  
  attr_accessor :name, :email, :age
  
  validates :name, presence: true, length: {minimum: 2}
  validates :email, presence: true, format: /@/
  validates :age, presence: true
end

user = User.new
user.name = "A"
user.email = "invalid"
user.age = 25
puts user.valid?  # false (name too short, email invalid format)
```

## Building Domain-Specific Languages (DSLs)

Ruby's flexibility makes it perfect for creating DSLs—mini-languages for specific domains:

### Configuration DSL

```ruby
class Configuration
  def self.configure(&block)
    config = new
    config.instance_eval(&block)
    config
  end
  
  def server(url = nil)
    @server = url if url
    @server
  end
  
  def port(number = nil)
    @port = number if number
    @port
  end
  
  def database(&block)
    @database ||= DatabaseConfig.new
    @database.instance_eval(&block) if block
    @database
  end
  
  class DatabaseConfig
    attr_accessor :host, :name, :pool
    
    def host(value = nil)
      @host = value if value
      @host
    end
    
    def name(value = nil)
      @name = value if value
      @name
    end
    
    def pool(size = nil)
      @pool = size if size
      @pool
    end
  end
end

# Using the DSL
config = Configuration.configure do
  server 'https://api.example.com'
  port 3000
  
  database do
    host 'localhost'
    name 'myapp_development'
    pool 5
  end
end

puts config.server  # "https://api.example.com"
puts config.database.name  # "myapp_development"
```

### Testing DSL

```ruby
class TestFramework
  def self.describe(description, &block)
    suite = TestSuite.new(description)
    suite.instance_eval(&block)
    suite.run
  end
  
  class TestSuite
    def initialize(description)
      @description = description
      @tests = []
      @before_each = nil
    end
    
    def before(&block)
      @before_each = block
    end
    
    def it(description, &block)
      @tests << {description: description, block: block}
    end
    
    def run
      puts "#{@description}"
      @tests.each do |test|
        @before_each&.call
        begin
          test[:block].call
          puts "  ✓ #{test[:description]}"
        rescue => e
          puts "  ✗ #{test[:description]}: #{e.message}"
        end
      end
    end
  end
end

# Using the testing DSL
TestFramework.describe "Calculator" do
  before do
    @calc = Calculator.new
  end
  
  it "adds two numbers" do
    raise "Failed" unless @calc.add(2, 3) == 5
  end
  
  it "subtracts two numbers" do
    raise "Failed" unless @calc.subtract(5, 3) == 2
  end
end
```

## Advanced Block Techniques

### Block Local Variables

```ruby
# Variables in block parameters are local to the block
x = 10
[1, 2, 3].each do |x|  # This x shadows the outer x
  puts x  # 1, 2, 3
end
puts x  # Still 10

# Explicit block-local variables
total = 0
[1, 2, 3].each do |num; temp|  # temp is block-local
  temp = num * 2
  total += temp
end
# temp doesn't exist here
```

### Proc Composition

```ruby
# Ruby 2.6+ proc composition
add_one = proc { |x| x + 1 }
multiply_two = proc { |x| x * 2 }

# Composition with >>
transform = add_one >> multiply_two
transform.call(3)  # (3 + 1) * 2 = 8

# Composition with <<
transform = multiply_two << add_one
transform.call(3)  # Same result, different order

# Building pipelines
class Pipeline
  def initialize
    @steps = []
  end
  
  def add_step(&block)
    @steps << block
    self
  end
  
  def process(input)
    @steps.reduce(input) { |data, step| step.call(data) }
  end
end

pipeline = Pipeline.new
  .add_step { |x| x.strip }
  .add_step { |x| x.downcase }
  .add_step { |x| x.gsub(/[^a-z]/, '') }

pipeline.process("  Hello, World!  ")  # "helloworld"
```

## Refinements: Monkey Patching with Manners

Refinements let you modify classes in a limited scope:

```ruby
module StringExtensions
  refine String do
    def shout
      upcase + "!!!"
    end
    
    def words
      split(/\s+/)
    end
  end
end

# Without using the refinement
"hello".shout  # NoMethodError

# Using the refinement
class Message
  using StringExtensions
  
  def self.process(text)
    puts text.shout
    puts "Word count: #{text.words.length}"
  end
end

Message.process("hello world")  # "HELLO WORLD!!!" and "Word count: 2"

# Outside the class, refinement doesn't apply
"hello".shout  # Still NoMethodError
```

## Method Hooks: Watching Things Happen

```ruby
module Trackable
  def self.included(base)
    base.extend(ClassMethods)
  end
  
  module ClassMethods
    def method_added(name)
      return if @adding_method  # Prevent infinite loop
      return if name.to_s.end_with?('_without_tracking')
      
      @adding_method = true
      
      alias_method "#{name}_without_tracking", name
      
      define_method(name) do |*args, &block|
        puts "Calling method: #{name}"
        start_time = Time.now
        result = send("#{name}_without_tracking", *args, &block)
        puts "Method #{name} took #{Time.now - start_time} seconds"
        result
      end
      
      @adding_method = false
    end
  end
end

class Calculator
  include Trackable
  
  def add(a, b)
    sleep 0.1  # Simulate work
    a + b
  end
  
  def multiply(a, b)
    sleep 0.2  # Simulate work
    a * b
  end
end

calc = Calculator.new
calc.add(2, 3)      # Logs method call and timing
calc.multiply(4, 5)  # Logs method call and timing
```

## ObjectSpace: Exploring Ruby's Universe

```ruby
# Count all objects
puts "Total objects: #{ObjectSpace.count_objects[:TOTAL]}"

# Find all instances of a class
class Person
  attr_reader :name
  
  def initialize(name)
    @name = name
  end
end

alice = Person.new("Alice")
bob = Person.new("Bob")

people = ObjectSpace.each_object(Person).to_a
puts "Found #{people.length} Person objects"
people.each { |person| puts "  - #{person.name}" }

# Track object allocation
require 'objspace'

ObjectSpace.trace_object_allocations_start

str = "Hello, World!"
arr = [1, 2, 3]

puts ObjectSpace.allocation_sourcefile(str)
puts ObjectSpace.allocation_sourceline(str)

ObjectSpace.trace_object_allocations_stop

# Memory size of objects
ObjectSpace.memsize_of("small")  # Small memory
ObjectSpace.memsize_of("x" * 1000)  # Larger memory
```

## Fiber: Lightweight Concurrency

```ruby
# Basic fiber
fiber = Fiber.new do
  puts "Hello from fiber"
  Fiber.yield "First yield"
  puts "Back in fiber"
  Fiber.yield "Second yield"
  puts "Fiber ending"
  "Final value"
end

puts fiber.resume  # "Hello from fiber" then "First yield"
puts fiber.resume  # "Back in fiber" then "Second yield"
puts fiber.resume  # "Fiber ending" then "Final value"

# Producer-consumer with fibers
def producer
  Fiber.new do
    10.times do |i|
      puts "Producing: #{i}"
      Fiber.yield i
    end
  end
end

def consumer(producer_fiber)
  while producer_fiber.alive?
    value = producer_fiber.resume
    puts "Consuming: #{value}"
    sleep 0.1
  end
end

consumer(producer)

# Fiber-based enumerator
class FibonacciEnumerator
  def each
    a, b = 0, 1
    loop do
      yield a
      a, b = b, a + b
    end
  end
end

fib = FibonacciEnumerator.new
enum = fib.enum_for(:each)

10.times do
  puts enum.next
end
```

## TracePoint: Debugging Magic

```ruby
# Trace method calls
trace = TracePoint.new(:call) do |tp|
  puts "Called: #{tp.defined_class}##{tp.method_id}"
end

trace.enable
"hello".upcase
[1, 2, 3].sum
trace.disable

# Trace specific events
class Calculator
  def add(a, b)
    a + b
  end
end

trace = TracePoint.new(:call, :return) do |tp|
  case tp.event
  when :call
    puts "Calling #{tp.method_id} with #{tp.binding.local_variables}"
  when :return
    puts "Returning #{tp.return_value}"
  end
end

trace.enable
calc = Calculator.new
result = calc.add(3, 4)
trace.disable

# Performance profiling with TracePoint
class Profiler
  def self.profile(&block)
    results = Hash.new(0)
    
    trace = TracePoint.new(:call, :return) do |tp|
      if tp.event == :call
        results[tp.method_id] -= Time.now.to_f
      else
        results[tp.method_id] += Time.now.to_f
      end
    end
    
    trace.enable
    block.call
    trace.disable
    
    results.sort_by { |_, time| -time }.each do |method, time|
      puts "#{method}: #{(time * 1000).round(2)}ms"
    end
  end
end

Profiler.profile do
  1000.times do
    "hello".upcase.reverse.downcase
  end
end
```

## Singleton Classes and Eigenclasses

```ruby
# Every object has a singleton class
str = "hello"
singleton = str.singleton_class

# Add methods to singleton class
singleton.define_method(:shout) do
  upcase + "!"
end

str.shout  # "HELLO!"
"other".shout  # NoMethodError

# Class methods are singleton methods
class MyClass
  def self.class_method
    "I'm a class method"
  end
end

# This is actually
MyClass.singleton_class.define_method(:class_method) do
  "I'm a class method"
end

# Eigenclass chain
obj = "hello"
puts obj.class  # String
puts obj.singleton_class  # #<Class:#<String:0x00...>>
puts obj.singleton_class.superclass  # String
```

## Prepend: The Method Resolution Modifier

```ruby
module Loggable
  def process(data)
    puts "Before processing: #{data}"
    result = super(data)
    puts "After processing: #{result}"
    result
  end
end

class DataProcessor
  prepend Loggable  # Inserts module BEFORE the class in hierarchy
  
  def process(data)
    data.upcase
  end
end

processor = DataProcessor.new
processor.process("hello")
# Before processing: hello
# After processing: HELLO
# => "HELLO"

# Method resolution order
p DataProcessor.ancestors
# [Loggable, DataProcessor, Object, Kernel, BasicObject]
```

## Binding: Capturing Context

```ruby
def create_binding(x)
  y = 20
  binding  # Captures current context
end

b = create_binding(10)

# Evaluate code in that context
eval("x + y", b)  # 30

# Access variables
b.local_variable_get(:x)  # 10
b.local_variable_set(:x, 50)
eval("x", b)  # 50

# ERB templates use bindings
require 'erb'

template = ERB.new("Hello <%= name %>, you are <%= age %> years old")
name = "Alice"
age = 30
result = template.result(binding)
puts result  # "Hello Alice, you are 30 years old"
```

## Delegators and Forwardable

```ruby
require 'delegate'
require 'forwardable'

# SimpleDelegator
class TimestampedArray < SimpleDelegator
  def initialize(array)
    super(array)
    @created_at = Time.now
  end
  
  def <<(item)
    puts "Adding #{item} at #{Time.now}"
    super
  end
  
  def created_at
    @created_at
  end
end

arr = TimestampedArray.new([1, 2, 3])
arr << 4  # Logs addition
puts arr.created_at

# Forwardable
class TodoList
  extend Forwardable
  
  def_delegators :@items, :size, :empty?, :first, :last
  def_delegator :@items, :push, :add_item
  
  def initialize
    @items = []
    @completed = []
  end
  
  def complete(index)
    item = @items.delete_at(index)
    @completed << item if item
  end
end

list = TodoList.new
list.add_item("Learn Ruby")
puts list.size  # 1 (delegated to @items.size)
```

## Your Turn: Fancy Ruby Challenges

1. **DSL Builder**: Create a DSL for defining state machines
2. **Method Tracer**: Build a tool that traces all method calls
3. **Dynamic ORM**: Create a simple ORM with metaprogramming
4. **Configuration System**: Build a flexible configuration DSL
5. **Aspect-Oriented Programming**: Implement AOP in Ruby

## What You've Learned

You now understand:
- Metaprogramming techniques and when to use them
- Building Domain-Specific Languages
- Advanced block and proc techniques
- Refinements for safe monkey patching
- Method hooks and introspection
- Fibers for lightweight concurrency
- TracePoint for debugging and profiling
- The Ruby object model in depth

## What's Next?

Next, we'll explore how HTTP and the internet work, preparing you for web development with Sinatra. You'll understand what happens when you type a URL and hit enter.

Remember: With great power comes great responsibility. These fancy features are powerful, but they can make code hard to understand if overused. Use them when they genuinely make your code better, not just to show off. The best Ruby code is the code that your teammates can understand and maintain. Sometimes boring is better than clever!