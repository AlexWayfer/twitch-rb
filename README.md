# Twitch API

This gem simplifies the Twitch API for Ruby users.

Currently uses [Kraken (5) version](https://dev.twitch.tv/docs/v5) because of
missing some data in [new (Helix) API](https://dev.twitch.tv/docs/api),
at least [`User#created_at`](https://github.com/twitchdev/issues/issues/84).

## Install

Add this line to your application's Gemfile:

```ruby
gem 'twitch'
```

And then execute:

```
bundle install
```

Or install it yourself as:

```
gem install twitch
```

## Authentication

You have to initialize `Twitch::Client` with `TwitchOAuth2::Tokens` object,
it can be with or without `:access_token` and `:refresh_token`:

```ruby
tokens = TwitchOAuth2::Tokens.new(
  client: {
    client_id: client_id,
    client_secret: client_secret,
    redirect_uri: redirect_uri
  },
  scopes: scopes,
  # token_type: :user, # default is non-personal `:application`
  # access_token: access_token,
  # refresh_token: refresh_token
)

# If you need for User Access Token:
unless access_token
  link = tokens.authorize_link
  puts "Please, open this link, authenticate and paste `code` query argument here:"
  tokens.code = gets.chomp
end

# Otherwise it will raise the exception for `token_type: :user`
twitch_client = Twitch::Client.new(
  tokens: tokens
)
```

If you want to make requests depending on `access_token`,
you should make sure that tokens are actual or you ready to authenticate,
otherwise direct user to `tokens.authorize_link`.
Please, read [`twitch_oauth2`](https://github.com/AlexWayfer/twitch_oauth2) documentation about it.

If you've passed `refresh_token` to initialization and your `access_token`
is invalid, requests that require `access_token` will automatically refresh it.

Later you can access tokens:

```ruby
twitch_client.tokens # => `TwitchOAuth2::Tokens` instance
twitch_client.tokens.access_token # => 'abcdef'
twitch_client.tokens.refresh_token # => 'ghijkl'
```

## Calls

Calls will return a `Faraday::Response` instance with `#body`
(parsed and symbolized), `#status`, etc.:

```ruby
twitch_client.user('day9tv').body

{
  display_name: 'dustinlakin',
  logo: nil,
  created_at: Time.parse('2011-12-18T18:42:09Z'),
  staff: false,
  updated_at: Time.parse('2013-02-11T23:48:11Z'),
  _id: 26883731,
  name: 'dustinlakin',
  _links: {
    self: 'https://api.twitch.tv/kraken/users/dustinlakin'
  }
}
```

## Usage

### Users

```ruby
# `access_token` is not required
twitch_client.user 'day9tv'
```

----

```ruby
# `access_token` is required
twitch_client.your_user

```

### Teams

```ruby
twitch_client.teams
```

----

```ruby
twitch_client.team 'eg'
```

### Channels

```ruby
twitch_client.channel '44322889'
```

----

```ruby
# `access_token` is required
twitch_client.your_channel
```

----

```ruby
# `access_token` and `channel_editor` scope are required

twitch_client.update_channel(
  '44322889',
  status: 'Ranked Solo Queue',
  game: 'League of Legends'
)
```

----

```ruby
# `access_token` and `channel_commercial` scope are required
# *this is untested*

twitch_client.run_commercial 'lethalfrag', 30
```

### Follows

```ruby
twitch_client.following 'channel_id'
```

----

```ruby
twitch_client.followed 'user_id'
```

----

```ruby
twitch_client.follow_status 'user_id', 'channel_id'
```


### Streams

```ruby
twitch_client.stream 'lethalfrag'
```

----

```ruby
twitch_client.streams
```

----

```ruby
twitch_client.featured_streams
```

----

```ruby
twitch_client.your_featured_streams
```

----

```ruby
# `access_token` is required
twitch_client.your_followed_streams
```

### Games

```ruby
twitch_client.top_games
```

### Search

```ruby
twitch_client.search_streams
```
----

```ruby
twitch_client.search_games
```

### Videos


```ruby
twitch_client.channel_videos 'lethalfrag'
```

----

```ruby
twitch_client.video 12345
```

### Adapters

This gem uses [Faraday](https://lostisland.github.io/faraday/),
which supports different adapters. You can use non-default adapter like:

```ruby
Twitch::Client::CONNECTION.adapter :httpclient
```

[Faraday Adapters documentation]
(https://lostisland.github.io/faraday/adapters/).

Feel free to contribute or add functionality!
