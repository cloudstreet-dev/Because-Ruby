# Chapter 23: Routes and Requests - Traffic Control for Your Web App

Routes are like a restaurant menu - they tell incoming requests where to go to get what they want. "You want the homepage? Right this way! Looking for user profiles? Second door on the left!" Let's learn how Sinatra handles the traffic flow of the web.

## Basic Routing

Every route in Sinatra has three parts: an HTTP method, a path, and a block:

```ruby
require 'sinatra'

# Method + Path + Block = Route
get '/' do
  "Welcome to the homepage!"
end

post '/users' do
  "Creating a new user..."
end

put '/users/123' do
  "Updating user 123..."
end

delete '/users/123' do
  "Deleting user 123..."
end

# You can also use patch, options, link, unlink
patch '/users/123' do
  "Partially updating user 123..."
end
```

## Route Patterns

Sinatra supports several pattern types:

```ruby
# Static routes - exact match
get '/about' do
  "About page"
end

# Named parameters - capture values
get '/users/:id' do
  "User ID: #{params[:id]}"
end

# Splat parameters - match anything
get '/files/*' do
  "You requested: #{params[:splat].first}"
end

# Multiple parameters
get '/posts/:year/:month/:day/:slug' do
  "Post from #{params[:year]}/#{params[:month]}/#{params[:day]}: #{params[:slug]}"
end

# Optional parameters with splat
get '/download/*.*' do
  # /download/path/to/file.txt
  # params[:splat] => ["path/to/file", "txt"]
  "File: #{params[:splat][0]}.#{params[:splat][1]}"
end
```

## Advanced Pattern Matching

```ruby
# Regular expression routes
get %r{/users/(\d+)} do
  "User with numeric ID: #{params[:captures].first}"
end

get %r{/posts/(\d{4})/(\d{2})/(\d{2})} do
  year, month, day = params[:captures]
  "Posts from #{year}-#{month}-#{day}"
end

# Named captures in regex
get %r{/products/(?<category>\w+)/(?<id>\d+)} do
  "Category: #{params[:category]}, ID: #{params[:id]}"
end

# Constraints on named parameters
get '/users/:id', constraints: { id: /\d+/ } do
  "User with numeric ID: #{params[:id]}"
end
```

## The Request Object

Sinatra provides a `request` object with tons of useful information:

```ruby
get '/inspect' do
  content_type :json
  
  {
    # Request line
    method: request.request_method,
    path: request.path,
    url: request.url,
    
    # Parameters
    params: params,
    query_string: request.query_string,
    
    # Headers
    user_agent: request.user_agent,
    accept: request.accept,
    content_type: request.content_type,
    
    # Network info
    ip: request.ip,
    port: request.port,
    scheme: request.scheme,  # http or https
    
    # Checks
    xhr?: request.xhr?,  # Is it AJAX?
    secure?: request.secure?,  # Is it HTTPS?
    forwarded?: request.forwarded?,  # Behind proxy?
    
    # Body
    body: request.body.read
  }.to_json
end
```

## Working with Parameters

Parameters can come from multiple sources:

```ruby
# Query parameters: /search?q=ruby&limit=10
get '/search' do
  query = params[:q]
  limit = params[:limit] || 10
  "Searching for '#{query}' with limit #{limit}"
end

# Route parameters: /users/42
get '/users/:id' do
  user_id = params[:id]
  "Showing user #{user_id}"
end

# Form parameters (POST body)
post '/login' do
  username = params[:username]
  password = params[:password]
  "Logging in #{username}..."
end

# JSON body parameters
post '/api/users' do
  request.body.rewind
  data = JSON.parse(request.body.read)
  "Creating user: #{data['name']}"
end

# File uploads
post '/upload' do
  if params[:file]
    filename = params[:file][:filename]
    file = params[:file][:tempfile]
    
    File.open("./uploads/#{filename}", 'wb') do |f|
      f.write(file.read)
    end
    
    "Uploaded #{filename}"
  else
    "No file uploaded"
  end
end
```

## Route Conditions

Add conditions to routes:

```ruby
# Host-based routing
get '/', host: 'admin.example.com' do
  "Admin interface"
end

get '/', host: 'api.example.com' do
  "API endpoint"
end

# Custom conditions
set :valid_api_key do |value|
  condition do
    request.env['HTTP_X_API_KEY'] == value
  end
end

get '/api/secret', valid_api_key: 'secret123' do
  "Secret data!"
end

# User agent conditions
get '/mobile', agent: /Mobile|Android|iPhone/ do
  "Mobile version"
end

get '/desktop' do
  "Desktop version"
end

# Method-based conditions
def authenticated?
  !session[:user_id].nil?
end

get '/profile', :auth => :authenticated? do
  "User profile"
end
```

## Before and After Filters

Run code before or after routes:

```ruby
# Run before every route
before do
  @start_time = Time.now
  puts "Request: #{request.path}"
end

# Run after every route
after do
  puts "Response time: #{Time.now - @start_time}s"
end

# Filters with patterns
before '/admin/*' do
  unless session[:admin]
    halt 403, "Admin access required"
  end
end

after '/api/*' do
  headers['X-API-Version'] = '1.0'
end

# Multiple filters
before do
  headers['X-Frame-Options'] = 'DENY'
end

before '/secure/*' do
  require_login!
end

# Conditional filters
before agent: /Googlebot/ do
  @bot_visit = true
end
```

## Route Helpers

Create reusable route logic:

```ruby
helpers do
  def require_login!
    unless session[:user_id]
      redirect '/login'
    end
  end
  
  def current_user
    @current_user ||= User.find(session[:user_id]) if session[:user_id]
  end
  
  def json_response(data)
    content_type :json
    data.to_json
  end
end

get '/dashboard' do
  require_login!
  "Welcome, #{current_user.name}!"
end

get '/api/users' do
  users = User.all
  json_response(users)
end
```

## Pass and Halt

Control route flow:

```ruby
# Pass to the next matching route
get '/guess/:who' do
  pass unless params[:who] == 'Frank'
  "You got me!"
end

get '/guess/*' do
  "You missed!"
end

# Halt stops processing immediately
get '/secret' do
  halt 401 unless authorized?
  "Secret content"
end

# Halt with custom response
get '/premium' do
  halt 402, {'Content-Type' => 'text/plain'}, 'Payment Required'
end

# Halt with status and body
get '/maintenance' do
  halt 503, '<h1>Under Maintenance</h1>'
end
```

## Streaming Responses

Send data as it's generated:

```ruby
get '/stream' do
  stream do |out|
    out << "Starting...\n"
    sleep 1
    out << "Processing...\n"
    sleep 1
    out << "Almost done...\n"
    sleep 1
    out << "Complete!\n"
  end
end

# Server-sent events
get '/events', provides: 'text/event-stream' do
  stream :keep_open do |out|
    EventMachine::PeriodicTimer.new(1) do
      out << "data: #{Time.now}\n\n"
    end
  end
end

# Streaming large files
get '/download/:file' do
  stream do |out|
    File.open("files/#{params[:file]}", 'rb') do |file|
      while chunk = file.read(1024)
        out << chunk
      end
    end
  end
end
```

## Redirects

Send users elsewhere:

```ruby
# Simple redirect
get '/old-page' do
  redirect '/new-page'
end

# Permanent redirect
get '/old-blog' do
  redirect '/blog', 301
end

# Redirect back
post '/login' do
  # Process login...
  redirect back  # Go back where they came from
end

# Redirect with flash message (requires sessions)
post '/contact' do
  # Process form...
  session[:flash] = "Message sent!"
  redirect '/'
end

# Conditional redirect
get '/profile' do
  redirect '/login' unless logged_in?
  # Show profile...
end
```

## Error Handling

Handle errors gracefully:

```ruby
# 404 Not Found
not_found do
  '<h1>404</h1><p>Page not found!</p>'
end

# 500 Server Error
error do
  '<h1>500</h1><p>Something went wrong!</p>'
end

# Specific error types
error 403 do
  '<h1>403</h1><p>Access forbidden!</p>'
end

# Handle specific exceptions
error MyCustomError do
  'Something specific went wrong: ' + env['sinatra.error'].message
end

# Development vs Production
configure :development do
  error do
    "<h1>Error</h1><pre>#{env['sinatra.error'].backtrace.join("\n")}</pre>"
  end
end

configure :production do
  error do
    "We're sorry. Something went wrong."
  end
end
```

## Content Types

Handle different response formats:

```ruby
get '/data' do
  # Check what the client wants
  case request.accept.first.to_s
  when 'application/json'
    content_type :json
    {name: 'Ruby', awesome: true}.to_json
  when 'application/xml'
    content_type :xml
    '<data><name>Ruby</name><awesome>true</awesome></data>'
  else
    content_type :html
    '<h1>Ruby is awesome!</h1>'
  end
end

# Or use provides
get '/users', provides: [:html, :json, :xml] do
  users = User.all
  
  respond_to do |format|
    format.html { erb :users }
    format.json { users.to_json }
    format.xml  { users.to_xml }
  end
end

# Force content type
get '/api/users' do
  content_type :json
  User.all.to_json
end
```

## Advanced Routing Patterns

```ruby
class BlogApp < Sinatra::Base
  # RESTful routes
  get '/posts' do
    @posts = Post.all
    erb :index
  end
  
  get '/posts/new' do
    @post = Post.new
    erb :new
  end
  
  post '/posts' do
    @post = Post.create(params[:post])
    redirect "/posts/#{@post.id}"
  end
  
  get '/posts/:id' do
    @post = Post.find(params[:id])
    erb :show
  end
  
  get '/posts/:id/edit' do
    @post = Post.find(params[:id])
    erb :edit
  end
  
  put '/posts/:id' do
    @post = Post.find(params[:id])
    @post.update(params[:post])
    redirect "/posts/#{@post.id}"
  end
  
  delete '/posts/:id' do
    Post.find(params[:id]).destroy
    redirect '/posts'
  end
end

# Namespace routes with Sinatra::Namespace
class API < Sinatra::Base
  register Sinatra::Namespace
  
  namespace '/api' do
    namespace '/v1' do
      get '/users' do
        json User.all
      end
      
      get '/users/:id' do
        json User.find(params[:id])
      end
    end
    
    namespace '/v2' do
      before do
        authenticate!
      end
      
      get '/users' do
        json current_user.visible_users
      end
    end
  end
end
```

## Testing Routes

```ruby
require 'minitest/autorun'
require 'rack/test'

class AppTest < Minitest::Test
  include Rack::Test::Methods
  
  def app
    Sinatra::Application
  end
  
  def test_homepage
    get '/'
    assert last_response.ok?
    assert_includes last_response.body, 'Welcome'
  end
  
  def test_user_route
    get '/users/42'
    assert last_response.ok?
    assert_includes last_response.body, '42'
  end
  
  def test_post_route
    post '/users', name: 'Alice', email: 'alice@example.com'
    assert last_response.redirect?
    follow_redirect!
    assert_includes last_response.body, 'Alice'
  end
  
  def test_not_found
    get '/nonexistent'
    assert_equal 404, last_response.status
  end
end
```

## Your Turn: Route Challenges

1. **RESTful API**: Build a complete REST API with all CRUD operations
2. **Multi-format**: Support JSON, XML, and HTML responses
3. **Authentication**: Implement route-based authentication
4. **API Versioning**: Create versioned API routes
5. **Rate Limiting**: Add rate limiting to routes

## What You've Learned

You now know how to:
- Define routes with different HTTP methods
- Use parameters and patterns in routes
- Access request information
- Handle different types of parameters
- Use filters and helpers
- Control route flow with pass and halt
- Stream responses
- Handle errors and redirects
- Work with different content types

## What's Next?

Next, we'll explore templates and views, learning how to generate dynamic HTML and separate our presentation logic from our route handlers. Get ready to make your web apps look good!