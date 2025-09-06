# Chapter 9: Classes - Building Your Own Objects

So far, we've been using Ruby's built-in objects like strings and arrays. But what if you want to create your own types of objects? What if Ruby doesn't have a `Dragon` class or a `Pizza` class? Well, that's where you come in. Time to play deity and create your own objects!

## Your First Class

A class is like a blueprint or a cookie cutter. It defines what attributes and behaviors all objects of that type will have:

```ruby
# Define a class
class Dog
  def initialize(name, breed)
    @name = name
    @breed = breed
  end
  
  def bark
    puts "#{@name} says Woof!"
  end
  
  def introduce
    puts "Hi, I'm #{@name}, a #{@breed}."
  end
end

# Create instances (objects) from the class
fido = Dog.new("Fido", "Golden Retriever")
buddy = Dog.new("Buddy", "Beagle")

fido.bark       # "Fido says Woof!"
buddy.introduce # "Hi, I'm Buddy, a Beagle."
```

## The Initialize Method: The Birth of an Object

The `initialize` method is special—it runs automatically when you create a new object with `new`:

```ruby
class Person
  def initialize(name, age)
    @name = name  # @ means instance variable
    @age = age
    @created_at = Time.now
    puts "A new person named #{@name} has been created!"
  end
end

alice = Person.new("Alice", 30)
# Prints: "A new person named Alice has been created!"

# You can have default parameters
class Coffee
  def initialize(size = "medium", type = "latte", extra_shot = false)
    @size = size
    @type = type
    @extra_shot = extra_shot
  end
end

morning_coffee = Coffee.new
fancy_coffee = Coffee.new("large", "cappuccino", true)
```

## Instance Variables: An Object's Memory

Instance variables (prefixed with `@`) belong to each individual object:

```ruby
class BankAccount
  def initialize(owner, balance = 0)
    @owner = owner
    @balance = balance
    @transactions = []
  end
  
  def deposit(amount)
    @balance += amount
    @transactions << { type: "deposit", amount: amount, time: Time.now }
    puts "Deposited $#{amount}. New balance: $#{@balance}"
  end
  
  def withdraw(amount)
    if amount <= @balance
      @balance -= amount
      @transactions << { type: "withdrawal", amount: amount, time: Time.now }
      puts "Withdrew $#{amount}. New balance: $#{@balance}"
    else
      puts "Insufficient funds!"
    end
  end
  
  def statement
    puts "Account owner: #{@owner}"
    puts "Current balance: $#{@balance}"
    puts "Transaction history:"
    @transactions.each do |t|
      puts "  #{t[:type]}: $#{t[:amount]} at #{t[:time].strftime('%Y-%m-%d %H:%M')}"
    end
  end
end

account = BankAccount.new("Alice", 100)
account.deposit(50)
account.withdraw(30)
account.statement
```

## Attr_accessor and Friends: Getters and Setters Made Easy

Writing getter and setter methods is tedious. Ruby gives us shortcuts:

```ruby
# The hard way
class PersonHardWay
  def initialize(name, age)
    @name = name
    @age = age
  end
  
  def name  # Getter
    @name
  end
  
  def name=(new_name)  # Setter
    @name = new_name
  end
  
  def age  # Getter
    @age
  end
  
  def age=(new_age)  # Setter
    @age = new_age
  end
end

# The Ruby way
class Person
  attr_accessor :name, :age  # Creates both getters and setters
  
  def initialize(name, age)
    @name = name
    @age = age
  end
end

# Other options
class Product
  attr_reader :id, :created_at    # Read-only (getter only)
  attr_writer :internal_notes      # Write-only (setter only)
  attr_accessor :name, :price      # Read and write
  
  def initialize(id, name, price)
    @id = id
    @name = name
    @price = price
    @created_at = Time.now
    @internal_notes = ""
  end
end

product = Product.new(1, "Ruby Book", 29.99)
product.name              # "Ruby Book" (reader)
product.name = "New Name" # Works (accessor)
product.id = 2            # Error! (read-only)
```

## Class Variables and Methods: Shared Among All Instances

Sometimes you want data or behavior that belongs to the class itself, not individual instances:

```ruby
class User
  @@user_count = 0  # Class variable (shared by all instances)
  
  attr_reader :username
  
  def initialize(username)
    @username = username
    @@user_count += 1
  end
  
  # Class method
  def self.total_users
    @@user_count
  end
  
  # Another way to define class methods
  class << self
    def find_by_username(username)
      # In real app, this would query a database
      puts "Searching for #{username}..."
    end
    
    def create_guest_user
      new("guest_#{rand(1000)}")
    end
  end
end

alice = User.new("alice")
bob = User.new("bob")

User.total_users  # 2
User.find_by_username("alice")
guest = User.create_guest_user
```

## Method Visibility: Public, Private, and Protected

Control who can call your methods:

```ruby
class Diary
  def initialize
    @entries = []
  end
  
  # Public methods (default)
  def add_entry(text)
    entry = format_entry(text)
    @entries << encrypt(entry)
    puts "Entry added!"
  end
  
  def read_entries
    @entries.map { |e| decrypt(e) }
  end
  
  private  # Everything below is private
  
  def format_entry(text)
    {
      content: text,
      timestamp: Time.now,
      word_count: text.split.length
    }
  end
  
  def encrypt(data)
    # Super secure encryption (not really)
    data.to_s.reverse
  end
  
  def decrypt(data)
    data.reverse
  end
  
  protected  # Can be called by other instances of same class
  
  def share_with(other_diary)
    other_diary.receive_entries(@entries)
  end
  
  def receive_entries(entries)
    @entries.concat(entries)
  end
end

diary = Diary.new
diary.add_entry("Dear diary, today I learned about classes!")
diary.read_entries
# diary.encrypt("test")  # Error! Private method
```

## Class Constants

Constants defined in a class are accessible with `::`:

```ruby
class HttpClient
  DEFAULT_TIMEOUT = 30
  MAX_RETRIES = 3
  
  STATUS_CODES = {
    200 => "OK",
    404 => "Not Found",
    500 => "Internal Server Error"
  }.freeze
  
  def self.status_message(code)
    STATU_CODES[code] || "Unknown"
  end
end

puts HttpClient::DEFAULT_TIMEOUT  # 30
puts HttpClient::STATU_CODES[200]  # "OK"
```

## Self: Know Thyself

Inside a class, `self` means different things in different contexts:

```ruby
class SelfDemo
  puts "In class definition, self is: #{self}"  # SelfDemo (the class)
  
  def instance_method
    puts "In instance method, self is: #{self}"  # The instance
  end
  
  def self.class_method
    puts "In class method, self is: #{self}"  # The class
  end
  
  def call_another_method
    self.instance_method  # self is optional here
    instance_method       # Same thing
  end
end

SelfDemo.class_method
obj = SelfDemo.new
obj.instance_method
```

## Operator Overloading: Making Your Objects Mathematical

You can define how operators work with your objects:

```ruby
class Vector
  attr_accessor :x, :y
  
  def initialize(x, y)
    @x = x
    @y = y
  end
  
  def +(other)
    Vector.new(@x + other.x, @y + other.y)
  end
  
  def -(other)
    Vector.new(@x - other.x, @y - other.y)
  end
  
  def *(scalar)
    Vector.new(@x * scalar, @y * scalar)
  end
  
  def ==(other)
    @x == other.x && @y == other.y
  end
  
  def [](index)
    case index
    when 0 then @x
    when 1 then @y
    else nil
    end
  end
  
  def to_s
    "(#{@x}, #{@y})"
  end
  
  def magnitude
    Math.sqrt(@x**2 + @y**2)
  end
end

v1 = Vector.new(3, 4)
v2 = Vector.new(1, 2)

v3 = v1 + v2  # Vector.new(4, 6)
v4 = v1 * 2   # Vector.new(6, 8)
puts v3       # "(4, 6)"
v1[0]         # 3
v1.magnitude  # 5.0
```

## Comparable: Teaching Your Objects to Sort Themselves

Include the Comparable module and define `<=>` to get sorting for free:

```ruby
class Version
  include Comparable
  
  attr_reader :major, :minor, :patch
  
  def initialize(version_string)
    @major, @minor, @patch = version_string.split('.').map(&:to_i)
  end
  
  def <=>(other)
    [major, minor, patch] <=> [other.major, other.minor, other.patch]
  end
  
  def to_s
    "#{major}.#{minor}.#{patch}"
  end
end

v1 = Version.new("2.1.3")
v2 = Version.new("2.0.5")
v3 = Version.new("2.1.3")

v1 > v2   # true
v1 == v3  # true
v1 < v2   # false

versions = [
  Version.new("1.0.0"),
  Version.new("2.1.3"),
  Version.new("1.2.0"),
  Version.new("2.0.0")
]

versions.sort.map(&:to_s)
# ["1.0.0", "1.2.0", "2.0.0", "2.1.3"]
```

## Class Composition: Objects Containing Objects

Complex objects are often built from simpler ones:

```ruby
class Address
  attr_accessor :street, :city, :state, :zip
  
  def initialize(street, city, state, zip)
    @street = street
    @city = city
    @state = state
    @zip = zip
  end
  
  def to_s
    "#{street}, #{city}, #{state} #{zip}"
  end
end

class PhoneNumber
  attr_reader :number, :type
  
  def initialize(number, type = :mobile)
    @number = number
    @type = type
  end
  
  def to_s
    "#{type.capitalize}: #{formatted}"
  end
  
  def formatted
    # Format as (XXX) XXX-XXXX
    digits = @number.gsub(/\D/, '')
    "(#{digits[0..2]}) #{digits[3..5]}-#{digits[6..9]}"
  end
end

class Contact
  attr_accessor :name, :email, :address
  attr_reader :phone_numbers
  
  def initialize(name, email)
    @name = name
    @email = email
    @phone_numbers = []
    @address = nil
  end
  
  def add_phone(number, type = :mobile)
    @phone_numbers << PhoneNumber.new(number, type)
  end
  
  def set_address(street, city, state, zip)
    @address = Address.new(street, city, state, zip)
  end
  
  def to_s
    output = ["Contact: #{@name}"]
    output << "Email: #{@email}"
    output << "Address: #{@address}" if @address
    @phone_numbers.each { |phone| output << phone.to_s }
    output.join("\n")
  end
end

contact = Contact.new("Alice Smith", "alice@example.com")
contact.add_phone("555-123-4567", :mobile)
contact.add_phone("555-987-6543", :work)
contact.set_address("123 Ruby Lane", "Codeville", "CA", "90210")
puts contact
```

## Practical Example: Building a Game Character System

Let's build something fun—a character system for an RPG:

```ruby
class Character
  attr_reader :name, :level, :health, :max_health, :experience
  attr_accessor :weapon, :armor
  
  BASE_HEALTH = 100
  LEVEL_HEALTH_BONUS = 20
  
  def initialize(name, character_class)
    @name = name
    @character_class = character_class
    @level = 1
    @experience = 0
    @max_health = calculate_max_health
    @health = @max_health
    @weapon = nil
    @armor = nil
    @inventory = []
    @skills = initialize_skills
  end
  
  def attack(target)
    damage = calculate_damage
    puts "#{@name} attacks #{target.name} for #{damage} damage!"
    target.take_damage(damage)
    
    # Chance for critical hit
    if rand(100) < critical_chance
      bonus = damage / 2
      puts "Critical hit! +#{bonus} damage!"
      target.take_damage(bonus)
    end
  end
  
  def take_damage(amount)
    reduced = amount - defense_value
    reduced = 1 if reduced < 1  # Minimum 1 damage
    
    @health -= reduced
    @health = 0 if @health < 0
    
    puts "#{@name} takes #{reduced} damage! (#{@health}/#{@max_health} HP)"
    check_death
  end
  
  def heal(amount)
    @health += amount
    @health = @max_health if @health > @max_health
    puts "#{@name} heals for #{amount}! (#{@health}/#{@max_health} HP)"
  end
  
  def gain_experience(amount)
    @experience += amount
    puts "#{@name} gains #{amount} XP!"
    
    while @experience >= experience_for_next_level
      level_up!
    end
  end
  
  def add_to_inventory(item)
    @inventory << item
    puts "#{@name} obtained #{item}!"
  end
  
  def use_skill(skill_name, target = nil)
    skill = @skills[skill_name]
    return puts "Unknown skill!" unless skill
    
    if @health < skill[:cost]
      return puts "Not enough HP to use #{skill_name}!"
    end
    
    @health -= skill[:cost]
    puts "#{@name} uses #{skill_name}!"
    skill[:effect].call(self, target)
  end
  
  def status
    puts "="*40
    puts "#{@name} (Level #{@level} #{@character_class})"
    puts "HP: #{@health}/#{@max_health}"
    puts "XP: #{@experience}/#{experience_for_next_level}"
    puts "Weapon: #{@weapon || 'None'}"
    puts "Armor: #{@armor || 'None'}"
    puts "Inventory: #{@inventory.empty? ? 'Empty' : @inventory.join(', ')}"
    puts "="*40
  end
  
  def alive?
    @health > 0
  end
  
  protected
  
  def check_death
    if @health <= 0
      puts "#{@name} has been defeated!"
    end
  end
  
  private
  
  def calculate_max_health
    BASE_HEALTH + ((@level - 1) * LEVEL_HEALTH_BONUS)
  end
  
  def calculate_damage
    base = 10 + (@level * 2)
    weapon_bonus = @weapon ? 10 : 0
    base + weapon_bonus + rand(1..6)
  end
  
  def defense_value
    base = @level
    armor_bonus = @armor ? 5 : 0
    base + armor_bonus
  end
  
  def critical_chance
    10 + (@level * 2)  # percentage
  end
  
  def experience_for_next_level
    @level * 100
  end
  
  def level_up!
    @level += 1
    old_max = @max_health
    @max_health = calculate_max_health
    health_gain = @max_health - old_max
    @health += health_gain
    
    puts "🎉 LEVEL UP! #{@name} is now level #{@level}!"
    puts "Max health increased by #{health_gain}!"
  end
  
  def initialize_skills
    case @character_class
    when "Warrior"
      {
        power_strike: {
          cost: 10,
          effect: ->(user, target) {
            damage = 30 + user.level * 3
            puts "Power Strike deals #{damage} damage!"
            target.take_damage(damage) if target
          }
        },
        battle_cry: {
          cost: 5,
          effect: ->(user, target) {
            heal_amount = 15
            puts "Battle Cry restores #{heal_amount} HP!"
            user.heal(heal_amount)
          }
        }
      }
    when "Mage"
      {
        fireball: {
          cost: 15,
          effect: ->(user, target) {
            damage = 25 + user.level * 4
            puts "Fireball deals #{damage} damage!"
            target.take_damage(damage) if target
          }
        },
        heal: {
          cost: 10,
          effect: ->(user, target) {
            heal_amount = 30
            target ||= user
            puts "Healing #{target.name} for #{heal_amount} HP!"
            target.heal(heal_amount)
          }
        }
      }
    else
      {}
    end
  end
end

# Let's play!
hero = Character.new("Aldric", "Warrior")
villain = Character.new("Morgoth", "Mage")

hero.status
hero.weapon = "Legendary Sword"
hero.armor = "Dragon Scale Mail"

# Battle!
while hero.alive? && villain.alive?
  hero.attack(villain)
  break unless villain.alive?
  
  villain.use_skill(:fireball, hero)
  break unless hero.alive?
  
  # Chance to use special skills
  if rand(3) == 0
    hero.use_skill(:power_strike, villain)
  end
  
  puts "-"*40
  sleep 1
end

# Victory rewards
if hero.alive?
  hero.gain_experience(150)
  hero.add_to_inventory("Magic Potion")
end

hero.status
```

## Class Design Best Practices

1. **Single Responsibility**: Each class should do one thing well
2. **Encapsulation**: Hide internal details, expose clean interfaces
3. **Composition over Inheritance**: Favor has-a over is-a relationships
4. **Tell, Don't Ask**: Objects should tell each other what to do, not ask for data
5. **Keep It Simple**: Start simple, refactor as needed

## Your Turn: Class Challenges

1. **Library System**: Create Book, Author, and Library classes
2. **Shopping Cart**: Build Product, Cart, and Order classes
3. **Social Network**: Design User, Post, and Comment classes
4. **Restaurant**: Create Menu, Dish, and Order classes
5. **Task Manager**: Build Task, Project, and Team classes

## What You've Learned

You can now:
- Define your own classes and create objects
- Use instance variables and methods
- Work with class variables and methods
- Control method visibility
- Overload operators
- Compose complex objects from simpler ones

## What's Next?

Next, we'll explore modules—Ruby's way of sharing functionality across classes. Think of them as mix-ins that add superpowers to your classes. It's like DLC for your objects!

Remember: Classes are how you model the world in your programs. Every class you create is a new type of thing your program understands. Start simple, and let your classes grow organically as your needs evolve. And always remember: in Ruby, you're not just writing code—you're designing objects that interact with each other to solve problems.