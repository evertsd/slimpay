[![Build Status](https://travis-ci.org/novagile/slimpay.svg)](https://travis-ci.org/novagile/slimpay)
[![Gem Version](https://badge.fury.io/rb/slimpay.svg)](https://badge.fury.io/rb/slimpay)

# Slimpay

Ruby implementation of the [Slimpay Hypermedia API](https://api.preprod.slimpay.com/docs/).

Slimpay website : [slimpay.com](https://www.slimpay.com/)

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'slimpay'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install slimpay


## Usage

**API Docs: https://api.preprod.slimpay.com/docs/**

### Configuration

If you use _Rails_ place this code in _config/initializers/slimpay.rb_:

```ruby
Slimpay.configure do |config|
  config.client_id = "your_client_id"
  config.client_secret = "your_client_secret"
  config.creditor_reference = "your_creditor_reference"
  config.sandbox = true
  config.notify_url = 'you_notifications_url'
  config.return_url = 'your_return_url'
end
```

### Notification and return URLs

The notification URL is the point of your application where Slimpay will send a POST request with the result of the current order.
The state of the order will be either **close.completed** or **closed.aborted.aborted_byclient** if everything worked normally, or one of those quoted in the [API Order documentation](https://api.preprod.slimpay.com/docs/alps/v1/orders/).

The return URL is the point of your application where your user/customer will be redirected to when he will have finish signing its mandate with Slimpay.

### The root endpoint:

The Slimpay API uses self-discovery. It means that each time you will perform a request, the answer will be a Hash of links to follow in order to perform more requestq.

The more you do requests, the more methods will appear.

When you emplement any class, it will inherits from the root-endpoint and thus already have available methods.

```ruby
slimpay = Slimpay::Base.new(client_id = '1234', client_secret = '987654321', creditor_reference = 'azerty')
slimpay.api_methods
```
:warning: If you call ```Slimpay::Base.new``` without arguments, the _Sandbox_ credentials will be used.

Result will be a Hash:

```ruby
{"self"=>"https://api.preprod.slimpay.com/"
"post_token"=>"https://api.preprod.slimpay.com/oauth/token",
"create_orders"=>"https://api.preprod.slimpay.com/orders",
"get_creditors"=>"https://api.preprod.slimpay.com/creditors{?reference}",
"get_orders"=>"https://api.preprod.slimpay.com/orders{?creditorReference,reference}",
"get_mandates"=>"https://api.preprod.slimpay.com/mandates{?creditorReference,rum}",
"create_documents"=>"https://api.preprod.slimpay.com/documents",
"get_documents"=>"https://api.preprod.slimpay.com/documents{?creditorReference,entityReference,reference}",
"create_direct_debits"=>"https://api.preprod.slimpay.com/direct-debits",
"get_direct_debits"=>"https://api.preprod.slimpay.com/direct-debits{?id}",
"create_recurrent_direct_debits"=>"https://api.preprod.slimpay.com/recurrent-direct-debits",
"get_recurrent_direct_debits"=>"https://api.preprod.slimpay.com/recurrent-direct-debits{?id}",
"get_card_transactions"=>"https://api.preprod.slimpay.com/card-transactions{?id}",
"get_card_transaction_issues"=>"https://api.preprod.slimpay.com/card-transaction-issues{?id}",
"profile"=>"https://api.preprod.slimpay.com/alps/v1"}
```

The keys of this Hash are the methods name you can call on the class instance (here Slimpay::Base).
The value is the URL that will be used, with its arguments.

**Example:**

```ruby
"get_orders"=>"https://api.preprod.slimpay.com/orders{?creditorReference,reference}",
```

The arguments will be _creditorReference_ and _reference_. You can give them as a Hash.
See below for an example.

Some methods as been added to this gem as shortcuts to these root methods.

### Available resources :
**Order**, **Mandate**, **App**

Each resource inherit from _Resource_ wich itself inherits from _Base_.
_Base_ defines root methods according to the Slimpay API.

**Example:**

The official API method:

```ruby
orders = Slimpay::Order.new(client_id = '1234', client_secret = '987654321', creditor_reference = 'azerty')
orders.get_orders({creditorReference: 'mysellername', reference: 1234})
```

The shortcut:

```ruby
orders = Slimpay::Order.new(client_id = '1234', client_secret = '987654321', creditor_reference = 'azerty')
orders.get_one(1234)
```

Result will be a Hash:

```json
{"_links"=>
  {"self"=>{"href"=>"https://api.preprod.slimpay.com/creditors/democreditor/orders/1"},
   "https://api.slimpay.net/alps#get-creditor"=>{"href"=>"https://api.preprod.slimpay.com/creditors/democreditor"},
   "https://api.slimpay.net/alps#get-subscriber"=>{"href"=>"https://api.preprod.slimpay.com/creditors/democreditor/orders/1/subscribers/subscriber01"},
   "https://api.slimpay.net/alps#user-approval"=>{"href"=>"https://slimpay.net/slimpaytpe16/userApproval?accessCode=spK534N0cuZztBGwj2FjC6eKzcsKFRzXbfy8buloUHiZV6p9PhIfcPgV7c507R"},
   "https://api.slimpay.net/alps#get-order-items"=>{"href"=>"https://api.preprod.slimpay.com/creditors/democreditor/orders/1/items"},
   "https://api.slimpay.net/alps#get-mandate"=>{"href"=>"https://api.preprod.slimpay.com/creditors/democreditor/mandates/1"}},
 "reference"=>"1",
 "state"=>"closed.completed",
 "started"=>true,
 "dateCreated"=>"2014-12-12T09:35:39.000+0000",
 "mandateReused"=>false}
```

Now you can call new methods : get_creditor, get_subscriber, user_approval, get_order_items, get_mandate

**NB:** Note that the methods in the resulting Hash are dashed-named, but the generated methods are camelcased.

## Credentials
The sanbox let you test credentials connection and few methods.

You will need a test environment setted up by Slimpay to go further.

### SANDBOX

* client_id =  'democreditor01'
* client_secret = 'demosecret01'
* creditor_reference : democreditor

### Test

* IBAN : FR1420041010050500013M02606
* BIC : PSSTFRPP
* Code for phone verification : 0000

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake rspec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/novagile/slimpay. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](contributor-covenant.org) code of conduct.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

