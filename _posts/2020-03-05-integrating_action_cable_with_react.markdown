---
layout: post
title:      "Integrating Action Cable with React"
date:       2020-03-05 18:01:56 +0000
permalink:  integrating_action_cable_with_react
---


My final React project for my Flatiron School bootcamp was [fuddll](https://github.com/tan-nate/fuddll)). I wanted to make a turn-based game kind of like Battleship, but when I got to the part of the project where I was faced with creating the turn-based game flow, I quickly realized I was stumped. How would I force a refresh of the opponent's page when one player submitted a move? 

After Googling around, I discovered WebSockets, and Rails Action Cable, which is the Rails implementation of WebSockets. I came across this excellent article discussing how to implement Action Cable with a React front end: [Using Action Cable With React](https://medium.com/@dakota.lillie/using-action-cable-with-react-c37df065f296). However, there were a few tweaks that I needed to make to the instructions given in this article in order to get it to work with my app. 

Without repeating the article too much, I'll go over the main takeaways. To get Action Cable to work, you need to uncomment the 'redis' and 'rack-cors' gems in your Gemfile. Then, the file 'config/initializers/cors.rb' must be configured similarly to the following: 

```
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'localhost:3001', 'www.fuddll.com'

    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head],
      credentials: true
  end
end

```

The first origin, 'localhost:3001', was the endpoint for my React frontend. 'www.fuddll.com' was my production endpoint. 

'credentials: true' is written to enable Rails sessions, which I'll discuss now. I found that I needed to enable sessions in order to match each WebSockets request to the correct authenticated user. Depending on your implementation of Action Cable, this may not be necessary. However, for my implementation, it was. If your app does not have to handle situations where an Action Cable broadcast must be sent if a user closes their browser, it may not be necessary. You can match broadcasts to an authenticated user in your Controller, using whatever current_user method you have defined there. However, if you are using Action Cable to, for example, render a dynamic lobby that shows individuals entering and leaving an app or message board, it will be necessary. 

To enable Rails sessions, navigate to 'config/application.rb'. Under `config.api_only = true`, paste the following two lines of code:

```
config.middleware.use ActionDispatch::Cookies
config.middleware.use ActionDispatch::Session::CookieStore, key: '_cookie_name'
```

Now, in all of your requests between React and Rails other than GET, you must include `credentials: "include"` as a header. For example:

```
  const headers = {
    method: 'POST',
    headers: {
      "Content-Type": "application/json",
      "Accept": "application/json"
    }, 
    credentials: "include",
    body: JSON.stringify({ board_id: boardId }),
  }

  fetch('/broadcast_fuddll', headers)
```

That's it as far as enabling sessions. Of course, at some point in your app, you need to store the user's id in a cookie. For example:

```
  def login_player(player)
    player.update(logged_in: true, in_game: false)
    session[:player_id] = player.id
    cookies.signed[:player_id] = player.id
  end
```

I found that the ActionCable::Connection class didn't have access to the session variable, which is the reason for the line, `cookies.signed[:player_id] = player.id`. I still used the session variable in my ActionController classes. 

Now, we need to set up our ActionCable::Connection Class to correctly identify the current user with their session. This way, if they close their browser, ActionCable will broadcast to all relevant subscribers that they have left. This was my implementation:

```
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_player

    def connect
      self.current_player = find_verified_player
    end

    private

    def find_verified_player
      if verified_player = Player.find(cookies.signed[:player_id])
        verified_player
      else
        reject_unauthorized_connection
      end
    end
  end
end
```

Then, in a given channel, for example a Game channel, if a player closes their browser, their opponent can be alerted of this in the following way:

```
class GamesChannel < ApplicationCable::Channel
  ...
	def unsubscribed
    ...
    GamesChannel.broadcast_to current_player.games.last, {out_of_game: current_player.id}.to_json
  end
end
```

The 'unsubscribed' method is the relevant method here. Because we identified our Action Cable connection with 'current_player' in the Connection class, we have access to this variable in the subsequent Channels that involve that player. When a player leaves a game, the 'unsubscribed' method is called, and a broadcast can be sent that refers to the 'current_player' that just left. 

I'm going to avoid delving further into implementing Channels, what the 'subscribed' method does, and how Controller actions can be integrated with Channel broadcasts to create real-time updates in your app, because I feel that the article cited at the top of this article explains all this very eloquently and simply. 

However, skipping forward into React, there were a few tweaks that I needed to make to the article's instructions in order to get it to work. 

The article references an npm package called 'react-actioncable-provider'. I unfortunately found that this package didn't work for me. 

Instead, I implemented the official Rails 'actioncable' package. You can do this by typing 'npm install actioncable' into your terminal. 

Then, in a React Component that needs to receive broadcasts, follow the following implementation:

```
import ActionCable from 'actioncable';

...

  componentDidMount() {
    const cable = ActionCable.createConsumer(WS_URL);

    cable.subscriptions.create({
      channel: 'RequestsChannel', 
      player: this.props.currentPlayer.id,
    }, {
      received: response => {this.handleReceived(response)},
    });

    cable.subscriptions.create({
      channel: 'ChallengesChannel',
      player: this.props.currentPlayer.id,
    }, {
      received: response => {this.handleChallenge(response)},
    });

    cable.subscriptions.create("PlayersChannel", {
      received: response => {this.handleInGame(response)},
    });
```

'WS-URL' was 'ws://localhost:3000/cable' in my case (be sure to add `mount ActionCable.server => '/cable'` to your 'config/routes.rb' file). 

Notice that when subscribing to a channel that uses 'stream_for' rather than 'stream_from', associating the channel with an object rather than a string, the syntax for subscribing to the channel in React is a bit different. The first argument in the 'subscriptions.create()' function is an object rather than a string, with two key-value pairs: the name of the channel, and the object passed to the channel in order to find the correct channel instance. Whereas, when subscribing to a 'stream_from' channel, the first argument is simply the string identifying the channel. This subtle difference caused me a lot of headache. 

That's basically it! Again, I strongly urge you to read the article posted at the top for a much better explanation of the fundamentals of Action Cable, and the basic way to use it with React, as it does this extremely well. 
