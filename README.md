# peach-curl

A little command line wrapper for `curl`ing the
[Peach](http://peach.cool/) API.

# Usage

1. Install [jq](https://stedolan.github.io/jq/).
1. Run once to log in.
2. Run again to get a list of known API endpoints and example commands.
3. Run again with some arguments to make a request.

# Thanks

Initial API exploration was done by @ummjackson in [this Gist](https://gist.github.com/ummjackson/4db1da44c509576c1d1b).

# TODO

- Multiple session file support
- Passwords with certain punctuation characters cause `/login` to fail
  (is the app escaping them?)
- Figure out if chat is really usable without connecting to `pubsub.pubnub.com`
- Figure out why `/user/hasVerifiedPhone` doesn't work for me
- Tests, somehow?
