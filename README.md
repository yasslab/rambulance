# Rambulance [![build](https://github.com/yuki24/rambulance/actions/workflows/tests.yml/badge.svg)](https://github.com/yuki24/rambulance/actions/workflows/tests.yml) [![Gem Version](https://badge.fury.io/rb/rambulance.svg)](https://rubygems.org/gems/rambulance)

A simple and safe way to dynamically render error pages for Rails apps.

## Features

### Simple and Safe

Rambulance's exceptions app is simple, skinny and well-tested. It  inherits from `ActionController::Base`, so it works fine even if your `ApplicationController` has an issue.

### Flexible

You have full control of which error page to show for a specific exception. It also json rendering (perfect for API apps). It even provides a way to create a custom exceptions app.

### Easy installation and development

You don't have to configure things that every single person has to do and Rambulance does everything for you.

## Installation and Usage

Add this line to your application's Gemfile:

```
gem 'rambulance'
```

And then execute:

```
$ rails g rambulance:install
```

Rambulance's generator can only generate `erb` templates. If you want to use haml or slim templates, please see [How to Convert Your `.erb` to `.slim`](https://github.com/slim-template/slim/wiki/Template-Converters-ERB-to-SLIM) or [html2haml](https://github.com/haml/html2haml).

Now you can start editing templates like `app/views/errors/not_found.html.erb`. Edit, run `rails server` and open [`localhost:3000/rambulance/not_found`](http://localhost:3000/rambulance/not_found)!

## Setting Pairs of Exceptions and HTTP Statuses

Open `config/initializers/rambulance.rb` and to configure the list of pairs of exception/corresponding http status.
For example, if you want to display:

 * 422(unprocessable entity) for `ActiveRecord::RecordNotUnique`
 * 403(forbidden) for `CanCan::AccessDenied`
 * 404(not found) for `YourCustomException`

Then do the following:

```ruby
# config/initializers/rambulance.rb
config.rescue_responses = {
  "ActiveRecord::RecordNotUnique" => :unprocessable_entity,
  "CanCan::AccessDenied"          => :forbidden,
  "YourCustomException"           => :not_found
}
```

## Local Development

### Open `localhost:3000/rambulance/***` in Your Browser

Just open one of the error pages via Rambulance:

 * [`localhost:3000/rambulance/not_found`](http://localhost:3000/rambulance/not_found) or
 * [`localhost:3000/rambulance/internal_server_error`](http://localhost:3000/rambulance/internal_server_error)

This is useful when you want to edit templates without changing Rails configuration.

### Set `consider_all_requests_local` to _false_

Change `config.consider_all_requests_local` to _false_ in `config/environments/development.rb`.

```ruby
config.consider_all_requests_local = false
```

This simulates how your production app displays error pages so you can actually raise an exception in your app and see how it works. Don't forget to change `consider_all_requests_local` back to _true_ after you tried this strategy.

## Custom Exceptions App

If you want to do some more things in a exceptions app, you can also write your own custom exceptions app:

```sh
$ rails g rambulance:exceptions_app
```

It will generate your own custom exceptions app. You can use most techniques you want to use in controllers like `before_filter` and rendering views since it's a grandchild of `ActionController::Base`! However there are still some restrictions, e.g. setting a flash notice works when rendering directly but not when redirecting because the ActionDispatch::Flash middleware is never hit.

**Heavily customizing the exceptions app is strongly discouraged as there would be no guard against bugs that occur in the exceptions app.**

## Testing

Rambulance ships with a test helper that allows you to test an error page generated by Rails. All you have to do is to `include Rambulance::TestHelper` and you will be able to use the `with_exceptions_app` DSDL:

Rspec:

```ruby
include Rambulance::TestHelper

it "shows an error page" do
  with_exceptions_app do
    get '/does_not_exist'
  end

  assert_equal 404, response.status
end
```

Minitest:

```ruby
include Rambulance::TestHelper

test "it shows an error page" do
  with_exceptions_app do
    get '/does_not_exist'
  end

  assert_equal 404, response.status
end
```

Note that testing error pages is not encouraged in Rails as it leads to overuse of the `rescue_from` DSL in controllers.

## Supported Versions

* Ruby 2.3, 2,4, 2,5, 2.6, 2.7, 3.0, 3.1, and JRuby 9.3
* Rails 4.2, 5.0, 5.1, 5.2, 6.0, 6.1, 7.0 and edge

Rambulance doesn't work with Rails 3.1 and below since they don't provide a way to use a custom exceptions app.

## Contributing

1. Fork it ( https://github.com/yuki24/rambulance/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## License

Copyright (c) 2014-2015 Yuki Nishijima. See LICENSE.txt for further details.
