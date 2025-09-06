# Chapter 25: Forms and User Input - Making Your App Interactive

Forms are how users talk to your web app. They're like a conversation where users fill in the blanks and your app responds. Let's learn how to handle user input safely, validate it properly, and create delightful interactive experiences!

## Basic Forms

HTML forms send data to your server:

```erb
<!-- views/signup.erb -->
<form action="/signup" method="post">
  <label for="username">Username:</label>
  <input type="text" id="username" name="username" required>
  
  <label for="email">Email:</label>
  <input type="email" id="email" name="email" required>
  
  <label for="password">Password:</label>
  <input type="password" id="password" name="password" required>
  
  <button type="submit">Sign Up</button>
</form>
```

Handle the form submission:

```ruby
require 'sinatra'

get '/signup' do
  erb :signup
end

post '/signup' do
  # Form data is in params hash
  username = params[:username]
  email = params[:email]
  password = params[:password]
  
  # Process the signup...
  "Welcome, #{username}!"
end
```

## Form Input Types

HTML5 provides many input types:

```erb
<form action="/profile" method="post">
  <!-- Text inputs -->
  <input type="text" name="name" placeholder="Your name">
  <input type="email" name="email" placeholder="email@example.com">
  <input type="password" name="password" placeholder="Password">
  <input type="tel" name="phone" placeholder="555-1234">
  <input type="url" name="website" placeholder="https://example.com">
  
  <!-- Numbers and dates -->
  <input type="number" name="age" min="1" max="120">
  <input type="date" name="birthday">
  <input type="time" name="appointment">
  <input type="datetime-local" name="meeting">
  
  <!-- Choices -->
  <input type="checkbox" name="subscribe" value="yes"> Subscribe
  
  <input type="radio" name="plan" value="free"> Free
  <input type="radio" name="plan" value="pro"> Pro
  
  <select name="country">
    <option value="us">United States</option>
    <option value="uk">United Kingdom</option>
    <option value="ca">Canada</option>
  </select>
  
  <!-- Special inputs -->
  <input type="color" name="favorite_color">
  <input type="range" name="satisfaction" min="1" max="10">
  <input type="file" name="avatar">
  
  <!-- Multi-line text -->
  <textarea name="bio" rows="5" cols="50"></textarea>
  
  <!-- Hidden fields -->
  <input type="hidden" name="referrer" value="google">
  
  <button type="submit">Save Profile</button>
</form>
```

## Form Validation

### Client-Side Validation

```erb
<form action="/register" method="post" onsubmit="return validateForm()">
  <!-- HTML5 validation attributes -->
  <input type="text" name="username" 
         required 
         minlength="3" 
         maxlength="20"
         pattern="[a-zA-Z0-9_]+"
         title="Alphanumeric and underscore only">
  
  <input type="email" name="email" required>
  
  <input type="password" name="password" 
         required 
         minlength="8"
         pattern="(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,}"
         title="Must contain at least one number, one uppercase and lowercase letter, and at least 8 characters">
  
  <input type="number" name="age" min="13" max="120" required>
  
  <button type="submit">Register</button>
</form>

<script>
function validateForm() {
  const username = document.querySelector('[name="username"]').value;
  const password = document.querySelector('[name="password"]').value;
  
  if (username === password) {
    alert("Username and password cannot be the same!");
    return false;
  }
  
  return true;
}
</script>
```

### Server-Side Validation

Never trust client-side validation alone:

```ruby
require 'sinatra'

class FormValidator
  def self.validate_email(email)
    return false if email.nil? || email.empty?
    email.match?(/\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i)
  end
  
  def self.validate_username(username)
    return false if username.nil? || username.length < 3
    username.match?(/\A[a-zA-Z0-9_]+\z/)
  end
  
  def self.validate_password(password)
    return false if password.nil? || password.length < 8
    has_lowercase = password.match?(/[a-z]/)
    has_uppercase = password.match?(/[A-Z]/)
    has_number = password.match?(/\d/)
    has_lowercase && has_uppercase && has_number
  end
end

post '/register' do
  @errors = []
  
  # Validate inputs
  unless FormValidator.validate_username(params[:username])
    @errors << "Username must be at least 3 characters, alphanumeric only"
  end
  
  unless FormValidator.validate_email(params[:email])
    @errors << "Please enter a valid email address"
  end
  
  unless FormValidator.validate_password(params[:password])
    @errors << "Password must be at least 8 characters with uppercase, lowercase, and number"
  end
  
  if params[:password] != params[:password_confirmation]
    @errors << "Passwords don't match"
  end
  
  if User.exists?(username: params[:username])
    @errors << "Username already taken"
  end
  
  if @errors.empty?
    # Create user
    User.create(
      username: params[:username],
      email: params[:email],
      password: params[:password]  # Should be hashed!
    )
    redirect '/welcome'
  else
    # Show form with errors
    erb :register
  end
end
```

## File Uploads

Handle file uploads safely:

```ruby
# HTML form must have enctype="multipart/form-data"
```

```erb
<form action="/upload" method="post" enctype="multipart/form-data">
  <label for="file">Choose file:</label>
  <input type="file" name="file" id="file" accept="image/*">
  <button type="submit">Upload</button>
</form>
```

```ruby
post '/upload' do
  # Check if file was uploaded
  unless params[:file] && params[:file][:tempfile]
    @error = "Please select a file"
    return erb :upload
  end
  
  file = params[:file]
  
  # Validate file type
  allowed_types = ['image/jpeg', 'image/png', 'image/gif']
  unless allowed_types.include?(file[:type])
    @error = "Only JPEG, PNG, and GIF files are allowed"
    return erb :upload
  end
  
  # Validate file size (5MB max)
  max_size = 5 * 1024 * 1024  # 5MB in bytes
  if file[:tempfile].size > max_size
    @error = "File too large (max 5MB)"
    return erb :upload
  end
  
  # Generate safe filename
  filename = file[:filename]
  safe_filename = filename.gsub(/[^\w\.]/, '_')
  timestamp = Time.now.to_i
  final_filename = "#{timestamp}_#{safe_filename}"
  
  # Save file
  File.open("./uploads/#{final_filename}", 'wb') do |f|
    f.write(file[:tempfile].read)
  end
  
  "File uploaded successfully: #{final_filename}"
end
```

## CSRF Protection

Protect against Cross-Site Request Forgery:

```ruby
require 'sinatra'
require 'securerandom'

enable :sessions

helpers do
  def csrf_token
    session[:csrf] ||= SecureRandom.hex(32)
  end
  
  def csrf_tag
    "<input type='hidden' name='csrf_token' value='#{csrf_token}'>"
  end
  
  def verify_csrf!
    unless params[:csrf_token] == session[:csrf]
      halt 403, "CSRF token mismatch"
    end
  end
end

# In your forms
get '/transfer' do
  erb :transfer
end

post '/transfer' do
  verify_csrf!
  
  # Process the transfer safely
  amount = params[:amount]
  to_account = params[:to_account]
  
  # ... transfer logic ...
end
```

```erb
<!-- views/transfer.erb -->
<form action="/transfer" method="post">
  <%= csrf_tag %>
  
  <input type="number" name="amount" step="0.01" required>
  <input type="text" name="to_account" required>
  <button type="submit">Transfer</button>
</form>
```

## Form Builders

Create helpers to generate forms:

```ruby
helpers do
  class FormBuilder
    def initialize(action, method = 'post')
      @action = action
      @method = method
      @fields = []
    end
    
    def text_field(name, options = {})
      attrs = build_attributes(options.merge(type: 'text', name: name))
      @fields << "<input #{attrs}>"
      self
    end
    
    def email_field(name, options = {})
      attrs = build_attributes(options.merge(type: 'email', name: name))
      @fields << "<input #{attrs}>"
      self
    end
    
    def password_field(name, options = {})
      attrs = build_attributes(options.merge(type: 'password', name: name))
      @fields << "<input #{attrs}>"
      self
    end
    
    def select(name, choices, options = {})
      html = "<select name='#{name}'>"
      choices.each do |value, text|
        selected = options[:selected] == value ? 'selected' : ''
        html << "<option value='#{value}' #{selected}>#{text}</option>"
      end
      html << "</select>"
      @fields << html
      self
    end
    
    def submit(text = 'Submit')
      @fields << "<button type='submit'>#{text}</button>"
      self
    end
    
    def render
      <<~HTML
        <form action="#{@action}" method="#{@method}">
          #{@fields.join("\n  ")}
        </form>
      HTML
    end
    
    private
    
    def build_attributes(options)
      options.map { |k, v| "#{k}='#{v}'" }.join(' ')
    end
  end
  
  def form_for(action, &block)
    builder = FormBuilder.new(action)
    yield(builder) if block_given?
    builder.render
  end
end

# Usage in template
# <%= form_for('/signup') do |f|
#   f.text_field('username', placeholder: 'Username', required: true)
#   f.email_field('email', placeholder: 'Email', required: true)
#   f.password_field('password', placeholder: 'Password', required: true)
#   f.submit('Sign Up')
# end %>
```

## AJAX Forms

Submit forms without page reload:

```erb
<form id="contact-form">
  <input type="text" name="name" required>
  <input type="email" name="email" required>
  <textarea name="message" required></textarea>
  <button type="submit">Send</button>
  <div id="result"></div>
</form>

<script>
document.getElementById('contact-form').addEventListener('submit', async (e) => {
  e.preventDefault();
  
  const formData = new FormData(e.target);
  const data = Object.fromEntries(formData);
  
  try {
    const response = await fetch('/contact', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    
    const result = await response.json();
    
    if (result.success) {
      document.getElementById('result').innerHTML = 
        '<p class="success">Message sent!</p>';
      e.target.reset();
    } else {
      document.getElementById('result').innerHTML = 
        `<p class="error">${result.error}</p>`;
    }
  } catch (error) {
    document.getElementById('result').innerHTML = 
      '<p class="error">Failed to send message</p>';
  }
});
</script>
```

```ruby
post '/contact' do
  content_type :json
  
  data = JSON.parse(request.body.read)
  
  # Validate
  if data['name'].empty? || data['email'].empty? || data['message'].empty?
    return { success: false, error: 'All fields are required' }.to_json
  end
  
  # Process (send email, save to database, etc.)
  # ...
  
  { success: true }.to_json
end
```

## Multi-Step Forms

Break complex forms into steps:

```ruby
enable :sessions

get '/wizard' do
  session[:wizard] ||= {}
  @step = params[:step] || '1'
  erb :"wizard_step_#{@step}"
end

post '/wizard/:step' do
  # Save current step data
  session[:wizard] ||= {}
  session[:wizard]["step_#{params[:step]}"] = params
  
  # Determine next step
  next_step = params[:step].to_i + 1
  
  if next_step > 3  # Assuming 3 steps total
    # Process complete form
    process_wizard_data(session[:wizard])
    session.delete(:wizard)
    redirect '/wizard/complete'
  else
    redirect "/wizard?step=#{next_step}"
  end
end

def process_wizard_data(data)
  # Combine all steps and process
  user_info = data['step_1']
  preferences = data['step_2']
  payment = data['step_3']
  
  # Create user, process payment, etc.
end
```

## Sanitizing User Input

Never trust user input:

```ruby
require 'sanitize'

helpers do
  def sanitize_html(html)
    Sanitize.fragment(html, 
      elements: ['b', 'i', 'em', 'strong', 'p', 'br'],
      attributes: {}
    )
  end
  
  def escape_sql(string)
    # Better: use parameterized queries
    string.gsub(/['";\\]/, '')
  end
  
  def clean_filename(filename)
    # Remove dangerous characters
    filename.gsub(/[^\w\.\-]/, '_')
  end
  
  def validate_url(url)
    uri = URI.parse(url)
    uri.is_a?(URI::HTTP) || uri.is_a?(URI::HTTPS)
  rescue URI::InvalidURIError
    false
  end
end

post '/comment' do
  # Sanitize HTML content
  content = sanitize_html(params[:content])
  
  # Validate and clean other inputs
  name = params[:name].strip.slice(0, 100)
  email = params[:email].downcase.strip
  
  # Save sanitized data
  Comment.create(
    name: name,
    email: email,
    content: content
  )
end
```

## Real-Time Validation

Validate as users type:

```erb
<form id="signup-form">
  <div class="form-group">
    <input type="text" name="username" id="username" required>
    <span id="username-feedback"></span>
  </div>
  
  <div class="form-group">
    <input type="email" name="email" id="email" required>
    <span id="email-feedback"></span>
  </div>
  
  <div class="form-group">
    <input type="password" name="password" id="password" required>
    <div id="password-strength"></div>
  </div>
</form>

<script>
// Check username availability
document.getElementById('username').addEventListener('blur', async (e) => {
  const username = e.target.value;
  if (username.length < 3) return;
  
  const response = await fetch(`/check-username?username=${username}`);
  const result = await response.json();
  
  const feedback = document.getElementById('username-feedback');
  if (result.available) {
    feedback.textContent = '✓ Available';
    feedback.className = 'success';
  } else {
    feedback.textContent = '✗ Already taken';
    feedback.className = 'error';
  }
});

// Password strength meter
document.getElementById('password').addEventListener('input', (e) => {
  const password = e.target.value;
  let strength = 0;
  
  if (password.length >= 8) strength++;
  if (password.match(/[a-z]/)) strength++;
  if (password.match(/[A-Z]/)) strength++;
  if (password.match(/[0-9]/)) strength++;
  if (password.match(/[^a-zA-Z0-9]/)) strength++;
  
  const meter = document.getElementById('password-strength');
  const strengths = ['', 'Weak', 'Fair', 'Good', 'Strong', 'Very Strong'];
  const colors = ['', 'red', 'orange', 'yellow', 'lightgreen', 'green'];
  
  meter.textContent = strengths[strength];
  meter.style.color = colors[strength];
});
</script>
```

```ruby
get '/check-username' do
  content_type :json
  username = params[:username]
  
  available = !User.exists?(username: username)
  { available: available }.to_json
end
```

## Your Turn: Form Challenges

1. **Survey Builder**: Create a dynamic survey form generator
2. **Payment Form**: Build a secure payment form with validation
3. **Image Upload**: Create an image uploader with preview
4. **Search Filters**: Build advanced search with multiple filters
5. **Form Analytics**: Track form completion and abandonment

## What You've Learned

You now know how to:
- Create HTML forms
- Handle form submissions
- Validate input client-side and server-side
- Upload files safely
- Protect against CSRF attacks
- Build form helpers
- Create AJAX forms
- Implement multi-step forms
- Sanitize user input
- Add real-time validation

## What's Next?

Next, we'll explore sessions and cookies, learning how to maintain state across requests and create personalized user experiences. Get ready to make your app remember things!