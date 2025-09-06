# Chapter 29: The Ruby Community - Your New Family

The Ruby community isn't just a group of programmers who happen to use the same language. It's a global family bound by shared values: kindness, creativity, and the belief that programming should bring joy. MINASWAN (Matz Is Nice And So We Are Nice) isn't just a cute acronym—it's a way of life that makes the Ruby community one of the most welcoming in tech.

## The Origins of Community

Ruby's community culture was set from the beginning by Yukihiro "Matz" Matsumoto. Unlike many language creators who focus purely on technical merit, Matz prioritized programmer happiness and explicitly encouraged kindness. This foundation shaped everything that followed.

```ruby
# The Ruby way extends beyond code
class RubyCommunity
  def initialize
    @motto = "Matz is nice and so we are nice"
    @values = [:kindness, :creativity, :fun, :helping_others]
    @everyone_welcome = true
  end
  
  def help_newcomer(person)
    greet_warmly(person)
    share_resources(person)
    encourage_questions(person)
    celebrate_small_wins(person)
  end
  
  def code_of_conduct
    {
      be: [:kind, :respectful, :helpful, :patient],
      dont_be: [:dismissive, :condescending, :exclusionary],
      remember: "Everyone was a beginner once"
    }
  end
end
```

## Where Rubyists Gather

### Online Communities

**Ruby Forum and Mailing Lists**
- ruby-talk: The original Ruby mailing list
- ruby-core: For Ruby language development
- ruby-doc: For documentation discussions

**Reddit**
- r/ruby: General Ruby discussions
- r/rails: Ruby on Rails specific
- r/learnruby: For beginners

**Discord and Slack**
- Ruby Discord: Real-time chat with thousands of members
- Local Ruby Slack teams: Many cities have their own

**Stack Overflow**
- Tag: [ruby] for language questions
- Tag: [ruby-on-rails] for Rails
- Always search before asking
- Help others when you can

**GitHub**
- Ruby source: github.com/ruby/ruby
- Contribute to open source projects
- Share your own gems and projects

### Conferences and Events

**Major Conferences**
- **RubyConf**: The original Ruby conference (US)
- **RubyKaigi**: Japan's premier Ruby conference
- **RailsConf**: Focused on Ruby on Rails
- **EuRuKo**: European Ruby Conference
- **RubyConf AU**: Australia's Ruby conference

**Regional Conferences**
- RubyConf India, Brazil, Kenya, and many more
- Brighton Ruby (UK)
- Rocky Mountain Ruby
- Southeast Ruby

**What to Expect at Conferences:**
```ruby
class RubyConference
  def schedule
    {
      keynotes: "Inspiring talks from community leaders",
      technical_talks: "Deep dives into Ruby features",
      workshops: "Hands-on learning",
      lightning_talks: "5-minute presentations",
      hallway_track: "The best part - meeting people!",
      parties: "Yes, Rubyists know how to have fun"
    }
  end
  
  def first_timer_tips
    [
      "Don't be shy - everyone is friendly",
      "Attend the newcomer orientation",
      "Ask speakers questions after talks",
      "Join group dinners",
      "Exchange contact info",
      "Take breaks when needed",
      "Have fun!"
    ]
  end
end
```

### Local Meetups

Almost every major city has a Ruby meetup:

```ruby
# Find your local Ruby meetup
def find_local_community
  resources = {
    meetup_com: "Search for Ruby or Rails groups",
    ruby_meetups: "ruby.meetups.com",
    twitter: "Search for #ruby + your city",
    github: "Look for local-ruby organizations"
  }
  
  typical_meetup_format = [
    "Welcome and introductions",
    "Main presentation (20-40 minutes)",
    "Lightning talks or show & tell",
    "Q&A and discussion",
    "Social time (often at a nearby venue)"
  ]
end
```

## Contributing to Ruby

### Contributing to Ruby Core

```bash
# Get the Ruby source
git clone https://github.com/ruby/ruby.git
cd ruby

# Build Ruby
./autogen.sh
./configure
make

# Run tests
make test-all

# Make your changes, then submit
# 1. Create a ticket at bugs.ruby-lang.org
# 2. Submit a pull request on GitHub
# 3. Be patient and responsive to feedback
```

### Contributing to Documentation

```ruby
# Ruby's documentation needs help!
# Ways to contribute:

# 1. Fix typos and errors
# 2. Add examples to methods
# 3. Improve explanations
# 4. Translate documentation

# Example: Adding documentation
class Array
  # Returns a new array containing elements common to both arrays.
  #
  # @param other_array [Array] the array to compare with
  # @return [Array] the intersection of both arrays
  #
  # @example
  #   [1, 2, 3] & [2, 3, 4]  #=> [2, 3]
  #   ["a", "b"] & ["b", "c"] #=> ["b"]
  def &(other_array)
    # implementation
  end
end
```

### Contributing to Gems

```ruby
# Find gems to contribute to:
# 1. Gems you use daily
# 2. Gems with "help wanted" labels
# 3. Abandoned gems that need maintenance

# Good first contributions:
class FirstContribution
  def ideas
    [
      "Add tests for untested code",
      "Update outdated dependencies",
      "Improve documentation",
      "Fix deprecation warnings",
      "Add examples to README",
      "Fix issues labeled 'good first issue'"
    ]
  end
  
  def process
    <<~STEPS
    1. Fork the repository
    2. Create a feature branch
    3. Make your changes
    4. Add tests
    5. Update documentation
    6. Submit a pull request
    7. Respond to feedback
    8. Celebrate when merged!
    STEPS
  end
end
```

## Ruby Heroes and Influencers

### The Creators and Core Team

**Yukihiro "Matz" Matsumoto**
- Creator of Ruby
- Still actively involved
- Sets the tone for the community

**Core Committers**
- Koichi Sasada (YARV, performance)
- Aaron Patterson (tenderlove, Rails/Ruby core)
- Nobuyoshi Nakada (nobu, prolific contributor)
- Samuel Williams (async Ruby)

### Community Leaders

**Sandi Metz**
- Author of "Practical Object-Oriented Design in Ruby"
- Conference speaker extraordinaire
- Teacher and mentor

**Avdi Grimm**
- RubyTapas screencasts
- "Confident Ruby" author
- Thoughtful takes on Ruby and programming

**Sarah Mei**
- Developer, speaker, and advocate
- Co-founder of RailsBridge
- Champion of diversity in tech

**David Heinemeier Hansson (DHH)**
- Creator of Ruby on Rails
- Basecamp CTO
- Controversial but influential

## Learning Resources

### Books That Shaped the Community

```ruby
ESSENTIAL_BOOKS = {
  beginner: [
    "Learn to Program" => "Chris Pine",
    "The Well-Grounded Rubyist" => "David A. Black",
    "Eloquent Ruby" => "Russ Olsen"
  ],
  intermediate: [
    "Practical Object-Oriented Design" => "Sandi Metz",
    "Confident Ruby" => "Avdi Grimm",
    "Effective Ruby" => "Peter Jones"
  ],
  advanced: [
    "Metaprogramming Ruby" => "Paolo Perrotta",
    "Ruby Under a Microscope" => "Pat Shaughnessy",
    "Ruby Performance Optimization" => "Alexander Dymo"
  ],
  classic: [
    "The Ruby Programming Language" => "Matz & Flanagan",
    "Programming Ruby" => "The Pickaxe Book",
    "Why's (Poignant) Guide to Ruby" => "_why"
  ]
}
```

### Online Learning

**Websites and Tutorials**
- Ruby Koans: Learn Ruby through testing
- Exercism.io: Practice problems with mentorship
- CodeWars: Ruby challenges
- RubyMonk: Interactive Ruby tutorials
- GoRails: Screencasts for Rails developers

**Podcasts**
```ruby
def ruby_podcasts
  {
    "Ruby Rogues" => "Panel discussions on Ruby topics",
    "Remote Ruby" => "Casual Ruby conversations",
    "Rails with Jason" => "Interviews with Ruby developers",
    "The Bike Shed" => "thoughtbot's podcast",
    "Ruby on Rails Podcast" => "Brittany Martin's show"
  }
end
```

**YouTube Channels**
- Drifting Ruby
- GoRails
- RubyConf recordings
- RailsConf recordings

## Diversity and Inclusion

The Ruby community actively works toward inclusion:

### RailsBridge

```ruby
class RailsBridge
  def mission
    "Teaching Ruby and Rails to underrepresented groups"
  end
  
  def workshops
    {
      format: "Free weekend workshops",
      target: "Women and non-binary people",
      curriculum: "From zero to first Rails app",
      philosophy: "Create safe learning environment"
    }
  end
  
  def impact
    "Thousands of new developers brought into the community"
  end
end
```

### RubyMe

Mentorship program connecting experienced developers with newcomers.

### Scholarships and Programs

Many conferences offer:
- Opportunity scholarships
- Diversity tickets
- Speaker mentorship
- Travel grants

## The Business of Ruby

### Companies Built on Ruby

```ruby
RUBY_COMPANIES = {
  giants: ["GitHub", "Shopify", "Stripe", "Airbnb", "Square"],
  established: ["Basecamp", "Heroku", "Twitch", "SoundCloud"],
  government: ["UK Government", "NASA", "US Government"],
  startups: "Thousands of startups choose Ruby for rapid development"
}
```

### Ruby Jobs

```ruby
def finding_ruby_jobs
  {
    job_boards: [
      "jobs.rubyonrails.org",
      "weworkremotely.com",
      "remoteok.io",
      "angel.co",
      "indeed.com"
    ],
    typical_roles: [
      "Ruby Developer",
      "Rails Engineer", 
      "Full-Stack Developer",
      "DevOps Engineer",
      "Site Reliability Engineer"
    ],
    salary_ranges: {
      junior: "$60k-$90k",
      mid: "$90k-$130k",
      senior: "$130k-$180k+",
      staff: "$180k-$250k+",
      note: "Varies greatly by location and company"
    }
  }
end
```

## Giving Back

### Ways to Contribute

```ruby
class CommunityContribution
  def non_code_contributions
    [
      "Answer questions on Stack Overflow",
      "Write blog posts about your learning",
      "Create tutorials or videos",
      "Organize local meetups",
      "Mentor newcomers",
      "Report bugs you find",
      "Improve documentation",
      "Share your projects",
      "Speak at meetups",
      "Live-tweet conferences for those who can't attend"
    ]
  end
  
  def start_small
    <<~ADVICE
    - Your first contribution doesn't have to be code
    - Every contribution matters, no matter how small
    - Share what you learn, even as a beginner
    - Your perspective as a newcomer is valuable
    - Help the next person who's where you were
    ADVICE
  end
end
```

## Community Traditions and Culture

### Ruby Culture

```ruby
module RubyTraditions
  INSIDE_JOKES = {
    "chunky_bacon" => "From _why's guide",
    "MINASWAN" => "Matz is nice and so we are nice",
    "Ruby Tuesday" => "Community Twitter tradition",
    "Refinements" => "The feature everyone argues about",
    "1.9" => "The traumatic upgrade",
    "symbol#to_proc" => "The &:method shorthand everyone loves"
  }
  
  COMMUNITY_VALUES = {
    developer_happiness: "Code should bring joy",
    creativity: "There's more than one way",
    pragmatism: "Get things done",
    mentorship: "Lift as you climb",
    fun: "Programming doesn't have to be serious"
  }
  
  CONFERENCE_TRADITIONS = {
    lightning_talks: "5-minute talks about anything",
    code_and_coffee: "Morning hacking sessions",
    ruby_karaoke: "Yes, this is a thing",
    sticker_trading: "Laptop stickers are currency",
    hallway_track: "The real conference"
  }
end
```

## Your Ongoing Journey

### Roadmap for Involvement

```ruby
class YourRubyJourney
  def year_one
    [
      "Join local Ruby meetup",
      "Attend a regional conference",
      "Contribute documentation fix",
      "Answer a Stack Overflow question",
      "Share your first project"
    ]
  end
  
  def year_two
    [
      "Give a lightning talk",
      "Contribute code to a gem",
      "Mentor a newcomer",
      "Write blog posts",
      "Attend RubyConf or RailsConf"
    ]
  end
  
  def year_three_and_beyond
    [
      "Speak at a conference",
      "Maintain a popular gem",
      "Organize community events",
      "Contribute to Ruby core",
      "Become the mentor you wished you had"
    ]
  end
  
  def remember
    "Everyone's journey is different. Go at your own pace."
  end
end
```

## The Future of Ruby Community

Ruby's community continues to evolve:

- **Performance**: Ruby 3x3 achieved, now pushing further
- **Typing**: Gradual typing with RBS and Steep
- **Concurrency**: Ractors and async Ruby
- **New developers**: Continuing to welcome newcomers
- **Global reach**: Growing communities worldwide
- **Sustainability**: Supporting maintainers

## Staying Connected

```ruby
def stay_connected
  {
    daily: [
      "Check Ruby Reddit",
      "Follow #ruby hashtag",
      "Read Ruby Weekly newsletter"
    ],
    weekly: [
      "Attend virtual meetups",
      "Watch a conference talk",
      "Try a Ruby puzzle"
    ],
    monthly: [
      "Attend local meetup",
      "Contribute to open source",
      "Write about Ruby"
    ],
    yearly: [
      "Attend a conference",
      "Level up your contributions",
      "Help organize an event"
    ]
  }
end
```

## Your Turn: Community Challenges

1. **First Contribution**: Make your first open source contribution
2. **Local Connection**: Attend or organize a local meetup
3. **Share Knowledge**: Write your first Ruby blog post
4. **Help Someone**: Answer questions or mentor a beginner
5. **Conference Talk**: Submit a talk proposal

## What You've Learned

You now know:
- The values that define Ruby community
- Where to find and connect with Rubyists
- How to contribute to Ruby and its ecosystem
- Resources for continued learning
- Ways to give back to the community
- The culture and traditions of Ruby

## Final Thoughts

The Ruby community is more than a professional network—it's a global family of people who believe programming should be joyful, that kindness matters, and that everyone deserves a chance to learn and grow.

You're not just learning a programming language; you're joining a movement. A movement that says code is creative expression, that programmers are people first, and that technology should serve humanity, not the other way around.

Welcome to the Ruby community. We've been waiting for you.

Remember: MINASWAN - Matz is nice and so we are nice. This isn't just a slogan; it's a promise we make to each other. Be kind, be helpful, be creative, and most importantly, be yourself.

Your journey with Ruby is just beginning, and the community will be with you every step of the way.

```ruby
class You
  include RubyCommunity
  
  def initialize
    @learning = true
    @contributing = true
    @helping_others = true
    @having_fun = true
  end
  
  def welcome_message
    <<~WELCOME
    Welcome to Ruby!
    You're part of something special now.
    We're glad you're here.
    
    Happy coding!
    ❤️ The Ruby Community
    WELCOME
  end
end

you = You.new
puts you.welcome_message
```