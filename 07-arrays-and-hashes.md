# Chapter 7: Arrays and Hashes - Ruby's Junk Drawers

If variables are like labeled boxes for single items, arrays and hashes are like those organizer drawers in your kitchen—the ones where you can store multiple things and (hopefully) find them again later. Arrays are for when you care about order, and hashes are for when you care about labels.

## Arrays: Lists That Actually Make Sense

An array is an ordered collection of objects. Think of it as a numbered list where Ruby handles the numbering for you (starting at 0, because programmers are weird like that).

```ruby
# Creating arrays
empty_array = []
numbers = [1, 2, 3, 4, 5]
mixed = [42, "hello", true, nil, 3.14]  # Ruby doesn't judge
words = %w[apple banana cherry]  # Shorthand for string arrays
symbols = %i[red green blue]     # Shorthand for symbol arrays

# Arrays can contain arrays (arrayception!)
matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
]
```

## Accessing Array Elements

Arrays are zero-indexed, which means counting starts at 0. This will either make perfect sense or haunt your dreams.

```ruby
fruits = ["apple", "banana", "cherry", "date", "elderberry"]

# Positive indexing
fruits[0]     # "apple" (first element)
fruits[2]     # "cherry" (third element)
fruits[4]     # "elderberry" (fifth element)
fruits[10]    # nil (doesn't exist)

# Negative indexing (from the end)
fruits[-1]    # "elderberry" (last element)
fruits[-2]    # "date" (second to last)

# Ranges
fruits[1..3]  # ["banana", "cherry", "date"]
fruits[1...3] # ["banana", "cherry"] (exclusive end)
fruits[2..-1] # ["cherry", "date", "elderberry"]

# Multiple elements
fruits.values_at(0, 2, 4)  # ["apple", "cherry", "elderberry"]
fruits.values_at(0, -1)    # ["apple", "elderberry"]

# First and last shortcuts
fruits.first     # "apple"
fruits.first(2)  # ["apple", "banana"]
fruits.last      # "elderberry"
fruits.last(2)   # ["date", "elderberry"]

# Sample random elements
fruits.sample    # Random fruit
fruits.sample(2) # Two random fruits
```

## Modifying Arrays

Arrays in Ruby are mutable, meaning you can change them after creation:

```ruby
colors = ["red", "green", "blue"]

# Adding elements
colors << "yellow"           # Append (["red", "green", "blue", "yellow"])
colors.push("orange")        # Same as << 
colors.unshift("purple")     # Add to beginning
colors.insert(2, "pink")     # Insert at specific position

# Removing elements
colors.pop                   # Remove and return last element
colors.shift                 # Remove and return first element
colors.delete("pink")        # Remove specific element
colors.delete_at(1)          # Remove element at index

# Changing elements
colors[0] = "crimson"        # Change single element
colors[1..2] = ["navy", "teal"]  # Change multiple elements

# Array operations
colors.clear                 # Remove all elements
colors.compact               # Remove nil values
colors.uniq                  # Remove duplicates
colors.reverse               # Reverse order
colors.sort                  # Sort elements
colors.shuffle               # Random order
```

## Array Iteration: The Fun Part

This is where Ruby really shines. Arrays have tons of methods for iteration:

```ruby
numbers = [1, 2, 3, 4, 5]

# Each - the workhorse
numbers.each do |num|
  puts "Number: #{num}"
end

# Each with index
numbers.each_with_index do |num, i|
  puts "#{i}: #{num}"
end

# Map - transform each element
doubled = numbers.map { |n| n * 2 }  # [2, 4, 6, 8, 10]
strings = numbers.map(&:to_s)        # ["1", "2", "3", "4", "5"]

# Select/Filter - keep matching elements
evens = numbers.select { |n| n.even? }  # [2, 4]
odds = numbers.reject { |n| n.even? }   # [1, 3, 5]

# Reduce/Inject - combine all elements
sum = numbers.reduce(0) { |total, n| total + n }  # 15
sum = numbers.reduce(:+)                          # 15 (shorter)
product = numbers.reduce(:*)                      # 120

# Find - first matching element
numbers.find { |n| n > 3 }     # 4
numbers.detect { |n| n > 10 }   # nil

# All? Any? None? One?
numbers.all? { |n| n > 0 }     # true
numbers.any? { |n| n > 4 }     # true
numbers.none? { |n| n < 0 }    # true
numbers.one? { |n| n == 3 }    # true

# Partition - split into two arrays
evens, odds = numbers.partition { |n| n.even? }
# evens = [2, 4], odds = [1, 3, 5]

# Group by
words = ["cat", "dog", "bird", "fish", "turtle"]
by_length = words.group_by { |w| w.length }
# {3=>["cat", "dog"], 4=>["bird", "fish"], 6=>["turtle"]}
```

## Hashes: Dictionaries for Humans

Hashes (also called dictionaries or maps in other languages) store key-value pairs. Think of them as a phonebook where you look up names (keys) to find phone numbers (values).

```ruby
# Creating hashes
empty_hash = {}
old_syntax = { "name" => "Alice", "age" => 30 }
new_syntax = { name: "Alice", age: 30 }  # For symbol keys
mixed_keys = { "string" => 1, :symbol => 2, 42 => "number" }

# Hash from arrays
keys = [:name, :age, :city]
values = ["Bob", 25, "NYC"]
person = keys.zip(values).to_h  # {name: "Bob", age: 25, city: "NYC"}

# Default values
scores = Hash.new(0)  # Default value is 0
scores["Alice"] += 10  # Works even though "Alice" doesn't exist yet
```

## Accessing Hash Values

```ruby
user = { name: "Charlie", age: 28, city: "Seattle", role: "developer" }

# Accessing values
user[:name]          # "Charlie"
user[:age]           # 28
user[:height]        # nil (doesn't exist)

# Fetch with defaults
user.fetch(:name)    # "Charlie"
user.fetch(:height, "Unknown")  # "Unknown" (default)
user.fetch(:height) { "Not specified" }  # Block default

# Multiple values
user.values_at(:name, :age)  # ["Charlie", 28]

# Keys and values
user.keys     # [:name, :age, :city, :role]
user.values   # ["Charlie", 28, "Seattle", "developer"]

# Checking existence
user.key?(:name)       # true
user.has_key?(:email)  # false
user.value?(28)        # true
user.has_value?(100)   # false
```

## Modifying Hashes

```ruby
person = { name: "Dave", age: 35 }

# Adding/updating values
person[:email] = "dave@example.com"
person[:age] = 36
person.merge!(city: "Boston", country: "USA")

# Removing values
person.delete(:email)           # Returns deleted value
person.delete(:phone) { "N/A" } # Returns default if not found

# Transform keys or values
person.transform_keys(&:to_s)   # Convert keys to strings
person.transform_values(&:to_s) # Convert values to strings

# Select/reject
person.select { |k, v| k == :name || k == :age }
person.reject { |k, v| v.nil? }

# Compact (remove nil values)
data = { a: 1, b: nil, c: 3, d: nil }
data.compact  # {a: 1, c: 3}
```

## Hash Iteration

```ruby
scores = { alice: 95, bob: 87, charlie: 92, diana: 88 }

# Each
scores.each do |name, score|
  puts "#{name}: #{score}"
end

# Each key or value separately
scores.each_key { |name| puts name }
scores.each_value { |score| puts score }

# Map/Transform
grades = scores.map do |name, score|
  grade = case score
  when 90..100 then "A"
  when 80..89 then "B"
  else "C"
  end
  [name, grade]
end.to_h

# Select/Filter
high_scores = scores.select { |name, score| score >= 90 }
# {alice: 95, charlie: 92}

# Sort (returns array of arrays)
sorted = scores.sort_by { |name, score| score }
# [[:bob, 87], [:diana, 88], [:charlie, 92], [:alice, 95]]

# Min/Max
scores.min_by { |name, score| score }  # [:bob, 87]
scores.max_by { |name, score| score }  # [:alice, 95]
```

## Nested Structures: Arrays and Hashes Together

Real-world data is often nested:

```ruby
# Array of hashes (common for API responses)
users = [
  { id: 1, name: "Alice", role: "admin" },
  { id: 2, name: "Bob", role: "user" },
  { id: 3, name: "Charlie", role: "user" }
]

# Find specific user
admin = users.find { |u| u[:role] == "admin" }

# Get all names
names = users.map { |u| u[:name] }

# Hash of arrays
inventory = {
  fruits: ["apple", "banana", "orange"],
  vegetables: ["carrot", "broccoli", "spinach"],
  dairy: ["milk", "cheese", "yogurt"]
}

# Access nested data
inventory[:fruits][0]  # "apple"
inventory[:dairy] << "butter"  # Add to array

# Complex nested structure
company = {
  name: "RubyCorp",
  employees: [
    {
      name: "Alice",
      department: "Engineering",
      skills: ["Ruby", "JavaScript", "SQL"]
    },
    {
      name: "Bob",
      department: "Marketing",
      skills: ["SEO", "Content", "Analytics"]
    }
  ],
  locations: {
    headquarters: {
      city: "San Francisco",
      employees: 100
    },
    branch: {
      city: "New York",
      employees: 50
    }
  }
}

# Navigate the structure
company[:employees][0][:skills][0]  # "Ruby"
company[:locations][:headquarters][:city]  # "San Francisco"

# Safe navigation
company.dig(:locations, :headquarters, :city)  # "San Francisco"
company.dig(:locations, :tokyo, :city)  # nil (doesn't blow up!)
```

## Set: The Unique Collection

Ruby also has Set, which is like an array that automatically prevents duplicates:

```ruby
require 'set'

# Creating sets
numbers = Set[1, 2, 3, 4, 5]
colors = Set.new(["red", "green", "blue"])

# No duplicates allowed
s = Set[1, 1, 2, 2, 3, 3]  # Set{1, 2, 3}

# Set operations
a = Set[1, 2, 3, 4]
b = Set[3, 4, 5, 6]

a | b  # Union: Set{1, 2, 3, 4, 5, 6}
a & b  # Intersection: Set{3, 4}
a - b  # Difference: Set{1, 2}
a ^ b  # Symmetric difference: Set{1, 2, 5, 6}

# Check membership
a.include?(3)  # true
a.subset?(b)   # false
```

## Practical Example: A Grade Book System

Let's build something useful:

```ruby
class GradeBook
  def initialize(class_name)
    @class_name = class_name
    @students = {}
    @assignments = []
  end
  
  def add_student(name, id)
    @students[id] = {
      name: name,
      grades: {},
      attendance: []
    }
    puts "Added student: #{name} (ID: #{id})"
  end
  
  def add_assignment(name, points)
    @assignments << { name: name, points: points }
    puts "Added assignment: #{name} (#{points} points)"
  end
  
  def record_grade(student_id, assignment_name, score)
    student = @students[student_id]
    return puts "Student not found" unless student
    
    assignment = @assignments.find { |a| a[:name] == assignment_name }
    return puts "Assignment not found" unless assignment
    
    student[:grades][assignment_name] = {
      score: score,
      percentage: (score.to_f / assignment[:points] * 100).round(2)
    }
    puts "Recorded grade for #{student[:name]}: #{score}/#{assignment[:points]}"
  end
  
  def record_attendance(student_id, date, status)
    student = @students[student_id]
    return puts "Student not found" unless student
    
    student[:attendance] << { date: date, status: status }
  end
  
  def student_report(student_id)
    student = @students[student_id]
    return puts "Student not found" unless student
    
    puts "\n" + "="*50
    puts "Student Report: #{student[:name]}"
    puts "="*50
    
    # Grades
    puts "\nGrades:"
    total_earned = 0
    total_possible = 0
    
    @assignments.each do |assignment|
      grade = student[:grades][assignment[:name]]
      if grade
        puts "  #{assignment[:name]}: #{grade[:score]}/#{assignment[:points]} (#{grade[:percentage]}%)"
        total_earned += grade[:score]
        total_possible += assignment[:points]
      else
        puts "  #{assignment[:name]}: Missing"
        total_possible += assignment[:points]
      end
    end
    
    if total_possible > 0
      overall = (total_earned.to_f / total_possible * 100).round(2)
      puts "\nOverall Grade: #{overall}% (#{letter_grade(overall)})"
    end
    
    # Attendance
    present = student[:attendance].count { |a| a[:status] == :present }
    total_days = student[:attendance].length
    
    if total_days > 0
      puts "\nAttendance: #{present}/#{total_days} days (#{(present.to_f/total_days*100).round}%)"
    end
  end
  
  def class_statistics
    puts "\n" + "="*50
    puts "Class Statistics: #{@class_name}"
    puts "="*50
    
    all_grades = []
    
    @students.each do |id, student|
      student_total = 0
      student_possible = 0
      
      @assignments.each do |assignment|
        if grade = student[:grades][assignment[:name]]
          student_total += grade[:score]
          student_possible += assignment[:points]
        else
          student_possible += assignment[:points]
        end
      end
      
      if student_possible > 0
        percentage = (student_total.to_f / student_possible * 100).round(2)
        all_grades << percentage
      end
    end
    
    if all_grades.any?
      puts "\nGrade Distribution:"
      puts "  Highest: #{all_grades.max}%"
      puts "  Lowest: #{all_grades.min}%"
      puts "  Average: #{(all_grades.sum / all_grades.length).round(2)}%"
      
      puts "\nLetter Grade Distribution:"
      distribution = all_grades.group_by { |g| letter_grade(g) }
      distribution.sort.each do |grade, students|
        puts "  #{grade}: #{students.length} student(s)"
      end
    end
  end
  
  private
  
  def letter_grade(percentage)
    case percentage
    when 90..100 then "A"
    when 80...90 then "B"
    when 70...80 then "C"
    when 60...70 then "D"
    else "F"
    end
  end
end

# Using our gradebook
gradebook = GradeBook.new("Ruby 101")

# Add students
gradebook.add_student("Alice Anderson", "S001")
gradebook.add_student("Bob Brown", "S002")
gradebook.add_student("Charlie Chen", "S003")

# Add assignments
gradebook.add_assignment("Homework 1", 100)
gradebook.add_assignment("Quiz 1", 50)
gradebook.add_assignment("Midterm", 200)

# Record grades
gradebook.record_grade("S001", "Homework 1", 95)
gradebook.record_grade("S001", "Quiz 1", 48)
gradebook.record_grade("S001", "Midterm", 185)

gradebook.record_grade("S002", "Homework 1", 88)
gradebook.record_grade("S002", "Quiz 1", 42)
gradebook.record_grade("S002", "Midterm", 165)

# Generate reports
gradebook.student_report("S001")
gradebook.class_statistics
```

## Common Array and Hash Patterns

### The Accumulator Pattern
```ruby
# Array accumulator
sum = numbers.reduce(0) { |total, n| total + n }

# Hash accumulator
word_count = words.each_with_object(Hash.new(0)) do |word, counts|
  counts[word] += 1
end
```

### The Transform and Filter Pattern
```ruby
# Transform then filter
result = data
  .map { |item| process(item) }
  .select { |item| valid?(item) }
  .sort_by { |item| item[:priority] }
```

### The Lookup Table Pattern
```ruby
COLORS = {
  r: "red",
  g: "green",
  b: "blue"
}.freeze

def expand_color(code)
  COLORS[code] || "unknown"
end
```

### The Cache Pattern
```ruby
@cache ||= {}

def expensive_operation(input)
  @cache[input] ||= begin
    # Expensive computation here
    sleep 2
    input * 2
  end
end
```

## Performance Tips

1. **Use symbols as hash keys** - They're faster and use less memory
2. **Freeze constants** - `CONSTANT = [1, 2, 3].freeze`
3. **Use Set for membership tests** - Much faster than `array.include?`
4. **Avoid nested loops** - Use hash lookups instead
5. **Pre-allocate arrays when size is known** - `Array.new(100)`

## Your Turn: Collection Challenges

1. **Frequency Counter**: Count occurrences of elements in an array
2. **Group Anagrams**: Group words that are anagrams of each other
3. **Shopping Cart**: Build a cart system with add, remove, and total
4. **Playlist Manager**: Create, shuffle, and manage music playlists
5. **Two Sum Problem**: Find pairs in array that sum to target value

## What You've Learned

You now know how to:
- Create and manipulate arrays
- Access elements with various indexing methods
- Iterate over arrays with powerful methods
- Create and modify hashes
- Work with nested data structures
- Use sets for unique collections
- Apply common patterns for data manipulation

## What's Next?

In the next chapter, we'll explore Ruby's object-oriented nature in depth. We'll see how literally everything in Ruby is an object and what that means for your code. It's time to peek behind the curtain and see how Ruby really works!

Remember: Arrays and hashes are your workhorses in Ruby. Master them, and you'll be able to handle 90% of your data manipulation needs. The other 10%? That's what Stack Overflow is for.