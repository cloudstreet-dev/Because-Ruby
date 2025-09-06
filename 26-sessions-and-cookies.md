# Chapter 26: Sessions and Cookies - Giving Your App a Memory

HTTP is stateless - it's like having a conversation with someone who has amnesia and forgets you after every sentence. Sessions and cookies are how we give web applications memory, allowing them to remember users between requests. Let's learn how to make your app remember things!

## Understanding Cookies

Cookies are small pieces of data stored in the user's browser:

```ruby
require 'sinatra'

# Setting a simple cookie
get '/set-cookie' do
  response.set_cookie('username', 'alice')
  "Cookie set!"
end

# Reading a cookie
get '/get-cookie' do
  username = request.cookies['username']
  "Hello, #{username || 'stranger'}!"
end

# Setting cookie with options
get '/secure-cookie' do
  response.set_cookie('session_id', {
    value: 'abc123',
    expires: Time.now + 3600,  # Expires in 1 hour
    domain: '.example.com',     # Available to subdomains
    path: '/',                   # Available site-wide
    secure: true,                # HTTPS only
    httponly: true              # Not accessible via JavaScript
  })
  "Secure cookie set!"
end

# Deleting a cookie
get '/logout' do
  response.delete_cookie('username')
  "Cookie deleted!"
end
```

## Sessions in Sinatra

Sessions provide server-side storage with a cookie-based identifier:

```ruby
require 'sinatra'

# Enable sessions
enable :sessions
set :session_secret, ENV['SESSION_SECRET'] || 'super_secret_key_change_in_production'

# Using sessions
get '/' do
  session[:visit_count] ||= 0
  session[:visit_count] += 1
  "You've visited this page #{session[:visit_count]} times"
end

# Store user data in session
post '/login' do
  user = User.authenticate(params[:username], params[:password])
  
  if user
    session[:user_id] = user.id
    session[:username] = user.username
    redirect '/dashboard'
  else
    @error = "Invalid credentials"
    erb :login
  end
end

# Check if user is logged in
get '/dashboard' do
  unless session[:user_id]
    redirect '/login'
  end
  
  @user = User.find(session[:user_id])
  erb :dashboard
end

# Clear session on logout
get '/logout' do
  session.clear
  redirect '/'
end
```

## Advanced Session Configuration

```ruby
require 'sinatra'
require 'sinatra/cookies'

class MyApp < Sinatra::Base
  # Configure sessions
  use Rack::Session::Cookie, {
    key: 'my_app_session',
    domain: '.example.com',
    path: '/',
    expire_after: 2592000,  # 30 days in seconds
    secret: ENV['SESSION_SECRET'],
    old_secret: ENV['OLD_SESSION_SECRET'],  # For rotation
    secure: production?,      # HTTPS only in production
    httponly: true,          # Prevent JS access
    same_site: :lax          # CSRF protection
  }
  
  helpers do
    def logged_in?
      !session[:user_id].nil?
    end
    
    def current_user
      @current_user ||= User.find(session[:user_id]) if logged_in?
    end
    
    def require_login!
      unless logged_in?
        session[:return_to] = request.path
        redirect '/login'
      end
    end
  end
  
  get '/protected' do
    require_login!
    "Secret content for #{current_user.name}"
  end
  
  post '/login' do
    # After successful login
    user = User.authenticate(params[:username], params[:password])
    
    if user
      session[:user_id] = user.id
      
      # Redirect to where they were trying to go
      redirect_to = session.delete(:return_to) || '/dashboard'
      redirect redirect_to
    else
      erb :login
    end
  end
end
```

## Flash Messages

Flash messages show temporary notifications:

```ruby
helpers do
  def flash(message = nil)
    if message
      session[:flash] = message
    else
      message = session[:flash]
      session.delete(:flash)
      message
    end
  end
  
  # More sophisticated flash with types
  def set_flash(type, message)
    session[:flash] = { type: type, message: message }
  end
  
  def get_flash
    return nil unless session[:flash]
    
    flash = session[:flash]
    session.delete(:flash)
    flash
  end
end

post '/contact' do
  # Process form...
  
  if save_message(params)
    set_flash(:success, "Your message has been sent!")
    redirect '/'
  else
    set_flash(:error, "Failed to send message. Please try again.")
    redirect '/contact'
  end
end

# In your layout template
```

```erb
<!-- views/layout.erb -->
<% if flash_data = get_flash %>
  <div class="flash <%= flash_data[:type] %>">
    <%= flash_data[:message] %>
  </div>
<% end %>
```

## Shopping Cart Example

A practical session example:

```ruby
class ShoppingCart
  def initialize(session)
    @session = session
    @session[:cart] ||= []
  end
  
  def add_item(product_id, quantity = 1)
    item = find_item(product_id)
    
    if item
      item['quantity'] += quantity
    else
      @session[:cart] << {
        'product_id' => product_id,
        'quantity' => quantity
      }
    end
  end
  
  def remove_item(product_id)
    @session[:cart].delete_if { |item| item['product_id'] == product_id }
  end
  
  def update_quantity(product_id, quantity)
    item = find_item(product_id)
    if item
      if quantity <= 0
        remove_item(product_id)
      else
        item['quantity'] = quantity
      end
    end
  end
  
  def items
    @session[:cart].map do |item|
      product = Product.find(item['product_id'])
      {
        product: product,
        quantity: item['quantity'],
        subtotal: product.price * item['quantity']
      }
    end
  end
  
  def total
    items.sum { |item| item[:subtotal] }
  end
  
  def count
    @session[:cart].sum { |item| item['quantity'] }
  end
  
  def clear
    @session[:cart] = []
  end
  
  private
  
  def find_item(product_id)
    @session[:cart].find { |item| item['product_id'] == product_id }
  end
end

# Using the shopping cart
get '/cart' do
  @cart = ShoppingCart.new(session)
  erb :cart
end

post '/cart/add' do
  cart = ShoppingCart.new(session)
  cart.add_item(params[:product_id].to_i, params[:quantity].to_i)
  
  flash("Item added to cart!")
  redirect back
end

post '/cart/update' do
  cart = ShoppingCart.new(session)
  cart.update_quantity(params[:product_id].to_i, params[:quantity].to_i)
  
  redirect '/cart'
end

post '/cart/remove' do
  cart = ShoppingCart.new(session)
  cart.remove_item(params[:product_id].to_i)
  
  flash("Item removed from cart")
  redirect '/cart'
end
```

## Remember Me Functionality

Implement "Remember Me" with persistent cookies:

```ruby
require 'securerandom'
require 'digest'

class User < ActiveRecord::Base
  def generate_remember_token
    token = SecureRandom.urlsafe_base64
    self.update(
      remember_token: Digest::SHA256.hexdigest(token),
      remember_created_at: Time.now
    )
    token
  end
  
  def self.find_by_remember_token(token)
    return nil if token.nil?
    
    hashed_token = Digest::SHA256.hexdigest(token)
    user = find_by(remember_token: hashed_token)
    
    # Check if token is still valid (30 days)
    if user && user.remember_created_at > 30.days.ago
      user
    else
      nil
    end
  end
  
  def forget_me
    update(remember_token: nil, remember_created_at: nil)
  end
end

post '/login' do
  user = User.authenticate(params[:username], params[:password])
  
  if user
    session[:user_id] = user.id
    
    # Handle "Remember Me"
    if params[:remember_me]
      token = user.generate_remember_token
      
      response.set_cookie('remember_token', {
        value: token,
        expires: Time.now + (30 * 24 * 3600),  # 30 days
        httponly: true,
        secure: production?
      })
    end
    
    redirect '/dashboard'
  else
    erb :login
  end
end

# Auto-login with remember token
before do
  if !logged_in? && request.cookies['remember_token']
    user = User.find_by_remember_token(request.cookies['remember_token'])
    
    if user
      session[:user_id] = user.id
      @current_user = user
    else
      # Invalid or expired token
      response.delete_cookie('remember_token')
    end
  end
end

get '/logout' do
  if current_user
    current_user.forget_me
    response.delete_cookie('remember_token')
  end
  
  session.clear
  redirect '/'
end
```

## Session Storage Alternatives

### Redis Sessions

```ruby
require 'sinatra'
require 'redis'
require 'rack/session/redis'

class App < Sinatra::Base
  use Rack::Session::Redis, {
    redis_server: ENV['REDIS_URL'] || 'redis://localhost:6379',
    expire_after: 86400,  # 1 day
    key: 'myapp.session',
    threadsafe: true
  }
  
  get '/' do
    session[:visits] ||= 0
    session[:visits] += 1
    "Visits: #{session[:visits]}"
  end
end
```

### Database Sessions

```ruby
require 'sinatra'
require 'active_record'

class SessionStore < ActiveRecord::Base
  def self.find_by_session_id(session_id)
    where(session_id: session_id).first
  end
  
  def self.create_or_update(session_id, data, expires_at = nil)
    session = find_by_session_id(session_id) || new(session_id: session_id)
    session.data = data.to_json
    session.expires_at = expires_at || 1.day.from_now
    session.save
  end
  
  def self.cleanup_expired
    where('expires_at < ?', Time.now).destroy_all
  end
end

class DatabaseSession
  def initialize(app)
    @app = app
  end
  
  def call(env)
    # Load session from database
    request = Rack::Request.new(env)
    session_id = request.cookies['session_id'] || SecureRandom.hex(16)
    
    session_record = SessionStore.find_by_session_id(session_id)
    env['rack.session'] = session_record ? JSON.parse(session_record.data) : {}
    
    status, headers, body = @app.call(env)
    
    # Save session to database
    SessionStore.create_or_update(session_id, env['rack.session'])
    
    # Set cookie
    Rack::Utils.set_cookie_header!(headers, 'session_id', {
      value: session_id,
      path: '/',
      httponly: true
    })
    
    [status, headers, body]
  end
end

use DatabaseSession
```

## User Preferences

Store user preferences in cookies or sessions:

```ruby
class PreferenceManager
  def initialize(request, response)
    @request = request
    @response = response
  end
  
  def get(key, default = nil)
    prefs = load_preferences
    prefs[key] || default
  end
  
  def set(key, value)
    prefs = load_preferences
    prefs[key] = value
    save_preferences(prefs)
  end
  
  def all
    load_preferences
  end
  
  private
  
  def load_preferences
    json = @request.cookies['preferences']
    json ? JSON.parse(json) : {}
  rescue JSON::ParserError
    {}
  end
  
  def save_preferences(prefs)
    @response.set_cookie('preferences', {
      value: prefs.to_json,
      expires: Time.now + (365 * 24 * 3600),  # 1 year
      httponly: true
    })
  end
end

helpers do
  def preferences
    @preferences ||= PreferenceManager.new(request, response)
  end
end

# Using preferences
get '/settings' do
  @theme = preferences.get('theme', 'light')
  @language = preferences.get('language', 'en')
  erb :settings
end

post '/settings' do
  preferences.set('theme', params[:theme])
  preferences.set('language', params[:language])
  preferences.set('timezone', params[:timezone])
  
  flash("Settings saved!")
  redirect '/settings'
end
```

## Security Best Practices

```ruby
class SecureApp < Sinatra::Base
  # 1. Use strong session secrets
  set :session_secret, ENV.fetch('SESSION_SECRET') { 
    raise "SESSION_SECRET environment variable must be set!" 
  }
  
  # 2. Rotate session IDs on login
  post '/login' do
    user = authenticate(params)
    
    if user
      # Clear old session
      session.clear
      
      # Generate new session ID (implementation depends on session store)
      env['rack.session.options'][:renew] = true
      
      # Set user data
      session[:user_id] = user.id
      session[:logged_in_at] = Time.now
      
      redirect '/dashboard'
    end
  end
  
  # 3. Implement session timeout
  before do
    if session[:logged_in_at]
      if Time.now - session[:logged_in_at] > 3600  # 1 hour timeout
        session.clear
        flash("Session expired. Please log in again.")
        redirect '/login'
      else
        session[:logged_in_at] = Time.now  # Update activity time
      end
    end
  end
  
  # 4. Validate session data
  helpers do
    def current_user
      return nil unless session[:user_id]
      
      # Validate user still exists and is active
      user = User.find_by(id: session[:user_id], active: true)
      
      unless user
        session.clear
        nil
      else
        user
      end
    end
  end
  
  # 5. Use secure cookies in production
  configure :production do
    use Rack::Session::Cookie, {
      secret: ENV['SESSION_SECRET'],
      secure: true,        # HTTPS only
      httponly: true,      # No JS access
      same_site: :strict   # CSRF protection
    }
  end
  
  # 6. Implement CSRF protection
  before do
    if request.post? || request.put? || request.delete?
      unless valid_csrf_token?
        halt 403, "Invalid CSRF token"
      end
    end
  end
  
  helpers do
    def csrf_token
      session[:csrf] ||= SecureRandom.hex(32)
    end
    
    def valid_csrf_token?
      params[:csrf_token] == session[:csrf]
    end
  end
end
```

## Testing Sessions and Cookies

```ruby
require 'minitest/autorun'
require 'rack/test'

class SessionTest < Minitest::Test
  include Rack::Test::Methods
  
  def app
    Sinatra::Application
  end
  
  def test_session_persistence
    get '/'
    assert last_response.ok?
    
    # Session should persist across requests
    get '/'
    assert_includes last_response.body, "2 times"
  end
  
  def test_login_sets_session
    post '/login', username: 'alice', password: 'secret'
    assert last_response.redirect?
    
    follow_redirect!
    assert_includes last_response.body, 'alice'
  end
  
  def test_logout_clears_session
    # Login first
    post '/login', username: 'alice', password: 'secret'
    
    # Then logout
    get '/logout'
    follow_redirect!
    
    # Should not be logged in
    get '/dashboard'
    assert last_response.redirect?
    assert_equal 'http://example.org/login', last_response.location
  end
  
  def test_cookie_setting
    get '/set-cookie'
    assert last_response.ok?
    
    # Check cookie was set
    cookie = last_response.cookies['username']
    assert_equal 'alice', cookie
  end
end
```

## Your Turn: Session Challenges

1. **Multi-Factor Auth**: Implement 2FA with session management
2. **Activity Tracker**: Track user activity across sessions
3. **A/B Testing**: Use cookies for feature flags
4. **Session Analytics**: Monitor session duration and patterns
5. **Guest Checkout**: Implement guest sessions that convert to user accounts

## What You've Learned

You now know how to:
- Set and read cookies
- Configure and use sessions
- Implement flash messages
- Build shopping carts
- Create "Remember Me" functionality
- Use alternative session stores
- Manage user preferences
- Secure sessions and cookies
- Test session functionality

## What's Next?

You've now learned all the fundamentals of web development with Sinatra! Next, we'll put it all together and build a complete web application from scratch. Get ready to combine everything you've learned into something awesome!