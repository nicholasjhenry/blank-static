task :default => :deploy
task :deploy => :"deploy:default"

namespace :deploy do

  desc "Deploy static site to Heroku. Run deploy:setup first"
  task :default do
    begin
      `mv .git .git-tmp`

      puts "Build static site..."
      `bundle exec staticmatic build .`

      puts "Preparing deployment..."
      `tar -zxf .deploy-git.tar.gz`
      `git add ./site`
      `git commit -m 'Update'`

      puts "Deploying to Heroku..."
      `git push heroku master --force`
      `tar -zcf .deploy-git.tar.gz .git`

    ensure
      puts "Cleaning up..."
      `rm -rf .git`
      `mv .git-tmp .git`
      puts "Completed."
    end
  end

  desc "Setup deploy to Heroku. Pass APP_NAME of the Heroku app to create"
  task :setup do
    begin
      app_name = ENV['APP_NAME']
      raise "App name cannot be blank" unless app_name

      `mv .git .git-tmp`
      `git init .`

      puts "Building static site..."
      `bundle exec staticmatic build .`
      
      puts "Creating Heroku app '#{app_name}'..."
      `heroku create #{app_name}`

      puts "Creating config files..."
      File.open(".gems", 'w') {|f| f.write(GEMS) }
      File.open("config.ru", 'w') {|f| f.write(CONFIG) }

      puts "Adding files to git..."
      `git add ./site`
      `git add .gems`
      `git add config.ru`
      `git commit -m "Initial import"`
      `tar -zcf .deploy-git.tar.gz .git`

    ensure
      puts "Cleaning up..."
      `rm -rf .git`
      `mv .git-tmp .git`
      puts "...Completed."
    end
  end

  GEMS  = <<-EOS 
rack --version 1.1.0
rack-try_static
  EOS

  CONFIG = <<-EOS
require 'rack'
require 'rack/contrib/try_static'

use Rack::TryStatic, 
  :root => "site",  # static files root dir
  :urls => %w[/],     # match all requests 
  :try => ['.html', 'index.html', '/index.html'] # try these postfixes sequentially

# otherwise 404 NotFound
run lambda { [404, {'Content-Type' => 'text/html'}, ['whoops! Not Found']]}
  EOS
end
