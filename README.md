# peach-curl

A little command line wrapper for `curl`ing the
[Peach](http://peach.cool/) API.

# Usage

1. Install [jq](https://stedolan.github.io/jq/).
2. Run once to log in (this stores the response in `~/.peach-session`).
3. Run again to get a list of known API endpoints and example commands.
4. Run again with some arguments to make a request: `peach [options] ENDPOINT [curl options]`

# Options

- `-f`: use a different session file (instead of `~/.peach-session`).
- `-n`: don't authenticate (if the session file exists), or try to login
  (if it does not). Useful for registering a new account, hopefully.

# Examples

For simple examples, run `peach` without any arguments. Here are some
ways you can combine it with `jq` to do more interesting things:

Make a text post:

    peach /post -d "$(jq -sRc '{message: [{type: "text", text: .}]}' < post-body.txt)"

List names of users you have blocked:

    peach /stream/block-list | jq -r '.data.blockList[].name'

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

Initial API exploration was done by @ummjackson in [this Gist](https://gist.github.com/ummjackson/4db1da44c509576c1d1b).

# TODO

- Passwords with certain punctuation characters cause `/login` to fail
  (is the app escaping them?)
- Figure out if chat is really usable without connecting to `pubsub.pubnub.com`
- Figure out why `/user/hasVerifiedPhone` doesn't work for me
- Tests, somehow?
