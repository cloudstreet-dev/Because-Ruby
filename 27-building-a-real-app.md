# Chapter 27: Building a Real App - Putting It All Together

Alright, time to build something real. No more toy examples or isolated concepts. We're going to build a complete web application that you could actually deploy and use. Our project: **RubyLinks** - a social link-sharing platform where developers can share and discuss interesting Ruby resources.

## The Plan

We're building a simplified Reddit/Hacker News clone focused on Ruby content. Users can:
- Submit links with titles and descriptions
- Vote links up or down
- Comment on links
- Create accounts and log in
- View trending and recent links
- Search for content

We'll use everything we've learned: classes, modules, Sinatra, databases, testing, and good Ruby practices.

## Project Structure

```
rubylinks/
├── Gemfile
├── Gemfile.lock
├── config.ru
├── app.rb
├── models/
│   ├── user.rb
│   ├── link.rb
│   ├── comment.rb
│   └── vote.rb
├── views/
│   ├── layout.erb
│   ├── index.erb
│   ├── link.erb
│   ├── submit.erb
│   ├── login.erb
│   └── register.erb
├── public/
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── app.js
├── db/
│   ├── schema.sql
│   └── seeds.rb
├── lib/
│   └── helpers.rb
└── spec/
    └── app_spec.rb
```

## Step 1: Setup and Dependencies

```ruby
# Gemfile
source 'https://rubygems.org'

# Core
gem 'sinatra'
gem 'sinatra-contrib'
gem 'puma'

# Database
gem 'sqlite3'
gem 'sequel'

# Authentication
gem 'bcrypt'

# Views
gem 'erubis'

# Development
group :development do
  gem 'rerun'
  gem 'pry'
end

# Testing
group :test do
  gem 'rspec'
  gem 'rack-test'
  gem 'database_cleaner-sequel'
end
```

## Step 2: Database Schema

```sql
-- db/schema.sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  password_hash VARCHAR(60) NOT NULL,
  karma INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE links (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  title VARCHAR(255) NOT NULL,
  url VARCHAR(500) NOT NULL,
  description TEXT,
  score INTEGER DEFAULT 0,
  comment_count INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE comments (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  link_id INTEGER NOT NULL,
  parent_id INTEGER,
  content TEXT NOT NULL,
  score INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (link_id) REFERENCES links(id),
  FOREIGN KEY (parent_id) REFERENCES comments(id)
);

CREATE TABLE votes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  votable_type VARCHAR(20) NOT NULL,
  votable_id INTEGER NOT NULL,
  value INTEGER NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(user_id, votable_type, votable_id),
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_links_score ON links(score DESC);
CREATE INDEX idx_links_created ON links(created_at DESC);
CREATE INDEX idx_votes_lookup ON votes(votable_type, votable_id);
```

## Step 3: Models

```ruby
# models/user.rb
require 'bcrypt'

class User < Sequel::Model
  one_to_many :links
  one_to_many :comments
  one_to_many :votes
  
  plugin :validation_helpers
  plugin :timestamps
  
  def validate
    super
    validates_presence [:username, :email]
    validates_unique [:username, :email]
    validates_format /\A[\w\-]+\z/, :username, message: 'can only contain letters, numbers, and underscores'
    validates_format /\A.+@.+\..+\z/, :email, message: 'is not a valid email'
  end
  
  def password=(new_password)
    self.password_hash = BCrypt::Password.create(new_password)
  end
  
  def authenticate(password)
    BCrypt::Password.new(password_hash) == password
  end
  
  def vote_for(votable, value)
    vote = Vote.find_or_create(
      user_id: id,
      votable_type: votable.class.name,
      votable_id: votable.id
    )
    
    old_value = vote.value || 0
    vote.value = value
    vote.save
    
    # Update score
    votable.score += (value - old_value)
    votable.save
    
    vote
  end
  
  def voted_on?(votable)
    Vote.where(
      user_id: id,
      votable_type: votable.class.name,
      votable_id: votable.id
    ).first
  end
  
  def update_karma!
    link_karma = links_dataset.sum(:score) || 0
    comment_karma = comments_dataset.sum(:score) || 0
    self.karma = link_karma + comment_karma
    save
  end
end

# models/link.rb
class Link < Sequel::Model
  many_to_one :user
  one_to_many :comments
  one_to_many :votes, key: :votable_id do |ds|
    ds.where(votable_type: 'Link')
  end
  
  plugin :validation_helpers
  plugin :timestamps
  
  def validate
    super
    validates_presence [:title, :url, :user_id]
    validates_max_length 255, :title
    validates_max_length 500, :url
    validates_format /\Ahttps?:\/\/.+\z/, :url, message: 'must be a valid URL'
  end
  
  def before_save
    self.title = title.strip if title
    self.url = url.strip if url
    self.description = description&.strip
    super
  end
  
  def domain
    URI.parse(url).host.sub(/^www\./, '') rescue url
  end
  
  def age_in_hours
    (Time.now - created_at) / 3600
  end
  
  def hotness_score
    # Hacker News algorithm
    (score - 1) / ((age_in_hours + 2) ** 1.5)
  end
  
  def root_comments
    comments_dataset.where(parent_id: nil).order(:score.desc, :created_at)
  end
  
  dataset_module do
    def hot
      order(Sequel.lit('(score - 1) / ((julianday("now") - julianday(created_at)) * 24 + 2) DESC'))
    end
    
    def recent
      order(:created_at.desc)
    end
    
    def top(period = nil)
      case period
      when :day
        where(created_at: Time.now - 86400..Time.now)
      when :week
        where(created_at: Time.now - 604800..Time.now)
      when :month
        where(created_at: Time.now - 2592000..Time.now)
      else
        self
      end.order(:score.desc)
    end
  end
end

# models/comment.rb
class Comment < Sequel::Model
  many_to_one :user
  many_to_one :link
  many_to_one :parent, class: self
  one_to_many :children, key: :parent_id, class: self
  one_to_many :votes, key: :votable_id do |ds|
    ds.where(votable_type: 'Comment')
  end
  
  plugin :validation_helpers
  plugin :timestamps
  plugin :tree
  
  def validate
    super
    validates_presence [:content, :user_id, :link_id]
    validates_max_length 10000, :content
  end
  
  def level
    parent ? parent.level + 1 : 0
  end
  
  def before_save
    self.content = content.strip if content
    super
  end
  
  def after_create
    link.update(comment_count: link.comments.count)
    super
  end
end

# models/vote.rb
class Vote < Sequel::Model
  many_to_one :user
  
  plugin :validation_helpers
  plugin :timestamps
  
  def validate
    super
    validates_presence [:user_id, :votable_type, :votable_id, :value]
    validates_includes [-1, 0, 1], :value
  end
end
```

## Step 4: Main Application

```ruby
# app.rb
require 'sinatra'
require 'sinatra/reloader' if development?
require 'sequel'
require 'bcrypt'
require 'erb'
require 'json'

# Database connection
DB = Sequel.sqlite(ENV['DATABASE_URL'] || 'db/rubylinks.db')

# Load models
require_relative 'models/user'
require_relative 'models/link'
require_relative 'models/comment'
require_relative 'models/vote'

# Configuration
configure do
  enable :sessions
  set :session_secret, ENV['SESSION_SECRET'] || 'development_secret_change_in_production'
  set :views, 'views'
  set :public_folder, 'public'
end

# Helpers
helpers do
  def current_user
    @current_user ||= User[session[:user_id]] if session[:user_id]
  end
  
  def logged_in?
    !!current_user
  end
  
  def require_login
    unless logged_in?
      session[:return_to] = request.path
      redirect '/login'
    end
  end
  
  def h(text)
    Rack::Utils.escape_html(text)
  end
  
  def time_ago(time)
    seconds = Time.now - time
    case seconds
    when 0..59 then "just now"
    when 60..3599 then "#{(seconds/60).round} minutes ago"
    when 3600..86399 then "#{(seconds/3600).round} hours ago"
    when 86400..2591999 then "#{(seconds/86400).round} days ago"
    else time.strftime("%B %d, %Y")
    end
  end
  
  def vote_button(votable, direction)
    return "" unless logged_in?
    
    vote = current_user.voted_on?(votable)
    voted = vote && vote.value == (direction == :up ? 1 : -1)
    
    %Q{
      <form method="post" action="/vote" class="vote-form">
        <input type="hidden" name="type" value="#{votable.class}">
        <input type="hidden" name="id" value="#{votable.id}">
        <input type="hidden" name="direction" value="#{direction}">
        <button type="submit" class="vote-btn #{direction} #{voted ? 'voted' : ''}">
          #{direction == :up ? '▲' : '▼'}
        </button>
      </form>
    }
  end
  
  def markdown(text)
    # Simple markdown-like formatting
    h(text).gsub(/\n\n/, '</p><p>')
           .gsub(/\n/, '<br>')
           .gsub(/\*\*(.*?)\*\*/, '<strong>\1</strong>')
           .gsub(/\*(.*?)\*/, '<em>\1</em>')
           .gsub(/`(.*?)`/, '<code>\1</code>')
           .gsub(/\[([^\]]+)\]\(([^)]+)\)/, '<a href="\2">\1</a>')
  end
end

# Routes

# Homepage
get '/' do
  @page = (params[:page] || 1).to_i
  @sort = params[:sort] || 'hot'
  
  dataset = case @sort
  when 'new' then Link.recent
  when 'top' then Link.top(params[:period]&.to_sym)
  else Link.hot
  end
  
  @links = dataset.paginate(@page, 25)
  erb :index
end

# Submit link
get '/submit' do
  require_login
  erb :submit
end

post '/submit' do
  require_login
  
  @link = Link.new(
    user_id: current_user.id,
    title: params[:title],
    url: params[:url],
    description: params[:description]
  )
  
  if @link.valid?
    @link.save
    redirect "/links/#{@link.id}"
  else
    @errors = @link.errors
    erb :submit
  end
end

# View link and comments
get '/links/:id' do
  @link = Link[params[:id]]
  halt 404, "Link not found" unless @link
  
  @comments = @link.root_comments
  erb :link
end

# Post comment
post '/links/:id/comment' do
  require_login
  
  link = Link[params[:id]]
  halt 404, "Link not found" unless link
  
  comment = Comment.new(
    user_id: current_user.id,
    link_id: link.id,
    parent_id: params[:parent_id],
    content: params[:content]
  )
  
  if comment.valid?
    comment.save
    redirect "/links/#{link.id}#comment-#{comment.id}"
  else
    session[:error] = comment.errors.full_messages.first
    redirect back
  end
end

# Voting
post '/vote' do
  require_login
  
  type = params[:type]
  id = params[:id].to_i
  direction = params[:direction]
  
  votable = type.constantize[id]
  halt 404, "Not found" unless votable
  
  value = direction == 'up' ? 1 : -1
  current_user.vote_for(votable, value)
  
  # Return JSON for AJAX requests
  if request.xhr?
    content_type :json
    { score: votable.reload.score }.to_json
  else
    redirect back
  end
end

# User profiles
get '/users/:username' do
  @user = User.where(username: params[:username]).first
  halt 404, "User not found" unless @user
  
  @links = @user.links_dataset.recent.limit(20)
  @comments = @user.comments_dataset.recent.limit(20)
  
  erb :profile
end

# Authentication
get '/register' do
  erb :register
end

post '/register' do
  @user = User.new(
    username: params[:username],
    email: params[:email]
  )
  @user.password = params[:password]
  
  if params[:password] != params[:password_confirmation]
    @error = "Passwords don't match"
    erb :register
  elsif @user.valid? && @user.save
    session[:user_id] = @user.id
    redirect session.delete(:return_to) || '/'
  else
    @error = @user.errors.full_messages.first
    erb :register
  end
end

get '/login' do
  erb :login
end

post '/login' do
  user = User.where(username: params[:username]).first
  
  if user && user.authenticate(params[:password])
    session[:user_id] = user.id
    redirect session.delete(:return_to) || '/'
  else
    @error = "Invalid username or password"
    erb :login
  end
end

get '/logout' do
  session.clear
  redirect '/'
end

# Search
get '/search' do
  @query = params[:q]
  if @query && @query.length > 2
    @links = Link.where(Sequel.like(:title, "%#{@query}%"))
                 .or(Sequel.like(:description, "%#{@query}%"))
                 .recent
                 .limit(50)
  end
  erb :search
end

# API endpoints
get '/api/links.json' do
  content_type :json
  
  links = Link.recent.limit(20).map do |link|
    {
      id: link.id,
      title: link.title,
      url: link.url,
      score: link.score,
      comments: link.comment_count,
      user: link.user.username,
      created_at: link.created_at
    }
  end
  
  links.to_json
end
```

## Step 5: Views

```erb
<!-- views/layout.erb -->
<!DOCTYPE html>
<html>
<head>
  <title>RubyLinks - Share Ruby Resources</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="/css/style.css">
</head>
<body>
  <header>
    <nav>
      <a href="/" class="logo">RubyLinks</a>
      <div class="nav-links">
        <a href="/?sort=hot">Hot</a>
        <a href="/?sort=new">New</a>
        <a href="/?sort=top">Top</a>
        <% if logged_in? %>
          <a href="/submit">Submit</a>
          <a href="/users/<%= current_user.username %>"><%= current_user.username %> (<%= current_user.karma %>)</a>
          <a href="/logout">Logout</a>
        <% else %>
          <a href="/login">Login</a>
          <a href="/register">Register</a>
        <% end %>
      </div>
    </nav>
  </header>
  
  <main>
    <% if session[:error] %>
      <div class="alert error"><%= session.delete(:error) %></div>
    <% end %>
    <% if session[:notice] %>
      <div class="alert notice"><%= session.delete(:notice) %></div>
    <% end %>
    
    <%= yield %>
  </main>
  
  <footer>
    <p>Built with Ruby and Sinatra | <a href="https://github.com/yourusername/rubylinks">Source Code</a></p>
  </footer>
  
  <script src="/js/app.js"></script>
</body>
</html>

<!-- views/index.erb -->
<div class="link-list">
  <% @links.each_with_index do |link, i| %>
    <article class="link-item">
      <div class="rank"><%= (@page - 1) * 25 + i + 1 %></div>
      <div class="voting">
        <%= vote_button(link, :up) %>
        <span class="score"><%= link.score %></span>
        <%= vote_button(link, :down) %>
      </div>
      <div class="content">
        <h2>
          <a href="<%= link.url %>" target="_blank"><%= h link.title %></a>
          <span class="domain">(<%= link.domain %>)</span>
        </h2>
        <% if link.description %>
          <p class="description"><%= h link.description %></p>
        <% end %>
        <div class="meta">
          by <a href="/users/<%= link.user.username %>"><%= link.user.username %></a>
          <%= time_ago(link.created_at) %> |
          <a href="/links/<%= link.id %>"><%= link.comment_count %> comments</a>
        </div>
      </div>
    </article>
  <% end %>
  
  <% if @links.empty? %>
    <p class="empty">No links found. <a href="/submit">Submit the first one!</a></p>
  <% end %>
  
  <div class="pagination">
    <% if @page > 1 %>
      <a href="?page=<%= @page - 1 %>&sort=<%= @sort %>">← Previous</a>
    <% end %>
    <% if @links.count == 25 %>
      <a href="?page=<%= @page + 1 %>&sort=<%= @sort %>">Next →</a>
    <% end %>
  </div>
</div>
```

## Step 6: Styling

```css
/* public/css/style.css */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  line-height: 1.6;
  color: #333;
  background: #f6f6f6;
}

header {
  background: #b91c1c;
  color: white;
  padding: 1rem 0;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

nav {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.logo {
  font-size: 1.5rem;
  font-weight: bold;
  color: white;
  text-decoration: none;
}

.nav-links a {
  color: white;
  text-decoration: none;
  margin-left: 1.5rem;
  opacity: 0.9;
}

.nav-links a:hover {
  opacity: 1;
  text-decoration: underline;
}

main {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 0 1rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.link-item {
  display: flex;
  padding: 1rem;
  border-bottom: 1px solid #e5e5e5;
}

.link-item:hover {
  background: #fafafa;
}

.rank {
  color: #999;
  margin-right: 1rem;
  min-width: 30px;
  text-align: right;
}

.voting {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-right: 1rem;
}

.vote-form {
  display: inline;
}

.vote-btn {
  background: none;
  border: none;
  color: #999;
  cursor: pointer;
  font-size: 0.8rem;
  padding: 0;
}

.vote-btn:hover {
  color: #b91c1c;
}

.vote-btn.voted {
  color: #b91c1c;
}

.score {
  font-weight: bold;
  margin: 0.25rem 0;
}

.content {
  flex: 1;
}

.content h2 {
  font-size: 1.1rem;
  margin-bottom: 0.25rem;
}

.content h2 a {
  color: #333;
  text-decoration: none;
}

.content h2 a:hover {
  text-decoration: underline;
}

.domain {
  color: #999;
  font-size: 0.9rem;
  font-weight: normal;
}

.description {
  color: #666;
  margin: 0.5rem 0;
  font-size: 0.95rem;
}

.meta {
  color: #999;
  font-size: 0.85rem;
}

.meta a {
  color: #999;
}

.meta a:hover {
  text-decoration: underline;
}

.comment {
  padding: 1rem;
  border-left: 2px solid #e5e5e5;
  margin: 1rem 0;
}

.comment.level-1 {
  margin-left: 2rem;
}

.comment.level-2 {
  margin-left: 4rem;
}

.comment-header {
  color: #999;
  font-size: 0.85rem;
  margin-bottom: 0.5rem;
}

.comment-content {
  color: #333;
}

form.main-form {
  padding: 2rem;
}

input[type="text"],
input[type="email"],
input[type="password"],
input[type="url"],
textarea {
  width: 100%;
  padding: 0.75rem;
  margin-bottom: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

textarea {
  min-height: 100px;
  resize: vertical;
}

button[type="submit"] {
  background: #b91c1c;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
}

button[type="submit"]:hover {
  background: #991b1b;
}

.alert {
  padding: 1rem;
  margin: 1rem 0;
  border-radius: 4px;
}

.alert.error {
  background: #fee;
  color: #c00;
  border: 1px solid #fcc;
}

.alert.notice {
  background: #efe;
  color: #060;
  border: 1px solid #cfc;
}

.pagination {
  padding: 2rem;
  text-align: center;
}

.pagination a {
  padding: 0.5rem 1rem;
  background: #f0f0f0;
  color: #333;
  text-decoration: none;
  border-radius: 4px;
  margin: 0 0.5rem;
}

.pagination a:hover {
  background: #e0e0e0;
}

footer {
  text-align: center;
  padding: 2rem;
  color: #999;
  font-size: 0.9rem;
}

footer a {
  color: #999;
}
```

## Step 7: JavaScript Enhancements

```javascript
// public/js/app.js
document.addEventListener('DOMContentLoaded', function() {
  // AJAX voting
  document.querySelectorAll('.vote-form').forEach(form => {
    form.addEventListener('submit', async (e) => {
      e.preventDefault();
      
      const formData = new FormData(form);
      const response = await fetch('/vote', {
        method: 'POST',
        body: formData,
        headers: {
          'X-Requested-With': 'XMLHttpRequest'
        }
      });
      
      if (response.ok) {
        const data = await response.json();
        const scoreElement = form.parentElement.querySelector('.score');
        scoreElement.textContent = data.score;
        
        // Toggle voted class
        const button = form.querySelector('.vote-btn');
        button.classList.toggle('voted');
      }
    });
  });
  
  // Auto-resize textareas
  document.querySelectorAll('textarea').forEach(textarea => {
    textarea.addEventListener('input', function() {
      this.style.height = 'auto';
      this.style.height = this.scrollHeight + 'px';
    });
  });
  
  // Confirm dangerous actions
  document.querySelectorAll('[data-confirm]').forEach(element => {
    element.addEventListener('click', (e) => {
      if (!confirm(element.dataset.confirm)) {
        e.preventDefault();
      }
    });
  });
  
  // Time ago updates
  function updateTimeAgo() {
    document.querySelectorAll('[data-time]').forEach(element => {
      const time = new Date(element.dataset.time);
      const seconds = Math.floor((new Date() - time) / 1000);
      
      let interval = Math.floor(seconds / 31536000);
      if (interval > 1) {
        element.textContent = interval + " years ago";
        return;
      }
      
      interval = Math.floor(seconds / 2592000);
      if (interval > 1) {
        element.textContent = interval + " months ago";
        return;
      }
      
      interval = Math.floor(seconds / 86400);
      if (interval > 1) {
        element.textContent = interval + " days ago";
        return;
      }
      
      interval = Math.floor(seconds / 3600);
      if (interval > 1) {
        element.textContent = interval + " hours ago";
        return;
      }
      
      interval = Math.floor(seconds / 60);
      if (interval > 1) {
        element.textContent = interval + " minutes ago";
        return;
      }
      
      element.textContent = "just now";
    });
  }
  
  updateTimeAgo();
  setInterval(updateTimeAgo, 60000); // Update every minute
});
```

## Step 8: Testing

```ruby
# spec/app_spec.rb
require_relative '../app'
require 'rspec'
require 'rack/test'

describe 'RubyLinks App' do
  include Rack::Test::Methods
  
  def app
    Sinatra::Application
  end
  
  describe 'Homepage' do
    it 'loads successfully' do
      get '/'
      expect(last_response).to be_ok
      expect(last_response.body).to include('RubyLinks')
    end
    
    it 'shows links' do
      user = User.create(username: 'testuser', email: 'test@example.com', password: 'password')
      link = Link.create(
        user_id: user.id,
        title: 'Test Link',
        url: 'https://example.com',
        description: 'A test link'
      )
      
      get '/'
      expect(last_response.body).to include('Test Link')
      expect(last_response.body).to include('example.com')
    end
  end
  
  describe 'Authentication' do
    it 'allows user registration' do
      post '/register', {
        username: 'newuser',
        email: 'new@example.com',
        password: 'password123',
        password_confirmation: 'password123'
      }
      
      expect(last_response).to be_redirect
      expect(User.where(username: 'newuser').count).to eq(1)
    end
    
    it 'allows user login' do
      user = User.create(username: 'testuser', email: 'test@example.com', password: 'password')
      
      post '/login', {
        username: 'testuser',
        password: 'password'
      }
      
      expect(last_response).to be_redirect
      expect(last_request.session[:user_id]).to eq(user.id)
    end
  end
  
  describe 'Link submission' do
    it 'requires login' do
      get '/submit'
      expect(last_response).to be_redirect
      expect(last_response.location).to include('/login')
    end
    
    it 'allows logged in users to submit links' do
      user = User.create(username: 'testuser', email: 'test@example.com', password: 'password')
      
      post '/login', { username: 'testuser', password: 'password' }
      
      post '/submit', {
        title: 'New Ruby Feature',
        url: 'https://ruby-lang.org',
        description: 'Check out this new Ruby feature!'
      }
      
      expect(last_response).to be_redirect
      expect(Link.where(title: 'New Ruby Feature').count).to eq(1)
    end
  end
end
```

## Step 9: Deployment

```ruby
# config.ru
require './app'
run Sinatra::Application

# Procfile (for Heroku)
web: bundle exec puma -C config/puma.rb

# config/puma.rb
workers Integer(ENV['WEB_CONCURRENCY'] || 2)
threads_count = Integer(ENV['RAILS_MAX_THREADS'] || 5)
threads threads_count, threads_count

preload_app!

rackup      DefaultRackup
port        ENV['PORT']     || 3000
environment ENV['RACK_ENV'] || 'development'
```

## Running the Application

```bash
# Install dependencies
bundle install

# Create database
sqlite3 db/rubylinks.db < db/schema.sql

# Run the app
ruby app.rb

# Or with auto-reload in development
rerun ruby app.rb

# Run tests
rspec

# Deploy to Heroku
heroku create rubylinks
git push heroku main
heroku run rake db:migrate
heroku open
```

## Features to Add

As you continue developing RubyLinks, consider adding:

1. **Email notifications** for replies to comments
2. **User karma** system based on contributions
3. **Tags/categories** for better organization
4. **RSS feeds** for different sections
5. **Admin panel** for moderation
6. **API rate limiting** to prevent abuse
7. **Full-text search** with PostgreSQL
8. **Real-time updates** with WebSockets
9. **Social login** (GitHub, Google)
10. **Dark mode** toggle

## Lessons Learned

Building this application, you've used:

- **Sinatra** for web framework
- **Sequel** for database ORM
- **BCrypt** for password security
- **Classes and modules** for organization
- **ERB templates** for views
- **CSS** for styling
- **JavaScript** for interactivity
- **RSpec** for testing
- **Git** for version control
- **SQL** for data persistence

## What You've Built

You've created a fully functional web application that:
- Handles user authentication securely
- Manages complex data relationships
- Provides a responsive user interface
- Includes AJAX for better UX
- Follows Ruby best practices
- Is ready for deployment

## What's Next?

This application is a solid foundation. You could:
- Add more features and polish
- Refactor to use Rails for comparison
- Build an API-only version
- Create a mobile app that consumes the API
- Open source it and build a community

Remember: Every large application started as a small one. Facebook was once just a simple PHP app. GitHub started as a Rails app. Your RubyLinks could become the next big thing—or at least a great portfolio piece!

The important thing is that you built something real, something that works, something you can be proud of. You're no longer just learning Ruby—you're a Ruby developer. Welcome to the community!