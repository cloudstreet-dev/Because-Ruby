# Chapter 24: Templates and Views - Making Your App Look Good

HTML in strings is like eating soup with a fork - technically possible but unnecessarily difficult. Templates let us separate our HTML from our Ruby code, making both cleaner and happier. Let's learn how to make beautiful web pages with Sinatra's templating system!

## ERB: Embedded Ruby

ERB (Embedded Ruby) is Ruby's built-in templating language:

```ruby
require 'sinatra'

# Inline templates
get '/' do
  @name = "World"
  erb "<h1>Hello, <%= @name %>!</h1>"
end

# Better: Use template files
get '/welcome' do
  @user = "Alice"
  erb :welcome  # Looks for views/welcome.erb
end
```

Create `views/welcome.erb`:
```erb
<!DOCTYPE html>
<html>
<head>
  <title>Welcome!</title>
</head>
<body>
  <h1>Welcome, <%= @user %>!</h1>
  <p>The time is <%= Time.now %></p>
</body>
</html>
```

## ERB Syntax

ERB has two main tags:

```erb
<!-- <%= %> outputs the result -->
<p>Hello, <%= @name %>!</p>
<p>2 + 2 = <%= 2 + 2 %></p>

<!-- <% %> executes code without output -->
<% if @user %>
  <p>Welcome back, <%= @user.name %>!</p>
<% else %>
  <p>Please log in</p>
<% end %>

<!-- Loops -->
<ul>
  <% @items.each do |item| %>
    <li><%= item %></li>
  <% end %>
</ul>

<!-- Comments -->
<%# This is a comment that won't appear in HTML %>

<!-- Escaping HTML (automatic in Sinatra) -->
<%= @user_input %>  <!-- HTML is escaped -->
<%== @trusted_html %>  <!-- HTML is NOT escaped -->
```

## Layouts

Layouts wrap your templates with common HTML:

```ruby
# app.rb
get '/' do
  @title = "Home"
  erb :index
end

get '/about' do
  @title = "About"
  erb :about
end
```

`views/layout.erb`:
```erb
<!DOCTYPE html>
<html>
<head>
  <title><%= @title %> - My App</title>
  <link rel="stylesheet" href="/styles.css">
</head>
<body>
  <header>
    <nav>
      <a href="/">Home</a>
      <a href="/about">About</a>
      <a href="/contact">Contact</a>
    </nav>
  </header>
  
  <main>
    <%= yield %>  <!-- This is where the view content goes -->
  </main>
  
  <footer>
    <p>&copy; 2025 My Awesome App</p>
  </footer>
</body>
</html>
```

`views/index.erb`:
```erb
<h1>Welcome to My App!</h1>
<p>This is the homepage.</p>
```

## Passing Data to Templates

```ruby
# Multiple instance variables
get '/profile/:username' do
  @user = User.find_by(username: params[:username])
  @posts = @user.posts.recent
  @followers = @user.followers.count
  
  erb :profile
end

# Locals hash (alternative approach)
get '/product/:id' do
  product = Product.find(params[:id])
  
  erb :product, locals: {
    product: product,
    related: product.related_items,
    reviews: product.reviews
  }
end

# Combining both approaches
get '/dashboard' do
  @current_user = session[:user]
  
  erb :dashboard, locals: {
    stats: calculate_stats,
    notifications: fetch_notifications
  }
end
```

## Partials

Reuse template fragments:

```ruby
# helpers for partials
helpers do
  def partial(template, locals = {})
    erb(template, layout: false, locals: locals)
  end
end

get '/blog' do
  @posts = Post.all
  erb :blog
end
```

`views/blog.erb`:
```erb
<h1>Blog Posts</h1>
<% @posts.each do |post| %>
  <%= partial :_post, post: post %>
<% end %>
```

`views/_post.erb`:
```erb
<article>
  <h2><%= post.title %></h2>
  <p class="meta">By <%= post.author %> on <%= post.date %></p>
  <div class="content">
    <%= post.content %>
  </div>
</article>
```

## Other Template Engines

### Haml - The Minimal Template Engine

```ruby
# Add to Gemfile: gem 'haml'
require 'haml'

get '/haml-example' do
  @title = "Haml is neat!"
  haml :example
end
```

`views/example.haml`:
```haml
!!!
%html
  %head
    %title= @title
  %body
    %h1 Welcome to Haml
    %p.intro
      Haml uses indentation instead of closing tags.
    
    %ul
      - @items.each do |item|
        %li= item
    
    #footer
      %p.copyright
        &copy; 2025
```

### Slim - Even More Minimal

```ruby
# Add to Gemfile: gem 'slim'
require 'slim'

get '/slim-example' do
  slim :example
end
```

`views/example.slim`:
```slim
doctype html
html
  head
    title = @title
  body
    h1 Welcome to Slim
    p.intro Slim is even more minimal than Haml
    
    ul
      - @items.each do |item|
        li = item
    
    #footer
      p.copyright &copy; 2025
```

### Markdown Templates

```ruby
# Add to Gemfile: gem 'redcarpet'
require 'redcarpet'

get '/doc/:page' do
  markdown params[:page].to_sym
end
```

`views/readme.md`:
```markdown
# Documentation

This is written in **Markdown** and converted to HTML automatically!

## Features
- Easy to write
- Converts to clean HTML
- Great for documentation
```

## Template Helpers

Create custom helpers for templates:

```ruby
helpers do
  # Format helpers
  def format_date(date)
    date.strftime("%B %d, %Y")
  end
  
  def format_money(amount)
    "$%.2f" % amount
  end
  
  # HTML helpers
  def link_to(text, url, options = {})
    attrs = options.map { |k, v| "#{k}=\"#{v}\"" }.join(" ")
    "<a href=\"#{url}\" #{attrs}>#{text}</a>"
  end
  
  def image_tag(src, alt = "")
    "<img src=\"#{src}\" alt=\"#{alt}\">"
  end
  
  # Form helpers
  def text_field(name, value = "", options = {})
    attrs = options.map { |k, v| "#{k}=\"#{v}\"" }.join(" ")
    "<input type=\"text\" name=\"#{name}\" value=\"#{value}\" #{attrs}>"
  end
  
  def text_area(name, value = "", options = {})
    rows = options.delete(:rows) || 10
    cols = options.delete(:cols) || 50
    "<textarea name=\"#{name}\" rows=\"#{rows}\" cols=\"#{cols}\">#{value}</textarea>"
  end
  
  # Flash messages
  def flash_messages
    return "" unless session[:flash]
    
    message = session[:flash]
    session.delete(:flash)
    
    "<div class=\"flash\">#{message}</div>"
  end
  
  # Active navigation
  def nav_link(text, path)
    current = (request.path == path) ? 'class="active"' : ''
    "<a href=\"#{path}\" #{current}>#{text}</a>"
  end
end
```

Using helpers in templates:

```erb
<!-- views/layout.erb -->
<nav>
  <%= nav_link("Home", "/") %>
  <%= nav_link("About", "/about") %>
  <%= nav_link("Contact", "/contact") %>
</nav>

<%= flash_messages %>

<!-- views/post.erb -->
<article>
  <h1><%= @post.title %></h1>
  <p class="date">Posted on <%= format_date(@post.created_at) %></p>
  <%= image_tag(@post.image_url, @post.title) %>
  <div class="content">
    <%= @post.content %>
  </div>
</article>

<!-- views/product.erb -->
<div class="product">
  <h2><%= @product.name %></h2>
  <p class="price"><%= format_money(@product.price) %></p>
  <%= link_to("Buy Now", "/buy/#{@product.id}", class: "btn btn-primary") %>
</div>
```

## Asset Management

Handle CSS, JavaScript, and images:

```ruby
# Static files from public directory
# public/css/style.css
# public/js/app.js
# public/images/logo.png

# In your template:
```

```erb
<link rel="stylesheet" href="/css/style.css">
<script src="/js/app.js"></script>
<img src="/images/logo.png" alt="Logo">

<!-- Or with helpers -->
<%= stylesheet_tag 'style' %>
<%= javascript_tag 'app' %>
<%= image_tag '/images/logo.png' %>
```

Helper implementation:
```ruby
helpers do
  def stylesheet_tag(name)
    "<link rel=\"stylesheet\" href=\"/css/#{name}.css\">"
  end
  
  def javascript_tag(name)
    "<script src=\"/js/#{name}.js\"></script>"
  end
end
```

## Working with Forms

```erb
<!-- views/contact.erb -->
<h1>Contact Us</h1>

<form action="/contact" method="post">
  <div class="form-group">
    <label for="name">Name:</label>
    <input type="text" id="name" name="name" required>
  </div>
  
  <div class="form-group">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
  </div>
  
  <div class="form-group">
    <label for="subject">Subject:</label>
    <select id="subject" name="subject">
      <option value="general">General Inquiry</option>
      <option value="support">Support</option>
      <option value="billing">Billing</option>
    </select>
  </div>
  
  <div class="form-group">
    <label for="message">Message:</label>
    <textarea id="message" name="message" rows="5" required></textarea>
  </div>
  
  <button type="submit">Send Message</button>
</form>

<!-- With helpers -->
<%= form_tag("/contact", method: "post") do %>
  <%= text_field("name", @name, placeholder: "Your name") %>
  <%= email_field("email", @email, placeholder: "your@email.com") %>
  <%= text_area("message", @message, rows: 10) %>
  <%= submit_button("Send") %>
<% end %>
```

## AJAX and JSON Templates

```ruby
# Return JSON for AJAX requests
get '/api/users' do
  content_type :json
  @users = User.all
  
  # Use a JSON template
  erb :"users.json", layout: false
end
```

`views/users.json.erb`:
```erb
{
  "users": [
    <% @users.each_with_index do |user, i| %>
      {
        "id": <%= user.id %>,
        "name": "<%= user.name %>",
        "email": "<%= user.email %>"
      }<%= "," unless i == @users.length - 1 %>
    <% end %>
  ]
}
```

Or use Jbuilder:
```ruby
# Gemfile: gem 'sinatra-jbuilder'
require 'sinatra/jbuilder'

get '/api/users' do
  @users = User.all
  jbuilder :users
end
```

`views/users.jbuilder`:
```ruby
json.users @users do |user|
  json.id user.id
  json.name user.name
  json.email user.email
  json.url "/users/#{user.id}"
end
```

## Advanced Template Techniques

### Content Blocks

```ruby
helpers do
  def content_for(key, &block)
    @content_blocks ||= {}
    @content_blocks[key] = capture(&block)
  end
  
  def yield_content(key)
    @content_blocks && @content_blocks[key]
  end
  
  def capture(&block)
    block.call
  end
end
```

Usage:
```erb
<!-- views/page.erb -->
<% content_for :head do %>
  <script src="/special-script.js"></script>
<% end %>

<h1>Page Content</h1>

<% content_for :footer do %>
  <p>Special footer content for this page</p>
<% end %>

<!-- views/layout.erb -->
<head>
  <title>My App</title>
  <%= yield_content :head %>
</head>
<body>
  <%= yield %>
  
  <footer>
    <%= yield_content :footer %>
  </footer>
</body>
```

### Caching Templates

```ruby
require 'sinatra/cache'

class App < Sinatra::Base
  register Sinatra::Cache
  
  get '/expensive-page' do
    cache(erb(:expensive), expires: 3600)
  end
  
  # Fragment caching
  helpers do
    def cache_fragment(key, &block)
      @fragment_cache ||= {}
      @fragment_cache[key] ||= capture(&block)
    end
  end
end
```

## Template Best Practices

```ruby
class BlogApp < Sinatra::Base
  # Organize views in subdirectories
  set :views, Proc.new { File.join(root, "app/views") }
  
  # Different layouts for different sections
  get '/admin/*' do
    erb :admin_page, layout: :admin_layout
  end
  
  # Inline templates for small apps
  __END__

  @@ layout
  <!DOCTYPE html>
  <html>
    <body>
      <%= yield %>
    </body>
  </html>

  @@ index
  <h1>Welcome!</h1>
end
```

## Your Turn: Template Challenges

1. **Blog Theme**: Create a complete blog template with posts and comments
2. **Dashboard**: Build an admin dashboard with charts and tables
3. **Email Templates**: Create HTML email templates
4. **Multi-language**: Support multiple languages in templates
5. **Component Library**: Build reusable UI components

## What You've Learned

You now know how to:
- Use ERB templates in Sinatra
- Create layouts and partials
- Pass data to templates
- Use different template engines
- Create template helpers
- Manage assets
- Build forms
- Handle AJAX requests
- Implement advanced template patterns

## What's Next?

Next, we'll dive into forms and user input, learning how to create interactive web applications that respond to user actions. Get ready to make your apps truly interactive!