# Chapter 15: Error Handling - When Things Go Wrong (And They Will)

Errors are not your enemy. They're your teachers, your debuggers, your safety net. They're the difference between a program that crashes mysteriously and one that fails gracefully with helpful information. In Ruby, error handling isn't about preventing errors—it's about expecting them, learning from them, and recovering from them with style.

## Understanding Exceptions

In Ruby, errors are objects—instances of the Exception class or its descendants. When something goes wrong, Ruby "raises" an exception, which travels up through your program until it's either rescued or crashes the program.

```ruby
# This raises an exception
result = 10 / 0
# ZeroDivisionError: divided by 0

# This also raises an exception
[1, 2, 3].fetch(10)
# IndexError: index 10 outside of array bounds: -3...3

# And this
nil.upcase
# NoMethodError: undefined method `upcase' for nil:NilClass
```

## The Exception Hierarchy

Ruby's exceptions are organized in a hierarchy:

```
Exception
├── NoMemoryError
├── ScriptError
│   ├── LoadError
│   ├── NotImplementedError
│   └── SyntaxError
├── SecurityError
├── SignalException
│   └── Interrupt
├── StandardError (Most exceptions you'll work with)
│   ├── ArgumentError
│   ├── IOError
│   ├── IndexError
│   ├── LocalJumpError
│   ├── NameError
│   │   └── NoMethodError
│   ├── RangeError
│   ├── RegexpError
│   ├── RuntimeError (default for raise)
│   ├── SystemCallError
│   ├── ThreadError
│   ├── TypeError
│   └── ZeroDivisionError
├── SystemExit
└── SystemStackError
```

## Basic Exception Handling: Begin/Rescue/End

The fundamental way to handle exceptions is with `begin...rescue...end`:

```ruby
begin
  # Dangerous code goes here
  result = 10 / 0
rescue
  # This runs if an exception occurs
  puts "Oops! Something went wrong."
end

# With specific exception types
begin
  result = 10 / 0
rescue ZeroDivisionError
  puts "Can't divide by zero!"
end

# Capturing the exception object
begin
  result = 10 / 0
rescue ZeroDivisionError => e
  puts "Error: #{e.message}"
  puts "Backtrace: #{e.backtrace.first(3)}"
end

# Multiple rescue clauses
begin
  # Some operation
rescue ArgumentError => e
  puts "Bad argument: #{e.message}"
rescue TypeError => e
  puts "Type mismatch: #{e.message}"
rescue StandardError => e
  puts "Something else went wrong: #{e.message}"
end
```

## Ensure: Code That Always Runs

The `ensure` clause runs whether an exception occurs or not:

```ruby
def read_file(filename)
  file = File.open(filename)
  begin
    return file.read
  rescue IOError => e
    puts "Error reading file: #{e.message}"
    return nil
  ensure
    file.close if file  # Always close the file
    puts "File closed"
  end
end

# Database connection example
def query_database
  connection = Database.connect
  begin
    result = connection.execute("SELECT * FROM users")
    return result
  rescue DatabaseError => e
    log_error(e)
    return []
  ensure
    connection.close  # Always close the connection
  end
end
```

## Else: When No Exception Occurs

The `else` clause runs only if no exception was raised:

```ruby
begin
  puts "Trying something risky..."
  result = 10 / 2
rescue ZeroDivisionError
  puts "Division by zero!"
else
  puts "Success! Result is #{result}"
ensure
  puts "This always runs"
end

# Output:
# Trying something risky...
# Success! Result is 5
# This always runs
```

## Raising Exceptions

You can raise your own exceptions:

```ruby
# Raise with default message
raise "Something went wrong!"

# Raise specific exception type
raise ArgumentError, "Invalid argument provided"

# Raise with custom exception
class CustomError < StandardError; end
raise CustomError, "This is a custom error"

# Conditional raising
def withdraw(amount)
  raise ArgumentError, "Amount must be positive" if amount <= 0
  raise InsufficientFundsError, "Not enough money" if amount > balance
  
  @balance -= amount
end

# Re-raising exceptions
begin
  dangerous_operation
rescue => e
  log_error(e)
  raise  # Re-raises the same exception
end

# Raise with backtrace
raise ArgumentError, "Bad input", caller
```

## Custom Exception Classes

Create your own exception types for better error handling:

```ruby
# Simple custom exception
class ValidationError < StandardError; end

# Exception with additional data
class HttpError < StandardError
  attr_reader :status_code, :response_body
  
  def initialize(status_code, message = nil, response_body = nil)
    @status_code = status_code
    @response_body = response_body
    super(message || "HTTP Error: #{status_code}")
  end
end

# Usage
raise HttpError.new(404, "Page not found", "<h1>404 Not Found</h1>")

# Exception hierarchy for an application
module MyApp
  class Error < StandardError; end
  
  class ValidationError < Error
    attr_reader :errors
    
    def initialize(errors)
      @errors = errors
      super("Validation failed: #{errors.join(', ')}")
    end
  end
  
  class AuthenticationError < Error; end
  class AuthorizationError < Error; end
  class NotFoundError < Error; end
  
  class ConfigurationError < Error
    def initialize(key)
      super("Missing configuration: #{key}")
    end
  end
end

# Usage
begin
  raise MyApp::ValidationError.new(["Name is required", "Email is invalid"])
rescue MyApp::ValidationError => e
  puts "Validation errors: #{e.errors}"
rescue MyApp::Error => e
  puts "Application error: #{e.message}"
end
```

## Retry: Try, Try Again

The `retry` keyword restarts the begin block:

```ruby
retries = 0
begin
  puts "Attempting to connect... (attempt #{retries + 1})"
  connect_to_service  # This might fail
rescue ConnectionError => e
  retries += 1
  if retries < 3
    puts "Connection failed, retrying..."
    sleep(2 ** retries)  # Exponential backoff
    retry
  else
    puts "Failed after 3 attempts"
    raise
  end
end

# More sophisticated retry logic
class RetryableOperation
  def self.with_retry(max_attempts: 3, delay: 1, backoff: 2)
    attempts = 0
    begin
      attempts += 1
      yield
    rescue StandardError => e
      if attempts < max_attempts
        sleep(delay * (backoff ** (attempts - 1)))
        retry
      else
        raise e
      end
    end
  end
end

# Usage
RetryableOperation.with_retry(max_attempts: 5, delay: 1) do
  fetch_data_from_api
end
```

## Method-Level Rescue

You can use rescue in method definitions:

```ruby
def divide(a, b)
  a / b
rescue ZeroDivisionError
  Float::INFINITY
end

divide(10, 2)   # 5
divide(10, 0)   # Infinity

# Inline rescue (use sparingly)
result = 10 / 0 rescue "Can't divide by zero"

# Multiple rescues in method
def process_data(data)
  validate(data)
  transform(data)
  save(data)
rescue ValidationError => e
  log_validation_error(e)
  nil
rescue TransformError => e
  log_transform_error(e)
  nil
rescue SaveError => e
  log_save_error(e)
  nil
end
```

## Practical Error Handling Patterns

### The Circuit Breaker Pattern

```ruby
class CircuitBreaker
  attr_reader :failure_threshold, :timeout
  
  def initialize(failure_threshold: 5, timeout: 60)
    @failure_threshold = failure_threshold
    @timeout = timeout
    @failures = 0
    @last_failure_time = nil
    @state = :closed  # :closed, :open, :half_open
  end
  
  def call
    case @state
    when :open
      if Time.now - @last_failure_time > @timeout
        @state = :half_open
      else
        raise CircuitOpenError, "Circuit breaker is open"
      end
    end
    
    begin
      result = yield
      @failures = 0 if @state == :half_open
      @state = :closed
      result
    rescue => e
      record_failure
      raise e
    end
  end
  
  private
  
  def record_failure
    @failures += 1
    @last_failure_time = Time.now
    
    if @failures >= @failure_threshold
      @state = :open
    end
  end
end

# Usage
breaker = CircuitBreaker.new(failure_threshold: 3)

10.times do |i|
  begin
    breaker.call do
      unreliable_service_call
    end
  rescue CircuitOpenError
    puts "Service is down, using fallback"
    use_fallback_service
  end
end
```

### The Null Object Pattern for Error Prevention

```ruby
# Instead of raising exceptions for missing objects
class NullUser
  def name
    "Guest"
  end
  
  def email
    "no-email@example.com"
  end
  
  def premium?
    false
  end
  
  def method_missing(method, *args)
    nil
  end
end

def current_user
  @current_user || NullUser.new
end

# Now this never raises an exception
puts "Welcome, #{current_user.name}!"
if current_user.premium?
  show_premium_features
end
```

### Result Objects Instead of Exceptions

```ruby
class Result
  attr_reader :value, :error
  
  def self.success(value)
    new(value: value)
  end
  
  def self.failure(error)
    new(error: error)
  end
  
  def initialize(value: nil, error: nil)
    @value = value
    @error = error
  end
  
  def success?
    @error.nil?
  end
  
  def failure?
    !success?
  end
  
  def on_success
    yield(@value) if success? && block_given?
    self
  end
  
  def on_failure
    yield(@error) if failure? && block_given?
    self
  end
end

# Usage
def divide(a, b)
  return Result.failure("Cannot divide by zero") if b == 0
  Result.success(a.to_f / b)
end

divide(10, 2)
  .on_success { |value| puts "Result: #{value}" }
  .on_failure { |error| puts "Error: #{error}" }
```

## Error Handling in Real Applications

### Web Application Error Handler

```ruby
class ApplicationController < Sinatra::Base
  # Handle specific errors
  error MyApp::NotFoundError do
    status 404
    erb :not_found
  end
  
  error MyApp::AuthenticationError do
    status 401
    { error: "Authentication required" }.to_json
  end
  
  error MyApp::ValidationError do |e|
    status 422
    { errors: e.errors }.to_json
  end
  
  # Generic error handler
  error do
    error = env['sinatra.error']
    logger.error "#{error.class}: #{error.message}"
    logger.error error.backtrace.join("\n")
    
    if production?
      status 500
      "Something went wrong. We've been notified."
    else
      status 500
      erb :error, locals: { error: error }
    end
  end
end
```

### Background Job Error Handling

```ruby
class JobProcessor
  def self.perform(job)
    job.execute
  rescue StandardError => e
    handle_job_error(job, e)
  end
  
  def self.handle_job_error(job, error)
    job.retries += 1
    
    if job.retries < job.max_retries
      delay = calculate_retry_delay(job.retries)
      job.retry_at = Time.now + delay
      job.save
      
      logger.warn "Job #{job.id} failed, will retry in #{delay}s"
    else
      job.failed!
      notify_failure(job, error)
      
      logger.error "Job #{job.id} failed permanently: #{error.message}"
    end
  end
  
  def self.calculate_retry_delay(attempt)
    [attempt ** 2, 3600].min  # Exponential backoff, max 1 hour
  end
  
  def self.notify_failure(job, error)
    ErrorNotifier.notify(
      error: error,
      context: {
        job_id: job.id,
        job_class: job.class.name,
        attempts: job.retries
      }
    )
  end
end
```

## Logging and Monitoring Errors

```ruby
class ErrorLogger
  def self.log(error, context = {})
    logger.error build_error_message(error, context)
    
    if should_notify?(error)
      notify_team(error, context)
    end
    
    track_metrics(error)
  end
  
  private
  
  def self.build_error_message(error, context)
    message = ["Error: #{error.class} - #{error.message}"]
    message << "Context: #{context.inspect}" unless context.empty?
    message << "Backtrace:\n#{error.backtrace.first(10).join("\n")}"
    message.join("\n")
  end
  
  def self.should_notify?(error)
    case error
    when CriticalError, DataLossError
      true
    when ValidationError, UserError
      false
    else
      error.message !~ /temporary|timeout/i
    end
  end
  
  def self.notify_team(error, context)
    Slack.post(
      channel: "#errors",
      text: "Error in production: #{error.class}",
      attachments: [{
        color: "danger",
        fields: [
          { title: "Message", value: error.message },
          { title: "Context", value: context.to_json }
        ]
      }]
    )
  end
  
  def self.track_metrics(error)
    StatsD.increment("errors.#{error.class.name.underscore}")
  end
end
```

## Testing Error Handling

```ruby
require 'rspec'

RSpec.describe PaymentProcessor do
  describe '#process' do
    context 'when payment is successful' do
      it 'returns success result' do
        result = subject.process(valid_payment)
        expect(result).to be_success
      end
    end
    
    context 'when payment fails' do
      it 'raises PaymentError' do
        expect {
          subject.process(invalid_payment)
        }.to raise_error(PaymentError)
      end
      
      it 'includes error details' do
        expect {
          subject.process(invalid_payment)
        }.to raise_error(PaymentError) do |error|
          expect(error.message).to include("Invalid card")
          expect(error.code).to eq("INVALID_CARD")
        end
      end
    end
    
    context 'when network error occurs' do
      before do
        allow(PaymentGateway).to receive(:charge)
          .and_raise(Net::ReadTimeout)
      end
      
      it 'retries the payment' do
        expect(PaymentGateway).to receive(:charge).exactly(3).times
        
        expect {
          subject.process(valid_payment)
        }.to raise_error(Net::ReadTimeout)
      end
    end
  end
end
```

## Best Practices

### 1. Be Specific with Rescue
```ruby
# Bad - catches everything
begin
  dangerous_operation
rescue
  # This catches all StandardError subclasses
end

# Good - catch specific exceptions
begin
  dangerous_operation
rescue NetworkError, TimeoutError => e
  handle_network_error(e)
end
```

### 2. Don't Swallow Exceptions
```ruby
# Bad - silently ignores errors
def process
  do_something
rescue
  # Silent failure
end

# Good - at least log the error
def process
  do_something
rescue => e
  logger.error "Process failed: #{e.message}"
  raise  # Or return a meaningful value
end
```

### 3. Use Custom Exceptions
```ruby
# Bad - generic error
raise "User not found"

# Good - specific exception
class UserNotFoundError < StandardError; end
raise UserNotFoundError, "User with ID #{id} not found"
```

### 4. Fail Fast in Development
```ruby
# Different behavior for different environments
if Rails.env.development?
  # Fail immediately in development
  raise ConfigurationError, "API_KEY not set"
else
  # Use fallback in production
  logger.warn "API_KEY not set, using default"
  use_default_configuration
end
```

## Your Turn: Error Handling Challenges

1. **Robust File Processor**: Build a file processor that handles various I/O errors
2. **API Client**: Create an API client with retry logic and circuit breaker
3. **Validation Framework**: Design a validation system with detailed error reporting
4. **Error Tracking System**: Build a system that logs and categorizes errors
5. **Graceful Degradation**: Implement fallback mechanisms for service failures

## What You've Learned

You now understand:
- Ruby's exception hierarchy
- Begin/rescue/ensure/else blocks
- Raising and re-raising exceptions
- Custom exception classes
- Retry mechanisms
- Error handling patterns
- Logging and monitoring strategies

## What's Next?

In the next chapter, we'll explore advanced Ruby topics—metaprogramming, performance optimization, and other techniques that separate good Ruby code from great Ruby code.

Remember: Good error handling isn't about preventing all errors—it's about failing gracefully, providing useful information, and recovering when possible. Errors are not failures; they're feedback. Embrace them, learn from them, and use them to make your code more robust and user-friendly.