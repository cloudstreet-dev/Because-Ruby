# Chapter 5: Control Flow - Teaching Ruby to Make Decisions

So far, our Ruby programs have been like a train on a track—they start at the top, chug along line by line, and stop at the bottom. But real programs need to make decisions, repeat tasks, and occasionally jump around like a caffeinated squirrel. That's where control flow comes in.

## If Statements: The Decision Makers

The `if` statement is programming's way of saying "maybe." It's how we teach Ruby to make choices:

```ruby
age = 18

if age >= 18
  puts "You can vote!"
end

# The one-liner version (for when you're feeling concise)
puts "You can vote!" if age >= 18

# With else (for when there's a plan B)
if age >= 18
  puts "You can vote!"
else
  puts "Sorry, not yet!"
end

# With elsif (yes, it's spelled weird)
if age >= 65
  puts "Senior discount!"
elsif age >= 18
  puts "Regular price"
elsif age >= 13
  puts "Student discount"
else
  puts "Kids eat free!"
end
```

Ruby also has `unless`, which is `if`'s pessimistic sibling:

```ruby
unless hungry
  puts "I don't need food"
end

# Same as:
if !hungry
  puts "I don't need food"
end

# One-liner unless
puts "Let's eat!" unless full
```

Pro tip: Use `unless` sparingly. It's great for simple conditions, but `unless` with `else` will melt your brain.

## The Ternary Operator: If's Compact Cousin

When you need a quick if/else, there's the ternary operator:

```ruby
# Long way
if age >= 18
  status = "adult"
else
  status = "minor"
end

# Ternary way
status = age >= 18 ? "adult" : "minor"

# Great for simple assignments
message = score > 90 ? "Excellent!" : "Keep trying!"
price = is_member ? 9.99 : 14.99
```

Don't nest these. Seriously. Your future self will thank you.

## Case Statements: If's Organized Sibling

When you have multiple conditions checking the same variable, `case` is your friend:

```ruby
grade = 'B'

case grade
when 'A'
  puts "Outstanding!"
when 'B'
  puts "Good job!"
when 'C'
  puts "You passed"
when 'D'
  puts "Barely made it"
when 'F'
  puts "See you next semester"
else
  puts "Invalid grade"
end

# Case works with ranges too!
score = 85

case score
when 90..100
  puts "A"
when 80..89
  puts "B"
when 70..79
  puts "C"
when 60..69
  puts "D"
else
  puts "F"
end

# And with types!
value = "hello"

case value
when String
  puts "It's text"
when Numeric
  puts "It's a number"
when TrueClass, FalseClass
  puts "It's a boolean"
else
  puts "I don't know what this is"
end

# Case can even use regexes!
email = "user@example.com"

case email
when /.*@gmail\.com/
  puts "Gmail user"
when /.*@yahoo\.com/
  puts "Yahoo user"
when /.*@.*\..*/
  puts "Some other email"
else
  puts "That's not an email"
end
```

## Loops: Making Ruby Repeat Itself

### While Loops: Keep Going Until...

```ruby
# Basic while loop
counter = 0
while counter < 5
  puts "Count: #{counter}"
  counter += 1
end

# Until loop (while's opposite)
counter = 0
until counter == 5
  puts "Count: #{counter}"
  counter += 1
end

# Infinite loop (Ctrl+C to escape!)
while true
  puts "This will go forever!"
  sleep 1  # Let's not spam too hard
end
```

### For Loops: The Rarely-Used-in-Ruby Loop

```ruby
# For loop (exists but isn't very Ruby-like)
for i in 1..5
  puts "Number #{i}"
end

# Rubyists prefer each
(1..5).each do |i|
  puts "Number #{i}"
end
```

### Loop Method: The Simple Infinite Loop

```ruby
# Basic infinite loop
loop do
  puts "Type 'exit' to quit:"
  input = gets.chomp
  break if input == 'exit'
end

# With a counter
count = 0
loop do
  puts count
  count += 1
  break if count >= 10
end
```

## Iterators: The Ruby Way

This is where Ruby shines. Iterators are methods that loop for you:

```ruby
# Times iterator
5.times do
  puts "Hello!"
end

# With index
5.times do |i|
  puts "Iteration #{i}"
end

# Each iterator (for collections)
fruits = ["apple", "banana", "orange"]
fruits.each do |fruit|
  puts "I like #{fruit}"
end

# Each with index
fruits.each_with_index do |fruit, index|
  puts "#{index + 1}. #{fruit}"
end

# Map (transform each element)
numbers = [1, 2, 3, 4, 5]
doubled = numbers.map { |n| n * 2 }
# doubled is now [2, 4, 6, 8, 10]

# Select (filter elements)
evens = numbers.select { |n| n.even? }
# evens is now [2, 4]

# Reject (opposite of select)
odds = numbers.reject { |n| n.even? }
# odds is now [1, 3, 5]

# Any? and All?
numbers.any? { |n| n > 3 }  # true
numbers.all? { |n| n > 0 }  # true

# Find (first matching element)
numbers.find { |n| n > 3 }  # 4
```

## Break, Next, and Redo: Loop Control

Sometimes you need to control the flow within a loop:

```ruby
# Break: Exit the loop entirely
10.times do |i|
  puts i
  break if i == 5  # Stops at 5
end

# Next: Skip to the next iteration
10.times do |i|
  next if i.even?  # Skip even numbers
  puts i
end

# Redo: Repeat the current iteration
attempts = 0
5.times do |i|
  attempts += 1
  puts "Attempt #{attempts} for iteration #{i}"
  redo if attempts < 3 && i == 2  # Retry iteration 2 twice
end

# Return: Exit the entire method (not just the loop)
def find_number
  10.times do |i|
    return i if i == 5  # Exits the method, returning 5
  end
  "Not found"  # This won't execute if we return early
end
```

## Modifier Forms: Ruby's Syntactic Sugar

Ruby lets you put conditions and loops at the end of statements:

```ruby
# If modifier
puts "You're an adult" if age >= 18

# Unless modifier
puts "Come back later" unless store_open

# While modifier
counter += 1 while counter < 10

# Until modifier
sleep 1 until awake
```

These are great for simple, single-line operations. Don't use them for complex logic.

## Practical Example: A Text Adventure Game

Let's put it all together with a mini text adventure:

```ruby
# adventure.rb - A Ruby Control Flow Adventure

def adventure_game
  puts "="*50
  puts " Welcome to Ruby Adventure! "
  puts "="*50
  
  player_health = 100
  player_gold = 0
  game_over = false
  rooms_explored = 0
  
  until game_over
    puts "\n" + "-"*30
    puts "Health: #{player_health} | Gold: #{player_gold}"
    puts "You're in a dark room. What do you do?"
    puts "1. Go north"
    puts "2. Go south" 
    puts "3. Search room"
    puts "4. Check inventory"
    puts "5. Rest"
    puts "6. Quit"
    
    choice = gets.chomp.to_i
    
    case choice
    when 1  # Go north
      puts "\nYou go north..."
      
      encounter = rand(1..3)
      case encounter
      when 1
        puts "You found a treasure chest!"
        gold_found = rand(10..30)
        player_gold += gold_found
        puts "You gained #{gold_found} gold!"
      when 2
        puts "Oh no! A goblin appears!"
        if rand(1..2) == 1
          damage = rand(5..15)
          player_health -= damage
          puts "The goblin hits you for #{damage} damage!"
        else
          puts "You dodge the goblin's attack!"
          puts "The goblin runs away in fear!"
        end
      when 3
        puts "The room is empty, but peaceful."
        rooms_explored += 1
      end
      
    when 2  # Go south
      puts "\nYou go south..."
      
      if rand(1..10) > 7
        puts "You fell into a trap!"
        player_health -= 20
      else
        puts "You find a healing potion!"
        heal = rand(10..25)
        player_health += heal
        player_health = [player_health, 100].min  # Cap at 100
        puts "You heal for #{heal} points!"
      end
      
    when 3  # Search room
      puts "\nYou search the room carefully..."
      
      3.times do |i|
        sleep 0.5
        print "."
      end
      puts
      
      found_something = rand(1..10) > 5
      
      if found_something
        items = ["a rusty key", "5 gold coins", "an old map", "a magic scroll"]
        item = items.sample
        puts "You found #{item}!"
        player_gold += 5 if item.include?("gold")
      else
        puts "You found nothing of interest."
      end
      
    when 4  # Check inventory
      puts "\nInventory:"
      puts "- Wooden sword"
      puts "- Torch"
      puts "- #{player_gold} gold coins"
      puts "- Hope and determination"
      
    when 5  # Rest
      if player_health >= 100
        puts "\nYou're already at full health!"
      else
        puts "\nYou rest for a while..."
        heal_amount = 0
        
        3.times do
          sleep 0.5
          heal = rand(5..10)
          heal_amount += heal
          print "Zzz... (#{heal} HP) "
        end
        
        player_health += heal_amount
        player_health = [player_health, 100].min
        puts "\nYou healed for #{heal_amount} total HP!"
      end
      
    when 6  # Quit
      puts "\nAre you sure you want to quit? (y/n)"
      confirm = gets.chomp.downcase
      game_over = true if confirm == 'y'
      
    else
      puts "\nInvalid choice! Try again."
    end
    
    # Check game over conditions
    if player_health <= 0
      puts "\n" + "="*50
      puts " GAME OVER - You have died! "
      puts "="*50
      puts "Final stats:"
      puts "- Gold collected: #{player_gold}"
      puts "- Rooms explored: #{rooms_explored}"
      game_over = true
    elsif player_gold >= 100
      puts "\n" + "="*50
      puts " VICTORY! You collected 100 gold! "
      puts "="*50
      puts "Final health: #{player_health}"
      game_over = true
    end
  end
  
  puts "\nThanks for playing Ruby Adventure!"
end

# Start the game
adventure_game
```

## Control Flow Best Practices

### 1. Keep It Simple
```ruby
# Bad - too nested
if condition1
  if condition2
    if condition3
      do_something
    end
  end
end

# Good - use guard clauses
return unless condition1
return unless condition2
return unless condition3
do_something
```

### 2. Use the Right Tool
```ruby
# Bad - using if/elsif for many conditions
if day == "Monday"
  schedule = "Work"
elsif day == "Tuesday"
  schedule = "Work"
elsif day == "Saturday"
  schedule = "Rest"
# ... etc

# Good - use case
case day
when "Monday", "Tuesday", "Wednesday", "Thursday", "Friday"
  schedule = "Work"
when "Saturday", "Sunday"
  schedule = "Rest"
end
```

### 3. Prefer Iterators Over Loops
```ruby
# Bad - manual loop
i = 0
while i < array.length
  puts array[i]
  i += 1
end

# Good - use iterator
array.each { |item| puts item }
```

### 4. Exit Early
```ruby
# Bad - deeply nested
def process(user)
  if user
    if user.active?
      if user.subscription_valid?
        # do stuff
      end
    end
  end
end

# Good - exit early
def process(user)
  return unless user
  return unless user.active?
  return unless user.subscription_valid?
  
  # do stuff
end
```

## Common Control Flow Patterns

### The Guard Clause
```ruby
def divide(a, b)
  return "Error: Division by zero" if b == 0
  a / b
end
```

### The State Machine
```ruby
state = :pending

loop do
  case state
  when :pending
    puts "Processing..."
    state = :processing
  when :processing
    puts "Completing..."
    state = :complete
  when :complete
    puts "Done!"
    break
  end
end
```

### The Retry Pattern
```ruby
retries = 0
max_retries = 3

begin
  # Try something that might fail
  risky_operation
rescue => e
  retries += 1
  if retries <= max_retries
    puts "Attempt #{retries} failed, retrying..."
    retry
  else
    puts "Max retries reached"
    raise e
  end
end
```

## Your Turn: Control Flow Challenges

1. **Password Validator**: Check if a password meets multiple criteria
2. **Number Guesser**: Create a game where the computer guesses your number
3. **Menu System**: Build a interactive menu with nested options
4. **Pattern Printer**: Use loops to print various patterns (triangles, diamonds)
5. **FizzBuzz**: The classic interview question (divisible by 3: "Fizz", by 5: "Buzz", by both: "FizzBuzz")

## What You've Learned

You can now:
- Make decisions with if/elsif/else and unless
- Use case statements for multiple conditions
- Create loops with while, until, and loop
- Use Ruby's iterators (the Ruby way!)
- Control loop flow with break, next, and redo
- Write more complex, branching programs

## What's Next?

In the next chapter, we'll learn about methods—how to package up your code into reusable chunks. No more copying and pasting the same code over and over! Methods are like teaching Ruby new tricks that it can perform whenever you ask.

Remember: Control flow is what makes your programs smart. Without it, your code is just a list of instructions. With it, your code can adapt, respond, and make decisions. It's the difference between a recipe and a chef.

Now go write something with lots of branches and loops. Make Ruby jump through hoops (literally, with loops). The more you practice control flow, the more natural it becomes. Soon you'll be writing elegant conditionals and beautiful iterators without even thinking about it.

And remember: when in doubt, there's probably an iterator for that!