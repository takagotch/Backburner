### Backburner
---
https://github.com/nesquena/backburner

```
sudo apt-get install beanstalkd
gem 'backburner'
bundle
gem install backburner

QUEUE=newsletter-sender,push-notifier rake backburner:work
bundle exec backburner -q newsletter-sender,push-notifier -d -P /var/run/backburner.pid -l /var/log/backburner.log

QUEUE=newletter-sender,push-message THREADS=2 GARBAGE-1000 rake backburner:threads_on_fork:work
bundle exec backburer -q queue1:4

rake backburner:work
QUEUE=newsletter-sender,push-noifier rake backburner:work

backbuner -q send_mail,create_thumbnail


```

```ruby
Backburner.configure do |config|
  config.beanstalk_url = "beanstalk://127.0.0.1"
  config.tube_namespace = "some.app.production"
  config.namespace_separator = "."
  config.on_error = lambda { |e| puts e }
  config.max_job_retries = 3
  config.retry_delay = 2
  config.retry_delay_proc = lambda { |min_retry_delay, num_retries| min_retry_delay + (num_retries ** 3) }
  config.default_priority = 65536
  config.logger = Logger.new(STDOUT)
  config.primary_queue = "backburner-jobs"
  config.priority_labels = { :custom => 50, :useless => 1000 }
  config.reserve_timeout = nil
  config.job_serializer_proc = lamdba { |body| JSON.dump(body) }
  config.job_parser_proc = lambda { |body| JSON.parse(body) }
end

class NewsletterJob
  def self.perform(email, body)
    NewsletterMailer.deliver_text_to_email(email, body)
  end
  def self.queue
    "newsletter-sender"
  end
  def self.queue_priority
    1000
  end
  def self.queue_respond_timeout
    300
  end
end

class NewsletterJob
  include Backburner::Queue
  queue "newsletter-sender"
  queue_priority 1000
  queue_respond_timeout 300
  def self.perform(email, body)
    NewsletterMailer.deliver_text_to_email(email, body)
  end
end

Backburner.enqueue NewsletterJob, 'foo@admin.com', 'lorem ipsum...'

class User
  include Backburner::Performable
  queue "user-jobs"
  queue_priority 500
  queue_respond_timeout 300
  def activate(device_id)
    @device = Device.find(device_id)
  end
  def self.reset_password(user_id)
  end
end
@user = User.first
@user.async(:ttr => 100, :queue => "activate").activate(@device.id)
User.async(:pri => 100, :delay => 10.second).reset_password(@user.id)

User.async(:queue => lambda { |user_klass| ["queue1","queue2"].sample(1).first }).do_hard_work

class User
  include Backburner::Performable
  def send_welcome_email
  end
  handle_asynchronously :send_welcome_email, queue: 'send-mail', pri: 5000, ttr: 60
  def self.update_recent_visitors
  end
  handle_static_asynchronously :update_resent_visitors, queue: 'long-tasks', ttr: 300
end

Backburner::Performable.handle_asynchronously(User, :activate, ttr: 100, queue: 'activate')

Backburner.work

Backburner.work('newsletter-sender', 'push-notifier')

Backburner::Worker.enqueue(NewletterJob, ['foo@admin.com', 'lorem ipsum...'], :delay => 1.hour)
User.async(:delay => 1.hour).reset_password(@user.id)

User.async.reset_password(@user.id)

@device = Device.find(987)
@user = User.find(987)
@user.async.activate(@device.id)

Backburner.configure do |config|
  config.priority_labels = { :custom => 50, :useful => 5 }
  # config.priority_labels = Backburner::Configuration::PRIORITY_LABELS.merge(:foo => 5)
end

Backburner::Worker.enqueue NewsletterJob, ["foo", "bar"], :pri => :custom
User.async(:pri => :useful, :delay => 10.seconds).reset_password(@user.id)

Backburner.configure do |config|
  config.default_worker = Backburner::Workers::Forking
end

Backburner.work('newsletter-sender', :worker => Backburner::Workers::ThreadOnFork)

Backburner::Worker.know_queue_classes
Backburner.default_queues.concat(["foo", "bar"])

Backburner.default_queues << NewsletterJob.queue

Backburner.configure do |config|
  config.max_job_retries
  config.retry_delay
end

Backburner.configure do |config|
  config.retry_delay = 2
  config.retry_delay_proc = lambda { |min_retry_delay, num_retries| min_retry_delay + (num_retries ** 3) }
end

Backburner.configure do |config|
  config.on_error = lambda { |ex| Airbrake.notify(ex) }
end

Backburner.configure do |config|
  config.logger = Logger.new(STDOUT)
end

require 'backburner/tasks'

backburner -q send_mail,create_thumbnail

path="/var/www/my-app/current"
backburner -r "$path"

environment="production"
backburner -e $environment

```

```
```
