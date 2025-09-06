# Chapter 22: Sinatra - The Minimalist's Web Framework

Rails might be the famous one, but Sinatra is the cool, laid-back friend who gets things done without all the ceremony. It's a DSL (Domain Specific Language) for quickly creating web applications in Ruby with minimal effort. If Rails is a Swiss Army knife, Sinatra is a sharp blade—simple, elegant, and perfect for what it does.

## What Is Sinatra?

Sinatra is a web framework that lets you create web applications with just a few lines of code. No complex setup, no directory structures to memorize, no "convention over configuration" to learn. Just Ruby code that reads like what it does:

```ruby
# app.rb - A complete web application!
require 'sinatra'

get '/' do
  "Hello, World!"
end

get '/about' do
  "This is a Sinatra app!"
end

# Run with: ruby app.rb
# Visit: http://localhost:4567
```

That's it. That's a working web application. No scaffolding, no generators, no configuration files. Just Ruby.

## Installing Sinatra

```bash
# Install Sinatra
gem install sinatra

# For auto-reloading during development
gem install sinatra-contrib

# A better server
gem install puma
```

Or with a Gemfile:

```ruby
# Gemfile
source 'https://rubygems.org'

gem 'sinatra'
gem 'sinatra-contrib'
gem 'puma'
```

## Your First Sinatra App

Let's build something slightly more interesting:

```ruby
# hello_sinatra.rb
require 'sinatra'
require 'sinatra/reloader' if development?

# Configure Sinatra
set :port, 3000
set :bind, '0.0.0.0'

# In-memory data store (for now)
$messages = []

# Routes
get '/' do
  @title = "Welcome to Sinatra!"
  erb :index
end

get '/hello/:name' do
  "Hello, #{params['name']}!"
end

get '/time' do
  "The current time is #{Time.now}"
end

post '/messages' do
  $messages << params['message']
  redirect '/messages'
end

get '/messages' do
  @messages = $messages
  erb :messages
end

# 404 handler
not_found do
  "404 - This page doesn't exist!"
end

__END__

@@ layout
<!DOCTYPE html>
<html>
<head>
  <title><%= @title || "Sinatra App" %></title>
  <style>
    body { font-family: Arial, sans-serif; margin: 40px; }
    h1 { color: #333; }
  </style>
</head>
<body>
  <%= yield %>
</body>
</html>

@@ index
<h1><%= @title %></h1>
<p>This is your first Sinatra application!</p>
<ul>
  <li><a href="/hello/Ruby">Say hello</a></li>
  <li><a href="/time">Check the time</a></li>
  <li><a href="/messages">View messages</a></li>
</ul>

@@ messages
<h1>Messages</h1>
<form action="/messages" method="post">
  <input type="text" name="message" placeholder="Enter a message">
  <button type="submit">Send</button>
</form>
<ul>
  <% @messages.each do |msg| %>
    <li><%= msg %></li>
  <% end %>
</ul>
```

## Routes: The Heart of Sinatra

Routes define what happens when someone visits a URL:

```ruby
# HTTP verbs as methods
get '/articles' do
  "List of articles"
end

post '/articles' do
  "Create a new article"
end

put '/articles/:id' do
  "Update article #{params['id']}"
end

delete '/articles/:id' do
  "Delete article #{params['id']}"
end

patch '/articles/:id' do
  "Partially update article #{params['id']}"
end

# Route patterns
get '/users/:id' do
  "User #{params['id']}"
end

get '/posts/:year/:month/:day/:slug' do
  "Post from #{params['year']}/#{params['month']}/#{params['day']}: #{params['slug']}"
end

# Wildcards and regular expressions
get '/files/*' do
  "You requested: #{params['splat'][0]}"
end

get %r{/users/(\d+)} do
  "User with numeric ID: #{params['captures'][0]}"
end

# Optional parameters
get '/articles/:id.?:format?' do
  if params['format']
    "Article #{params['id']} in #{params['format']} format"
  else
    "Article #{params['id']} in HTML format"
  end
end
```

## Request and Response Objects

Sinatra gives you access to the request and response:

```ruby
get '/info' do
  # Request object
  "Method: #{request.request_method}<br>" +
  "Path: #{request.path}<br>" +
  "Query String: #{request.query_string}<br>" +
  "User Agent: #{request.user_agent}<br>" +
  "IP: #{request.ip}<br>" +
  "Referrer: #{request.referrer}"
end

get '/headers' do
  # Accessing headers
  auth = request.env['HTTP_AUTHORIZATION']
  content_type = request.content_type
  
  "Auth: #{auth}, Content-Type: #{content_type}"
end

post '/upload' do
  # Accessing POST data
  username = params['username']
  password = params['password']
  file = params['file']  # Uploaded file
  
  "Received: #{username}, File: #{file[:filename] if file}"
end

get '/custom-response' do
  # Modifying response
  status 201
  headers 'X-Custom-Header' => 'Custom Value'
  body 'Created successfully!'
end

get '/json' do
  content_type :json
  { message: 'Hello', time: Time.now }.to_json
end
```

## Sessions and Cookies

Managing user state:

```ruby
require 'sinatra'

# Enable sessions
enable :sessions
set :session_secret, 'super secret key here'

get '/login' do
  erb :login
end

post '/login' do
  if params['username'] == 'admin' && params['password'] == 'secret'
    session[:user] = params['username']
    redirect '/dashboard'
  else
    @error = "Invalid credentials"
    erb :login
  end
end

get '/dashboard' do
  redirect '/login' unless session[:user]
  
  "Welcome, #{session[:user]}!"
end

get '/logout' do
  session.clear
  redirect '/login'
end

# Cookies
get '/set-cookie' do
  response.set_cookie('user_preference', 
    value: 'dark_mode',
    expires: Time.now + 3600 * 24 * 30,  # 30 days
    httponly: true
  )
  "Cookie set!"
end

get '/read-cookie' do
  preference = request.cookies['user_preference']
  "Your preference: #{preference}"
end
```

## Filters: Before and After Hooks

Run code before or after requests:

```ruby
# Run before every request
before do
  @start_time = Time.now
  puts "Request starting: #{request.path}"
end

# Run after every request
after do
  puts "Request took: #{Time.now - @start_time} seconds"
end

# Filters for specific routes
before '/admin/*' do
  redirect '/login' unless session[:admin]
end

after '/api/*' do
  headers 'Access-Control-Allow-Origin' => '*'
end

# Conditional filters
before agent: /Googlebot/ do
  halt 403, "No bots allowed!"
end
```

## Static Files and Public Directory

Sinatra automatically serves static files from the `public` directory:

```ruby
# Directory structure:
# myapp/
#   app.rb
#   public/
#     css/
#       style.css
#     js/
#       app.js
#     images/
#       logo.png

# In app.rb
get '/' do
  erb :home
end

# In views/home.erb:
# <link rel="stylesheet" href="/css/style.css">
# <script src="/js/app.js"></script>
# <img src="/images/logo.png">

# Or configure a different directory
set :public_folder, File.dirname(__FILE__) + '/static'
```

## Error Handling

Gracefully handle errors:

```ruby
# Custom error pages
error 404 do
  erb :not_found
end

error 500 do
  erb :server_error
end

# Handle specific exceptions
error MyCustomError do
  'Sorry, this broke: ' + env['sinatra.error'].message
end

# Development vs Production
configure :development do
  enable :logging
  enable :dump_errors
  enable :show_exceptions
end

configure :production do
  disable :logging
  disable :dump_errors
  disable :show_exceptions
end

# Halting execution
get '/restricted' do
  halt 401, {'Content-Type' => 'text/plain'}, 'Unauthorized'
end

get '/redirect-me' do
  redirect '/new-location'
end

get '/permanent-redirect' do
  redirect '/new-location', 301
end
```

## Modular Applications

For larger apps, use the modular style:

```ruby
# app.rb
require 'sinatra/base'

class MyApp < Sinatra::Base
  configure do
    set :port, 3000
    enable :sessions
  end
  
  helpers do
    def current_user
      @current_user ||= User.find(session[:user_id]) if session[:user_id]
    end
    
    def logged_in?
      !!current_user
    end
  end
  
  get '/' do
    @posts = Post.recent
    erb :index
  end
  
  get '/posts/:id' do
    @post = Post.find(params['id'])
    erb :show
  end
  
  # Run the app
  run! if app_file == $0
end
```

## Practical Example: A Simple Blog

Let's build a complete blog application:

```ruby
# blog.rb
require 'sinatra'
require 'sinatra/reloader' if development?
require 'json'
require 'time'

# Simple file-based storage
class Post
  attr_accessor :id, :title, :content, :author, :created_at
  
  DATA_FILE = 'posts.json'
  
  def initialize(attributes = {})
    @id = attributes['id'] || Time.now.to_i
    @title = attributes['title']
    @content = attributes['content']
    @author = attributes['author']
    @created_at = attributes['created_at'] || Time.now
  end
  
  def to_h
    {
      id: @id,
      title: @title,
      content: @content,
      author: @author,
      created_at: @created_at
    }
  end
  
  def save
    posts = self.class.all
    if existing = posts.find { |p| p.id == @id }
      posts[posts.index(existing)] = self
    else
      posts << self
    end
    self.class.save_all(posts)
    self
  end
  
  def destroy
    posts = self.class.all
    posts.delete_if { |p| p.id == @id }
    self.class.save_all(posts)
  end
  
  def self.all
    return [] unless File.exist?(DATA_FILE)
    
    JSON.parse(File.read(DATA_FILE)).map do |data|
      Post.new(data)
    end
  rescue
    []
  end
  
  def self.find(id)
    all.find { |p| p.id == id.to_i }
  end
  
  def self.save_all(posts)
    File.write(DATA_FILE, JSON.pretty_generate(posts.map(&:to_h)))
  end
end

# Configuration
set :port, 3000
enable :sessions
set :session_secret, 'my_blog_secret_key_change_this'

# Helpers
helpers do
  def logged_in?
    session[:username]
  end
  
  def require_login
    redirect '/login' unless logged_in?
  end
  
  def h(text)
    Rack::Utils.escape_html(text)
  end
  
  def markdown(text)
    # Simple markdown-ish formatting
    text.gsub(/\*\*(.*?)\*\*/, '<strong>\1</strong>')
        .gsub(/\*(.*?)\*/, '<em>\1</em>')
        .gsub(/\n/, '<br>')
  end
end

# Routes
get '/' do
  @posts = Post.all.sort_by(&:created_at).reverse
  erb :index
end

get '/posts/new' do
  require_login
  erb :new_post
end

post '/posts' do
  require_login
  
  post = Post.new(
    'title' => params['title'],
    'content' => params['content'],
    'author' => session[:username]
  )
  post.save
  
  redirect "/posts/#{post.id}"
end

get '/posts/:id' do
  @post = Post.find(params['id'])
  halt 404, "Post not found" unless @post
  erb :show_post
end

get '/posts/:id/edit' do
  require_login
  @post = Post.find(params['id'])
  halt 404, "Post not found" unless @post
  erb :edit_post
end

put '/posts/:id' do
  require_login
  
  post = Post.find(params['id'])
  halt 404, "Post not found" unless post
  
  post.title = params['title']
  post.content = params['content']
  post.save
  
  redirect "/posts/#{post.id}"
end

delete '/posts/:id' do
  require_login
  
  post = Post.find(params['id'])
  halt 404, "Post not found" unless post
  
  post.destroy
  redirect '/'
end

get '/login' do
  erb :login
end

post '/login' do
  # Simple authentication (in real app, use proper auth!)
  if params['username'] && params['password'] == 'secret'
    session[:username] = params['username']
    redirect '/'
  else
    @error = "Invalid credentials"
    erb :login
  end
end

get '/logout' do
  session.clear
  redirect '/'
end

# Views
__END__

@@ layout
<!DOCTYPE html>
<html>
<head>
  <title>My Ruby Blog</title>
  <style>
    body {
      font-family: Georgia, serif;
      max-width: 800px;
      margin: 0 auto;
      padding: 20px;
      line-height: 1.6;
    }
    header {
      border-bottom: 2px solid #333;
      margin-bottom: 30px;
      padding-bottom: 10px;
    }
    nav a {
      margin-right: 20px;
      text-decoration: none;
      color: #333;
    }
    nav a:hover {
      text-decoration: underline;
    }
    .post {
      margin-bottom: 40px;
      padding-bottom: 20px;
      border-bottom: 1px solid #eee;
    }
    .meta {
      color: #666;
      font-size: 0.9em;
    }
    .button {
      background: #333;
      color: white;
      padding: 10px 20px;
      text-decoration: none;
      border: none;
      cursor: pointer;
    }
    .button:hover {
      background: #555;
    }
    input, textarea {
      width: 100%;
      padding: 10px;
      margin-bottom: 10px;
      border: 1px solid #ddd;
    }
    textarea {
      height: 200px;
      font-family: monospace;
    }
    .error {
      color: red;
      margin: 10px 0;
    }
  </style>
</head>
<body>
  <header>
    <h1>My Ruby Blog</h1>
    <nav>
      <a href="/">Home</a>
      <% if logged_in? %>
        <a href="/posts/new">New Post</a>
        <a href="/logout">Logout (<%= session[:username] %>)</a>
      <% else %>
        <a href="/login">Login</a>
      <% end %>
    </nav>
  </header>
  
  <%= yield %>
</body>
</html>

@@ index
<h2>Recent Posts</h2>
<% if @posts.empty? %>
  <p>No posts yet. <a href="/posts/new">Write the first one!</a></p>
<% else %>
  <% @posts.each do |post| %>
    <div class="post">
      <h3><a href="/posts/<%= post.id %>"><%= h post.title %></a></h3>
      <p class="meta">
        by <%= h post.author %> on <%= post.created_at.strftime('%B %d, %Y') %>
      </p>
      <p><%= markdown(h(post.content[0..200])) %>...</p>
      <a href="/posts/<%= post.id %>">Read more →</a>
    </div>
  <% end %>
<% end %>

@@ show_post
<article>
  <h2><%= h @post.title %></h2>
  <p class="meta">
    by <%= h @post.author %> on <%= @post.created_at.strftime('%B %d, %Y at %I:%M %p') %>
  </p>
  <div class="content">
    <%= markdown(h(@post.content)) %>
  </div>
  
  <% if logged_in? %>
    <div style="margin-top: 30px;">
      <a href="/posts/<%= @post.id %>/edit" class="button">Edit</a>
      <form method="post" action="/posts/<%= @post.id %>" style="display: inline;">
        <input type="hidden" name="_method" value="delete">
        <button type="submit" class="button" onclick="return confirm('Are you sure?')">Delete</button>
      </form>
    </div>
  <% end %>
</article>

@@ new_post
<h2>New Post</h2>
<form method="post" action="/posts">
  <input type="text" name="title" placeholder="Post title" required>
  <textarea name="content" placeholder="Write your post here..." required></textarea>
  <button type="submit" class="button">Publish</button>
</form>

@@ edit_post
<h2>Edit Post</h2>
<form method="post" action="/posts/<%= @post.id %>">
  <input type="hidden" name="_method" value="put">
  <input type="text" name="title" value="<%= h @post.title %>" required>
  <textarea name="content" required><%= h @post.content %></textarea>
  <button type="submit" class="button">Update</button>
</form>

@@ login
<h2>Login</h2>
<% if @error %>
  <p class="error"><%= @error %></p>
<% end %>
<form method="post" action="/login">
  <input type="text" name="username" placeholder="Username" required>
  <input type="password" name="password" placeholder="Password" required>
  <button type="submit" class="button">Login</button>
</form>
<p><em>Hint: Any username works with password "secret"</em></p>
```

## Sinatra vs Rails

**Use Sinatra when:**
- Building APIs or microservices
- Creating simple web applications
- Learning web development
- Prototyping quickly
- You want full control

**Use Rails when:**
- Building large, complex applications
- You need lots of built-in features
- Working with a team
- Following conventions is important
- You need an ORM, mailers, etc.

## Your Turn: Sinatra Challenges

1. **API Service**: Build a JSON API for a todo list
2. **URL Shortener**: Create a link shortening service
3. **Chat App**: Build a real-time chat with WebSockets
4. **File Upload**: Create an image gallery with uploads
5. **Authentication**: Add proper user authentication to the blog

## What You've Learned

You now know how to:
- Create web applications with Sinatra
- Define routes and handle requests
- Work with templates and views
- Manage sessions and cookies
- Build complete web applications
- Choose between Sinatra and Rails

## What's Next?

In the next chapters, we'll dive deeper into web development with Sinatra, covering templates, databases, and building a complete application. You'll see how Sinatra's simplicity makes it perfect for learning web development concepts without getting lost in framework magic.

Remember: Sinatra is about simplicity and clarity. It doesn't hide what's happening—every line of code has a clear purpose. This transparency makes it perfect for learning web development and for building applications where you want full control. Sometimes, less is more!