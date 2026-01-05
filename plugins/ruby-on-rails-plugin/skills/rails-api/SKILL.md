---
name: rails-api
description: Use this skill for ALL Ruby on Rails API development including models, controllers, services, jobs, and testing. Enforces Basecamp/37signals "vanilla Rails" patterns, Standard Ruby style, and TDD workflow. MUST be used proactively when working in services/streambridge/.
---

# Rails API Development Skill

You are an expert Rails engineer building a production API. You follow Basecamp's "vanilla Rails" philosophy - rich domain models, thin controllers, and minimal abstractions. Code should be a pleasure to read.

## CRITICAL: Test-First Development

**Write tests BEFORE implementation:**

```bash
# Run all tests
docker-compose exec api bundle exec rspec

# Run single file
docker-compose exec api bundle exec rspec spec/path/to/file_spec.rb

# Run single spec
docker-compose exec api bundle exec rspec spec/path/to/file_spec.rb:42

# Lint
docker-compose exec api bundle exec standardrb
```

Never commit code without passing tests. Never skip linting.

---

## Philosophy: Vanilla Rails

### Core Principles

From [Vanilla Rails is Plenty](https://dev.37signals.com/vanilla-rails-is-plenty/):

- **Rich domain models** - Business logic belongs in models
- **Thin controllers** - Coordinate, don't compute
- **Concerns for shared behavior** - Not services for everything
- **CRUD resources** - Model actions as resources, not custom endpoints

### When to Use Services

Services are justified for:
- Complex multi-model orchestration (e.g., `StreamOrchestration::StartService`)
- External API integrations (e.g., `LivekitService`)
- Workflows that span multiple bounded contexts

Services are NOT needed for:
- Simple model operations
- Single-model business logic (use model methods)
- Validation (use model validations)

---

## Code Style (Basecamp/37signals)

### Method Ordering

```ruby
class SomeClass
  # 1. Class methods first
  def self.find_by_token(token)
    # ...
  end

  # 2. Public methods (initialize at top)
  def initialize(params)
    @params = params
  end

  def perform
    step_one
    step_two
  end

  # 3. Private methods (ordered by invocation)
  private
    def step_one
      step_one_helper
    end

    def step_one_helper
      # ...
    end

    def step_two
      # ...
    end
end
```

### Private Indentation

Indent content under `private`:

```ruby
class User < ApplicationRecord
  def full_name
    "#{first_name} #{last_name}"
  end

  private
    def normalize_email
      self.email = email.downcase.strip
    end

    def set_defaults
      # ...
    end
end
```

### Conditional Returns

Prefer expanded conditionals over guard clauses:

```ruby
# Bad
def process
  return [] unless items.present?
  items.map(&:transform)
end

# Good
def process
  if items.present?
    items.map(&:transform)
  else
    []
  end
end
```

Exception: Early returns at method start for non-trivial methods:

```ruby
def after_stream_started(stream)
  return if stream.test_mode?

  # Complex logic follows...
  notify_viewers
  start_recording
  update_analytics
end
```

### Bang Methods

Only use `!` when there's a non-bang counterpart:

```ruby
# Good - save has save!
stream.save!

# Bad - no non-bang version exists
def start_stream!  # Don't do this
end

# Good
def start_stream
end
```

---

## Controllers

### CRUD Resources

Model actions as resources, not custom endpoints:

```ruby
# Bad
resources :streams do
  post :start
  post :stop
end

# Good
resources :streams do
  resource :activation, only: [:create, :destroy]
end
```

### Thin Controllers

Controllers coordinate, models compute:

```ruby
class StreamsController < ApplicationController
  def create
    @stream = Current.event.streams.create!(stream_params)
    render json: StreamSerializer.new(@stream)
  end

  def update
    @stream.update!(stream_params)
    render json: StreamSerializer.new(@stream)
  end

  private
    def set_stream
      @stream = Current.event.streams.find(params[:id])
    end

    def stream_params
      params.require(:stream).permit(:name, :srt_mode)
    end
end
```

### Complex Operations

For complex behavior, use intention-revealing model methods:

```ruby
# Controller
class Streams::ActivationsController < ApplicationController
  def create
    @stream.activate!
    render json: StreamSerializer.new(@stream)
  end
end

# Model
class Stream < ApplicationRecord
  def activate!
    transaction do
      update!(status: :active, activated_at: Time.current)
      start_ffmpeg_pod
      notify_viewers
    end
  end
end
```

---

## Models

### Rich Domain Models

Business logic belongs in models:

```ruby
class Stream < ApplicationRecord
  include Broadcastable, Activatable, LivekitIntegration

  belongs_to :event
  belongs_to :creator, class_name: "User"

  has_many :viewers, dependent: :destroy

  scope :active, -> { where(status: :active) }
  scope :for_event, ->(event) { where(event: event) }

  validates :name, presence: true
  validates :stream_key, uniqueness: true

  before_create :generate_stream_key

  def duration
    return 0 unless started_at && ended_at
    ended_at - started_at
  end

  def live?
    status == "active" && ffmpeg_pod_running?
  end

  private
    def generate_stream_key
      self.stream_key ||= SecureRandom.urlsafe_base64(16)
    end
end
```

### Concerns for Shared Behavior

Extract reusable logic into concerns:

```ruby
# app/models/concerns/activatable.rb
module Activatable
  extend ActiveSupport::Concern

  included do
    scope :active, -> { where(status: :active) }

    after_update :broadcast_status_change, if: :saved_change_to_status?
  end

  def activate!
    update!(status: :active, activated_at: Time.current)
  end

  def deactivate!
    update!(status: :inactive, deactivated_at: Time.current)
  end

  private
    def broadcast_status_change
      broadcast_replace_to "stream_#{id}", target: "stream_status"
    end
end
```

---

## Jobs

### Thin Jobs with _later/_now Pattern

Jobs delegate to models:

```ruby
# Model
module Stream::Recording
  extend ActiveSupport::Concern

  def start_recording_later
    Stream::RecordingJob.perform_later(self)
  end

  def start_recording_now
    # Actual recording logic
    update!(recording_started_at: Time.current)
    RecordingService.start(stream_key)
  end
end

# Job
class Stream::RecordingJob < ApplicationJob
  def perform(stream)
    stream.start_recording_now
  end
end
```

---

## Serializers

### Always Use Serializers

Never return raw models or build JSON in controllers:

```ruby
# Bad
render json: @stream.attributes

# Bad
render json: { id: @stream.id, name: @stream.name }

# Good
render json: StreamSerializer.new(@stream)
```

### JSONAPI::Serializer Pattern

```ruby
class StreamSerializer
  include JSONAPI::Serializer

  attributes :id, :name, :status, :stream_key

  attribute :duration do |stream|
    stream.duration.to_i
  end

  attribute :viewer_count do |stream|
    stream.viewers.count
  end

  belongs_to :event
  has_many :viewers
end
```

---

## Testing

### RSpec Style

```ruby
RSpec.describe Stream do
  describe "#activate!" do
    let(:stream) { create(:stream, status: :inactive) }

    it "sets status to active" do
      stream.activate!
      expect(stream.status).to eq("active")
    end

    it "records activation time" do
      freeze_time do
        stream.activate!
        expect(stream.activated_at).to eq(Time.current)
      end
    end
  end
end
```

### Request Specs for APIs

```ruby
RSpec.describe "Streams API" do
  describe "POST /api/v1/events/:event_id/streams" do
    let(:event) { create(:event) }
    let(:user) { create(:user) }

    it "creates a stream" do
      post "/api/v1/events/#{event.id}/streams",
        params: { stream: { name: "Main Camera" } },
        headers: auth_headers(user)

      expect(response).to have_http_status(:created)
      expect(json_response["data"]["attributes"]["name"]).to eq("Main Camera")
    end
  end
end
```

### Real Dependencies, No Mocks

Test with real database, real services where possible:

```ruby
# Good - real database
it "persists the stream" do
  expect { stream.activate! }.to change { stream.reload.status }
end

# Avoid excessive mocking
# Only mock external APIs that can't be tested locally
```

---

## StreamBridge Patterns

### Service Objects (When Justified)

For complex orchestration spanning multiple concerns:

```ruby
# app/services/stream_orchestration/start_service.rb
module StreamOrchestration
  class StartService
    def initialize(stream)
      @stream = stream
    end

    def call
      validate_stream
      start_ffmpeg_pod
      create_livekit_ingress
      update_stream_status
    end

    private
      attr_reader :stream

      def validate_stream
        raise InvalidStreamError unless stream.can_start?
      end

      def start_ffmpeg_pod
        Orchestrators::DockerOrchestrator.start_pod(stream)
      end

      def create_livekit_ingress
        LivekitService.create_ingress(stream)
      end

      def update_stream_status
        stream.update!(status: :starting, started_at: Time.current)
      end
  end
end
```

### Platform Abstraction

```ruby
# app/services/orchestrators/docker_orchestrator.rb
module Orchestrators
  class DockerOrchestrator
    def self.start_pod(stream)
      new(stream).start_pod
    end

    def initialize(stream)
      @stream = stream
    end

    def start_pod
      # Docker-specific implementation
    end
  end
end
```

---

## Checklist Before Completing Rails Work

1. [ ] Tests written and passing (`bundle exec rspec`)
2. [ ] Linting passes (`bundle exec standardrb`)
3. [ ] Controllers are thin - delegate to models
4. [ ] Business logic in models or justified services
5. [ ] Serializers used for all API responses
6. [ ] Private methods indented under `private`
7. [ ] Methods ordered by invocation flow
8. [ ] CRUD resources, not custom actions

---

## Quick Reference

```ruby
# Method ordering
class Foo
  def self.class_method; end
  def initialize; end
  def public_method; end
  private
    def private_method; end
end

# Concerns
module Activatable
  extend ActiveSupport::Concern
  included do
    scope :active, -> { where(status: :active) }
  end
end

# Jobs
def start_later
  StartJob.perform_later(self)
end
def start_now
  # actual work
end

# Serializers
render json: StreamSerializer.new(@stream)

# CRUD resources
resources :streams do
  resource :activation, only: [:create, :destroy]
end
```
