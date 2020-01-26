---
layout: post
title:      "Rails 6 API with Devise and Devise-JWT"
date:       2020-01-26 09:05:54 +0000
permalink:  rails_6_api_with_devise_and_devise-jwt
---

I recently had to create a Rails API with a JavaScript front end for a project.  After scouring through countless blogs, SO posts, videos, the Devise and Devise-JWT docs, and other resources, I was able to get a working app up and running.  I'm writting my final steps in this post to centralize the research into one post.  I'm sure the next time I return to this or if anyone else attempts to follow these steps there will still  be  much googling needed, but I  hope it will at least help.

## File structure:

Start with a directory for the app `mkdir app` and `cd app` into it.  There create the front end directory with `mkdir app-client`.  Choose the names you see fit.

Create the Rails app using the api flag
```
rails new app-api -api
```
Add any other flags you like.  These can be found using `rails new --help`.

## Gems

Open the Gemfile in the root of the rails app.  There, you will need to uncomment `gem 'rack-cors` and add
```
gem 'devise'
gem 'devise-jwt'
```
Also for utility and testing purposes, add the following gems for development and test
```
group :development, :test do
  gem 'pry'
  gem 'faker'
  gem 'fabrication'
  gem 'dotenv-rails'
  gem 'rspec-rails'
end
```

## Bundle

Now is a good time to run in the terminal
```
bundle isntall
```

## CORS

To allow your api to be accessed from an origin different from its own, you'll need to enable CORS.  Without going into what CORS is all about, here are the steps to do it for this occasion.

In your rails app directory, navigate to `config/initializers/cors.rb`.  There you will need to uncomment the  rack-cors middleware.  To expose the token, you'll also need to expose Authorization in resource.  The final product should look like this:
```
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins '*'

    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head],
      expose: ["Authorization"]
  end
```
If you also plan to use cookies, the `origins '*'` will have to be set to a specific server as wildcards are not permitted when credentials is set to true.  Other than origins, to enable cookies add `credentials: true` to resource

## Install Devise
Simple.  Install Devise with
```
rails g devise:install
```
Then, Devise instructions will print to the console.
```
Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. You can copy Devise views (for customization) to your app by running:

       rails g devise:views

```
As this is an api, 3 and 4 do not apply.  Follow 1 and  2 using the exaples for each.


## API Navigational Formats
Open `app-api/config/initializers/devise.rb`.  Locate `config.navigational_formats` using `ctrl-f` and set it to an empty array, `[]`.  This file is pretty huge, but its valuable.  All the details about configuring Devise can be found here.

## Create the Devise model
Create the model that will use Devise for authentication.  More than one model can be used, but we'll stick to one-a users mode-for now.
```
rails g devise user
```
Replace 'user' with whatever you choose to name your model.  This is also a pretty good time to generate any other models/controllers/resources you will need since migrating comes soon.

## Revocation Strategy
There a different methods for token revocation using Devise-JWT that are discussed in the Devise-JWT docs and elsewhere.  For this example, well focus on blacklisting.  Be sure to look into the pros and cons when you choose your strategy.  Or just follow along since, you know, this is all right here already.  Start by running
```
rails g model jwt_blacklist jti:string:index exp:datetime
```
You can verify the migration.  It should  look like this:
```
class CreateJwtBlacklists < ActiveRecord::Migration[6.0]
		    def change
		        create_table :jwt_blacklist do |t|
			t.string :jti, null: false
			t.datetime :exp, null: false
			t.timestamps
		        end
		        add_index :jwt_blacklist, :jti
		    end
		end
```
Then, add the following lines to the `jwt_blacklise.rb` model
```
include Devise::JWT::RevocationStrategies::Blacklist
self.table_name = ‘jwt_blacklist’
```

## Migrate
Run migrations using
```
rails db:migrate
```

## Configure the devise model
To specify the models will be jwt-authenticatable and will use the blacklist strategy, in your devise model-user in this guide-add the following two options:
```
:jwt_authenticatable, jwt_revocation_strategy: JwtBlacklist
```
The final devise options should look like this:
```
devise :database_authenticatable, :registerable,
         :jwt_authenticatable, jwt_revocation_strategy: JwtBlacklist
```
Warning!  These mess with migrations.  If you haven't migrated yet or you need to again after this step, you may need to remove the two options added here (or just comment them) until after the migration succeeds.

## Strong params
If you need to permit additional parameters (in addition to the key Devise uses for authentication-email by default-and password), you'll need to set up strong params.  Devise has a lazy version in the docs, which follows:
```
before_action :configure_permitted_parameters, if: :devise_controller?

		protected
		def configure_permitted_parameters
		    devise_parameter_sanitizer.permit(:signup, keys: [:username])
		end
```
Add this to ApplicationController in application_controller.rb.

## Other Validations and Associations
Now is a good time to add any other validations and/or associations to your models.

##  Secret
You will need a secret key for JWT and add it as environment variable.  Create your`.env` file in	your rails root folder (i.e. app/app-api, in this case).  Then generate a secret key using
```
rails secret
```
Copy the output to the `.env` file as:
```
DEVISE_JWT_SECRET_KEY=secret_output
```
Replacing `secret_output` with the output generated.  The name used here is unimportant as long as it is identical to the name used in the JWT configuration following next.

## JWT Config
Add the JWT configuration to the Devise config file `app/app-api/config/initializers/devise.rb` at the bottom but inside the  do/end.
```
config.jwt do |jwt|
	    jwt.secret = ENV['DEVISE_JWT_SECRET_KEY']
	    jwt.dispatch_requests = [
	      ['POST', %r{^/login$}],
	      ['POST', %r{^/login.json$}]
	    ]
	    jwt.revocation_requests = [
	      ['DELETE', %r{^/logout$}],
	      ['DELETE', %r{^/logout.json$}]
	    ]
	    jwt.expiration_time = 1.day.to_i
	 end
```
Here is where the name must match the key used in the `.env` file.

## Set route endpoints

This is optional step but be aware that future steps may assume you use the endpoint established here.  In the route draw method in `app/app-api/config/routes.rb` move/modify `devise_for :users` (or your :MODEL) to match the following
```
devise_for :users,
 				path: '',
				path_names: {
				    sign_in: 'login',
  				    sign_out: 'logout',
  				    registration: 'signup'
  				},
  				controllers: {
  				    sessions: 'sessions',
  				    registrations: 'registrations'
  				}
```

## Sessions Controller
Create a session controller `sessions_controller.rb` in the app's controllers and inherit Devise's session controller using
```
class SessionsController < Devise::SessionsController
    respond_to :json
end
```
The `respond_to :json` lets Devise know it can respond to json.

## Regisration Controller
This is where I may let you  down a bit.  You don't need to override the Devise Registrations Controller, except to tell it  to respond to json.  However, this is probably where you'll want to handle your ActiveRecord exceptions.  Really all that's needed is
```
Class RegistrationsController < Devise::RegistrationsController
   respond_to :json

```
Like I said, you may still need to do a good bit of googling to iron this (especially this part) out.

## Routes
The devise routes have already been taken care of, but now is a good time to set up your routes for any other controllers/actions you have.


## Profit

At this point you should have an API ready to go.  Test with `rails s`.  
