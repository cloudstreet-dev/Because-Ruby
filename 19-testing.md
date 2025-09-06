# Chapter 19: Testing - Making Sure Your Code Actually Works

Testing is like flossing—everyone knows they should do it, but most people don't until something goes wrong. The difference is that with testing, you can automate the process and never have to feel guilty again. Ruby's testing culture is strong, and RSpec makes testing so pleasant you might actually enjoy it. (I said might.)

## Why Test?

Before we dive into the how, let's talk about the why:

1. **Confidence**: Tests prove your code works
2. **Documentation**: Tests show how to use your code
3. **Refactoring Safety**: Change code without fear
4. **Design Tool**: Writing tests first improves your design
5. **Regression Prevention**: Old bugs stay fixed
6. **Collaboration**: Others can change your code safely

## Getting Started with RSpec

RSpec is Ruby's most popular testing framework. It reads like English and makes testing feel natural:

```bash
# Install RSpec
gem install rspec

# Initialize RSpec in your project
rspec --init

# This creates:
# .rspec (configuration file)
# spec/spec_helper.rb (test configuration)
```

## Your First Test

```ruby
# calculator.rb
class Calculator
  def add(a, b)
    a + b
  end
  
  def subtract(a, b)
    a - b
  end
  
  def multiply(a, b)
    a * b
  end
  
  def divide(a, b)
    raise ArgumentError, "Cannot divide by zero" if b == 0
    a.to_f / b
  end
end

# spec/calculator_spec.rb
require_relative '../calculator'

RSpec.describe Calculator do
  # Create a new calculator for each test
  let(:calculator) { Calculator.new }
  
  describe '#add' do
    it 'adds two positive numbers' do
      result = calculator.add(2, 3)
      expect(result).to eq(5)
    end
    
    it 'adds negative numbers' do
      result = calculator.add(-2, -3)
      expect(result).to eq(-5)
    end
    
    it 'adds zero' do
      result = calculator.add(5, 0)
      expect(result).to eq(5)
    end
  end
  
  describe '#divide' do
    it 'divides two numbers' do
      result = calculator.divide(10, 2)
      expect(result).to eq(5.0)
    end
    
    it 'raises an error when dividing by zero' do
      expect { calculator.divide(10, 0) }.to raise_error(ArgumentError)
    end
    
    it 'returns a float' do
      result = calculator.divide(5, 2)
      expect(result).to be_a(Float)
      expect(result).to eq(2.5)
    end
  end
end

# Run tests with: rspec spec/calculator_spec.rb
```

## RSpec Matchers

RSpec provides many matchers for different types of assertions:

```ruby
RSpec.describe "RSpec Matchers" do
  # Equality
  it "checks equality" do
    expect(2 + 2).to eq(4)         # ==
    expect(2 + 2).to eql(4)        # eql?
    expect("hello").to equal("hello".freeze)  # same object_id
  end
  
  # Comparison
  it "compares values" do
    expect(5).to be > 3
    expect(2).to be < 10
    expect(5).to be >= 5
    expect(3).to be <= 3
    expect(5).to be_between(1, 10).inclusive
  end
  
  # Truthiness
  it "checks truthiness" do
    expect(true).to be true
    expect(false).to be false
    expect(nil).to be nil
    expect(5).to be_truthy
    expect(nil).to be_falsy
  end
  
  # Types
  it "checks types" do
    expect("hello").to be_a(String)
    expect([1, 2]).to be_an(Array)
    expect(5).to be_a_kind_of(Numeric)
    expect("hello").to be_instance_of(String)
  end
  
  # Collections
  it "checks collections" do
    expect([1, 2, 3]).to include(2)
    expect([1, 2, 3]).to contain_exactly(3, 1, 2)
    expect([1, 2, 3]).to match_array([3, 2, 1])
    expect([1, 2, 3]).to start_with(1)
    expect([1, 2, 3]).to end_with(3)
    expect([]).to be_empty
    expect([1, 2, 3]).to all(be > 0)
  end
  
  # Strings and Regex
  it "matches strings" do
    expect("hello world").to include("world")
    expect("hello world").to start_with("hello")
    expect("hello world").to end_with("world")
    expect("hello world").to match(/w.rld/)
  end
  
  # Hashes
  it "checks hashes" do
    hash = {a: 1, b: 2}
    expect(hash).to include(a: 1)
    expect(hash).to have_key(:a)
    expect(hash).to have_value(2)
  end
  
  # Predicates
  it "uses predicate matchers" do
    expect([]).to be_empty     # calls empty?
    expect(5).to be_odd         # calls odd?
    expect(4).to be_even        # calls even?
    expect(nil).to be_nil       # calls nil?
  end
  
  # Change
  it "checks for changes" do
    array = []
    expect { array << 1 }.to change { array.size }.from(0).to(1)
    expect { array << 2 }.to change { array.size }.by(1)
  end
  
  # Errors
  it "checks for errors" do
    expect { 1 / 0 }.to raise_error(ZeroDivisionError)
    expect { raise "oops" }.to raise_error("oops")
    expect { raise ArgumentError, "bad arg" }.to raise_error(ArgumentError, /bad/)
  end
  
  # Output
  it "checks output" do
    expect { puts "hello" }.to output("hello\n").to_stdout
    expect { warn "danger" }.to output(/danger/).to_stderr
  end
end
```

## Testing Classes

```ruby
# user.rb
class User
  attr_reader :name, :email, :age
  
  def initialize(name, email, age)
    @name = name
    @email = email
    @age = age
    @created_at = Time.now
  end
  
  def adult?
    age >= 18
  end
  
  def display_name
    name.split.map(&:capitalize).join(' ')
  end
  
  def valid_email?
    email.match?(/\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i)
  end
  
  def account_age_in_days
    ((Time.now - @created_at) / 86400).to_i
  end
end

# spec/user_spec.rb
RSpec.describe User do
  # Setup
  let(:user) { User.new("john doe", "john@example.com", 25) }
  let(:teenager) { User.new("jane smith", "jane@example.com", 16) }
  
  describe '#initialize' do
    it 'sets the user attributes' do
      expect(user.name).to eq("john doe")
      expect(user.email).to eq("john@example.com")
      expect(user.age).to eq(25)
    end
  end
  
  describe '#adult?' do
    context 'when user is 18 or older' do
      it 'returns true' do
        expect(user).to be_adult
      end
    end
    
    context 'when user is under 18' do
      it 'returns false' do
        expect(teenager).not_to be_adult
      end
    end
  end
  
  describe '#display_name' do
    it 'capitalizes each word' do
      expect(user.display_name).to eq("John Doe")
    end
    
    it 'handles single names' do
      single_name_user = User.new("madonna", "madonna@example.com", 30)
      expect(single_name_user.display_name).to eq("Madonna")
    end
  end
  
  describe '#valid_email?' do
    it 'returns true for valid emails' do
      valid_emails = [
        "user@example.com",
        "user.name@example.co.uk",
        "user+tag@example.org"
      ]
      
      valid_emails.each do |email|
        user = User.new("Test", email, 25)
        expect(user.valid_email?).to be true
      end
    end
    
    it 'returns false for invalid emails' do
      invalid_emails = [
        "not_an_email",
        "@example.com",
        "user@",
        "user @example.com"
      ]
      
      invalid_emails.each do |email|
        user = User.new("Test", email, 25)
        expect(user.valid_email?).to be false
      end
    end
  end
  
  describe '#account_age_in_days' do
    it 'returns 0 for new accounts' do
      expect(user.account_age_in_days).to eq(0)
    end
    
    it 'calculates days correctly' do
      # Time travel!
      allow(Time).to receive(:now).and_return(Time.now + 86400 * 7)
      expect(user.account_age_in_days).to eq(7)
    end
  end
end
```

## Mocks and Stubs

Mocks and stubs let you isolate the code you're testing:

```ruby
# email_service.rb
class EmailService
  def self.send_email(to, subject, body)
    # Actually sends email
  end
end

# user_notifier.rb
class UserNotifier
  def initialize(user)
    @user = user
  end
  
  def send_welcome_email
    EmailService.send_email(
      @user.email,
      "Welcome!",
      "Welcome to our service, #{@user.name}!"
    )
  end
  
  def send_birthday_email
    return unless Date.today == @user.birthday
    
    EmailService.send_email(
      @user.email,
      "Happy Birthday!",
      "Happy birthday, #{@user.name}!"
    )
  end
end

# spec/user_notifier_spec.rb
RSpec.describe UserNotifier do
  let(:user) { double("User", name: "Alice", email: "alice@example.com") }
  let(:notifier) { UserNotifier.new(user) }
  
  describe '#send_welcome_email' do
    it 'sends an email with the correct parameters' do
      expect(EmailService).to receive(:send_email).with(
        "alice@example.com",
        "Welcome!",
        "Welcome to our service, Alice!"
      )
      
      notifier.send_welcome_email
    end
  end
  
  describe '#send_birthday_email' do
    context 'on user birthday' do
      before do
        allow(user).to receive(:birthday).and_return(Date.today)
      end
      
      it 'sends a birthday email' do
        expect(EmailService).to receive(:send_email).with(
          "alice@example.com",
          "Happy Birthday!",
          "Happy birthday, Alice!"
        )
        
        notifier.send_birthday_email
      end
    end
    
    context 'not on user birthday' do
      before do
        allow(user).to receive(:birthday).and_return(Date.today + 1)
      end
      
      it 'does not send an email' do
        expect(EmailService).not_to receive(:send_email)
        notifier.send_birthday_email
      end
    end
  end
end
```

## Testing Web Applications

```ruby
# app.rb (Sinatra app)
require 'sinatra'

get '/' do
  'Welcome!'
end

get '/hello/:name' do
  "Hello, #{params[:name]}!"
end

post '/users' do
  user = User.create(params)
  if user.valid?
    status 201
    user.to_json
  else
    status 422
    {errors: user.errors}.to_json
  end
end

# spec/app_spec.rb
require 'rack/test'
require_relative '../app'

RSpec.describe 'Sinatra App' do
  include Rack::Test::Methods
  
  def app
    Sinatra::Application
  end
  
  describe 'GET /' do
    it 'returns welcome message' do
      get '/'
      expect(last_response).to be_ok
      expect(last_response.body).to eq('Welcome!')
    end
  end
  
  describe 'GET /hello/:name' do
    it 'greets the user' do
      get '/hello/Alice'
      expect(last_response).to be_ok
      expect(last_response.body).to eq('Hello, Alice!')
    end
  end
  
  describe 'POST /users' do
    context 'with valid params' do
      it 'creates a user' do
        post '/users', name: 'Alice', email: 'alice@example.com'
        
        expect(last_response.status).to eq(201)
        json = JSON.parse(last_response.body)
        expect(json['name']).to eq('Alice')
      end
    end
    
    context 'with invalid params' do
      it 'returns errors' do
        post '/users', name: ''
        
        expect(last_response.status).to eq(422)
        json = JSON.parse(last_response.body)
        expect(json).to have_key('errors')
      end
    end
  end
end
```

## Test-Driven Development (TDD)

TDD means writing tests first, then code:

```ruby
# Step 1: Write a failing test
RSpec.describe StringCalculator do
  describe '#add' do
    it 'returns 0 for empty string' do
      calc = StringCalculator.new
      expect(calc.add("")).to eq(0)
    end
  end
end
# Test fails: uninitialized constant StringCalculator

# Step 2: Write minimal code to pass
class StringCalculator
  def add(numbers)
    0
  end
end
# Test passes!

# Step 3: Write another test
it 'returns the number for single number' do
  calc = StringCalculator.new
  expect(calc.add("5")).to eq(5)
end
# Test fails

# Step 4: Update code
class StringCalculator
  def add(numbers)
    return 0 if numbers.empty?
    numbers.to_i
  end
end
# Tests pass!

# Step 5: Continue the cycle
it 'adds two numbers separated by comma' do
  calc = StringCalculator.new
  expect(calc.add("1,2")).to eq(3)
end

# Step 6: Update code again
class StringCalculator
  def add(numbers)
    return 0 if numbers.empty?
    numbers.split(',').map(&:to_i).sum
  end
end
# All tests pass!
```

## Shared Examples

DRY up your tests with shared examples:

```ruby
# spec/support/shared_examples.rb
RSpec.shared_examples "a collection" do
  it { is_expected.to respond_to(:size) }
  it { is_expected.to respond_to(:each) }
  it { is_expected.to respond_to(:empty?) }
end

RSpec.shared_examples "cacheable" do
  it "caches the result" do
    expect(subject).to receive(:expensive_operation).once
    2.times { subject.cached_result }
  end
  
  it "expires the cache" do
    subject.cached_result
    subject.expire_cache
    expect(subject).to receive(:expensive_operation)
    subject.cached_result
  end
end

# Using shared examples
RSpec.describe Array do
  it_behaves_like "a collection"
end

RSpec.describe TodoList do
  it_behaves_like "a collection"
  it_behaves_like "cacheable"
end
```

## Custom Matchers

Create your own matchers for cleaner tests:

```ruby
# spec/support/matchers.rb
RSpec::Matchers.define :be_a_valid_email do
  match do |actual|
    actual.match?(/\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i)
  end
  
  failure_message do |actual|
    "expected '#{actual}' to be a valid email address"
  end
end

RSpec::Matchers.define :have_error_on do |attribute|
  match do |model|
    model.valid?
    model.errors[attribute].any?
  end
  
  failure_message do |model|
    "expected #{model} to have error on #{attribute}"
  end
end

# Using custom matchers
RSpec.describe User do
  it 'validates email format' do
    expect("user@example.com").to be_a_valid_email
    expect("invalid").not_to be_a_valid_email
  end
  
  it 'validates presence of name' do
    user = User.new(email: "test@example.com")
    expect(user).to have_error_on(:name)
  end
end
```

## Testing Best Practices

### 1. Arrange, Act, Assert
```ruby
it 'calculates the total' do
  # Arrange
  cart = ShoppingCart.new
  cart.add_item(Product.new("Book", 10))
  cart.add_item(Product.new("DVD", 15))
  
  # Act
  total = cart.total
  
  # Assert
  expect(total).to eq(25)
end
```

### 2. One Assertion Per Test
```ruby
# Bad
it 'creates a user' do
  user = User.create(name: "Alice", email: "alice@example.com")
  expect(user.name).to eq("Alice")
  expect(user.email).to eq("alice@example.com")
  expect(user).to be_valid
end

# Good
describe 'user creation' do
  let(:user) { User.create(name: "Alice", email: "alice@example.com") }
  
  it 'sets the name' do
    expect(user.name).to eq("Alice")
  end
  
  it 'sets the email' do
    expect(user.email).to eq("alice@example.com")
  end
  
  it 'is valid' do
    expect(user).to be_valid
  end
end
```

### 3. Use Descriptive Test Names
```ruby
# Bad
it 'works' do
  # ...
end

# Good
it 'returns the sum of positive integers' do
  # ...
end
```

### 4. Test Behavior, Not Implementation
```ruby
# Bad - testing implementation
it 'calls the calculate_tax private method' do
  expect(order).to receive(:calculate_tax)
  order.total
end

# Good - testing behavior
it 'includes tax in the total' do
  order = Order.new(subtotal: 100)
  expect(order.total).to eq(110)  # Assuming 10% tax
end
```

## Running Tests

```bash
# Run all tests
rspec

# Run specific file
rspec spec/user_spec.rb

# Run specific line
rspec spec/user_spec.rb:42

# Run with documentation format
rspec --format documentation

# Run only tagged tests
rspec --tag focus

# Run except tagged tests
rspec --tag ~slow

# Generate coverage report
rspec --require spec_helper --format html --out rspec_results.html
```

## Test Coverage

```ruby
# spec/spec_helper.rb
require 'simplecov'
SimpleCov.start do
  add_filter '/spec/'
  add_filter '/config/'
end

# Now run: rspec
# Open coverage/index.html to see report
```

## Your Turn: Testing Challenges

1. **Calculator TDD**: Build a calculator using TDD
2. **API Client Tests**: Test an HTTP API client with mocks
3. **Game Logic**: Test a card game with complex rules
4. **Data Validator**: Build and test a validation framework
5. **Integration Tests**: Test a full web application flow

## What You've Learned

You now know how to:
- Write tests with RSpec
- Use various matchers and assertions
- Mock and stub dependencies
- Practice Test-Driven Development
- Test web applications
- Create custom matchers
- Follow testing best practices

## What's Next?

In the next chapter, we'll explore debugging—how to find and fix problems when your tests are failing (or worse, passing when they shouldn't). You'll learn tools and techniques to track down bugs like a detective.

Remember: Tests are not about achieving 100% coverage or following rules blindly. They're about confidence—confidence that your code works, confidence to refactor, confidence to deploy on Friday afternoon (okay, maybe not that). Write tests that give you confidence, and skip the ones that don't. Quality over quantity, always!