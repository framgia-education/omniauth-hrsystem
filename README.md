# OmniAuth HrSystem

Framgia HrSystem OAuth2 Strategy for OmniAuth.

## Installing

Add to your `Gemfile`:

```ruby
gem "omniauth-hrsystem"
```

Then `bundle install`.

## Usage

`OmniAuth::Strategies::HrSystem` is simply a Rack middleware. Read the OmniAuth docs for detailed instructions: https://github.com/intridea/omniauth.

Here"s a quick example, adding the middleware to a Rails app in `config/initializers/omniauth.rb`:

```ruby
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :hr_system, ENV["HRSYSTEM_KEY"], ENV["HRSYSTEM_SECRET"]
end
```
