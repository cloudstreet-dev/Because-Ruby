# Chapter 3: Setting Up Your Ruby Playground

Alright, enough talk. Time to actually get Ruby on your computer so you can start breaking things—I mean, learning!

Setting up a programming environment used to be like assembling IKEA furniture blindfolded. These days, it's more like... assembling IKEA furniture with the lights on. Progress!

## First Things First: Do You Already Have Ruby?

Plot twist: you might already have Ruby! If you're on macOS or Linux, there's a decent chance Ruby came pre-installed, like those U2 songs nobody asked for.

Open your terminal (that scary black window that makes you look like a hacker) and type:

```bash
ruby --version
```

If you see something like `ruby 3.2.0` or higher, congratulations! You're already halfway there. If you see `command not found` or a version older than 3.0, keep reading.

**Windows users**: You definitely don't have Ruby pre-installed. Windows believes in making you work for everything.

## The Official "This Won't Hurt a Bit" Installation Guide

### macOS: The Homebrew Way

If you're on a Mac, you'll want to use Homebrew (the unofficial package manager that should be official).

1. **Install Homebrew** (if you haven't already):
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

This looks terrifying, but it's just downloading and running an installation script. It's like clicking "I agree" to terms and conditions, but geekier.

2. **Install Ruby**:
```bash
brew install ruby
```

3. **Add Ruby to your PATH** (so your computer knows where to find it):
```bash
echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Windows: The Ruby Installer Way

Windows users, you get the simplest installation! (It's about time Windows made something easier.)

1. Go to [RubyInstaller.org](https://rubyinstaller.org/)
2. Download the latest Ruby+Devkit version (the one with the biggest number)
3. Run the installer
4. Check all the boxes (especially "Add Ruby to PATH")
5. At the end, let it run `ridk install` - just press Enter when prompted

That's it! You've installed Ruby with approximately 5 clicks. Your ancestors who had to compile from source are jealous.

### Linux: The Package Manager Way

Linux users, you already know what you're doing, but here's the Ubuntu/Debian way:

```bash
sudo apt update
sudo apt install ruby-full
```

For other distributions, you know the drill. Use your package manager of choice.

## Version Managers: The Pro Move

Here's a secret: professional Ruby developers don't just install Ruby directly. They use version managers because different projects need different Ruby versions. It's like having multiple personalities, but for programming.

### rbenv: The Minimalist's Choice

```bash
# Install rbenv
brew install rbenv ruby-build  # macOS
# or
git clone https://github.com/rbenv/rbenv.git ~/.rbenv  # Linux

# Add to your shell
echo 'eval "$(rbenv init -)"' >> ~/.zshrc
source ~/.zshrc

# Install Ruby
rbenv install 3.3.0
rbenv global 3.3.0
```

### RVM: The Swiss Army Knife

```bash
# Install RVM
\curl -sSL https://get.rvm.io | bash -s stable

# Install Ruby
rvm install 3.3.0
rvm use 3.3.0 --default
```

**Pro tip**: Pick rbenv if you like things simple. Pick RVM if you like features you'll never use.

## Your First Ruby Playground: IRB

IRB (Interactive Ruby) is where you can experiment without commitment. It's like a sandbox, but for code.

Type `irb` in your terminal:

```ruby
$ irb
irb(main):001:0> puts "Hello, Ruby!"
Hello, Ruby!
=> nil
irb(main):002:0> 2 + 2
=> 4
irb(main):003:0> "stressed".reverse
=> "desserts"
irb(main):004:0> exit
```

That `=> nil` or `=> 4` is Ruby showing you what your code evaluated to. It's like Ruby's way of saying, "This is what I understood from that."

## Pry: IRB's Cooler Sibling

IRB is fine, but Pry is better. It's like IRB went to college and came back with syntax highlighting and better error messages.

```bash
gem install pry
pry
```

Now you can do everything IRB does, but prettier:

```ruby
[1] pry(main)> def greet(name)
[1] pry(main)*   "Hello, #{name}!"
[1] pry(main)* end
=> :greet
[2] pry(main)> greet("World")
=> "Hello, World!"
```

## Your First Ruby File

IRB is fun, but real programs live in files. Create a file called `hello.rb`:

```ruby
# hello.rb
# My first Ruby program!

def introduce_myself
  name = "Future Rubyist"
  age = "old enough to know better"
  mood = "cautiously optimistic"
  
  puts "Hi! I'm #{name}."
  puts "I'm #{age} years old."
  puts "I'm feeling #{mood} about learning Ruby!"
end

introduce_myself

# Let's do something fun
puts "\nHere's a joke:"
puts "Why do programmers prefer dark mode?"
puts "Because light attracts bugs! 🐛"

# And something useful
puts "\nToday is #{Time.now.strftime('%A, %B %d, %Y')}"
puts "You've been learning Ruby for approximately 0 days."
puts "You're already 5% cooler than you were yesterday."
```

Save it and run it:

```bash
ruby hello.rb
```

Congratulations! You've just run your first Ruby program. Your computer now respects you 3% more.

## Setting Up a Real Development Environment

### VS Code: The People's Champion

VS Code is free, powerful, and has more extensions than a 90s phone cord.

1. Download [VS Code](https://code.visualstudio.com/)
2. Install the Ruby extension (search for "Ruby" in extensions)
3. Install the "Ruby Solargraph" extension for super powers
4. Feel like a real developer

### RubyMine: The Luxury Option

If you want the Rolls-Royce of Ruby IDEs (and don't mind paying), get RubyMine. It knows what you want to type before you do. It's basically telepathic.

### Sublime Text: The Speed Demon

Fast, pretty, and minimalist. Like a sports car for your code.

### Vim/Neovim: The Final Boss

If you use Vim, you don't need my advice. You've already transcended to a higher plane of existence. The rest of us are still trying to figure out how to exit.

## Essential Ruby Gems for Beginners

Gems are Ruby's packages/libraries. Install them with the `gem` command:

```bash
gem install bundler  # Manages other gems
gem install rubocop  # Tells you when your code is ugly
gem install pry-byebug  # Debugging superpowers
gem install colorize  # Makes terminal output pretty
```

## Creating Your First Ruby Project

Let's set up a proper Ruby project structure:

```bash
mkdir my_awesome_project
cd my_awesome_project
```

Create a `Gemfile` (this tracks your project's dependencies):

```ruby
# Gemfile
source 'https://rubygems.org'

gem 'colorize'
gem 'pry'

group :development do
  gem 'rubocop'
end
```

Install the gems:

```bash
bundle install
```

Create your main Ruby file:

```ruby
# app.rb
require 'colorize'

puts "Welcome to Ruby!".colorize(:green)
puts "This text is blue".colorize(:blue)
puts "This is red on yellow".colorize(:red).on_yellow
puts "This is bold and underlined".bold.underline

# Let's make a rainbow!
text = "RAINBOW"
colors = [:red, :yellow, :green, :cyan, :blue, :magenta]

text.chars.each_with_index do |char, i|
  print char.colorize(colors[i % colors.length])
end
puts
```

Run it:

```bash
ruby app.rb
```

Look at that! You're already making terminal rainbows. Your parents would be so proud (or confused).

## Troubleshooting: When Things Go Wrong

### "Command not found"
Your computer can't find Ruby. Either:
- You didn't install it (go back to the installation section)
- It's installed but not in your PATH (Google "add ruby to PATH" + your operating system)

### "Permission denied"
You're trying to install gems system-wide. Add `sudo` on Mac/Linux or run as administrator on Windows. Or better yet, use a version manager.

### "LoadError: cannot load such file"
You're trying to use a gem you haven't installed. Run `gem install [gem-name]` or `bundle install` if you have a Gemfile.

### "Syntax error, unexpected end-of-input"
You forgot an `end` somewhere. Ruby needs closure. Check that all your `def`, `if`, `do`, and `class` statements have matching `end`s.

## Your Development Ritual

Every Ruby developer has a ritual. Here's a good starter one:

1. **Open terminal**
2. **Navigate to your project**: `cd ~/code/my_project`
3. **Open your editor**: `code .` (for VS Code)
4. **Start PRY for experimentation**: `pry` in another terminal tab
5. **Write code**
6. **Run code**: `ruby myfile.rb`
7. **Fix errors** (there will be errors)
8. **Repeat steps 5-7 until happy**
9. **Celebrate with coffee/tea/victory dance**

## The Ruby REPL Loop

Here's your learning loop for the next few chapters:

```ruby
loop do
  read_chapter
  try_examples
  break_something
  fix_it
  understand_why
  feel_accomplished
end
```

## Quick Sanity Check

Let's make sure everything's working. Create a file called `sanity_check.rb`:

```ruby
# sanity_check.rb
puts "Ruby version: #{RUBY_VERSION}"
puts "Platform: #{RUBY_PLATFORM}"
puts "Installation check: #{'✅' * 5}"

# Can we do math?
puts "2 + 2 = #{2 + 2}"

# Can we use methods?
def working?
  true
end

puts "Methods working: #{working? ? '✅' : '❌'}"

# Can we use gems?
begin
  require 'json'
  puts "Gems working: ✅"
rescue LoadError
  puts "Gems working: ❌ (but that's okay for now)"
end

puts "\nCongratulations! Your Ruby is ready to roll! 🚀"
```

Run it:
```bash
ruby sanity_check.rb
```

If you see checkmarks, you're golden!

## What's Next?

You now have:
- Ruby installed and running
- A text editor configured
- IRB/Pry for experimentation
- Your first programs working
- The ability to install and use gems

In the next chapter, we're going to learn about variables and data types. We'll teach Ruby to remember things, manipulate text, juggle numbers, and make lists. It's going to be like teaching a very fast, very literal child to organize their toys.

But first, take a moment to appreciate what you've accomplished. You've successfully set up a development environment. That's genuinely harder than most of the actual programming you'll do. It's all downhill from here (in a good way).

Remember: every error message is a learning opportunity, every bug is a puzzle, and every working program is a tiny miracle. Welcome to the wonderful world of Ruby development!

Now go make something that prints your name in rainbow colors. You've earned it.