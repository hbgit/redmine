Based on https://www.redmine.org/projects/redmine/wiki/HowTo_Install_Redmine_(3xx)_on_Heroku_

First, git clone to Redmine stable source

$ git clone https://github.com/redmine/redmine.git -b 4.2-stable

And move to redmine project directory

$ cd redmine

Edit .gitignroe file, and remove such line's

Gemfile.lock
Gemfile.local
public/plugin_assets 
config/initializers/session_store.rb 
config/initializers/secret_token.rb 
config/configuration.yml 
config/email.yml

Editing Gemfile, remove or comment this block

database_file = File.join(File.dirname(__FILE__), "config/database.yml")
 if File.exist?(database_file)
   database_config = YAML::load(ERB.new(IO.read(database_file)).result)
   adapters = database_config.values.map {|c| c['adapter']}.compact.uniq
   if adapters.any?
     adapters.each do |adapter|
       case adapter
       when 'mysql2'
         gem "mysql2", "~> 0.4.6", :platforms => [:mri, :mingw, :x64_mingw]
       when /postgresql/
         gem "pg", "~> 0.18.1", :platforms => [:mri, :mingw, :x64_mingw]
       when /sqlite3/
         gem "sqlite3", (RUBY_VERSION < "2.0" && RUBY_PLATFORM =~ /mingw/ ? "1.3.12" : "~>1.3.12"),
                        :platforms => [:mri, :mingw, :x64_mingw]
       when /sqlserver/
         gem "tiny_tds", (RUBY_VERSION >= "2.0" ? "~> 1.0.5" : "~> 0.7.0"), :platforms => [:mri, :mingw, :x64_mingw]
         gem "activerecord-sqlserver-adapter", :platforms => [:mri, :mingw, :x64_mingw]
       else
         warn("Unknown database adapter `#{adapter}` found in config/database.yml, use Gemfile.local to load your own database gems")
       end
     end
   else
    warn("No adapter found in config/database.yml, please configure it first")
   end
 else
  warn("Please configure your config/database.yml first")
 end

And add this block

group :production do
  gem 'pg', '~> 0.20'
end


install the gem's to bundle install

$ bundle install

get secret token with this command

$ rake generate_secret_token 

In config/enviroment.rb edit to "exit 1" (remove or comment)

 exit 1

Create to Heroku App

$ heroku apps:create -a APP_NAME

Add, postgreaql addon's

$ heroku addons:create heroku-postgresql -a APP_NAME

Link to remote (heroku)

$ heroku git:remote -a APP_NAME
$ heroku stack:set heroku-18

Deploy to Heroku these command

$ git add -A
$ git commit -m “Redmine for Heroku deployment”
$ git push heroku 3.4-stable:master

Finally, migrate & loding default data

$ heroku run rake db:migrate
$ heroku run rake redmine:load_default_data

===================================

Logging heroku via bash:
I can do with this commands

heroku login
heroku run bash -a APPNAME
$ cd app








