---
layout: post
title:      "Final project"
date:       2020-02-24 00:44:14 +0000
permalink:  final_project
---


I'm finally done with my final class project. It was a super informative process to build a React/Rails application. Here are some resources that were really helpful with making a game that makes use of Rails Action Cable. 

https://medium.com/@dakota.lillie/using-action-cable-with-react-c37df065f296
This was a great article with a good overview. The only problem with it is that the ActionCable Provider package is not the best. I found that I got better results with simply importing the official ActionCable package. Then I integrated it into my app as such: 

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
  }
	
	...
	
	handleReceived = (response) => {
    const json = JSON.parse(response);
    this.setState({
      challengerIds: [...this.state.challengerIds, json.challenger_id]
    });
  }

  handleChallenge = response => {
    const json = JSON.parse(response);
    if (json.accepter_board) {
      this.props.addBoard(json.accepter_board);
      this.props.addBoard(json.challenger_board);
      broadcastInGame(this.props.currentPlayer.id);
      this.props.storeOpponent(this.findChallenger(json.accepter_board.player_id));
    }
  }
	
	...
```

I found that I had to turn sessions on to get ActionCable to identify the connection as belonging to the current user. As a tutorial on turning sessions on, I found this video by Howard very useful:
https://www.youtube.com/watch?time_continue=814&v=t1n54yYgRXo&feature=emb_logo

This was also useful in that regard:
https://pragmaticstudio.com/tutorials/rails-session-cookies-for-api-authentication

Finally I needed to do some digging to figure out how to deploy my app and use ActionCable in the deployment. These articles were super helpful on those points:
https://medium.com/how-i-get-it/rails-react-js-heroku-deployment-43d7469e122e

https://blog.heroku.com/real_time_rails_implementing_websockets_in_rails_5_with_action_cable#deploying-our-application-to-heroku

All in all I came into this section finding React a bit daunting, but now I am quite enamored with how it can scaffold a project and make things easier. I'm really pleased with how the course was structured very thoughtfully and with the technologies it chose to teach; I think it did a great job integrating a lot of things, covering a lot of ground, and providing a solid yet approachable foundation. Well done Flatiron!
