# Chapter 13: Enumerable - Your New Superpower

If Ruby modules were superheroes, Enumerable would be Superman. It's the module that gives collections their powers—the ability to search, sort, transform, and manipulate data with elegant, expressive methods. Once you understand Enumerable, you'll write Ruby code that's not just functional, but beautiful.

## What Is Enumerable?

Enumerable is a module that provides collection classes with traversal, searching, and sorting methods. Any class that includes Enumerable and defines an `each` method gets over 50 powerful methods for free. It's like buying a swiss army knife and discovering it also makes coffee.

```ruby
# A simple class that includes Enumerable
class TodoList
  include Enumerable
  
  def initialize
    @items = []
  end
  
  def add(item)
    @items << item
  end
  
  # Just define 'each' and get 50+ methods for free!
  def each
    @items.each { |item| yield(item) }
  end
end

list = TodoList.new
list.add("Learn Ruby")
list.add("Build something awesome")
list.add("Share with the world")

# Now we can use ALL Enumerable methods!
list.map(&:upcase)    # ["LEARN RUBY", "BUILD SOMETHING AWESOME", "SHARE WITH THE WORLD"]
list.select { |item| item.length > 10 }  # ["Build something awesome", "Share with the world"]
list.any? { |item| item.include?("Ruby") }  # true
list.first  # "Learn Ruby"
```

## The Core Enumerable Methods

### Each: The Foundation

Every Enumerable method is built on top of `each`:

```ruby
[1, 2, 3].each { |n| puts n }

# With index
["a", "b", "c"].each_with_index do |letter, index|
  puts "#{index}: #{letter}"
end

# Each on different collections
{a: 1, b: 2}.each { |key, value| puts "#{key}: #{value}" }
(1..5).each { |n| puts n }
"hello".each_char { |c| puts c }
```

### Map/Collect: Transform Every Element

`map` transforms each element and returns a new array:

```ruby
# Basic transformation
[1, 2, 3, 4, 5].map { |n| n * 2 }  # [2, 4, 6, 8, 10]

# String manipulation
names = ["alice", "bob", "charlie"]
names.map(&:capitalize)  # ["Alice", "Bob", "Charlie"]

# Creating new objects
users = [
  {name: "Alice", age: 30},
  {name: "Bob", age: 25}
]
user_strings = users.map { |u| "#{u[:name]} (#{u[:age]})" }
# ["Alice (30)", "Bob (25)"]

# Nested mapping
matrix = [[1, 2], [3, 4], [5, 6]]
doubled = matrix.map { |row| row.map { |n| n * 2 } }
# [[2, 4], [6, 8], [10, 12]]

# Map with index
letters = ["a", "b", "c"]
letters.map.with_index { |letter, i| "#{i}: #{letter}" }
# ["0: a", "1: b", "2: c"]
```

### Select/Filter/Reject: Choose What You Want

```ruby
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Select (keep matching elements)
evens = numbers.select { |n| n.even? }  # [2, 4, 6, 8, 10]
evens = numbers.filter { |n| n.even? }  # Same as select

# Reject (remove matching elements)
odds = numbers.reject { |n| n.even? }  # [1, 3, 5, 7, 9]

# More complex filtering
words = ["apple", "banana", "cherry", "date", "elderberry"]
long_words = words.select { |w| w.length > 5 }  # ["banana", "cherry", "elderberry"]
starts_with_vowel = words.select { |w| w.match(/^[aeiou]/i) }  # ["apple", "elderberry"]

# Chaining filters
numbers.select(&:even?).reject { |n| n > 6 }  # [2, 4, 6]
```

### Reduce/Inject: Combine All Elements

`reduce` combines all elements into a single value:

```ruby
# Sum
[1, 2, 3, 4, 5].reduce(:+)  # 15
[1, 2, 3, 4, 5].reduce(0) { |sum, n| sum + n }  # 15

# Product
[1, 2, 3, 4, 5].reduce(:*)  # 120

# Finding maximum
[5, 2, 8, 1, 9].reduce { |max, n| n > max ? n : max }  # 9

# Building a hash
pairs = [[:a, 1], [:b, 2], [:c, 3]]
hash = pairs.reduce({}) do |result, (key, value)|
  result[key] = value
  result
end
# {:a=>1, :b=>2, :c=>3}

# Word frequency counter
words = ["ruby", "python", "ruby", "javascript", "ruby", "python"]
frequency = words.reduce(Hash.new(0)) do |counts, word|
  counts[word] += 1
  counts
end
# {"ruby"=>3, "python"=>2, "javascript"=>1}

# With initial value
[1, 2, 3].reduce(10, :+)  # 16 (10 + 1 + 2 + 3)
```

### Find/Detect: Get the First Match

```ruby
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Find first even number
numbers.find { |n| n.even? }  # 2
numbers.detect { |n| n > 5 }   # 6 (same as find)

# Find with default
numbers.find(-> { "not found" }) { |n| n > 100 }  # "not found"

# Find index
numbers.find_index { |n| n > 5 }  # 5 (index of 6)

# Find all (use select for this)
numbers.find_all { |n| n.even? }  # [2, 4, 6, 8, 10]
```

### All?, Any?, None?, One?: Boolean Checks

```ruby
numbers = [2, 4, 6, 8, 10]

# All elements match?
numbers.all? { |n| n.even? }  # true
numbers.all? { |n| n > 5 }    # false

# Any element matches?
numbers.any? { |n| n > 8 }    # true
numbers.any? { |n| n.odd? }   # false

# No elements match?
numbers.none? { |n| n.odd? }  # true
numbers.none? { |n| n > 0 }   # false

# Exactly one element matches?
[1, 2, 3].one? { |n| n == 2 }  # true
[1, 2, 2].one? { |n| n == 2 }  # false

# Without block (checks truthiness)
[true, true, true].all?   # true
[true, false, true].all?  # false
[nil, false, nil].none?   # true
```

### Group_by and Partition

```ruby
# Group elements by a criteria
people = [
  {name: "Alice", age: 30, dept: "Engineering"},
  {name: "Bob", age: 25, dept: "Marketing"},
  {name: "Charlie", age: 35, dept: "Engineering"},
  {name: "Diana", age: 28, dept: "Marketing"}
]

by_dept = people.group_by { |p| p[:dept] }
# {
#   "Engineering" => [{name: "Alice"...}, {name: "Charlie"...}],
#   "Marketing" => [{name: "Bob"...}, {name: "Diana"...}]
# }

# Partition into two groups
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
evens, odds = numbers.partition { |n| n.even? }
# evens = [2, 4, 6, 8, 10]
# odds = [1, 3, 5, 7, 9]

# Group by multiple criteria
words = ["apple", "anchor", "banana", "berry", "cherry", "carrot"]
grouped = words.group_by { |w| [w[0], w.length] }
# {
#   ["a", 5]=>["apple"],
#   ["a", 6]=>["anchor"],
#   ["b", 6]=>["banana"],
#   ["b", 5]=>["berry"],
#   ["c", 6]=>["cherry", "carrot"]
# }
```

### Sort and Sort_by

```ruby
# Basic sorting
[3, 1, 4, 1, 5, 9, 2, 6].sort  # [1, 1, 2, 3, 4, 5, 6, 9]

# Sort with block
["apple", "pie", "zoo", "banana"].sort { |a, b| a.length <=> b.length }
# ["pie", "zoo", "apple", "banana"]

# Sort_by (more efficient for complex operations)
people = [
  {name: "Alice", age: 30},
  {name: "Bob", age: 25},
  {name: "Charlie", age: 35}
]

people.sort_by { |p| p[:age] }
# [{name: "Bob", age: 25}, {name: "Alice", age: 30}, {name: "Charlie", age: 35}]

# Reverse sort
[1, 2, 3, 4, 5].sort { |a, b| b <=> a }  # [5, 4, 3, 2, 1]
[1, 2, 3, 4, 5].sort.reverse  # Same result

# Multiple criteria sorting
students = [
  {name: "Alice", grade: "B", score: 85},
  {name: "Bob", grade: "A", score: 92},
  {name: "Charlie", grade: "B", score: 88},
  {name: "Diana", grade: "A", score: 95}
]

students.sort_by { |s| [s[:grade], -s[:score]] }
# Sorts by grade (A before B), then by score descending within grade
```

### Take, Drop, and Slice

```ruby
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Take elements from the beginning
numbers.take(3)  # [1, 2, 3]
numbers.take_while { |n| n < 5 }  # [1, 2, 3, 4]

# Drop elements from the beginning
numbers.drop(3)  # [4, 5, 6, 7, 8, 9, 10]
numbers.drop_while { |n| n < 5 }  # [5, 6, 7, 8, 9, 10]

# First and last
numbers.first     # 1
numbers.first(3)  # [1, 2, 3]
numbers.last      # 10
numbers.last(3)   # [8, 9, 10]
```

### Zip: Combine Arrays

```ruby
letters = ["a", "b", "c"]
numbers = [1, 2, 3]

letters.zip(numbers)  # [["a", 1], ["b", 2], ["c", 3]]

# Multiple arrays
colors = ["red", "green", "blue"]
letters.zip(numbers, colors)
# [["a", 1, "red"], ["b", 2, "green"], ["c", 3, "blue"]]

# With a block
letters.zip(numbers) { |l, n| puts "#{l}: #{n}" }
# a: 1
# b: 2
# c: 3

# Creating a hash from arrays
keys = [:name, :age, :city]
values = ["Alice", 30, "NYC"]
Hash[keys.zip(values)]  # {:name=>"Alice", :age=>30, :city=>"NYC"}
```

### Lazy Evaluation

For large or infinite collections, use lazy evaluation:

```ruby
# Without lazy (processes everything immediately)
(1..Float::INFINITY).select { |n| n % 3 == 0 }.take(5)  # This would hang!

# With lazy (processes on demand)
(1..Float::INFINITY).lazy.select { |n| n % 3 == 0 }.take(5).to_a
# [3, 6, 9, 12, 15]

# Chaining lazy operations
result = (1..1000000).lazy
  .select { |n| n.even? }
  .map { |n| n * 2 }
  .reject { |n| n % 6 == 0 }
  .take(10)
  .to_a
# Only processes enough elements to get 10 results

# Reading large files lazily
File.foreach("huge_file.txt").lazy
  .select { |line| line.include?("ERROR") }
  .map { |line| line.strip }
  .take(100)
  .to_a
```

## Advanced Enumerable Techniques

### Method Chaining

```ruby
# Complex data transformation pipeline
sales_data = [
  {product: "Widget", price: 10, quantity: 5, date: "2024-01-01"},
  {product: "Gadget", price: 20, quantity: 3, date: "2024-01-01"},
  {product: "Widget", price: 10, quantity: 8, date: "2024-01-02"},
  {product: "Doohickey", price: 15, quantity: 2, date: "2024-01-02"}
]

revenue_by_product = sales_data
  .map { |sale| sale.merge(revenue: sale[:price] * sale[:quantity]) }
  .group_by { |sale| sale[:product] }
  .transform_values { |sales| sales.sum { |s| s[:revenue] } }
  .sort_by { |product, revenue| -revenue }
  .to_h

# {"Widget"=>130, "Gadget"=>60, "Doohickey"=>30}
```

### Custom Enumerable Classes

```ruby
class Fibonacci
  include Enumerable
  
  def initialize(limit = Float::INFINITY)
    @limit = limit
  end
  
  def each
    a, b = 0, 1
    while a <= @limit
      yield a
      a, b = b, a + b
    end
  end
end

fib = Fibonacci.new(100)
fib.select(&:even?)  # [0, 2, 8, 34]
fib.take(10)  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# Tree traversal
class TreeNode
  include Enumerable
  attr_accessor :value, :children
  
  def initialize(value)
    @value = value
    @children = []
  end
  
  def each(&block)
    yield @value
    @children.each { |child| child.each(&block) }
  end
end

root = TreeNode.new(1)
root.children << TreeNode.new(2)
root.children << TreeNode.new(3)
root.children[0].children << TreeNode.new(4)

root.to_a  # [1, 2, 4, 3]
root.max   # 4
```

### Enumerator Objects

```ruby
# Creating enumerators
enum = [1, 2, 3].each  # Returns an Enumerator
enum.next  # 1
enum.next  # 2

# External iteration
colors = ["red", "green", "blue"].to_enum
loop do
  puts colors.next
end
# StopIteration exception when done

# Enumerator with size
enum = Enumerator.new(3) do |yielder|
  yielder << "a"
  yielder << "b"
  yielder << "c"
end

enum.to_a  # ["a", "b", "c"]

# Infinite enumerator
infinite = Enumerator.new do |yielder|
  n = 0
  loop do
    yielder << n
    n += 1
  end
end

infinite.take(5)  # [0, 1, 2, 3, 4]
```

## Practical Example: Data Analysis Pipeline

```ruby
# Analyzing server logs
class LogAnalyzer
  include Enumerable
  
  def initialize(log_file)
    @log_file = log_file
  end
  
  def each
    File.foreach(@log_file) do |line|
      if match = line.match(/(\d{4}-\d{2}-\d{2}) (\d{2}:\d{2}:\d{2}) \[(\w+)\] (.+)/)
        yield({
          date: match[1],
          time: match[2],
          level: match[3],
          message: match[4]
        })
      end
    end
  end
  
  def errors
    select { |log| log[:level] == "ERROR" }
  end
  
  def warnings
    select { |log| log[:level] == "WARNING" }
  end
  
  def by_date(date)
    select { |log| log[:date] == date }
  end
  
  def stats
    group_by { |log| log[:level] }
      .transform_values(&:count)
  end
  
  def most_common_errors(n = 10)
    errors
      .map { |log| log[:message] }
      .group_by(&:itself)
      .transform_values(&:count)
      .sort_by { |message, count| -count }
      .take(n)
      .to_h
  end
end

# Usage
analyzer = LogAnalyzer.new("server.log")

# Get error count
puts "Errors today: #{analyzer.by_date('2024-03-15').errors.count}"

# Get statistics
puts analyzer.stats
# {"INFO"=>1523, "WARNING"=>87, "ERROR"=>23}

# Find patterns
analyzer.errors.each do |error|
  if error[:message].include?("database")
    puts "Database error at #{error[:time]}: #{error[:message]}"
  end
end
```

## Performance Considerations

```ruby
# Bad: Multiple iterations
numbers = (1..1000000).to_a
evens = numbers.select(&:even?)
doubled = evens.map { |n| n * 2 }
sum = doubled.reduce(:+)

# Good: Single iteration with reduce
sum = (1..1000000).reduce(0) do |total, n|
  n.even? ? total + (n * 2) : total
end

# Better: Use lazy for large datasets
sum = (1..1000000).lazy
  .select(&:even?)
  .map { |n| n * 2 }
  .reduce(:+)

# Benchmark different approaches
require 'benchmark'

n = 1000000
Benchmark.bm do |x|
  x.report("select + map") do
    (1..n).select(&:even?).map { |i| i * 2 }
  end
  
  x.report("lazy") do
    (1..n).lazy.select(&:even?).map { |i| i * 2 }.to_a
  end
  
  x.report("single loop") do
    result = []
    (1..n).each { |i| result << i * 2 if i.even? }
    result
  end
end
```

## Common Enumerable Patterns

### The Accumulator Pattern
```ruby
# Building a result incrementally
result = data.reduce(initial_value) do |accumulator, element|
  # Process element and update accumulator
  accumulator
end
```

### The Filter-Map Pattern
```ruby
# Transform only certain elements
data.filter_map { |item| transform(item) if condition(item) }
```

### The Partition Pattern
```ruby
# Split into groups
valid, invalid = data.partition { |item| item.valid? }
```

### The Window Pattern
```ruby
# Process in sliding windows
data.each_cons(3) do |prev, current, next|
  # Process with context
end
```

## Your Turn: Enumerable Challenges

1. **Word Frequency**: Count word frequency in a text file
2. **CSV Processor**: Parse and analyze CSV data using Enumerable
3. **Prime Generator**: Create an Enumerable that generates prime numbers
4. **Tournament Scorer**: Calculate standings from match results
5. **Log Parser**: Build a log analysis tool using Enumerable methods

## What You've Learned

You now understand:
- How Enumerable provides 50+ methods from just defining `each`
- Core iteration methods: map, select, reduce, etc.
- Boolean predicates: all?, any?, none?, one?
- Grouping and partitioning data
- Lazy evaluation for large datasets
- Creating custom Enumerable classes
- Performance considerations and patterns

## What's Next?

In the next chapter, we'll explore regular expressions—Ruby's powerful pattern matching system. You'll learn how to search, extract, and manipulate text like a pro.

Remember: Enumerable is what makes Ruby collections so powerful. Master these methods, and you'll write code that's not just correct, but elegant. You'll solve complex data problems in a single line that would take dozens of lines in other languages. That's the Ruby way—expressive, powerful, and beautiful.