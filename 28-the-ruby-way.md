# Chapter 28: The Ruby Way - Writing Code That Doesn't Suck

There's writing code that works, and then there's writing Ruby. The Ruby Way isn't just about syntax—it's a philosophy, a set of principles that make Ruby code not just functional, but beautiful, maintainable, and a joy to work with. This chapter is about writing code that would make Matz smile.

## The Ruby Philosophy

Ruby was designed with these principles:

1. **Optimize for programmer happiness**
2. **Principle of Least Surprise (POLS)**
3. **There's more than one way to do it (TMTOWTDI)**
4. **Convention over Configuration**
5. **DRY (Don't Repeat Yourself)**
6. **YAGNI (You Aren't Gonna Need It)**

## Writing Idiomatic Ruby

### Use the Right Iterator

```ruby
# Bad - C-style loop
for i in 0..array.length-1
  puts array[i]
end

# Bad - while loop
i = 0
while i < array.length
  puts array[i]
  i += 1
end

# Good - Ruby way
array.each { |element| puts element }

# Even better when appropriate
puts array  # puts calls to_s on each element
```

### Leverage Ruby's Expressiveness

```ruby
# Bad - verbose
if user.nil?
  name = "Guest"
else
  name = user.name
end

# Good - idiomatic
name = user&.name || "Guest"

# Bad - explicit return
def adult?(age)
  if age >= 18
    return true
  else
    return false
  end
end

# Good - implicit return and boolean expression
def adult?(age)
  age >= 18
end
```

### Use Ruby's Built-in Methods

```ruby
# Bad - reinventing the wheel
def first_n_elements(array, n)
  result = []
  n.times do |i|
    result << array[i] if i < array.length
  end
  result
end

# Good - use built-in
array.take(n)

# Bad - manual nil checking
values.each do |value|
  results << value unless value.nil?
end

# Good - use compact
results = values.compact

# Bad - manual transformation
numbers = []
strings.each do |str|
  numbers << str.to_i
end

# Good - use map
numbers = strings.map(&:to_i)
```

## Ruby Style Guide Essentials

### Naming Conventions

```ruby
# Classes and modules: CamelCase
class UserAccount
  # Constants: SCREAMING_SNAKE_CASE
  MAX_LOGIN_ATTEMPTS = 3
  
  # Methods and variables: snake_case
  def validate_password(input_password)
    @last_login_attempt = Time.now
    valid_password?(input_password)
  end
  
  # Predicate methods: end with ?
  def logged_in?
    !@session_token.nil?
  end
  
  # Dangerous methods: end with !
  def reset_password!
    @password = generate_random_password
    save!
  end
end
```

### Method Definitions

```ruby
# Good - parentheses for methods with arguments
def greet(name)
  "Hello, #{name}"
end

# Good - no parentheses for methods without arguments
def current_time
  Time.now
end

# Good - parentheses optional for DSL-style methods
class User < ActiveRecord::Base
  validates :name, presence: true
  has_many :posts
  belongs_to :organization
end

# Good - use keyword arguments for clarity
def create_user(name:, email:, role: "member")
  User.create(name: name, email: email, role: role)
end

# Bad - too many positional arguments
def create_user(name, email, age, city, country, role, active)
  # Hard to remember order
end
```

### Conditional Expressions

```ruby
# Good - simple if modifier
puts "Welcome!" if user.logged_in?

# Good - unless for negative conditions
launch_rockets unless system.safe_mode?

# Bad - unless with else (confusing)
unless user.admin?
  restricted_access
else
  full_access
end

# Good - use if for clarity
if user.admin?
  full_access
else
  restricted_access
end

# Good - ternary for simple conditionals
status = user.active? ? "Active" : "Inactive"

# Bad - nested ternary (unreadable)
status = user.active? ? (user.admin? ? "Active Admin" : "Active User") : "Inactive"

# Good - case statement for multiple conditions
status = case user.role
         when "admin" then "Administrator"
         when "moderator" then "Moderator"
         when "user" then "Regular User"
         else "Guest"
         end
```

## The Art of Ruby Objects

### Embrace Duck Typing

```ruby
# Bad - type checking
def process(obj)
  if obj.is_a?(String)
    obj.upcase
  elsif obj.is_a?(Array)
    obj.map(&:upcase)
  else
    raise "Unsupported type"
  end
end

# Good - duck typing
def process(obj)
  if obj.respond_to?(:upcase)
    obj.upcase
  elsif obj.respond_to?(:map)
    obj.map { |item| process(item) }
  else
    obj.to_s.upcase
  end
end
```

### Tell, Don't Ask

```ruby
# Bad - asking for data
if user.account.balance > 0
  user.account.balance -= amount
  user.account.save
end

# Good - telling object what to do
user.account.withdraw(amount)

# Inside Account class
def withdraw(amount)
  raise InsufficientFunds if balance < amount
  self.balance -= amount
  save
end
```

### Null Object Pattern

```ruby
# Bad - nil checks everywhere
if current_user
  puts "Welcome, #{current_user.name}"
  if current_user.preferences
    apply_theme(current_user.preferences.theme)
  end
end

# Good - Null Object
class NullUser
  def name
    "Guest"
  end
  
  def preferences
    NullPreferences.new
  end
  
  def logged_in?
    false
  end
end

class NullPreferences
  def theme
    "default"
  end
end

def current_user
  @current_user || NullUser.new
end

# Now no nil checks needed
puts "Welcome, #{current_user.name}"
apply_theme(current_user.preferences.theme)
```

## Functional Programming in Ruby

### Immutability and Pure Functions

```ruby
# Bad - mutating arguments
def process_data!(data)
  data.map! { |x| x * 2 }
  data.select! { |x| x > 10 }
  data
end

# Good - pure function
def process_data(data)
  data
    .map { |x| x * 2 }
    .select { |x| x > 10 }
end

# Bad - side effects
$total = 0
def add_to_total(amount)
  $total += amount
end

# Good - return new value
def add_to_total(total, amount)
  total + amount
end
```

### Function Composition

```ruby
# Composable functions
add_tax = ->(price) { price * 1.08 }
add_shipping = ->(price) { price + 10 }
apply_discount = ->(price) { price * 0.9 }

# Compose them
calculate_total = ->(price) {
  [add_tax, add_shipping, apply_discount]
    .reduce(price) { |result, fn| fn.call(result) }
}

# Or with a helper
def compose(*functions)
  ->(x) { functions.reduce(x) { |result, fn| fn.call(result) } }
end

calculate_total = compose(add_tax, add_shipping, apply_discount)
final_price = calculate_total.call(100)
```

## Metaprogramming: Use with Care

### Dynamic Method Definition

```ruby
# Good use case - reducing boilerplate
class Settings
  ATTRIBUTES = %i[theme language timezone]
  
  ATTRIBUTES.each do |attr|
    define_method attr do
      @settings[attr]
    end
    
    define_method "#{attr}=" do |value|
      @settings[attr] = value
    end
  end
end

# Bad - too clever
class Mystery
  def method_missing(method, *args)
    if method.to_s.start_with?("find_by_")
      attribute = method.to_s.sub("find_by_", "")
      find_where(attribute => args.first)
    else
      super
    end
  end
end
```

## Common Ruby Patterns

### Memoization

```ruby
class ExpensiveCalculation
  # Good - simple memoization
  def result
    @result ||= perform_calculation
  end
  
  # Good - memoization with nil handling
  def result
    return @result if defined?(@result)
    @result = perform_calculation
  end
  
  # Good - memoization with parameters
  def factorial(n)
    @factorial_cache ||= {}
    @factorial_cache[n] ||= calculate_factorial(n)
  end
end
```

### Builder Pattern

```ruby
class EmailBuilder
  def initialize
    @email = {}
  end
  
  def to(recipient)
    @email[:to] = recipient
    self
  end
  
  def subject(text)
    @email[:subject] = text
    self
  end
  
  def body(text)
    @email[:body] = text
    self
  end
  
  def send
    EmailService.deliver(@email)
  end
end

# Usage
EmailBuilder.new
  .to("user@example.com")
  .subject("Hello!")
  .body("How are you?")
  .send
```

### Service Objects

```ruby
class AuthenticateUser
  def initialize(email, password)
    @email = email
    @password = password
  end
  
  def call
    user = User.find_by(email: @email)
    return failure("User not found") unless user
    return failure("Invalid password") unless user.valid_password?(@password)
    
    success(user)
  end
  
  private
  
  def success(user)
    OpenStruct.new(success?: true, user: user)
  end
  
  def failure(message)
    OpenStruct.new(success?: false, error: message)
  end
end

# Usage
result = AuthenticateUser.new(email, password).call
if result.success?
  session[:user_id] = result.user.id
else
  flash[:error] = result.error
end
```

## Writing Maintainable Ruby

### Self-Documenting Code

```ruby
# Bad - needs comments to explain
def calc(p, r, t)
  # Calculate compound interest
  # p is principal, r is rate, t is time
  p * (1 + r) ** t
end

# Good - self-documenting
def calculate_compound_interest(principal:, annual_rate:, years:)
  principal * (1 + annual_rate) ** years
end
```

### Fail Fast

```ruby
# Good - validate early
def process_order(order)
  raise ArgumentError, "Order cannot be nil" if order.nil?
  raise InvalidOrder, "Order must have items" if order.items.empty?
  raise InsufficientInventory unless inventory.can_fulfill?(order)
  
  # Main logic here
  fulfill_order(order)
end
```

### Use Constants for Magic Values

```ruby
# Bad - magic numbers
if user.login_attempts > 3
  lock_account(user, 3600)
end

# Good - named constants
class SecurityPolicy
  MAX_LOGIN_ATTEMPTS = 3
  ACCOUNT_LOCK_DURATION = 1.hour
  
  def lock_account_if_needed(user)
    if user.login_attempts > MAX_LOGIN_ATTEMPTS
      user.lock_account(ACCOUNT_LOCK_DURATION)
    end
  end
end
```

## Ruby Code Smells and How to Fix Them

### Long Methods

```ruby
# Smell - method doing too much
def process_data(input)
  # 50 lines of code
end

# Fix - extract methods
def process_data(input)
  validated = validate(input)
  transformed = transform(validated)
  persist(transformed)
end
```

### Feature Envy

```ruby
# Smell - method uses another object's data extensively
def calculate_charge(order)
  if order.customer.membership.premium?
    order.items.sum(&:price) * 0.8
  else
    order.items.sum(&:price)
  end
end

# Fix - move logic to appropriate class
class Order
  def total_charge
    base_price * customer.discount_rate
  end
  
  def base_price
    items.sum(&:price)
  end
end
```

### Primitive Obsession

```ruby
# Smell - using primitives for domain concepts
def create_user(name, email, phone)
  # What format is phone? String? Number?
end

# Fix - use value objects
class PhoneNumber
  def initialize(number)
    @number = validate_and_format(number)
  end
  
  def to_s
    @number
  end
  
  def international
    "+1 #{@number}"
  end
end

def create_user(name, email, phone_number)
  # phone_number is now a PhoneNumber object
end
```

## Testing the Ruby Way

```ruby
# Good - descriptive test names
RSpec.describe ShoppingCart do
  describe "#add_item" do
    context "when item is in stock" do
      it "adds the item to the cart" do
        # test
      end
      
      it "updates the total price" do
        # test
      end
    end
    
    context "when item is out of stock" do
      it "raises an OutOfStockError" do
        # test
      end
    end
  end
end

# Good - test behavior, not implementation
# Bad
it "calls the calculate_tax method" do
  expect(order).to receive(:calculate_tax)
  order.total
end

# Good
it "includes tax in the total" do
  order = Order.new(subtotal: 100)
  expect(order.total).to eq(108)  # 8% tax
end
```

## Performance the Ruby Way

```ruby
# Good - use lazy evaluation for large datasets
def process_large_file(filename)
  File.foreach(filename).lazy
    .select { |line| line.include?("ERROR") }
    .map { |line| parse_error(line) }
    .take(100)
    .to_a
end

# Good - use symbols for hash keys
# Bad
{"name" => "Alice", "age" => 30}

# Good
{name: "Alice", age: 30}

# Good - freeze constants
COLORS = ["red", "green", "blue"].freeze
SETTINGS = {timeout: 30, retries: 3}.freeze
```

## The Zen of Ruby

1. **Beautiful is better than ugly**
2. **Explicit is sometimes better than implicit**
3. **Simple is better than complex**
4. **Complex is better than complicated**
5. **Readability counts**
6. **Special cases aren't special enough to break the rules**
7. **Although practicality beats purity**
8. **Errors should never pass silently**
9. **In the face of ambiguity, refuse the temptation to guess**
10. **There should be one-- and preferably many --obvious ways to do it**
11. **Now is better than never**
12. **If the implementation is hard to explain, it's a bad idea**
13. **If the implementation is easy to explain, it may be a good idea**

## Your Turn: Ruby Way Challenges

1. **Refactor Legacy Code**: Take ugly code and make it beautiful
2. **Build a DSL**: Create a domain-specific language
3. **Functional Pipeline**: Build a data processing pipeline
4. **Object Design**: Design a system using SOLID principles
5. **Performance Optimization**: Make slow code fast, idiomatically

## What You've Learned

You now understand:
- Ruby's core philosophy and principles
- How to write idiomatic Ruby code
- Common patterns and anti-patterns
- Object-oriented design in Ruby
- Functional programming techniques
- When and how to use metaprogramming
- Code smells and refactoring

## What's Next?

In the next chapter, we'll explore the Ruby community—the people, places, and resources that make Ruby special. You'll learn how to continue your journey and become part of this amazing community.

Remember: The Ruby Way isn't about following rules blindly. It's about understanding the spirit of Ruby—making programmers happy, writing expressive code, and building software that's a joy to work with. As you write more Ruby, you'll develop your own style within these principles. The goal isn't perfection; it's code that you and others will enjoy working with.