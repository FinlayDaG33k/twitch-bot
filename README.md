# twitch-bot [![CircleCI](https://circleci.com/gh/kritzware/twitch-bot.svg?style=svg&circle-token=3d338af28058e84dde13bee88751a50f55aefab3)](https://circleci.com/gh/kritzware/twitch-bot)
Easily create chat bots for Twitch.tv

## Install
Install via NPM
```
$ npm install twitch-bot
```

## Example
```javascript
const TwitchBot = require('twitch-bot')

const Bot = new TwitchBot({
  username: 'Kappa_Bot'
  oauth: 'oauth:dwiaj91j1KKona9j9d1420',
  channel: 'twitch'
})

Bot.on('join', () => {

  Bot.on('message', chatter => {
    if(chatter.message === '!test') {
      Bot.say('Command executed! PogChamp')
    }
  })
})

Bot.on('error', err => {
  console.log(err)
})
```

## Index
- [Events](https://github.com/kritzware/twitch-bot#events)
  - [`join`](https://github.com/kritzware/twitch-bot#join---)
  - [`message`](https://github.com/kritzware/twitch-bot#message---chatter-object)
  - [`timeout`](https://github.com/kritzware/twitch-bot#timeout---event-object)
  - [`ban`](https://github.com/kritzware/twitch-bot#ban---event-object)
  - [`error`](https://github.com/kritzware/twitch-bot#error---err-object)
  - [`close`](https://github.com/kritzware/twitch-bot#close---)
- [Methods](https://github.com/kritzware/twitch-bot#methods)
  - [`say()`](https://github.com/kritzware/twitch-bot#saymessage-string-err-callback)
  - [`timeout()`](https://github.com/kritzware/twitch-bot#timeoutusername-string-duration-int-reason-string)
  - [`ban()`](https://github.com/kritzware/twitch-bot#banusername-string-reason-string)
  - [`close()`](https://github.com/kritzware/twitch-bot#close)
- [Tests](https://github.com/kritzware/twitch-bot#running-tests)

## Events
### `join - ()`
This event is emitted when a connection has been made to the the channel
#### Usage
```javascript
Bot.on('join', () => ... )
```


### `message - (chatter: Object)`
Emitted when a `PRIVSMSG` event is sent over IRC. Chatter object attributes can be found on the [Twitch developers site](https://dev.twitch.tv/docs/v5/guides/irc/#privmsg-twitch-tags)

#### Usage
```javascript
Bot.on('message', chatter => ... )
```

#### Example Response
```javascript
{ color: '#3C78FD',
  display_name: 'kritzware',
  emotes: '88:18-25',
  id: 'c5ee7248-3cea-43f5-ae44-2916d9a1325a',
  mod: true,
  room_id: 44667418,
  sent_ts: 1501541672959,
  subscriber: true,
  tmi_sent_ts: 1501541673368,
  turbo: false,
  user_id: 44667418,
  user_type: 'mod',
  badges: { broadcaster: 1, subscriber: 0 },
  channel: '#kritzware',
  message: 'This is a message PogChamp',
  username: 'Kritzware' }
  ```

### `timeout - (event: Object)`
Emitted when a user is timed out in the chat. The `ban_reason` attribute is `null` when no reason message is used.

#### Chat Trigger
```javascript
kritzware: "/timeout {user} {duration} {reason}"
```

#### Usage
```javascript
Bot.on('timeout', event => ... )
```

#### Example Response
```javascript
{ ban_duration: 10, // seconds
  ban_reason: 'Using a banned word',
  room_id: 44667418,
  target_user_id: 37798112,
  tmi_sent_ts: 1503346029068,
  type: 'timeout',
  channel: '#kritzware',
  target_username: 'blarev' }
```

### `ban - (event: Object)`
Emitted when a user is permanently banned from the chat. The `ban_reason` attribute is `null` when no reason message is used.

#### Usage
```javascript
Bot.on('ban', event => ... )
```

#### Chat Trigger
```javascript
kritzware: "/ban {user} {reason}"
```

#### Example Response
```javascript
{ ban_reason: 'Using a banned word',
  room_id: 44667418,
  target_user_id: 37798112,
  tmi_sent_ts: 1503346078025,
  type: 'ban',
  channel: '#kritzware',
  target_username: 'blarev' }
```

### `error - (err: Object)`
Emitted when any errors occurs in the Twitch IRC channel, or when attempting to connect to a channel.

#### Error types
##### `Login authentication failed`
This error occurs when either your twitch username or oauth are incorrect/invalid.

Response:
```javscript
{ message: 'Login authentication failed' }
```

##### `Improperly formatted auth`
This error occurs when your oauth password is not formatted correctly. The valid format should be `"oauth:your-oauth-password-123"`.

Response:
```javscript
{ message: 'Improperly formatted auth' }
```

#### Usage
```javascript
Bot.on('error', err => ... )
```

#### Example Response
```javascript
{ message: 'Some error happened in the IRC channel' }
```

### `close - ()`
This event is emitted when the irc connection is destroyed via the `Bot.close()` method.
#### Usage
```javascript
Bot.on('close', () => {
  console.log('closed bot irc connection')
})
```

## Methods
### `say(message: String, err: Callback)`
Send a message in the currently connected Twitch channel. An optional callback is provided for validating if the message was sent correctly.

#### Example
```javascript
Bot.say('This is a message')

Bot.say('Pretend this message is over 500 characters', err => {
  sent: false,
  message: 'Exceeded PRIVMSG character limit (500)'
  ts: '2017-08-13T16:38:54.989Z'
})
```

### `timeout(username: String, duration: int, reason: String)`
Timeout a user from the chat. Default `duration` is 600 seconds. Optional `reason` message.

#### Example
```javascript
Bot.timeout('kritzware', 10)
// "kritzware was timed out for 10 seconds"

Bot.timeout('kritzware', 5, 'Using a banned word')
// "kritzware was timed out for 5 seconds, reason: 'Using a banned word'"

Bot.on('message', chatter => {
  if(chatter.message === 'xD') Bot.timeout(chatter.username, 10)
})
```

### `ban(username: String, reason: String)`
Permanently ban a user from the chat. Optional `reason` message.

#### Example
```javascript
Bot.ban('kritzware')
// "kritzware is now banned from the room"

Bot.timeout('kritzware', 'Using a banned word')
// "kritzware is now banned from the room, reason: 'Using a banned word'"

Bot.on('message', chatter => {
  if(chatter.message === 'Ban me!') Bot.ban(chatter.username)
})
``` 

### `close()`
Closes the Twitch irc connection. Bot will be removed from the Twitch channel AND the irc server.

#### Example
```javascript
Bot.close()
```

## Running Tests
Running the test suite requires at least two twitch accounts, one moderator account and one normal account. The channel used must be the same - This is so timeout/ban methods can be tested with the mod account. Using these two accounts, set the following environment variables:
```javascript
TWITCHBOT_USERNAME=mod_username
TWITCHBOT_OAUTH=oauth:mod-oauth-token
TWITCHBOT_CHANNEL=mod_channel
TWITCHBOT_USERNAME_NON_MOD=non_mod_username
TWITCHBOT_OAUTH_NON_MOD=oauth:non-mod-oauth-token
TWITCHBOT_CHANNEL_NON_MOD=mod_channel
```
To run the tests (powered with [Mocha](https://mochajs.org/)), use the following command:
```bash
yarn test
```
