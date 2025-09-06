# Chapter 14: Regular Expressions - Pattern Matching Without the Headache

Regular expressions (regex) are like a secret language for finding patterns in text. They look like someone fell asleep on their keyboard: `/^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$/i`. But once you understand them, they're incredibly powerful. Think of them as wildcards on steroids.

## What Are Regular Expressions?

A regular expression is a pattern that describes a set of strings. Ruby uses them to search, match, extract, and replace text. They're the difference between writing 50 lines of string manipulation code and writing one elegant line.

```ruby
# Without regex (the painful way)
def valid_email?(email)
  return false unless email.include?("@")
  parts = email.split("@")
  return false unless parts.length == 2
  return false if parts[0].empty? || parts[1].empty?
  return false unless parts[1].include?(".")
  domain_parts = parts[1].split(".")
  return false if domain_parts.last.length < 2
  true
end

# With regex (the elegant way)
def valid_email?(email)
  email.match?(/\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i)
end
```

## Basic Pattern Matching

In Ruby, regular expressions are enclosed in forward slashes or created with `Regexp.new`:

```ruby
# Literal matching
"Hello, World!" =~ /World/  # 7 (position where match starts)
"Hello, World!" =~ /world/  # nil (case sensitive)
"Hello, World!" =~ /world/i # 7 (i flag makes it case insensitive)

# Using match method
"Hello, World!".match(/World/)  # #<MatchData "World">
"Hello, World!".match(/xyz/)    # nil

# Using match? for boolean result
"Hello, World!".match?(/World/) # true
"Hello, World!".match?(/xyz/)   # false

# Scan for all matches
"The quick brown fox".scan(/[aeiou]/)  # ["e", "u", "i", "o", "o"]
```

## Character Classes

Character classes match any one character from a set:

```ruby
# Basic character classes
/[aeiou]/     # Any vowel
/[0-9]/       # Any digit
/[a-z]/       # Any lowercase letter
/[A-Z]/       # Any uppercase letter
/[a-zA-Z]/    # Any letter
/[^aeiou]/    # Anything EXCEPT vowels (^ negates)

# Predefined character classes
/\d/          # Any digit (same as [0-9])
/\D/          # Any non-digit
/\w/          # Any word character [a-zA-Z0-9_]
/\W/          # Any non-word character
/\s/          # Any whitespace (space, tab, newline)
/\S/          # Any non-whitespace
/./           # Any character except newline

# Examples
"Price: $42.99".scan(/\d/)     # ["4", "2", "9", "9"]
"Price: $42.99".scan(/\d+/)    # ["42", "99"]
"Hello World".scan(/\w+/)      # ["Hello", "World"]
"a1b2c3".scan(/[a-z]\d/)      # ["a1", "b2", "c3"]
```

## Anchors and Boundaries

Anchors don't match characters; they match positions:

```ruby
# Line anchors
/^Hello/      # Start of line
/World$/      # End of line
/^Hello$/     # Entire line is "Hello"

# String anchors (more reliable)
/\AHello/     # Start of string
/World\z/     # End of string
/\AHello\z/   # Entire string is "Hello"

# Word boundaries
/\bword\b/    # Whole word "word"
/\Bword\B/    # "word" inside another word

# Examples
"Hello World".match?(/^Hello/)     # true
"Hello World".match?(/^World/)     # false
"Hello World".match?(/World$/)     # true

# Word boundary examples
"The word is here".match(/\bword\b/)     # matches "word"
"The sword is here".match(/\bword\b/)    # nil (word is part of sword)
"password123".match(/\bword\b/)          # nil
```

## Quantifiers: How Many?

Quantifiers specify how many times a pattern should match:

```ruby
# Basic quantifiers
/a*/          # 0 or more 'a's
/a+/          # 1 or more 'a's
/a?/          # 0 or 1 'a'
/a{3}/        # Exactly 3 'a's
/a{3,}/       # 3 or more 'a's
/a{3,5}/      # Between 3 and 5 'a's

# Examples
"".match?(/a*/)           # true (0 matches)
"aaa".match?(/a+/)        # true (1 or more)
"".match?(/a+/)           # false (needs at least 1)
"color".match?(/colou?r/) # true (u is optional)
"colour".match?(/colou?r/) # true

# Greedy vs non-greedy
text = "<tag>content</tag>"
text.match(/<.*>/)[0]     # "<tag>content</tag>" (greedy - takes everything)
text.match(/<.*?>/)[0]    # "<tag>" (non-greedy - takes minimum)

# Real examples
"Call me at 555-1234 or 555-5678".scan(/\d{3}-\d{4}/)
# ["555-1234", "555-5678"]

"aaabbbccc".match(/a{3}b{3}c{3}/)  # matches
"abc".match(/a{3}b{3}c{3}/)        # nil
```

## Groups and Captures

Parentheses create groups that can be captured and reused:

```ruby
# Basic grouping
/(Ruby|Python|JavaScript)/  # Matches any of these languages

# Capturing groups
phone = "Call me at (555) 123-4567"
if match = phone.match(/\((\d{3})\) (\d{3})-(\d{4})/)
  puts "Area code: #{match[1]}"  # "555"
  puts "Exchange: #{match[2]}"   # "123"
  puts "Number: #{match[3]}"     # "4567"
  puts "Full match: #{match[0]}" # "(555) 123-4567"
end

# Named captures (Ruby 1.9+)
pattern = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/
if match = "2024-03-15".match(pattern)
  puts match[:year]   # "2024"
  puts match[:month]  # "03"
  puts match[:day]    # "15"
end

# Non-capturing groups (?: )
# Use when you need grouping but don't need to capture
/(?:Mr|Ms|Mrs|Dr)\. (\w+)/  # Only captures the name, not the title

# Backreferences
/(\w+) \1/  # Matches repeated words: "the the", "is is"
"the the quick brown fox".gsub(/(\w+) \1/, '\1')  # "the quick brown fox"

# Example: matching HTML tags
html = "<b>Bold</b> and <i>Italic</i>"
html.scan(/<(\w+)>.*?<\/\1>/)  # Uses \1 to match closing tag
# [["b"], ["i"]]
```

## Character Escaping

Some characters have special meaning and need escaping:

```ruby
# Special characters that need escaping
# . ^ $ * + ? { } [ ] \ | ( )

# Escape with backslash
/\./          # Literal period
/\$/          # Literal dollar sign
/\[/          # Literal bracket

# Examples
"Price: $9.99".match(/\$\d+\.\d{2}/)  # Matches dollar amounts
"2 + 2 = 4".match(/\d+ \+ \d+/)       # Matches addition

# Regexp.escape for dynamic patterns
user_input = "What's (up)?"
pattern = Regexp.escape(user_input)  # "What's \\(up\\)\\?"
text.match(/#{pattern}/)

# Quote method (alias for escape)
Regexp.quote("Price: $10.00")  # "Price:\\ \\$10\\.00"
```

## Practical Examples

### Email Validation

```ruby
class EmailValidator
  # Simple version
  SIMPLE = /\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i
  
  # More complete version (RFC 5322 simplified)
  COMPLETE = /\A[a-zA-Z0-9.!#$%&'*+\/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*\z/
  
  def self.valid?(email)
    email.match?(SIMPLE)
  end
  
  def self.extract_parts(email)
    if match = email.match(/\A([\w+\-.]+)@([\w\-]+(?:\.[\w\-]+)*\.[a-z]+)\z/i)
      {
        local: match[1],
        domain: match[2]
      }
    end
  end
end

EmailValidator.valid?("user@example.com")  # true
EmailValidator.valid?("invalid.email")     # false
EmailValidator.extract_parts("alice@company.co.uk")
# {:local=>"alice", :domain=>"company.co.uk"}
```

### URL Parsing

```ruby
class URLParser
  URL_PATTERN = %r{
    \A
    (?<protocol>https?):\/\/
    (?<host>[^\/\?:]+)
    (?::(?<port>\d+))?
    (?<path>\/[^\?]*)?
    (?:\?(?<query>[^#]*))?
    (?:#(?<fragment>.*))?
    \z
  }x  # x flag allows comments and whitespace
  
  def self.parse(url)
    if match = url.match(URL_PATTERN)
      {
        protocol: match[:protocol],
        host: match[:host],
        port: match[:port] || (match[:protocol] == "https" ? "443" : "80"),
        path: match[:path] || "/",
        query: match[:query],
        fragment: match[:fragment]
      }
    end
  end
end

URLParser.parse("https://example.com:8080/path/to/page?q=ruby#section")
# {
#   :protocol=>"https",
#   :host=>"example.com",
#   :port=>"8080",
#   :path=>"/path/to/page",
#   :query=>"q=ruby",
#   :fragment=>"section"
# }
```

### Text Processing

```ruby
class TextProcessor
  # Remove extra whitespace
  def self.clean_whitespace(text)
    text.gsub(/\s+/, ' ').strip
  end
  
  # Convert camelCase to snake_case
  def self.to_snake_case(text)
    text.gsub(/([A-Z])/, '_\1').downcase.gsub(/^_/, '')
  end
  
  # Extract hashtags
  def self.extract_hashtags(text)
    text.scan(/#\w+/)
  end
  
  # Censor profanity
  def self.censor(text, words)
    pattern = Regexp.union(words.map { |w| /\b#{Regexp.escape(w)}\b/i })
    text.gsub(pattern) { |match| '*' * match.length }
  end
  
  # Extract quoted strings
  def self.extract_quotes(text)
    text.scan(/"([^"]*)"/).flatten
  end
  
  # Format phone numbers
  def self.format_phone(number)
    cleaned = number.gsub(/\D/, '')
    if match = cleaned.match(/\A(\d{3})(\d{3})(\d{4})\z/)
      "(#{match[1]}) #{match[2]}-#{match[3]}"
    else
      number
    end
  end
end

TextProcessor.clean_whitespace("  Too    many     spaces  ")
# "Too many spaces"

TextProcessor.to_snake_case("getUserName")
# "get_user_name"

TextProcessor.extract_hashtags("Check out #Ruby and #Rails!")
# ["#Ruby", "#Rails"]

TextProcessor.format_phone("5551234567")
# "(555) 123-4567"
```

### Log File Analysis

```ruby
class LogAnalyzer
  # Apache/Nginx log format
  LOG_PATTERN = /
    (?<ip>\d+\.\d+\.\d+\.\d+)\s+
    \S+\s+\S+\s+
    \[(?<timestamp>[^\]]+)\]\s+
    "(?<method>\w+)\s+(?<path>[^\s]+)\s+[^"]+"\s+
    (?<status>\d+)\s+
    (?<size>\d+|-)\s+
    "(?<referrer>[^"]*)"\s+
    "(?<user_agent>[^"]*)"
  /x
  
  def initialize(log_file)
    @log_file = log_file
  end
  
  def parse_entries
    File.readlines(@log_file).map do |line|
      if match = line.match(LOG_PATTERN)
        {
          ip: match[:ip],
          timestamp: Time.parse(match[:timestamp]),
          method: match[:method],
          path: match[:path],
          status: match[:status].to_i,
          size: match[:size] == '-' ? 0 : match[:size].to_i,
          referrer: match[:referrer],
          user_agent: match[:user_agent]
        }
      end
    end.compact
  end
  
  def find_404s
    parse_entries.select { |entry| entry[:status] == 404 }
  end
  
  def top_ips(n = 10)
    parse_entries
      .group_by { |entry| entry[:ip] }
      .transform_values(&:count)
      .sort_by { |ip, count| -count }
      .take(n)
  end
  
  def bot_traffic
    parse_entries.select do |entry|
      entry[:user_agent].match?(/bot|crawler|spider|scraper/i)
    end
  end
end
```

## Advanced Regex Features

### Lookahead and Lookbehind

```ruby
# Positive lookahead (?=pattern)
# Matches position followed by pattern
"password123".match(/\w+(?=\d)/)  # Matches "password" (followed by digit)

# Negative lookahead (?!pattern)
# Matches position NOT followed by pattern
"test.rb".match(/\w+(?!\.rb)/)    # Matches "tes" (not followed by .rb)

# Positive lookbehind (?<=pattern)
# Matches position preceded by pattern
"$100".match(/(?<=\$)\d+/)        # Matches "100" (preceded by $)

# Negative lookbehind (?<!pattern)
# Matches position NOT preceded by pattern
"100 dollars".match(/(?<!\$)\d+/) # Matches "100" (not preceded by $)

# Password validation with lookahead
def strong_password?(password)
  password.match?(/
    \A
    (?=.*[a-z])        # At least one lowercase
    (?=.*[A-Z])        # At least one uppercase
    (?=.*\d)           # At least one digit
    (?=.*[@$!%*?&])    # At least one special char
    [A-Za-z\d@$!%*?&]{8,}  # 8+ chars from allowed set
    \z
  /x)
end
```

### Conditional Patterns

```ruby
# Match different patterns based on previous captures
pattern = /(Mr|Ms)\. (\w+) (?(1)Smith|Jones)/
# If group 1 matches, look for Smith, else look for Jones

# More practical: match quotes
quote_pattern = /(['"])(.*?)\1/  # \1 refers back to the quote type
"'single quotes'".match(quote_pattern)  # Works
'"double quotes"'.match(quote_pattern)  # Works
"'mismatched\"".match(quote_pattern)    # Doesn't match
```

### Regex Options

```ruby
# Case insensitive
/ruby/i.match?("RUBY")  # true

# Multiline mode (. matches newline)
/start.*end/m.match?("start\nmiddle\nend")  # true

# Extended mode (ignore whitespace, allow comments)
pattern = /
  \A              # Start of string
  [\w+\-.]+       # Local part
  @               # At sign
  [a-z\d\-]+      # Domain name
  (\.[a-z]+)+     # TLD
  \z              # End of string
/ix  # Case insensitive and extended

# Free-spacing mode for complex patterns
complex = %r{
  \A                          # Start of string
  (?<protocol>https?)://      # Protocol
  (?<domain>[^/]+)           # Domain
  (?<path>/.*)?              # Optional path
  \z                          # End of string
}x
```

## String Methods with Regex

```ruby
# gsub - global substitution
"Hello World".gsub(/[aeiou]/, '*')  # "H*ll* W*rld"
"Hello World".gsub(/(\w)/, '[\1]')  # "[H][e][l][l][o] [W][o][r][l][d]"

# sub - single substitution
"Hello Hello".sub(/Hello/, 'Hi')    # "Hi Hello"

# scan - find all matches
"a1b2c3".scan(/[a-z]\d/)           # ["a1", "b2", "c3"]
"test@example.com".scan(/\w+/)      # ["test", "example", "com"]

# split - divide string
"one,two,three".split(/,/)          # ["one", "two", "three"]
"CamelCaseString".split(/(?=[A-Z])/) # ["Camel", "Case", "String"]

# Block forms
text = "Price: $10, Cost: $5"
text.gsub(/\$(\d+)/) do |match|
  amount = $1.to_i * 2
  "$#{amount}"
end
# "Price: $20, Cost: $10"
```

## Performance Tips

```ruby
# Compile once, use many times
EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i

# Bad - compiles regex each time
def validate_email(email)
  email.match?(/\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i)
end

# Good - uses precompiled regex
def validate_email(email)
  email.match?(EMAIL_REGEX)
end

# Use non-capturing groups when you don't need the capture
/(?:Mr|Ms|Mrs)\. (\w+)/  # Only captures the name

# Anchor patterns when possible
/\A\d+\z/  # Much faster than /^\d+$/ for validation

# Use character classes instead of alternation
/[aeiou]/  # Faster than /(a|e|i|o|u)/
```

## Common Gotchas

```ruby
# Dot doesn't match newline by default
"line1\nline2".match(/line1.line2/)   # nil
"line1\nline2".match(/line1.line2/m)  # matches with multiline flag

# ^ and $ match line boundaries, not string boundaries
"first\nsecond".match?(/^second$/)    # true (matches line)
"first\nsecond".match?(/\Asecond\z/)  # false (doesn't match string)

# Greedy quantifiers can match too much
"<tag>text</tag>".match(/<.*>/)[0]    # "<tag>text</tag>" (too much!)
"<tag>text</tag>".match(/<.*?>/)[0]   # "<tag>" (non-greedy)

# Remember to escape special characters
"price: $10.99".match(/\$\d+\.\d+/)   # Must escape $ and .
```

## Your Turn: Regex Challenges

1. **Markdown Parser**: Extract headers, links, and emphasis from Markdown
2. **Credit Card Validator**: Validate and identify card types
3. **Code Syntax Highlighter**: Identify keywords, strings, and comments
4. **Data Extractor**: Pull structured data from unstructured text
5. **HTML Tag Stripper**: Remove or extract HTML tags from text

## What You've Learned

You now understand:
- Basic pattern matching with regex
- Character classes and predefined patterns
- Anchors and boundaries
- Quantifiers and greediness
- Groups, captures, and backreferences
- Lookahead and lookbehind assertions
- String methods that use regex
- Performance considerations

## What's Next?

In the next chapter, we'll explore error handling—how to gracefully handle things when they go wrong (and they will). You'll learn about exceptions, rescue blocks, and how to write robust code that doesn't crash at the first sign of trouble.

Remember: Regular expressions are powerful but can quickly become unreadable. When a regex gets complex, consider breaking it into smaller pieces or using multiple simpler patterns. And always comment your regex patterns—future you will thank present you!