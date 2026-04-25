# Tighten Examples

Before/after examples for reference.

## Over-Parameterized Constructor

```ruby
# BEFORE (Score: -5)
def initialize(mode: "standard", provider: nil, guard: nil,
               approval_system: nil, cost_tracker: nil,
               checkpoint_interval: 5, max_concurrency: 5)
  @mode = mode                           # -3 (might need), -2 (flexible)
  @provider = provider                   # -1 (abstraction)
  @guard = guard                         # -3 (might need)
  @approval_system = approval_system     # -3 (might need)
  @cost_tracker = cost_tracker           # -3 (might need)
  @checkpoint_interval = checkpoint_interval
  @max_concurrency = max_concurrency     # +3 (actually used)
end

# AFTER
def initialize(max_concurrency: 5)
  @max_concurrency = max_concurrency
end
```

## Defensive Error Handling

```ruby
# BEFORE - Have you seen these errors?
begin
  process_step
rescue NetworkError, TimeoutError, ParseError, ValidationError => e
  logger.error("Error in process_step: #{e.message}")
  handle_error(e)
end

# AFTER - Let it fail, fix when it does
process_step
```

## Single-Implementation Interface

```ruby
# BEFORE (Score: -3)
class PaymentProcessor
  def initialize(gateway: StripeGateway.new)
    @gateway = gateway  # -2 (best practice), -1 (abstraction)
  end

  def charge(amount)
    @gateway.charge(amount)
  end
end

class StripeGateway
  def charge(amount)
    Stripe::Charge.create(amount: amount)
  end
end

# AFTER
class PaymentProcessor
  def charge(amount)
    Stripe::Charge.create(amount: amount)
  end
end
```

## AI Comment Slop

```ruby
# BEFORE
# This method calculates the total price for the order
# It iterates through all items and sums their prices
# Returns the total as a Money object
def total_price
  items.sum(&:price)
end

# AFTER
def total_price
  items.sum(&:price)
end
```

## Log-and-Rethrow

```ruby
# BEFORE
def fetch_user(id)
  User.find(id)
rescue ActiveRecord::RecordNotFound => e
  Rails.logger.error("User not found: #{id}")
  raise e
end

# AFTER
def fetch_user(id)
  User.find(id)
end
```

## Single-Includer Concern

```ruby
# BEFORE
module Sluggable
  extend ActiveSupport::Concern

  included do
    before_save :generate_slug
  end

  def generate_slug
    self.slug = name.parameterize
  end
end

class Church < ApplicationRecord
  include Sluggable  # Only includer
end

# AFTER
class Church < ApplicationRecord
  before_save :generate_slug

  def generate_slug
    self.slug = name.parameterize
  end
end
```

## Premature useCallback

```jsx
// BEFORE - No measured perf issue
const handleClick = useCallback(() => {
  setCount((c) => c + 1);
}, []);

// AFTER
const handleClick = () => {
  setCount((c) => c + 1);
};
```

## Over-Configured Service

```ruby
# BEFORE
class EmailService
  def initialize(
    provider: :sendgrid,
    retry_count: 3,
    timeout: 30,
    batch_size: 100,
    rate_limit: 1000
  )
    @provider = provider        # Only one provider exists
    @retry_count = retry_count  # Never retried
    @timeout = timeout          # Default always used
    @batch_size = batch_size    # Never batched
    @rate_limit = rate_limit    # Never rate limited
  end
end

# AFTER
class EmailService
  TIMEOUT = 30

  def send(to:, subject:, body:)
    SendGrid.send(to: to, subject: subject, body: body, timeout: TIMEOUT)
  end
end
```
