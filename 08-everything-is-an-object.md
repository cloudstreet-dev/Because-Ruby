# Chapter 8: Everything Is an Object (No, Really, Everything)

When Ruby says "everything is an object," it's not being metaphorical or trying to sound deep. It literally means everything. That number 42? Object. The word "hello"? Object. The concept of nothingness (nil)? Yep, that's an object too. Even classes are objects. It's objects all the way down, and then the turtles are objects too.

## What Even Is an Object?

An object is a bundle of data (attributes) and behavior (methods). Think of it like a smartphone: it has properties (color, screen size, battery level) and things it can do (make calls, take photos, drain battery at inconvenient times).

In Ruby, every piece of data is an object that knows things about itself and can do things:

```ruby
# Numbers are objects
42.class          # Integer
42.methods.count  # 131 (that's a lot of methods!)
42.even?          # true
42.next           # 43
42.to_s           # "42"

# Strings are objects
"hello".class     # String
"hello".upcase    # "HELLO"
"hello".length    # 5
"hello".reverse   # "olleh"

# Even nil is an object
nil.class         # NilClass
nil.to_s          # ""
nil.nil?          # true
nil.to_i          # 0

# Classes are objects too!
String.class      # Class
String.ancestors  # [String, Comparable, Object, Kernel, BasicObject]
```

## The Object Hierarchy: Ruby's Family Tree

Ruby has a hierarchy of objects, like a family tree where everyone inherits traits from their parents:

```ruby
# Let's explore the hierarchy
5.class                    # Integer
Integer.superclass         # Numeric
Numeric.superclass         # Object
Object.superclass          # BasicObject
BasicObject.superclass     # nil (the top!)

# Everything inherits from Object (well, almost everything)
"hello".is_a?(Object)      # true
42.is_a?(Object)           # true
[1, 2, 3].is_a?(Object)    # true
{ a: 1 }.is_a?(Object)     # true
nil.is_a?(Object)          # true

# You can check the family tree
String.ancestors
# [String, Comparable, Object, Kernel, BasicObject]

Array.ancestors
# [Array, Enumerable, Object, Kernel, BasicObject]
```

## Methods: What Objects Can Do

Every object comes with a set of methods it inherits from its class and parent classes:

```ruby
# See what an object can do
"hello".methods                # All methods (there are many!)
"hello".public_methods          # Just public ones
"hello".methods.grep(/case/)   # Methods with 'case' in the name
# [:downcase, :upcase, :swapcase, :casecmp, :downcase!, :upcase!, :swapcase!, :casecmp?]

# Check if an object responds to a method
"hello".respond_to?(:upcase)   # true
"hello".respond_to?(:fly)      # false (strings can't fly... yet)

# Get method objects
method_obj = "hello".method(:upcase)
method_obj.call                # "HELLO"
method_obj.owner               # String
method_obj.arity               # 0 (takes no arguments)
```

## The `self` Keyword: An Object's Identity

Every object has a sense of self. When you're inside an object's method, `self` refers to the current object:

```ruby
class Dog
  def whoami
    puts "self is: #{self}"
    puts "self's class is: #{self.class}"
  end
  
  def bark
    puts "#{self} says Woof!"  # self is the current dog instance
  end
end

fido = Dog.new
fido.whoami
# self is: #<Dog:0x00007f8b2a0c5e38>
# self's class is: Dog

# At the top level, self is 'main'
puts self  # main
puts self.class  # Object
```

## Object Identity and Equality

Every object has a unique identity, but Ruby has multiple ways to check equality:

```ruby
# Object ID - unique identifier
a = "hello"
b = "hello"
c = a

a.object_id  # 70235465789320 (your number will differ)
b.object_id  # 70235465789300 (different object!)
c.object_id  # 70235465789320 (same as a!)

# Different equality checks
a == b       # true (same value)
a.equal?(b)  # false (not the same object)
a.equal?(c)  # true (same object)

# With symbols (always the same object)
:ruby.object_id  # 1120988 (same every time)
:ruby.object_id  # 1120988 (yep, same)

# Practical example
str1 = "Ruby"
str2 = "Ruby"
str3 = str1

str1 == str2      # true (same content)
str1.equal?(str2) # false (different objects)
str1.equal?(str3) # true (same object)

# This is why we use symbols for hash keys!
# Every time you use :name, it's the same object
# Every time you use "name", it's a new object
```

## Duck Typing: If It Quacks Like a Duck

Ruby doesn't care what class an object is, only what it can do. This is called "duck typing"—if it walks like a duck and quacks like a duck, it's a duck.

```ruby
# These are all different classes
string = "hello"
array = ["h", "e", "l", "l", "o"]
range = (1..5)

# But they all respond to 'each'
[string, array, range].each do |thing|
  print "#{thing.class} can iterate: "
  thing.each { |item| print "#{item} " }
  puts
end

# Duck typing in action
def make_it_loud(something)
  # We don't care what it is, just that it responds to upcase
  something.upcase
end

make_it_loud("hello")        # "HELLO"
make_it_loud(:symbol)        # "SYMBOL"
# make_it_loud(42)           # Error! Numbers don't have upcase

# Better with checking
def make_it_loud_safely(something)
  if something.respond_to?(:upcase)
    something.upcase
  else
    something.to_s.upcase
  end
end

make_it_loud_safely("hello")  # "HELLO"
make_it_loud_safely(42)       # "42"
```

## Modifying Objects: Mutability

Some objects can be changed (mutable), others can't (immutable):

```ruby
# Strings are mutable
str = "hello"
str.upcase!         # Modifies in place
puts str            # "HELLO"

# Numbers are immutable
num = 42
num + 1             # Returns 43, doesn't change num
puts num            # Still 42

# Freeze makes objects immutable
str = "hello".freeze
# str.upcase!       # Error! Can't modify frozen string

# Check if frozen
str.frozen?         # true

# Arrays and hashes can be frozen too
arr = [1, 2, 3].freeze
# arr << 4          # Error!
```

## Object Conversion: Shape-Shifting

Ruby objects can often convert themselves to other types:

```ruby
# To string
42.to_s              # "42"
[1, 2, 3].to_s       # "[1, 2, 3]"
{ a: 1 }.to_s        # "{:a=>1}"
nil.to_s             # ""

# To integer
"42".to_i            # 42
"42.5".to_i          # 42
"hello".to_i         # 0
true.to_i            # NoMethodError (booleans can't convert)

# To array
"hello".to_a         # NoMethodError
"hello".chars        # ["h", "e", "l", "l", "o"] (better way)
(1..5).to_a          # [1, 2, 3, 4, 5]
{ a: 1, b: 2 }.to_a  # [[:a, 1], [:b, 2]]

# To symbol
"hello".to_sym       # :hello
42.to_sym            # NoMethodError

# Custom conversion
class Dog
  def initialize(name)
    @name = name
  end
  
  def to_s
    "Dog named #{@name}"
  end
  
  def to_h
    { type: "dog", name: @name }
  end
end

fido = Dog.new("Fido")
puts fido            # "Dog named Fido"
fido.to_h            # {:type=>"dog", :name=>"Fido"}
```

## Singleton Methods: Special Snowflake Methods

Ruby lets you add methods to individual objects:

```ruby
# Regular string
str = "hello"

# Add a method just to this string
def str.shout
  self.upcase + "!!!"
end

str.shout            # "HELLO!!!"

# Other strings don't have this method
"world".shout        # NoMethodError

# You can even add methods to classes (they're objects too!)
def String.random_greeting
  ["Hello", "Hi", "Hey", "Howdy"].sample
end

String.random_greeting  # "Hey" (random each time)
```

## The Eigenclass: The Secret Class

Every object has a hidden singleton class (eigenclass) where its singleton methods live:

```ruby
str = "hello"

# Get the singleton class
singleton = str.singleton_class
puts singleton  # #<Class:#<String:0x00007f8b2a0c5e38>>

# Add method via singleton class
singleton.define_method(:whisper) do
  self.downcase
end

str.whisper  # "hello"

# Check the method lookup chain
p str.singleton_class.ancestors
# [#<Class:#<String:0x00007f8b2a0c5e38>>, String, Comparable, Object, Kernel, BasicObject]
```

## Practical Example: Building a Customizable Logger

Let's use our object knowledge to build something useful:

```ruby
class SmartLogger
  attr_accessor :level, :format
  
  LEVELS = { debug: 0, info: 1, warn: 2, error: 3, fatal: 4 }
  
  def initialize(level = :info)
    @level = level
    @format = :standard
    @outputs = []
    @filters = []
  end
  
  LEVELS.each do |level_name, level_value|
    define_method level_name do |message, &block|
      return if LEVELS[@level] > level_value
      
      message = block.call if block_given?
      message = apply_filters(message)
      formatted = format_message(level_name, message)
      
      output(formatted)
    end
  end
  
  def add_output(output)
    # Duck typing - anything that responds to puts works
    @outputs << output if output.respond_to?(:puts)
    self
  end
  
  def add_filter(&block)
    @filters << block
    self
  end
  
  def format_message(level, message)
    timestamp = Time.now.strftime("%Y-%m-%d %H:%M:%S")
    
    case @format
    when :standard
      "[#{timestamp}] #{level.upcase}: #{message}"
    when :json
      { timestamp: timestamp, level: level, message: message }.to_json
    when :minimal
      "#{level[0].upcase}: #{message}"
    else
      message
    end
  end
  
  def apply_filters(message)
    @filters.reduce(message) { |msg, filter| filter.call(msg) }
  end
  
  def with_context(context)
    # Return a modified version for this context
    contextual = self.dup
    
    # Add context to all messages
    contextual.add_filter { |msg| "[#{context}] #{msg}" }
    contextual
  end
  
  private
  
  def output(message)
    if @outputs.empty?
      puts message
    else
      @outputs.each { |out| out.puts message }
    end
  end
end

# Using our logger
logger = SmartLogger.new(:debug)

# Basic logging
logger.debug "Starting application"
logger.info "User logged in"
logger.warn "Low memory"
logger.error "Failed to connect"

# Add custom output
class SlackNotifier
  def puts(message)
    # Imagine this sends to Slack
    puts "📱 SLACK: #{message}"
  end
end

logger.add_output(SlackNotifier.new)

# Add filters
logger.add_filter { |msg| msg.gsub(/password=\w+/, 'password=***') }

# Log with sensitive data
logger.info "Login attempt with password=secret123"
# Output: [timestamp] INFO: Login attempt with password=***

# Contextual logging
api_logger = logger.with_context("API")
api_logger.info "Request received"
# Output: [timestamp] INFO: [API] Request received

# Block form for expensive operations
logger.debug { "Expensive calculation: #{expensive_method}" }
# Only calls expensive_method if debug level is active

# Change format on the fly
logger.format = :minimal
logger.info "Quick message"
# Output: I: Quick message
```

## Object Inspection and Debugging

Ruby gives you tools to inspect objects:

```ruby
class Person
  attr_accessor :name, :age
  
  def initialize(name, age)
    @name = name
    @age = age
    @secret = "shh"
  end
  
  def inspect
    "#<Person name=#{@name} age=#{@age}>"
  end
  
  def to_s
    "#{@name} (#{@age})"
  end
end

person = Person.new("Alice", 30)

# Different inspection methods
person.inspect          # "#<Person name=Alice age=30>"
person.to_s            # "Alice (30)"
p person               # Calls inspect
puts person            # Calls to_s

# Instance variables
person.instance_variables  # [:@name, :@age, :@secret]
person.instance_variable_get(:@secret)  # "shh"
person.instance_variable_set(:@secret, "new secret")

# Method introspection
person.methods.grep(/name/)  # [:name, :name=]
person.method(:name).source_location  # File and line where defined
```

## Common Object Patterns

### The Null Object Pattern
```ruby
class NullUser
  def name; "Guest"; end
  def logged_in?; false; end
  def permissions; []; end
end

def current_user
  @current_user || NullUser.new
end

# No need for nil checks!
puts "Welcome, #{current_user.name}"
```

### The Value Object Pattern
```ruby
class Money
  attr_reader :amount, :currency
  
  def initialize(amount, currency = "USD")
    @amount = amount
    @currency = currency
    freeze  # Make immutable
  end
  
  def +(other)
    raise "Currency mismatch" unless currency == other.currency
    Money.new(amount + other.amount, currency)
  end
  
  def to_s
    "#{currency} #{amount}"
  end
end

price = Money.new(100, "USD")
tax = Money.new(10, "USD")
total = price + tax  # Money.new(110, "USD")
```

## Your Turn: Object Challenges

1. **Object Inspector**: Build a tool that shows an object's complete method lookup path
2. **Method Logger**: Create a module that logs all method calls on an object
3. **Immutable Hash**: Build a hash-like object that can't be modified after creation
4. **Type Checker**: Create a method that validates objects match expected interfaces
5. **Object Pool**: Implement an object pool pattern for expensive objects

## What You've Learned

You now understand:
- Everything in Ruby truly is an object
- Objects have identity, state, and behavior
- Duck typing and why it matters
- How to inspect and modify objects
- Singleton methods and the eigenclass
- Object conversion and mutability
- Common object-oriented patterns

## What's Next?

In the next chapter, we'll learn how to create our own classes—essentially designing our own types of objects. You'll go from using objects to creating them, from player to game designer. It's where the real fun begins!

Remember: Understanding that everything is an object isn't just philosophical—it's practical. It means you can call methods on anything, inspect anything, and extend anything. It's what makes Ruby so flexible and powerful. Once you internalize this, Ruby starts to feel less like a programming language and more like a conversation with objects.