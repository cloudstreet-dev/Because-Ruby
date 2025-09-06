# Chapter 17: Working with Files - Reading, Writing, and Not Losing Data

Files are where data goes to live permanently. Unlike variables that disappear when your program ends, files stick around. They're how programs remember things, share information, and generally act like they have a memory longer than a goldfish. Ruby makes working with files surprisingly pleasant—no arcane incantations required.

## Opening and Closing Files

The most basic file operation is opening a file, doing something with it, and closing it:

```ruby
# The manual way (don't do this)
file = File.open("example.txt", "r")
content = file.read
file.close  # Don't forget this!

# The better way (with block - auto-closes)
File.open("example.txt", "r") do |file|
  content = file.read
  # File automatically closes when block ends
end

# File modes
"r"   # Read only (default)
"w"   # Write only (creates new or overwrites)
"a"   # Append (adds to end)
"r+"  # Read and write
"w+"  # Read and write (overwrites existing)
"a+"  # Read and append

# Binary mode (for non-text files)
"rb"  # Read binary
"wb"  # Write binary

# Examples
File.open("data.txt", "w") do |file|
  file.puts "Hello, World!"
end

File.open("data.txt", "a") do |file|
  file.puts "Adding a new line"
end
```

## Reading Files

Ruby provides many ways to read files:

```ruby
# Read entire file at once
content = File.read("data.txt")

# Read into an array of lines
lines = File.readlines("data.txt")

# Read line by line (memory efficient for large files)
File.foreach("data.txt") do |line|
  puts line
end

# Read with encoding
content = File.read("data.txt", encoding: "UTF-8")

# Read limited bytes
first_100_bytes = File.read("data.txt", 100)

# Read from offset
File.read("data.txt", 100, 50)  # Read 100 bytes starting at byte 50

# Using IO methods
File.open("data.txt") do |file|
  # Read one line
  first_line = file.gets
  
  # Read specific number of characters
  chars = file.read(10)
  
  # Read rest of file
  rest = file.read
  
  # Read byte by byte
  file.each_byte { |byte| print byte, " " }
  
  # Read character by character
  file.each_char { |char| print char }
end
```

## Writing Files

Multiple ways to write to files:

```ruby
# Simple write (overwrites)
File.write("output.txt", "Hello, World!")

# Append to file
File.write("output.txt", "More content", mode: "a")

# Write with block
File.open("output.txt", "w") do |file|
  file.puts "Line 1"  # Adds newline
  file.print "Line 2" # No newline
  file.write " continued\n"  # Raw write
  file << "Line 3\n"  # Append operator
end

# Write an array of lines
lines = ["First line", "Second line", "Third line"]
File.open("output.txt", "w") do |file|
  lines.each { |line| file.puts line }
end

# Or more concisely
File.write("output.txt", lines.join("\n"))

# Writing binary data
File.open("data.bin", "wb") do |file|
  file.write([1, 2, 3, 4].pack("C*"))  # Write bytes
end

# Printf-style formatting
File.open("report.txt", "w") do |file|
  file.printf("Name: %-20s Age: %3d\n", "Alice", 30)
  file.printf("Name: %-20s Age: %3d\n", "Bob", 25)
end
```

## File Information and Metadata

```ruby
# Check if file exists
File.exist?("data.txt")    # true/false
File.file?("data.txt")     # true if regular file
File.directory?("data")    # true if directory

# File size
File.size("data.txt")      # Size in bytes
File.size?("data.txt")     # nil if empty, size otherwise
File.zero?("data.txt")     # true if empty

# File times
File.mtime("data.txt")     # Last modified time
File.atime("data.txt")     # Last accessed time
File.ctime("data.txt")     # Last status change time

# File permissions
File.readable?("data.txt")
File.writable?("data.txt")
File.executable?("script.sh")

# File type
File.symlink?("link")      # Is it a symbolic link?
File.pipe?("mypipe")       # Is it a pipe?
File.socket?("mysocket")   # Is it a socket?

# File stats
stats = File.stat("data.txt")
puts "Size: #{stats.size}"
puts "Modified: #{stats.mtime}"
puts "Permissions: #{stats.mode.to_s(8)}"
puts "Owner UID: #{stats.uid}"
puts "Blocks: #{stats.blocks}"

# Comparing files
File.identical?("file1.txt", "file2.txt")  # Same file?
```

## Working with Directories

```ruby
# Current directory
puts Dir.pwd  # Print working directory
Dir.chdir("/tmp")  # Change directory

# List directory contents
Dir.entries(".")  # Returns array including . and ..
Dir.children(".")  # Without . and ..
Dir.glob("*.rb")  # Files matching pattern

# Iterate through directory
Dir.foreach(".") do |filename|
  next if filename.start_with?(".")
  puts filename
end

# Create and remove directories
Dir.mkdir("new_folder")
Dir.mkdir("path/to/folder")  # Error if parent doesn't exist
FileUtils.mkdir_p("path/to/folder")  # Creates parent dirs

Dir.rmdir("empty_folder")  # Only if empty
FileUtils.rm_rf("folder")  # Remove recursively (dangerous!)

# Get specific directories
Dir.home  # User's home directory
Dir.tmpdir  # System temp directory

# Pattern matching with glob
Dir.glob("**/*.rb")  # All .rb files recursively
Dir.glob("*.{rb,txt}")  # .rb or .txt files
Dir.glob("[a-z]*")  # Files starting with lowercase
Dir.glob("*", File::FNM_DOTMATCH)  # Include hidden files

# Using blocks with glob
Dir.glob("*.txt") do |filename|
  puts "Processing #{filename}"
  content = File.read(filename)
  # Process content
end
```

## File Paths

```ruby
require 'pathname'

# Join paths (platform independent)
File.join("path", "to", "file.txt")  # "path/to/file.txt"

# Expand paths
File.expand_path("~/Documents")  # "/Users/username/Documents"
File.expand_path("../file.txt", "/current/dir")  # "/current/file.txt"

# Path manipulation
path = "/path/to/file.txt"
File.dirname(path)   # "/path/to"
File.basename(path)  # "file.txt"
File.basename(path, ".txt")  # "file"
File.extname(path)   # ".txt"

# Absolute vs relative paths
File.absolute_path("file.txt")  # Full path
File.realdirpath("../file.txt")  # Resolved path

# Using Pathname (more OO approach)
path = Pathname.new("/path/to/file.txt")
path.dirname    # Pathname:/path/to
path.basename   # Pathname:file.txt
path.extname    # ".txt"
path.exist?     # true/false
path.directory? # false
path.file?      # true
path.parent     # Pathname:/path/to
path.children   # If directory, returns children

# Path arithmetic
base = Pathname.new("/usr/local")
full = base + "bin/ruby"  # /usr/local/bin/ruby
relative = full.relative_path_from(base)  # bin/ruby
```

## CSV Files

```ruby
require 'csv'

# Reading CSV
CSV.foreach("data.csv", headers: true) do |row|
  puts "Name: #{row['name']}, Age: #{row['age']}"
end

# Parse CSV from string
csv_text = "name,age\nAlice,30\nBob,25"
CSV.parse(csv_text, headers: true) do |row|
  puts row.to_h  # Convert to hash
end

# Writing CSV
CSV.open("output.csv", "w") do |csv|
  csv << ["Name", "Age", "City"]  # Header
  csv << ["Alice", 30, "NYC"]
  csv << ["Bob", 25, "LA"]
end

# Generate CSV string
data = [
  ["Name", "Score"],
  ["Alice", 95],
  ["Bob", 87]
]
csv_string = CSV.generate do |csv|
  data.each { |row| csv << row }
end

# Advanced CSV options
CSV.foreach("data.csv",
  headers: true,
  col_sep: ";",           # Semicolon separator
  quote_char: "'",        # Single quote for quoting
  skip_blanks: true,      # Skip empty lines
  converters: :numeric    # Auto-convert numbers
) do |row|
  # Process row
end

# Custom converters
CSV.foreach("data.csv",
  headers: true,
  converters: lambda { |field| field.strip }
) do |row|
  # Fields are stripped of whitespace
end
```

## JSON Files

```ruby
require 'json'

# Reading JSON
data = JSON.parse(File.read("data.json"))

# Writing JSON
data = {
  name: "Alice",
  age: 30,
  hobbies: ["reading", "coding"]
}

File.write("output.json", JSON.pretty_generate(data))

# Streaming large JSON files
require 'json/streaming'

File.open("large.json") do |file|
  JSON::Streaming::Parser.parse(file) do |event, value|
    case event
    when :start_object
      # Handle object start
    when :key
      # Handle key
    when :value
      # Handle value
    end
  end
end
```

## YAML Files

```ruby
require 'yaml'

# Reading YAML
config = YAML.load_file("config.yml")

# Writing YAML
data = {
  database: {
    host: "localhost",
    port: 5432,
    name: "myapp"
  },
  features: ["auth", "api", "admin"]
}

File.write("config.yml", data.to_yaml)

# Safe loading (prevents code execution)
YAML.safe_load(File.read("untrusted.yml"))

# Custom classes in YAML
class Person
  attr_accessor :name, :age
  
  def initialize(name, age)
    @name = name
    @age = age
  end
end

person = Person.new("Alice", 30)
File.write("person.yml", person.to_yaml)

# Loading with permitted classes
loaded = YAML.safe_load(File.read("person.yml"), [Person])
```

## File Locking

```ruby
# Exclusive lock for writing
File.open("data.txt", "w") do |file|
  file.flock(File::LOCK_EX)  # Exclusive lock
  file.puts "Writing with lock"
  # Lock released when file closes
end

# Shared lock for reading
File.open("data.txt", "r") do |file|
  file.flock(File::LOCK_SH)  # Shared lock
  content = file.read
end

# Non-blocking lock
File.open("data.txt", "w") do |file|
  if file.flock(File::LOCK_EX | File::LOCK_NB)
    # Got the lock
    file.puts "Writing"
  else
    puts "File is locked by another process"
  end
end

# Lock with timeout
def with_file_lock(filename, timeout = 5)
  File.open(filename, "r+") do |file|
    Timeout.timeout(timeout) do
      file.flock(File::LOCK_EX)
      yield file
    end
  end
rescue Timeout::Error
  raise "Could not acquire lock on #{filename}"
end
```

## Temporary Files

```ruby
require 'tempfile'

# Create temporary file
temp = Tempfile.new('myapp')
temp.write("Temporary data")
temp.rewind
content = temp.read
temp.close
temp.unlink  # Delete the file

# With block (auto-cleanup)
Tempfile.create('myapp') do |temp|
  temp.write("Temporary data")
  temp.rewind
  # File deleted when block ends
end

# Temporary directory
require 'tmpdir'

Dir.mktmpdir do |dir|
  File.write("#{dir}/temp.txt", "data")
  # Directory and contents deleted when block ends
end

# Custom temporary file
require 'securerandom'

tmp_name = "tmp_#{SecureRandom.hex(8)}.txt"
tmp_path = File.join(Dir.tmpdir, tmp_name)

begin
  File.write(tmp_path, "temporary data")
  # Use the file
ensure
  File.delete(tmp_path) if File.exist?(tmp_path)
end
```

## File Watching

```ruby
# Simple file monitoring
last_modified = File.mtime("watched.txt")

loop do
  sleep 1
  current_modified = File.mtime("watched.txt")
  
  if current_modified > last_modified
    puts "File changed!"
    last_modified = current_modified
    # Process changes
  end
end

# Using Listen gem for better watching
require 'listen'

listener = Listen.to('.', only: /\.rb$/) do |modified, added, removed|
  puts "Modified: #{modified}" unless modified.empty?
  puts "Added: #{added}" unless added.empty?
  puts "Removed: #{removed}" unless removed.empty?
end

listener.start
sleep  # Keep running
```

## Practical Example: Log File Analyzer

```ruby
class LogAnalyzer
  attr_reader :log_file, :stats
  
  def initialize(log_file)
    @log_file = log_file
    @stats = {
      total_lines: 0,
      error_count: 0,
      warning_count: 0,
      by_date: Hash.new(0),
      by_level: Hash.new(0),
      top_errors: Hash.new(0)
    }
  end
  
  def analyze
    File.foreach(@log_file) do |line|
      @stats[:total_lines] += 1
      process_line(line)
    end
    
    generate_report
  end
  
  private
  
  def process_line(line)
    # Parse log line: [2024-03-15 10:30:45] ERROR: Something went wrong
    if match = line.match(/\[(\d{4}-\d{2}-\d{2}) .*?\] (\w+): (.*)/)
      date = match[1]
      level = match[2]
      message = match[3]
      
      @stats[:by_date][date] += 1
      @stats[:by_level][level] += 1
      
      case level
      when "ERROR"
        @stats[:error_count] += 1
        @stats[:top_errors][message] += 1
      when "WARNING"
        @stats[:warning_count] += 1
      end
    end
  end
  
  def generate_report
    File.open("log_report.txt", "w") do |report|
      report.puts "Log Analysis Report"
      report.puts "=" * 50
      report.puts "Total lines: #{@stats[:total_lines]}"
      report.puts "Errors: #{@stats[:error_count]}"
      report.puts "Warnings: #{@stats[:warning_count]}"
      
      report.puts "\nEntries by date:"
      @stats[:by_date].sort.each do |date, count|
        report.puts "  #{date}: #{count}"
      end
      
      report.puts "\nTop 5 errors:"
      @stats[:top_errors]
        .sort_by { |msg, count| -count }
        .take(5)
        .each do |message, count|
          report.puts "  #{count}x: #{message[0..50]}..."
        end
    end
    
    puts "Report generated: log_report.txt"
  end
end

# Usage
analyzer = LogAnalyzer.new("application.log")
analyzer.analyze
```

## File Processing Pipeline

```ruby
class FileProcessor
  def self.process_directory(input_dir, output_dir)
    FileUtils.mkdir_p(output_dir)
    
    Dir.glob("#{input_dir}/**/*.txt") do |input_file|
      relative_path = Pathname.new(input_file)
        .relative_path_from(Pathname.new(input_dir))
      
      output_file = File.join(output_dir, relative_path)
      FileUtils.mkdir_p(File.dirname(output_file))
      
      process_file(input_file, output_file)
    end
  end
  
  def self.process_file(input_file, output_file)
    puts "Processing: #{input_file}"
    
    File.open(input_file, "r") do |input|
      File.open(output_file, "w") do |output|
        input.each_line.with_index do |line, index|
          processed = process_line(line, index + 1)
          output.puts processed
        end
      end
    end
    
    puts "  -> #{output_file}"
  end
  
  def self.process_line(line, line_number)
    # Add line numbers and clean up
    cleaned = line.strip.gsub(/\s+/, ' ')
    "#{line_number.to_s.rjust(4)}: #{cleaned}"
  end
end

# Usage
FileProcessor.process_directory("input", "output")
```

## Best Practices

### 1. Always Use Blocks for File Operations
```ruby
# Good - file automatically closed
File.open("file.txt") do |file|
  file.read
end

# Bad - might forget to close
file = File.open("file.txt")
content = file.read
file.close  # Easy to forget
```

### 2. Handle Encoding Properly
```ruby
# Specify encoding when needed
File.read("file.txt", encoding: "UTF-8")
File.open("file.txt", "r:UTF-8") do |file|
  # ...
end
```

### 3. Check File Existence
```ruby
if File.exist?(filename)
  content = File.read(filename)
else
  puts "File not found: #{filename}"
end
```

### 4. Use Appropriate Methods for File Size
```ruby
# For small files
content = File.read("small.txt")

# For large files
File.foreach("large.txt") do |line|
  # Process line by line
end
```

## Your Turn: File Challenges

1. **File Synchronizer**: Build a tool that syncs files between directories
2. **Log Rotator**: Create a log rotation system with compression
3. **Configuration Manager**: Build a system to manage app configs
4. **File Differ**: Implement a file comparison tool
5. **Backup System**: Create an incremental backup solution

## What You've Learned

You now know how to:
- Read and write files in various ways
- Work with directories and paths
- Handle CSV, JSON, and YAML files
- Use temporary files safely
- Lock files for concurrent access
- Process files efficiently
- Follow file handling best practices

## What's Next?

In the next chapter, we'll dive into testing with RSpec—how to ensure your code works correctly and continues to work as you make changes. Testing isn't just about finding bugs; it's about building confidence in your code.

Remember: Files are how programs persist data and communicate with each other. Handle them carefully—lost data is hard to recover. Always consider what happens if your program crashes mid-write, if multiple processes access the same file, or if the disk is full. Defensive file handling isn't paranoia; it's professionalism!