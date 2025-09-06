# Chapter 21: HTTP - How the Internet Actually Works

Before we dive into building web applications with Sinatra, let's understand how the web actually works. It's all just text flying through tubes. Seriously. The entire internet is basically computers sending carefully formatted text messages to each other. Let's demystify the magic!

## What Is HTTP?

HTTP (HyperText Transfer Protocol) is how browsers talk to servers. It's like a very polite conversation:

- Browser: "Hello server, may I please have the homepage?"
- Server: "Certainly! Here's the HTML you requested."
- Browser: "Thank you! Oh, I see you mentioned a stylesheet?"
- Server: "Yes, here's the CSS file."
- Browser: "Splendid! And this JavaScript?"
- Server: "Coming right up!"

## The Request-Response Cycle

Every HTTP interaction follows this pattern:

```ruby
# 1. Client makes a REQUEST
# 2. Server processes the request
# 3. Server sends a RESPONSE
# 4. Client does something with the response

# Let's simulate this in Ruby!
require 'net/http'
require 'uri'

# Making a request (what your browser does)
uri = URI('http://example.com')
response = Net::HTTP.get_response(uri)

puts "Status: #{response.code}"
puts "Body: #{response.body[0..100]}..."  # First 100 chars
```

## Anatomy of an HTTP Request

An HTTP request looks like this:

```
GET /about HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
Cookie: session=abc123
```

Let's break it down:

```ruby
# Building a request manually
require 'net/http'

uri = URI('http://httpbin.org/get')
http = Net::HTTP.new(uri.host, uri.port)

request = Net::HTTP::Get.new(uri)
request['User-Agent'] = 'Ruby Tutorial Bot 1.0'
request['Accept'] = 'application/json'
request['Custom-Header'] = 'I can add whatever I want!'

response = http.request(request)
puts response.body
```

## HTTP Methods (Verbs)

HTTP has different methods for different actions:

```ruby
require 'net/http'
require 'json'
require 'uri'

class HTTPDemo
  BASE_URL = 'https://jsonplaceholder.typicode.com'
  
  # GET - Retrieve data
  def get_example
    uri = URI("#{BASE_URL}/posts/1")
    response = Net::HTTP.get_response(uri)
    JSON.parse(response.body)
  end
  
  # POST - Create new data
  def post_example
    uri = URI("#{BASE_URL}/posts")
    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true
    
    request = Net::HTTP::Post.new(uri)
    request['Content-Type'] = 'application/json'
    request.body = {
      title: 'My New Post',
      body: 'This is the content',
      userId: 1
    }.to_json
    
    response = http.request(request)
    JSON.parse(response.body)
  end
  
  # PUT - Update existing data (replace completely)
  def put_example(id)
    uri = URI("#{BASE_URL}/posts/#{id}")
    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true
    
    request = Net::HTTP::Put.new(uri)
    request['Content-Type'] = 'application/json'
    request.body = {
      id: id,
      title: 'Updated Title',
      body: 'Updated content',
      userId: 1
    }.to_json
    
    response = http.request(request)
    JSON.parse(response.body)
  end
  
  # PATCH - Partial update
  def patch_example(id)
    uri = URI("#{BASE_URL}/posts/#{id}")
    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true
    
    request = Net::HTTP::Patch.new(uri)
    request['Content-Type'] = 'application/json'
    request.body = { title: 'Just updating the title' }.to_json
    
    response = http.request(request)
    JSON.parse(response.body)
  end
  
  # DELETE - Remove data
  def delete_example(id)
    uri = URI("#{BASE_URL}/posts/#{id}")
    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true
    
    request = Net::HTTP::Delete.new(uri)
    response = http.request(request)
    
    { status: response.code, message: 'Deleted' }
  end
end

demo = HTTPDemo.new
puts demo.get_example
puts demo.post_example
```

## Status Codes: What They Mean

HTTP status codes tell you what happened:

```ruby
class StatusCodeExplainer
  CODES = {
    # 1xx - Informational (rare)
    100 => "Continue - Keep going!",
    
    # 2xx - Success
    200 => "OK - Everything worked!",
    201 => "Created - New resource created",
    204 => "No Content - Success, but nothing to return",
    
    # 3xx - Redirection
    301 => "Moved Permanently - New address!",
    302 => "Found - Temporarily moved",
    304 => "Not Modified - Use your cache",
    
    # 4xx - Client Errors (You messed up)
    400 => "Bad Request - Your request doesn't make sense",
    401 => "Unauthorized - Who are you?",
    403 => "Forbidden - I know who you are, but no",
    404 => "Not Found - That doesn't exist",
    418 => "I'm a teapot - Yes, this is real",
    422 => "Unprocessable Entity - I understand but can't process",
    429 => "Too Many Requests - Slow down!",
    
    # 5xx - Server Errors (Server messed up)
    500 => "Internal Server Error - We broke something",
    502 => "Bad Gateway - Server got confused",
    503 => "Service Unavailable - We're down",
  }
  
  def self.explain(code)
    CODES[code] || "Unknown code: #{code}"
  end
end

# Testing different responses
require 'net/http'

urls = [
  'https://httpbin.org/status/200',
  'https://httpbin.org/status/404',
  'https://httpbin.org/status/500'
]

urls.each do |url|
  uri = URI(url)
  response = Net::HTTP.get_response(uri)
  puts "#{response.code}: #{StatusCodeExplainer.explain(response.code.to_i)}"
end
```

## Headers: The Metadata

Headers carry additional information:

```ruby
class HeaderExplorer
  def examine_request_headers
    # Common request headers
    {
      'User-Agent' => 'Tells server what browser/client you are',
      'Accept' => 'What content types you can handle',
      'Accept-Language' => 'Preferred languages',
      'Cookie' => 'Session and preference data',
      'Authorization' => 'Authentication credentials',
      'Content-Type' => 'Format of data you're sending',
      'Host' => 'Which website you want',
      'Referer' => 'Where you came from'
    }
  end
  
  def examine_response_headers
    # Common response headers
    {
      'Content-Type' => 'Format of the response',
      'Content-Length' => 'Size of the response',
      'Set-Cookie' => 'Store this for later',
      'Cache-Control' => 'How long to cache this',
      'Location' => 'Redirect to this URL',
      'Server' => 'What server software',
      'Date' => 'When this response was created'
    }
  end
  
  def real_example
    uri = URI('https://httpbin.org/headers')
    response = Net::HTTP.get_response(uri)
    
    puts "Response Headers:"
    response.each_header do |key, value|
      puts "  #{key}: #{value}"
    end
    
    puts "\nResponse Body (shows request headers):"
    puts response.body
  end
end

explorer = HeaderExplorer.new
explorer.real_example
```

## Cookies: The Web's Memory

HTTP is stateless - each request knows nothing about previous ones. Cookies fix this:

```ruby
class CookieJar
  def demonstrate_cookies
    require 'net/http'
    uri = URI('https://httpbin.org/cookies/set?session=abc123&user=alice')
    
    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true
    
    # First request sets cookies
    response = http.get(uri.path + '?' + uri.query)
    
    # Server sends back Set-Cookie header
    cookies = response.get_fields('set-cookie')
    puts "Server set these cookies:"
    cookies&.each { |cookie| puts "  #{cookie}" }
    
    # Next request should include cookies
    uri2 = URI('https://httpbin.org/cookies')
    request = Net::HTTP::Get.new(uri2)
    request['Cookie'] = cookies&.join('; ') || ''
    
    response2 = http.request(request)
    puts "\nServer sees these cookies:"
    puts response2.body
  end
end
```

## HTTPS: The Secure Version

HTTPS is HTTP with encryption:

```ruby
require 'net/http'
require 'openssl'

class SecureConnection
  def compare_http_vs_https
    # HTTP (insecure)
    http_uri = URI('http://httpbin.org/get')
    
    # HTTPS (secure)
    https_uri = URI('https://httpbin.org/get')
    
    # Making HTTPS request
    http = Net::HTTP.new(https_uri.host, https_uri.port)
    http.use_ssl = true
    http.verify_mode = OpenSSL::SSL::VERIFY_PEER
    
    request = Net::HTTP::Get.new(https_uri)
    response = http.request(request)
    
    puts "Secure response received!"
    puts "SSL/TLS ensures:"
    puts "  1. Data is encrypted in transit"
    puts "  2. Server identity is verified"
    puts "  3. Data integrity is maintained"
  end
end
```

## Building a Simple HTTP Server

Let's build our own HTTP server to see the other side:

```ruby
require 'socket'

class TinyHTTPServer
  def initialize(port = 3000)
    @port = port
  end
  
  def start
    server = TCPServer.new(@port)
    puts "Server running on http://localhost:#{@port}"
    puts "Press Ctrl+C to stop"
    
    loop do
      client = server.accept
      request = client.gets
      
      # Parse the request line
      method, path, version = request.split
      
      puts "#{Time.now} - #{method} #{path}"
      
      # Send response
      response = case path
      when '/'
        build_response(200, '<h1>Welcome to Ruby Server!</h1>')
      when '/about'
        build_response(200, '<h1>About Page</h1><p>Built with Ruby!</p>')
      when '/api/time'
        build_response(200, { time: Time.now.to_s }.to_json,
                      'application/json')
      else
        build_response(404, '<h1>404 Not Found</h1>')
      end
      
      client.print response
      client.close
    end
  end
  
  private
  
  def build_response(status, body, content_type = 'text/html')
    status_text = {
      200 => 'OK',
      404 => 'Not Found'
    }[status]
    
    <<~RESPONSE
      HTTP/1.1 #{status} #{status_text}
      Content-Type: #{content_type}
      Content-Length: #{body.bytesize}
      Connection: close
      
      #{body}
    RESPONSE
  end
end

# Run with: TinyHTTPServer.new.start
# Then visit http://localhost:3000 in your browser!
```

## REST: A Way of Thinking

REST (Representational State Transfer) is a pattern for designing web APIs:

```ruby
class RESTfulAPI
  # Resources are nouns, HTTP methods are verbs
  
  def rest_patterns
    {
      'GET /users'        => 'List all users',
      'GET /users/123'    => 'Show user 123',
      'POST /users'       => 'Create new user',
      'PUT /users/123'    => 'Update user 123 completely',
      'PATCH /users/123'  => 'Update user 123 partially',
      'DELETE /users/123' => 'Delete user 123'
    }
  end
  
  def good_rest_design
    # Good: Resources are nouns
    '/articles'
    '/articles/42'
    '/articles/42/comments'
    
    # Bad: Actions in URLs
    '/getArticle'
    '/articles/delete/42'
    '/CreateNewArticle'
  end
  
  def status_codes_matter
    {
      'GET success'    => 200,  # OK
      'POST success'   => 201,  # Created
      'DELETE success' => 204,  # No Content
      'Not found'      => 404,  # Not Found
      'Validation fail' => 422, # Unprocessable Entity
      'Server error'   => 500   # Internal Server Error
    }
  end
end
```

## Working with APIs

Most modern web services provide JSON APIs:

```ruby
require 'net/http'
require 'json'

class GitHubAPI
  BASE_URL = 'https://api.github.com'
  
  def get_user(username)
    uri = URI("#{BASE_URL}/users/#{username}")
    response = Net::HTTP.get_response(uri)
    
    if response.code == '200'
      user = JSON.parse(response.body)
      {
        name: user['name'],
        bio: user['bio'],
        repos: user['public_repos'],
        followers: user['followers']
      }
    else
      { error: "User not found" }
    end
  end
  
  def get_repos(username)
    uri = URI("#{BASE_URL}/users/#{username}/repos")
    response = Net::HTTP.get_response(uri)
    
    if response.code == '200'
      repos = JSON.parse(response.body)
      repos.map { |r| { name: r['name'], stars: r['stargazers_count'] } }
           .sort_by { |r| -r[:stars] }
           .first(5)
    else
      []
    end
  end
end

# Usage
api = GitHubAPI.new
puts api.get_user('matz')  # Ruby's creator!
```

## HTTP in the Real World

```ruby
class RealWorldHTTP
  def common_scenarios
    {
      authentication: <<~AUTH,
        # Basic Auth
        request['Authorization'] = 'Basic ' + Base64.encode64('user:pass')
        
        # Bearer Token
        request['Authorization'] = 'Bearer your-api-token-here'
        
        # API Key
        request['X-API-Key'] = 'your-api-key'
      AUTH
      
      pagination: <<~PAGE,
        # Using query parameters
        GET /posts?page=2&per_page=20
        
        # Response headers tell you more
        Link: <http://example.com/posts?page=3>; rel="next"
        X-Total-Count: 200
      PAGE
      
      rate_limiting: <<~RATE,
        # Server tells you limits
        X-RateLimit-Limit: 60
        X-RateLimit-Remaining: 42
        X-RateLimit-Reset: 1234567890
      RATE
      
      caching: <<~CACHE,
        # Client sends
        If-None-Match: "abc123"
        
        # Server responds
        304 Not Modified  # Use your cached version
        # OR
        200 OK
        ETag: "def456"   # New version tag
      CACHE
    }
  end
end
```

## Debugging HTTP

Tools for understanding HTTP:

```ruby
class HTTPDebugger
  def debug_with_curl
    # curl is your friend!
    commands = [
      "curl -v https://example.com",           # Verbose output
      "curl -I https://example.com",           # Headers only
      "curl -X POST https://example.com/api",  # Different method
      "curl -H 'Accept: application/json' ...", # Custom headers
      "curl -d 'data=value' ...",              # Send data
      "curl -b cookies.txt -c cookies.txt ...", # Handle cookies
    ]
  end
  
  def debug_in_ruby
    require 'net/http'
    
    uri = URI('https://httpbin.org/get')
    
    Net::HTTP.start(uri.host, uri.port, use_ssl: true) do |http|
      http.set_debug_output($stdout)  # See everything!
      
      request = Net::HTTP::Get.new(uri)
      response = http.request(request)
    end
  end
  
  def useful_services
    {
      'httpbin.org' => 'Test HTTP requests',
      'requestbin.com' => 'Inspect webhooks',
      'webhook.site' => 'Debug webhooks',
      'postman.com' => 'API development platform'
    }
  end
end
```

## Your Turn: HTTP Challenges

1. **Build an API Client**: Create a wrapper for your favorite API
2. **Web Scraper**: Extract data from websites
3. **Webhook Receiver**: Build a server that handles webhooks
4. **REST API**: Design and implement a RESTful API
5. **HTTP Monitor**: Track response times and status codes

## What You've Learned

You now understand:
- How HTTP requests and responses work
- The different HTTP methods and when to use them
- Status codes and what they mean
- Headers and their purposes
- Cookies and sessions
- HTTPS and security
- REST principles
- How to work with APIs
- Debugging HTTP interactions

## What's Next?

Now that you understand HTTP, you're ready to build web applications! In the next chapter, we'll introduce Sinatra, a delightful Ruby web framework that makes building web apps fun and simple.

Remember: The entire web is just computers sending text to each other following agreed-upon formats. There's no magic, just protocols. And now you understand the most important protocol of them all!