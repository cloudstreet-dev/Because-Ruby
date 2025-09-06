# Chapter 18: Gems - Standing on the Shoulders of Giants

Why write code that already exists? Ruby gems are pre-packaged libraries that solve common problems, so you can focus on building your unique application instead of reinventing the wheel. Think of RubyGems as a massive library where generous programmers have already solved your problems and wrapped them up with a bow.

## What Are Gems?

Gems are Ruby's way of packaging and distributing code. They're like apps for your code—installable packages that add functionality to your Ruby programs:

```ruby
# Without gems (reinventing the wheel)
def colorize_text(text, color)
  color_codes = {
    red: 31,
    green: 32,
    blue: 34
  }
  "\e[#{color_codes[color]}m#{text}\e[0m"
end

puts colorize_text("Hello", :red)

# With gems (using colorize gem)
require 'colorize'
puts "Hello".red
puts "World".blue.on_yellow.bold
```

## Installing and Using Gems

### Basic Gem Commands

```bash
# Install a gem
gem install colorize

# List installed gems
gem list

# Search for gems
gem search json

# Get info about a gem
gem info rails

# Uninstall a gem
gem uninstall colorize

# Update gems
gem update
```

### Using Gems in Your Code

```ruby
# Require the gem
require 'colorize'
require 'faker'
require 'httparty'

# Now use them!
puts "Error!".red.bold
puts Faker::Name.name
response = HTTParty.get('https://api.github.com')
```

## Bundler: The Gem Manager

Bundler manages gem dependencies for your project. It's like a shopping list for your application's gems:

### Setting Up Bundler

```ruby
# Gemfile - your project's gem shopping list
source 'https://rubygems.org'

gem 'sinatra'
gem 'puma'
gem 'json'

# Specify versions
gem 'rails', '~> 7.0.0'
gem 'pg', '>= 1.0', '< 2.0'

# Groups for different environments
group :development do
  gem 'pry'
  gem 'rubocop'
end

group :test do
  gem 'rspec'
  gem 'capybara'
end

# Git sources
gem 'my_gem', git: 'https://github.com/user/my_gem.git'

# Local gems
gem 'my_local_gem', path: '../my_local_gem'
```

### Bundler Commands

```bash
# Install gems from Gemfile
bundle install

# Update gems to latest allowed versions
bundle update

# Execute commands in bundle context
bundle exec ruby app.rb

# Create a new Gemfile
bundle init

# Show gem dependencies
bundle show

# Check for security vulnerabilities
bundle audit
```

## Essential Gems Every Rubyist Should Know

### Web Development

```ruby
# Gemfile
gem 'sinatra'      # Lightweight web framework
gem 'rails'        # Full-featured web framework
gem 'puma'         # Web server
gem 'rack'         # Web server interface

# Example: Quick API with Sinatra
require 'sinatra'
require 'json'

get '/api/greeting' do
  content_type :json
  { message: "Hello, World!", time: Time.now }.to_json
end
```

### Database

```ruby
gem 'pg'           # PostgreSQL
gem 'mysql2'       # MySQL
gem 'sqlite3'      # SQLite
gem 'redis'        # Redis client
gem 'sequel'       # Database toolkit

# Example with Sequel
require 'sequel'

DB = Sequel.sqlite('blog.db')

DB.create_table :posts do
  primary_key :id
  String :title
  Text :content
  DateTime :created_at
end

posts = DB[:posts]
posts.insert(title: "Hello", content: "World", created_at: Time.now)
```

### HTTP and APIs

```ruby
gem 'httparty'     # Simple HTTP client
gem 'faraday'      # Advanced HTTP client
gem 'rest-client'  # RESTful client

# HTTParty example
require 'httparty'

class GitHub
  include HTTParty
  base_uri 'https://api.github.com'
  
  def self.user(username)
    get("/users/#{username}")
  end
  
  def self.repos(username)
    get("/users/#{username}/repos")
  end
end

user = GitHub.user('matz')
puts "#{user['name']} has #{user['public_repos']} public repos"
```

### Testing

```ruby
gem 'rspec'        # BDD testing framework
gem 'minitest'     # Built-in testing framework
gem 'factory_bot'  # Test data generation
gem 'faker'        # Fake data generation

# RSpec example
# spec/calculator_spec.rb
require 'rspec'

class Calculator
  def add(a, b)
    a + b
  end
end

RSpec.describe Calculator do
  describe '#add' do
    it 'adds two numbers' do
      calc = Calculator.new
      expect(calc.add(2, 3)).to eq(5)
    end
  end
end
```

### Development Tools

```ruby
gem 'pry'          # Better REPL
gem 'rubocop'      # Code linter
gem 'byebug'       # Debugger
gem 'better_errors' # Better error pages

# Pry example
require 'pry'

def mysterious_method(data)
  result = data.map { |x| x * 2 }
  binding.pry  # Execution stops here, drops into REPL
  result.sum
end

mysterious_method([1, 2, 3])
```

### Data Processing

```ruby
gem 'nokogiri'     # HTML/XML parsing
gem 'csv'          # CSV processing (built-in)
gem 'json'         # JSON processing (built-in)

# Nokogiri web scraping example
require 'nokogiri'
require 'open-uri'

doc = Nokogiri::HTML(URI.open('https://news.ycombinator.com'))
doc.css('.titleline > a').each do |link|
  puts link.content
  puts link['href']
  puts "---"
end
```

## Creating Your Own Gem

Let's create a simple gem from scratch:

### 1. Generate the Gem Structure

```bash
bundle gem my_awesome_gem
cd my_awesome_gem
```

### 2. Edit the Gemspec

```ruby
# my_awesome_gem.gemspec
Gem::Specification.new do |spec|
  spec.name          = "my_awesome_gem"
  spec.version       = "0.1.0"
  spec.authors       = ["Your Name"]
  spec.email         = ["you@example.com"]

  spec.summary       = "A gem that does awesome things"
  spec.description   = "Longer description of awesome things"
  spec.homepage      = "https://github.com/you/my_awesome_gem"
  spec.license       = "MIT"

  spec.files         = Dir["lib/**/*", "README.md", "LICENSE.txt"]
  spec.require_paths = ["lib"]

  spec.add_dependency "colorize", "~> 0.8"
  spec.add_development_dependency "rspec", "~> 3.0"
end
```

### 3. Write Your Gem Code

```ruby
# lib/my_awesome_gem.rb
require "my_awesome_gem/version"
require "colorize"

module MyAwesomeGem
  class Error < StandardError; end
  
  class Greeter
    def self.greet(name, options = {})
      message = "Hello, #{name}!"
      
      if options[:shout]
        message = message.upcase
      end
      
      if options[:color]
        message = message.send(options[:color])
      end
      
      puts message
    end
  end
  
  class Calculator
    def self.fibonacci(n)
      return n if n <= 1
      fibonacci(n - 1) + fibonacci(n - 2)
    end
    
    def self.factorial(n)
      return 1 if n <= 1
      n * factorial(n - 1)
    end
  end
end
```

### 4. Test Your Gem

```ruby
# spec/my_awesome_gem_spec.rb
require 'my_awesome_gem'

RSpec.describe MyAwesomeGem::Calculator do
  describe '.fibonacci' do
    it 'calculates fibonacci numbers' do
      expect(described_class.fibonacci(0)).to eq(0)
      expect(described_class.fibonacci(1)).to eq(1)
      expect(described_class.fibonacci(5)).to eq(5)
      expect(described_class.fibonacci(10)).to eq(55)
    end
  end
end
```

### 5. Build and Install Locally

```bash
# Build the gem
gem build my_awesome_gem.gemspec

# Install locally
gem install ./my_awesome_gem-0.1.0.gem

# Or use it locally without installing
bundle exec irb
require './lib/my_awesome_gem'
```

### 6. Publish to RubyGems

```bash
# Create account at rubygems.org first
gem push my_awesome_gem-0.1.0.gem
```

## Practical Example: Building a Task Manager with Gems

Let's build a real application using popular gems:

```ruby
# Gemfile
source 'https://rubygems.org'

gem 'thor'         # CLI framework
gem 'colorize'     # Colored output
gem 'terminal-table' # Pretty tables
gem 'json'         # JSON storage
gem 'chronic'      # Natural language date parsing

# tasks.rb
require 'thor'
require 'colorize'
require 'terminal-table'
require 'json'
require 'chronic'
require 'fileutils'

class TaskManager < Thor
  DATA_FILE = File.expand_path('~/.tasks.json')
  
  def initialize(*args)
    super
    load_tasks
  end
  
  desc "add TASK", "Add a new task"
  option :due, aliases: '-d', desc: "Due date"
  option :priority, aliases: '-p', desc: "Priority (high, medium, low)"
  def add(task)
    new_task = {
      id: next_id,
      task: task,
      created_at: Time.now,
      completed: false,
      priority: options[:priority] || 'medium',
      due: options[:due] ? Chronic.parse(options[:due]) : nil
    }
    
    @tasks << new_task
    save_tasks
    
    puts "Task added successfully!".green
    puts "ID: #{new_task[:id]}, Task: #{new_task[:task]}"
  end
  
  desc "list", "List all tasks"
  option :all, type: :boolean, aliases: '-a', desc: "Show completed tasks"
  option :sort, aliases: '-s', desc: "Sort by: id, priority, due"
  def list
    tasks_to_show = options[:all] ? @tasks : @tasks.reject { |t| t[:completed] }
    
    if options[:sort]
      tasks_to_show = sort_tasks(tasks_to_show, options[:sort])
    end
    
    if tasks_to_show.empty?
      puts "No tasks found!".yellow
      return
    end
    
    table = Terminal::Table.new do |t|
      t.headings = ['ID', 'Task', 'Priority', 'Due', 'Status']
      t.rows = tasks_to_show.map do |task|
        [
          task[:id],
          task[:task][0..40],
          colorize_priority(task[:priority]),
          format_due_date(task[:due]),
          task[:completed] ? "✓".green : "○"
        ]
      end
    end
    
    puts table
  end
  
  desc "complete ID", "Mark a task as complete"
  def complete(id)
    task = find_task(id.to_i)
    
    if task
      task[:completed] = true
      task[:completed_at] = Time.now
      save_tasks
      puts "Task #{id} marked as complete!".green
    else
      puts "Task #{id} not found!".red
    end
  end
  
  desc "delete ID", "Delete a task"
  def delete(id)
    task = find_task(id.to_i)
    
    if task
      @tasks.delete(task)
      save_tasks
      puts "Task #{id} deleted!".green
    else
      puts "Task #{id} not found!".red
    end
  end
  
  desc "stats", "Show task statistics"
  def stats
    total = @tasks.count
    completed = @tasks.count { |t| t[:completed] }
    pending = total - completed
    
    by_priority = @tasks.group_by { |t| t[:priority] }
    
    puts "\n📊 Task Statistics".bold
    puts "="*30
    puts "Total tasks: #{total}"
    puts "Completed: #{completed} (#{percentage(completed, total)}%)".green
    puts "Pending: #{pending} (#{percentage(pending, total)}%)".yellow
    puts "\nBy Priority:"
    %w[high medium low].each do |priority|
      count = by_priority[priority]&.count || 0
      puts "  #{priority.capitalize}: #{count}"
    end
    
    # Overdue tasks
    overdue = @tasks.select do |t|
      t[:due] && !t[:completed] && t[:due] < Time.now
    end
    
    unless overdue.empty?
      puts "\n⚠️  Overdue tasks: #{overdue.count}".red
      overdue.each do |task|
        puts "  - #{task[:task][0..40]}".red
      end
    end
  end
  
  private
  
  def load_tasks
    if File.exist?(DATA_FILE)
      data = JSON.parse(File.read(DATA_FILE), symbolize_names: true)
      @tasks = data.map do |task|
        task[:created_at] = Time.parse(task[:created_at]) if task[:created_at]
        task[:due] = Time.parse(task[:due]) if task[:due]
        task[:completed_at] = Time.parse(task[:completed_at]) if task[:completed_at]
        task
      end
    else
      @tasks = []
    end
  end
  
  def save_tasks
    FileUtils.mkdir_p(File.dirname(DATA_FILE))
    File.write(DATA_FILE, JSON.pretty_generate(@tasks))
  end
  
  def find_task(id)
    @tasks.find { |t| t[:id] == id }
  end
  
  def next_id
    @tasks.empty? ? 1 : @tasks.map { |t| t[:id] }.max + 1
  end
  
  def colorize_priority(priority)
    case priority
    when 'high' then priority.red
    when 'medium' then priority.yellow
    when 'low' then priority.green
    else priority
    end
  end
  
  def format_due_date(due)
    return "-" unless due
    
    days_until = ((due - Time.now) / 86400).round
    
    if days_until < 0
      "Overdue by #{-days_until} days".red
    elsif days_until == 0
      "Today".yellow
    elsif days_until == 1
      "Tomorrow".yellow
    else
      due.strftime("%Y-%m-%d")
    end
  end
  
  def sort_tasks(tasks, sort_by)
    case sort_by
    when 'priority'
      priority_order = { 'high' => 0, 'medium' => 1, 'low' => 2 }
      tasks.sort_by { |t| priority_order[t[:priority]] }
    when 'due'
      tasks.sort_by { |t| t[:due] || Time.now + 1000000 }
    else
      tasks.sort_by { |t| t[:id] }
    end
  end
  
  def percentage(part, whole)
    return 0 if whole == 0
    ((part.to_f / whole) * 100).round
  end
end

TaskManager.start(ARGV)
```

## Gem Best Practices

### 1. Use Semantic Versioning
```ruby
# MAJOR.MINOR.PATCH
"1.0.0"  # First stable release
"1.0.1"  # Backward compatible bug fix
"1.1.0"  # Backward compatible new feature
"2.0.0"  # Breaking changes
```

### 2. Lock Your Dependencies
```ruby
# Good - specific versions in Gemfile.lock
gem 'rails', '~> 7.0.0'  # Accept 7.0.x but not 7.1

# Bad - too loose
gem 'rails'  # Could break with major updates
```

### 3. Use Groups Wisely
```ruby
# Don't install test gems in production
group :development, :test do
  gem 'rspec'
  gem 'pry'
end

group :production do
  gem 'pg'
  gem 'puma'
end
```

## Finding Gems

- **RubyGems.org**: The official gem repository
- **Ruby Toolbox**: Categories and popularity metrics
- **GitHub**: Search for Ruby repositories
- **Awesome Ruby**: Curated list of gems by category

## Your Turn: Gem Challenges

1. **Create a Weather CLI**: Build a gem that fetches weather data
2. **Text Formatter**: Create a gem for formatting text (ASCII art, tables)
3. **API Wrapper**: Write a gem that wraps your favorite API
4. **File Organizer**: Build a gem that organizes files by type/date
5. **Password Generator**: Create a secure password generator gem

## What You've Learned

You now know how to:
- Install and use gems
- Manage dependencies with Bundler
- Create your own gems
- Publish gems to RubyGems
- Use essential gems for common tasks
- Build real applications with gems

## What's Next?

Next, we'll dive into testing with RSpec, one of Ruby's most popular gems. You'll learn how to write tests that ensure your code works correctly and continues to work as you make changes.

Remember: Gems are the power-ups of Ruby programming. They let you build on the work of thousands of talented developers. Don't reinvent the wheel—check if there's a gem for that! But also don't be afraid to create your own gems when you solve a unique problem. Who knows? Your gem might be exactly what someone else needs.