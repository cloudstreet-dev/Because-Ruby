# Chapter 10: Modules - Ruby's Mix-and-Match System

If classes are like cookie cutters that create objects, modules are like sprinkles you can add to any cookie. They're collections of methods and constants that can be mixed into classes to add functionality. Think of them as Ruby's answer to "I want this AND that."

## What's a Module?

A module is a collection of methods, constants, and other module definitions. Unlike classes, you can't create instances of modules. Instead, you mix them into classes:

```ruby
module Greetings
  def say_hello
    puts "Hello!"
  end
  
  def say_goodbye
    puts "Goodbye!"
  end
  
  def say_hi
    puts "Hi there!"
  end
end

# You can't do this:
# greeter = Greetings.new  # Error! No instances allowed

# But you can do this:
class Person
  include Greetings  # Mix in the module
  
  def initialize(name)
    @name = name
  end
end

person = Person.new("Alice")
person.say_hello    # "Hello!"
person.say_goodbye  # "Goodbye!"
```

## Include, Prepend, and Extend: The Three Ways to Mix

Ruby gives you three ways to add module functionality to classes:

```ruby
module Shout
  def yell(text)
    text.upcase + "!!!"
  end
end

# Include: adds module methods as instance methods
class Person
  include Shout
end

Person.new.yell("hello")  # "HELLO!!!"

# Extend: adds module methods as class methods
class Announcement
  extend Shout
end

Announcement.yell("attention")  # "ATTENTION!!!"

# Prepend: like include, but inserts earlier in method lookup
module Polite
  def greet
    "Excuse me, " + super  # Calls the original method
  end
end

class Greeting
  prepend Polite
  
  def greet
    "Hello!"
  end
end

Greeting.new.greet  # "Excuse me, Hello!"
```

## Mixins: Adding Abilities to Classes

Modules are perfect for adding shared behavior to multiple classes:

```ruby
module Flyable
  def fly
    puts "#{self.class.name} is flying!"
  end
  
  def land
    puts "#{self.class.name} has landed."
  end
  
  def altitude
    @altitude ||= 0
  end
  
  def altitude=(height)
    @altitude = height
    puts "Flying at #{height} feet"
  end
end

module Swimmable
  def swim
    puts "#{self.class.name} is swimming!"
  end
  
  def dive(depth)
    puts "Diving to #{depth} meters"
  end
end

class Bird
  include Flyable
  
  def initialize(species)
    @species = species
  end
end

class Fish
  include Swimmable
  
  def initialize(species)
    @species = species
  end
end

class Duck
  include Flyable
  include Swimmable  # Ducks can do both!
  
  def initialize(name)
    @name = name
  end
  
  def quack
    puts "Quack!"
  end
end

sparrow = Bird.new("Sparrow")
sparrow.fly  # "Bird is flying!"

salmon = Fish.new("Salmon")
salmon.swim  # "Fish is swimming!"

donald = Duck.new("Donald")
donald.fly   # "Duck is flying!"
donald.swim  # "Duck is swimming!"
donald.quack # "Quack!"
```

## Module Constants and Nested Modules

Modules can contain constants and other modules:

```ruby
module MathHelpers
  PI = 3.14159
  E = 2.71828
  
  module Trigonometry
    def sin_degrees(angle)
      Math.sin(angle * Math::PI / 180)
    end
    
    def cos_degrees(angle)
      Math.cos(angle * Math::PI / 180)
    end
  end
  
  module Statistics
    def mean(numbers)
      numbers.sum.to_f / numbers.size
    end
    
    def median(numbers)
      sorted = numbers.sort
      mid = sorted.length / 2
      sorted.length.odd? ? sorted[mid] : (sorted[mid-1] + sorted[mid]) / 2.0
    end
  end
end

class Calculator
  include MathHelpers::Trigonometry
  include MathHelpers::Statistics
  
  def circle_area(radius)
    MathHelpers::PI * radius ** 2
  end
end

calc = Calculator.new
calc.mean([1, 2, 3, 4, 5])  # 3.0
calc.sin_degrees(30)         # 0.5
calc.circle_area(5)          # 78.53975
```

## Enumerable: The Most Famous Module

The Enumerable module is Ruby's Swiss Army knife for collections. Include it and define `each`, and you get dozens of methods for free:

```ruby
class TodoList
  include Enumerable
  
  def initialize
    @items = []
  end
  
  def add(item)
    @items << item
  end
  
  # Define each to get Enumerable methods
  def each
    @items.each { |item| yield(item) }
  end
end

list = TodoList.new
list.add("Buy milk")
list.add("Walk dog")
list.add("Learn Ruby")

# Now we get all these for free!
list.map(&:upcase)         # ["BUY MILK", "WALK DOG", "LEARN RUBY"]
list.select { |i| i.include?("dog") }  # ["Walk dog"]
list.any? { |i| i.include?("Ruby") }   # true
list.first                 # "Buy milk"
list.count                 # 3
```

## Module Methods: Namespacing and Utilities

Modules can have their own methods, useful for namespacing:

```ruby
module FileHelper
  # Module method (called on the module itself)
  def self.sanitize_filename(filename)
    filename.gsub(/[^\w\.\-]/, '_')
  end
  
  def self.ensure_extension(filename, ext)
    filename.end_with?(ext) ? filename : "#{filename}.#{ext}"
  end
  
  # Alternative syntax for module methods
  module_function
  
  def timestamp
    Time.now.strftime("%Y%m%d_%H%M%S")
  end
  
  def temp_file_name
    "temp_#{timestamp}_#{rand(1000)}.txt"
  end
end

# Use module methods directly
FileHelper.sanitize_filename("my file!.txt")  # "my_file_.txt"
FileHelper.ensure_extension("report", ".pdf")  # "report.pdf"
FileHelper.timestamp                           # "20240315_143022"
```

## Module Callbacks: Hook Into the Mix

Modules can know when they're included:

```ruby
module Trackable
  def self.included(base)
    base.extend(ClassMethods)
    base.class_eval do
      attr_accessor :tracked_at
    end
  end
  
  module ClassMethods
    def track_method(method_name)
      alias_method "#{method_name}_without_tracking", method_name
      
      define_method method_name do |*args|
        puts "Calling #{method_name} at #{Time.now}"
        result = send("#{method_name}_without_tracking", *args)
        @tracked_at = Time.now
        result
      end
    end
  end
  
  def tracking_info
    "Last tracked at: #{@tracked_at || 'Never'}"
  end
end

class Order
  include Trackable
  
  def initialize(id)
    @id = id
  end
  
  def process
    puts "Processing order #{@id}"
  end
  
  track_method :process  # Add tracking to the process method
end

order = Order.new(123)
order.process
order.tracking_info
```

## Comparable: Make Your Objects Sortable

We touched on this before, but Comparable is a perfect example of module power:

```ruby
class Score
  include Comparable
  
  attr_reader :points, :player
  
  def initialize(player, points)
    @player = player
    @points = points
  end
  
  # Define <=> and get <, <=, ==, >=, >, between? for free
  def <=>(other)
    @points <=> other.points
  end
  
  def to_s
    "#{@player}: #{@points}"
  end
end

scores = [
  Score.new("Alice", 100),
  Score.new("Bob", 85),
  Score.new("Charlie", 95)
]

scores.sort.each { |s| puts s }
# Bob: 85
# Charlie: 95
# Alice: 100

scores.max  # Alice: 100
scores.min  # Bob: 85
```

## Module Composition: Building Complex Behaviors

Modules can include other modules:

```ruby
module Timestamps
  def touch
    @updated_at = Time.now
  end
  
  def last_updated
    @updated_at || "Never"
  end
end

module Versioned
  def increment_version
    @version = (@version || 0) + 1
  end
  
  def version
    @version || 1
  end
end

module Auditable
  include Timestamps
  include Versioned
  
  def audit_log
    @audit_log ||= []
  end
  
  def record_change(change)
    audit_log << {
      change: change,
      version: version,
      timestamp: Time.now
    }
    touch
    increment_version
  end
  
  def audit_report
    puts "Audit Report:"
    puts "Version: #{version}"
    puts "Last updated: #{last_updated}"
    puts "Changes:"
    audit_log.each do |entry|
      puts "  v#{entry[:version]}: #{entry[:change]} at #{entry[:timestamp]}"
    end
  end
end

class Document
  include Auditable
  
  attr_reader :content
  
  def initialize(content)
    @content = content
    record_change("Document created")
  end
  
  def update(new_content)
    old_content = @content
    @content = new_content
    record_change("Content changed from '#{old_content}' to '#{new_content}'")
  end
end

doc = Document.new("First draft")
doc.update("Second draft")
doc.update("Final version")
doc.audit_report
```

## Practical Example: Building a Plugin System

Let's build a flexible reporting system using modules:

```ruby
module Formattable
  module JSON
    def format(data)
      require 'json'
      data.to_json
    end
    
    def extension
      ".json"
    end
  end
  
  module CSV
    def format(data)
      return "" if data.empty?
      
      headers = data.first.keys
      rows = data.map { |row| row.values.join(",") }
      
      ([headers.join(",")] + rows).join("\n")
    end
    
    def extension
      ".csv"
    end
  end
  
  module Markdown
    def format(data)
      return "" if data.empty?
      
      headers = data.first.keys
      
      output = []
      output << "| #{headers.join(' | ')} |"
      output << "| #{headers.map { '-' * 10 }.join(' | ')} |"
      
      data.each do |row|
        output << "| #{row.values.join(' | ')} |"
      end
      
      output.join("\n")
    end
    
    def extension
      ".md"
    end
  end
end

module Exportable
  def export_to_file(filename = nil)
    filename ||= "export_#{Time.now.to_i}#{extension}"
    
    File.write(filename, format(to_export_data))
    puts "Exported to #{filename}"
    filename
  end
  
  def preview(lines = 5)
    formatted = format(to_export_data)
    preview_lines = formatted.split("\n").first(lines)
    
    puts "Preview (first #{lines} lines):"
    puts preview_lines.join("\n")
    puts "..." if formatted.lines.count > lines
  end
end

class SalesReport
  include Exportable
  include Formattable::CSV  # Default to CSV format
  
  def initialize
    @sales = []
  end
  
  def add_sale(product, amount, date = Date.today)
    @sales << {
      product: product,
      amount: amount,
      date: date.to_s,
      quarter: "Q#{((date.month - 1) / 3) + 1}"
    }
  end
  
  def to_export_data
    @sales
  end
  
  def total
    @sales.sum { |s| s[:amount] }
  end
  
  def by_product
    @sales.group_by { |s| s[:product] }
          .transform_values { |sales| sales.sum { |s| s[:amount] } }
  end
end

class CustomerReport
  include Exportable
  
  def initialize
    @customers = []
    @format_module = Formattable::JSON  # Dynamic format selection
  end
  
  def set_format(format)
    @format_module = case format
    when :json then Formattable::JSON
    when :csv then Formattable::CSV
    when :markdown then Formattable::Markdown
    else Formattable::JSON
    end
    
    # Remove old format module and include new one
    singleton_class.include(@format_module)
  end
  
  def add_customer(name, email, signup_date)
    @customers << {
      name: name,
      email: email,
      signup_date: signup_date.to_s,
      id: @customers.size + 1
    }
  end
  
  def to_export_data
    @customers
  end
end

# Using our reports
sales = SalesReport.new
sales.add_sale("Widget", 100, Date.new(2024, 1, 15))
sales.add_sale("Gadget", 200, Date.new(2024, 2, 20))
sales.add_sale("Widget", 150, Date.new(2024, 3, 10))

sales.preview(3)
sales.export_to_file("sales_report.csv")

customers = CustomerReport.new
customers.add_customer("Alice", "alice@example.com", Date.today)
customers.add_customer("Bob", "bob@example.com", Date.today - 30)

customers.set_format(:markdown)
customers.preview
customers.export_to_file("customers.md")
```

## Module Best Practices

### 1. Single Responsibility
```ruby
# Good - focused module
module Timestampable
  def touch
    @updated_at = Time.now
  end
end

# Bad - doing too much
module UtilityBelt
  def timestamp
  def validate
  def format
  # Too many unrelated things
end
```

### 2. Use Modules for Shared Behavior
```ruby
# Good - behavior used by multiple classes
module Cacheable
  def cache_key
    "#{self.class.name.downcase}_#{id}"
  end
end

class User
  include Cacheable
end

class Product
  include Cacheable
end
```

### 3. Namespace with Modules
```ruby
module MyApp
  module Models
    class User
    end
  end
  
  module Controllers
    class UserController
    end
  end
end

user = MyApp::Models::User.new
```

## Module vs Class: When to Use Which?

**Use a Module when:**
- You want to share functionality across multiple classes
- You need a namespace
- You want to group related methods
- Multiple inheritance of behavior is needed

**Use a Class when:**
- You need to create instances
- You have a clear "is-a" relationship
- You need inheritance hierarchies
- You're modeling a concrete concept

## Your Turn: Module Challenges

1. **Validation Module**: Create a module that adds validation methods to any class
2. **Searchable Module**: Build a module that makes collections searchable
3. **State Machine Module**: Design a module that adds state management
4. **Serializable Module**: Create export/import functionality for objects
5. **Observable Module**: Implement the observer pattern as a module

## What You've Learned

You now know how to:
- Create and use modules
- Mix modules into classes with include, prepend, and extend
- Build reusable functionality with mixins
- Use modules for namespacing
- Create module methods
- Compose complex behaviors from simple modules

## What's Next?

Next chapter, we'll explore inheritance—how classes can have parents and children. We'll see how Ruby's single inheritance model works and why modules often provide a better solution than complex inheritance hierarchies.

Remember: Modules are about sharing behavior, not creating objects. They're the ingredients you mix into your classes to give them superpowers. Use them liberally, and your code will be more modular, reusable, and maintainable. Plus, you'll finally understand what people mean when they say "composition over inheritance!"