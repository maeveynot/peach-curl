# peach-curl

A little command line wrapper for `curl`ing the (not officially
documented) [Peach](http://peach.cool/) API.

# Quick Start

1. Install [jq](https://stedolan.github.io/jq/).
2. Put `peach` somewhere in your PATH: `install peach /usr/local/bin`
3. Run `peach` once to log in (this stores the response in `~/.peach-session`).
4. Run it again to get a list of known API endpoints and a few example
   commands. For more details on the examples, see below.

# Options

- `-f`: use a different session file (instead of `~/.peach-session`).
- `-n`: don't authenticate (if the session file exists), or try to login
  (if it does not). Useful for registering a new account.

# Examples

Get the contents of your friends' streams (this also updates the time
you were seen "online"):

    peach /connections

Mark your activity as read:

    peach /activity/read -X PUT

Make an unauthenticated request to check if an account is available
(this example is the official Team Peach account, so it's not):

    peach -n /precheck/create/stream -d '{"name": "peach"}'

Make a text post:

    peach /post -d "$(jq -sRc '{message: [{type: "text", text: .}]}' post-body.txt)"

List names of users you have blocked:

    peach /stream/block-list | jq -r '.data.blockList[].name'

Get your unread activity, saving previous activity in `activity.json` to
show only new items:

    if peach /activity/isUnread | jq -e '.data.isUnreadActivity' >/dev/null; then
        last_read="$(jq '.data.activityItems[0].createdTime' activity.json 2>/dev/null)"
        peach /activity | tee activity.json \
            | jq --arg last_read "${last_read:-0}" -c '.data.activityItems[] | select(.createdTime > ($last_read | tonumber))' &&
            peach /activity/read -X PUT >/dev/null
    fi

# Endpoint list notes

The endpoint list is given very tersely. The three columns are:

- HTTP method
- URL relative to `https://v1.peachapi.com/`
- (for `POST` and `PUT` requests only) description of request body

If part of the URL looks like a `{{template}}`, that means it needs to
be filled in with an appropriate value.

If there is a request body, it is described in shorthand (not valid
JSON): keys are unquoted, and type names are used as placeholders for
values. For example, `{public: boolean}` would mean you need to supply
a JSON body of `{"public": true}` or `{"public": false}`. If the
shorthand includes an array, the array can contain multiple values.

# API Notes

User streams that allow pagination will return a `cursor` key in the
stream data. Make another request with this value added as a query
parameter (so, `GET /stream/id/{{id}}?cursor={{cursor}}`) to return
earlier items in the stream.

Some parts of the API seem to have hidden state:

- As mentioned above, `GET /connections` updates the time you were last
  seen "online". Other endpoints do not have this effect.

- In order to register a new account, you must request (without
  authenticating) the correct `precheck` endpoint and get a success
  response for all three pieces of information (email, stream name,
  password) shortly before doing `POST /register`.

- `POST /stream/device` does not do anything observable. However, there
  are two not-yet-documented endpoints, `GET /user/hasVerifiedPhone` and
  `DELETE /user` which I suspect require some kind of device state to
  have been set (otherwise they return 401 even with a valid session).
  The device ID is a 64-digit hex number.

Although chat endpoints are documented, I don't know if it's possible to
fully use chat through the API only. The app also connects to
`pubsub.pubnub.com` for push notifications and I have not looked at this
traffic closely.

There is an additional pseudo-endpoint, `GET
/web/email/unsubscribe?token={{token}}`, which is sent to you as a
regular URL in the activity emails you receive when logged out.
Requesting it returns an HTML response meant for a web browser. The
token is the same in every email but not used anywhere else. It is not
clear how you would re-subscribe to these emails.

# Output

If `peach`'s stdout is a terminal, output will be filtered through `jq
.`. Otherwise, responses will be output without any filtering. So:

    peach /connections                      # will be pretty-printed and colorized
    peach /connections > c.json             # will be compact
    peach /connections | less               # will also be compact
    peach /connections | jq . > c.json      # will be pretty-printed, but not colorized
    peach /connections | jq -C . | less -R  # will again be pretty-printed and colorized

In the last example, the `-C` option to `jq` is necessary because its
stdout is not a terminal either.

# Thanks

I used
[this Gist](https://gist.github.com/ummjackson/4db1da44c509576c1d1b) as
a starting point for exploring the API.

Peach's [Community Guidelines](http://peach.cool/guidelines.html) say
"If you act like a creep, we'll ban you" and explicitly call out
"discrimination, bigotry, racism, hatred, harassment or harm" as
prohibited. I appreciate that. Do not use any of the knowledge about the
API embodied in this script to do any of those things.

# Bot Gallery

If you create a bot using this script, I'd love to hear about it.
Currently I am using it to run `@gifs_galore`.

# TODO

- A more pleasant way to automatically select different session files.
  Saying "current directory if exists else home directory" is not good
  because then it becomes ambiguous where to create the file when logging
  in. Environment variables are kind of tacky.
- Tests, somehow?
