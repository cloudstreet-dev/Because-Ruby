# Chapter 11: Inheritance - It's a Family Thing

Inheritance is how classes can have parents and children, passing down attributes and behaviors through generations. It's like genetics, but for code, and thankfully without any of the awkward family dinners.

## Basic Inheritance: Like Parent, Like Child

In Ruby, a class can inherit from another class using the `<` symbol:

```ruby
# Parent class (superclass)
class Animal
  attr_reader :name, :species
  
  def initialize(name, species)
    @name = name
    @species = species
    @age = 0
  end
  
  def speak
    "Some generic animal sound"
  end
  
  def eat(food)
    puts "#{@name} is eating #{food}"
  end
  
  def sleep
    puts "#{@name} is sleeping... Zzz"
  end
  
  def birthday
    @age += 1
    puts "#{@name} is now #{@age} years old!"
  end
end

# Child class (subclass)
class Dog < Animal
  def initialize(name, breed)
    super(name, "Canine")  # Call parent's initialize
    @breed = breed
  end
  
  def speak
    "Woof!"  # Override parent method
  end
  
  def fetch(item)
    puts "#{@name} fetches the #{item}!"
  end
end

# Another child class
class Cat < Animal
  def initialize(name, indoor = true)
    super(name, "Feline")
    @indoor = indoor
  end
  
  def speak
    "Meow"
  end
  
  def scratch(item)
    puts "#{@name} scratches the #{item}"
  end
end

# Using inheritance
dog = Dog.new("Buddy", "Golden Retriever")
cat = Cat.new("Whiskers")

dog.speak        # "Woof!" (overridden method)
dog.eat("kibble") # "Buddy is eating kibble" (inherited method)
dog.fetch("ball") # "Buddy fetches the ball!" (new method)

cat.speak        # "Meow"
cat.sleep        # "Whiskers is sleeping... Zzz"
cat.birthday     # "Whiskers is now 1 years old!"
```

## The Super Keyword: Calling Mom and Dad

The `super` keyword calls the parent class's version of the current method:

```ruby
class Vehicle
  def initialize(make, model, year)
    @make = make
    @model = model
    @year = year
    @mileage = 0
  end
  
  def description
    "#{@year} #{@make} #{@model}"
  end
  
  def drive(miles)
    @mileage += miles
    puts "Drove #{miles} miles"
  end
end

class Car < Vehicle
  def initialize(make, model, year, doors = 4)
    super(make, model, year)  # Calls parent's initialize with arguments
    @doors = doors
  end
  
  def description
    super + " with #{@doors} doors"  # Calls parent's description and adds to it
  end
end

class Motorcycle < Vehicle
  def initialize(make, model, year, type)
    super(make, model, year)  # Pass first 3 args to parent
    @type = type  # sport, cruiser, touring, etc.
  end
  
  def drive(miles)
    puts "Revving engine..."
    super  # Calls parent's drive method
    puts "What a ride!"
  end
end

car = Car.new("Toyota", "Camry", 2024)
puts car.description  # "2024 Toyota Camry with 4 doors"

bike = Motorcycle.new("Harley", "Sportster", 2023, "cruiser")
bike.drive(100)
# Revving engine...
# Drove 100 miles
# What a ride!
```

## Method Lookup Chain: Following the Family Tree

Ruby looks for methods by climbing up the inheritance chain:

```ruby
class Grandparent
  def family_method
    "From Grandparent"
  end
end

class Parent < Grandparent
  def family_method
    "From Parent"
  end
end

class Child < Parent
end

child = Child.new
child.family_method  # "From Parent" (found in Parent, stops looking)

# See the lookup chain
p Child.ancestors
# [Child, Parent, Grandparent, Object, Kernel, BasicObject]

# Check where a method comes from
p child.method(:family_method).owner  # Parent
p child.method(:to_s).owner          # Object
```

## Protected and Private Inheritance

Inherited methods maintain their visibility:

```ruby
class BankAccount
  def initialize(balance)
    @balance = balance
  end
  
  def public_balance
    "$#{@balance}"
  end
  
  protected
  
  def protected_balance
    @balance
  end
  
  private
  
  def private_transaction_log
    "Secret transaction data"
  end
end

class SavingsAccount < BankAccount
  def initialize(balance, interest_rate)
    super(balance)
    @interest_rate = interest_rate
  end
  
  def apply_interest
    # Can access protected method from parent
    interest = protected_balance * @interest_rate
    @balance += interest
    puts "Applied #{@interest_rate * 100}% interest: +$#{interest}"
  end
  
  def try_private
    # This would cause an error - private methods aren't inherited
    # private_transaction_log
  end
  
  def compare_balance(other_account)
    # Protected methods can be called on other instances of same class family
    if other_account.protected_balance > protected_balance
      "Other account has more money"
    else
      "This account has more money"
    end
  end
end

savings = SavingsAccount.new(1000, 0.05)
savings.apply_interest  # Works - can access protected method
```

## Class Variables in Inheritance: Shared Family Secrets

Class variables are shared across the entire inheritance hierarchy:

```ruby
class Animal
  @@total_animals = 0
  
  def initialize(name)
    @name = name
    @@total_animals += 1
  end
  
  def self.population
    @@total_animals
  end
end

class Dog < Animal
  @@total_dogs = 0
  
  def initialize(name)
    super
    @@total_dogs += 1
  end
  
  def self.dog_count
    @@total_dogs
  end
end

class Cat < Animal
  @@total_cats = 0
  
  def initialize(name)
    super
    @@total_cats += 1
  end
  
  def self.cat_count
    @@total_cats
  end
end

Dog.new("Buddy")
Dog.new("Max")
Cat.new("Whiskers")

puts Animal.population  # 3 (all animals)
puts Dog.dog_count     # 2
puts Cat.cat_count     # 1
```

## Abstract Classes: Templates for Children

Ruby doesn't have true abstract classes, but we can simulate them:

```ruby
class Shape
  def initialize
    raise NotImplementedError, "Cannot instantiate abstract class" if self.class == Shape
  end
  
  def area
    raise NotImplementedError, "Subclass must implement area method"
  end
  
  def perimeter
    raise NotImplementedError, "Subclass must implement perimeter method"
  end
  
  def description
    "This is a #{self.class.name} with area #{area} and perimeter #{perimeter}"
  end
end

class Rectangle < Shape
  def initialize(width, height)
    super()  # Calls Shape's initialize
    @width = width
    @height = height
  end
  
  def area
    @width * @height
  end
  
  def perimeter
    2 * (@width + @height)
  end
end

class Circle < Shape
  def initialize(radius)
    super()
    @radius = radius
  end
  
  def area
    Math::PI * @radius ** 2
  end
  
  def perimeter
    2 * Math::PI * @radius
  end
end

# shape = Shape.new  # Error! Cannot instantiate abstract class
rect = Rectangle.new(5, 10)
puts rect.description  # Works!

circle = Circle.new(7)
puts circle.description  # Also works!
```

## Multiple Inheritance: Why Ruby Says No (But Modules Say Yes)

Ruby only allows single inheritance, but modules provide multiple inheritance of behavior:

```ruby
# Can't do this:
# class FlyingCar < Car, Airplane  # Error!

# But can do this:
module Flyable
  def fly
    puts "Taking off!"
  end
  
  def land
    puts "Landing safely"
  end
end

module Driveable
  def drive
    puts "Driving on the road"
  end
  
  def park
    puts "Parking"
  end
end

class FlyingCar
  include Driveable
  include Flyable
  
  def initialize(model)
    @model = model
  end
  
  def travel_mode
    puts "Choose your travel mode:"
    puts "1. Drive"
    puts "2. Fly"
    choice = gets.chomp.to_i
    
    case choice
    when 1 then drive
    when 2 then fly
    else puts "Invalid choice"
    end
  end
end

flying_car = FlyingCar.new("AeroMobile")
flying_car.drive
flying_car.fly
```

## Composition vs Inheritance: Has-a vs Is-a

Sometimes composition (has-a) is better than inheritance (is-a):

```ruby
# Inheritance approach (is-a relationship)
class Car < Vehicle
  # Car IS A Vehicle
end

# Composition approach (has-a relationship)
class Engine
  attr_reader :horsepower, :type
  
  def initialize(horsepower, type)
    @horsepower = horsepower
    @type = type
  end
  
  def start
    puts "Engine starting... Vroom!"
  end
  
  def stop
    puts "Engine stopping"
  end
end

class Transmission
  attr_reader :type, :gears
  
  def initialize(type, gears)
    @type = type  # manual or automatic
    @gears = gears
  end
  
  def shift(gear)
    puts "Shifting to gear #{gear}"
  end
end

class Car
  attr_reader :make, :model, :engine, :transmission
  
  def initialize(make, model)
    @make = make
    @model = model
    @engine = Engine.new(200, "V6")  # Car HAS AN Engine
    @transmission = Transmission.new("automatic", 6)  # Car HAS A Transmission
  end
  
  def start
    @engine.start
    puts "#{@make} #{@model} is ready to drive!"
  end
  
  def drive
    @engine.start unless @engine_running
    @transmission.shift(1)
    puts "Driving..."
    @transmission.shift(2)
  end
end

car = Car.new("Toyota", "Camry")
car.start
car.drive
```

## Practical Example: Building a Game Class Hierarchy

Let's create a more complex example with multiple levels of inheritance:

```ruby
# Base entity class
class Entity
  attr_reader :x, :y, :health, :max_health
  
  def initialize(x, y, health)
    @x = x
    @y = y
    @health = health
    @max_health = health
    @alive = true
  end
  
  def move(dx, dy)
    @x += dx
    @y += dy
    puts "#{self.class.name} moved to (#{@x}, #{@y})"
  end
  
  def take_damage(amount)
    @health -= amount
    @health = 0 if @health < 0
    puts "#{self.class.name} takes #{amount} damage! (#{@health}/#{@max_health} HP)"
    die if @health <= 0
  end
  
  def alive?
    @alive
  end
  
  protected
  
  def die
    @alive = false
    puts "#{self.class.name} has died!"
  end
end

# Character class (NPCs and Players)
class Character < Entity
  attr_reader :name, :level
  
  def initialize(name, x, y, health, level = 1)
    super(x, y, health)
    @name = name
    @level = level
    @inventory = []
  end
  
  def speak(message)
    puts "#{@name}: #{message}"
  end
  
  def pick_up(item)
    @inventory << item
    puts "#{@name} picked up #{item}"
  end
  
  def show_inventory
    if @inventory.empty?
      puts "#{@name}'s inventory is empty"
    else
      puts "#{@name}'s inventory: #{@inventory.join(', ')}"
    end
  end
end

# Player class
class Player < Character
  attr_reader :experience, :skill_points
  
  def initialize(name, x = 0, y = 0)
    super(name, x, y, 100, 1)
    @experience = 0
    @skill_points = 0
    @skills = {}
  end
  
  def gain_experience(amount)
    @experience += amount
    puts "Gained #{amount} XP! (Total: #{@experience})"
    check_level_up
  end
  
  def learn_skill(skill_name)
    return puts "Not enough skill points!" if @skill_points < 1
    
    @skills[skill_name] = true
    @skill_points -= 1
    puts "Learned new skill: #{skill_name}!"
  end
  
  private
  
  def check_level_up
    required_xp = @level * 100
    if @experience >= required_xp
      @level += 1
      @skill_points += 2
      @max_health += 10
      @health = @max_health
      puts "🎉 LEVEL UP! You are now level #{@level}!"
      puts "Gained 2 skill points and 10 max HP!"
    end
  end
end

# Enemy class
class Enemy < Character
  attr_reader :damage, :reward_xp
  
  def initialize(name, x, y, health, damage, reward_xp)
    super(name, x, y, health)
    @damage = damage
    @reward_xp = reward_xp
    @aggressive = true
  end
  
  def attack(target)
    return puts "#{@name} is dead and cannot attack!" unless alive?
    
    puts "#{@name} attacks #{target.name}!"
    target.take_damage(@damage)
    
    # Reward player if enemy dies
    if !target.alive? && target.is_a?(Enemy)
      gain_experience(target.reward_xp) if self.is_a?(Player)
    end
  end
  
  def patrol
    direction = rand(4)
    case direction
    when 0 then move(0, 1)   # North
    when 1 then move(1, 0)   # East
    when 2 then move(0, -1)  # South
    when 3 then move(-1, 0)  # West
    end
  end
end

# Boss class (special enemy)
class Boss < Enemy
  def initialize(name, x, y)
    super(name, x, y, 500, 25, 500)
    @phase = 1
    @special_attacks = ["Fireball", "Lightning Strike", "Earthquake"]
  end
  
  def take_damage(amount)
    super
    check_phase_change
  end
  
  def special_attack(target)
    attack_name = @special_attacks.sample
    damage = @damage * 2
    puts "#{@name} uses #{attack_name}!"
    target.take_damage(damage)
  end
  
  private
  
  def check_phase_change
    health_percentage = (@health.to_f / @max_health) * 100
    
    if health_percentage <= 50 && @phase == 1
      @phase = 2
      @damage *= 1.5
      puts "#{@name} enters phase 2! Damage increased!"
    elsif health_percentage <= 25 && @phase == 2
      @phase = 3
      @damage *= 1.5
      puts "#{@name} enters final phase! MAXIMUM POWER!"
    end
  end
  
  def die
    super
    puts "💀 The mighty #{@name} has been defeated!"
    puts "🎯 BOSS DEFEATED! Massive XP reward!"
  end
end

# Game simulation
player = Player.new("Hero", 5, 5)
goblin = Enemy.new("Goblin", 3, 3, 30, 10, 25)
boss = Boss.new("Dark Lord", 10, 10)

player.speak("Time to save the kingdom!")
player.pick_up("Sword")
player.pick_up("Health Potion")
player.show_inventory

# Battle!
goblin.attack(player)
player.take_damage(5)

# Simulate defeating the goblin
goblin.take_damage(35)
player.gain_experience(25)

# Boss battle!
boss.special_attack(player)
player.learn_skill("Healing")

# Show inheritance chain
p Player.ancestors[0..5]
# [Player, Character, Entity, Object, Kernel, BasicObject]
```

## When to Use Inheritance

**Use inheritance when:**
- There's a clear "is-a" relationship
- You want to share implementation, not just interface
- The child truly specializes the parent
- The relationship is stable and unlikely to change

**Avoid inheritance when:**
- The relationship is "has-a" (use composition)
- You need multiple inheritance (use modules)
- The hierarchy gets deeper than 3 levels
- You're inheriting just to reuse code (use modules)

## Your Turn: Inheritance Challenges

1. **Vehicle Hierarchy**: Create Car, Truck, and Motorcycle classes
2. **Employee System**: Build Employee, Manager, and Executive classes
3. **Media Library**: Design MediaItem, Book, Movie, and Music classes
4. **Restaurant Orders**: Create Order, DineIn, Takeout, and Delivery classes
5. **School System**: Build Person, Student, Teacher, and Principal classes

## What You've Learned

You now understand:
- How to create class hierarchies with inheritance
- Using super to call parent methods
- Method lookup chains
- When to use inheritance vs composition
- How to combine inheritance with modules
- Protected vs private inheritance

## What's Next?

Next, we'll dive into blocks, procs, and lambdas—Ruby's powerful way of passing code around like data. These are the features that make Ruby feel magical and enable its elegant, expressive style.

Remember: Inheritance is powerful, but with great power comes great responsibility. Deep inheritance hierarchies can become hard to understand and maintain. Often, composition and modules provide cleaner solutions. Think of inheritance as one tool in your toolbox, not the only tool. And remember the golden rule: prefer composition over inheritance!