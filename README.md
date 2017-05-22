# OmniAuth HrSystem

Framgia HrSystem OAuth2 Strategy for OmniAuth.

## Installing

Add to your `Gemfile`:

```ruby
gem "omniauth-hrsystem", git: "git@github.com:dieunb/omniauth-hrsystem.git", branch: "master"
```

Then `bundle install`.

## Usage

`OmniAuth::Strategies::HrSystem` is simply a Rack middleware. Read the OmniAuth docs for detailed instructions: https://github.com/intridea/omniauth.

### With Devise gem
Here is a possible configuration for `config/initializers/devise.rb:

```ruby
config.omniauth :hr_system, ENV["APP_ID"], ENV["APP_SECRET"]
```

`routes.rb`

```ruby
devise_for :users,
  controllers: {
    omniauth_callbacks: "omniauth_callbacks"
  }
```

`omniauth_callbacks_controller.rb`

```ruby
class OmniauthCallbacksController < Devise::OmniauthCallbacksController
  skip_before_action :authenticate_user!

  def create
    auth = request.env["omniauth.auth"]
    @user = User.from_omniauth(auth)

    if @user.persisted?
      set_flash_message(:notice, :success, kind: auth.provider) if is_navigational_format?
      sign_in_and_redirect user
    else
      flash[:notice] = "Auth failure"
      redirect_to root_path
    end
  end

  def failure
    flash[:notice] = "Auth failure"
    redirect_to root_path
  end

  alias_method :hr_system, :create
end
```

### Without Devise gem

Here"s a quick example, adding the middleware to a Rails app in `config/initializers/omniauth.rb`:

```ruby
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :hr_system, ENV["HRSYSTEM_KEY"], ENV["HRSYSTEM_SECRET"]
end
```

`routes.rb`

```ruby
get "/auth/:provider/callback", to: "omniauth_callbacks#create"
get "/auth/failure", to: "omniauth_callbacks#failure"
```

`omniauth_callbacks_controller.rb`

```ruby
class OmniauthCallbacksController < ApplicationController
  skip_before_action :authenticate_user!

  def create
    auth = request.env["omniauth.auth"]
    @user = User.from_omniauth(auth)

    if @user.persisted?
      log_in @user
      flash[:success] = "Sign in with #{auth.provider}"
      redirect_to root_path
    else
      flash[:notice] = "Auth failure"
      redirect_to root_path
    end
  end

  def failure
    flash[:notice] = "Auth failure"
    redirect_to root_path
  end
end
```

`from_omniauth` function

```ruby
class << self
  def from_omniauth auth
    user = find_or_initialize_by(email: auth.info.email)
    user.full_name = auth.info.name
    user.first_name = auth.info.name.split(" ").last
    user.provider = auth.provider
    user.password = User.generate_unique_secure_token if user.new_record?
    user.token = auth.credentials.token
    user.refresh_token = auth.credentials.refresh_token
    user.save
    user
  end
end
```
