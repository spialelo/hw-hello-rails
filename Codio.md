# Running and Interacting with a Sinatra or Rails SaaS App In Development

The process for starting an app server and visiting the app from a
browser are slightly different if you are doing purely local
development (Ruby/Rails running on your own computer) vs. using Codio.

If you're comfortable with networking and tunneling, skip down to
"Why the difference" below.

## If developing in Codio

Obtain your Codio subdomain name by going into any Codio terminal
window and saying `hostname`.  Your subdomain will be a pair of random
words--in this example we'll pretend it's `luminous-coconut`.

1. Start the app in a terminal:  `rails server -b 0.0.0.0`
2. Open a regular browser window to  `luminous-coconut-3000.codio.io` to visit the app's home page

## If developing Locally

If you're developing locally, the steps would instead be these:

1. Start the app in a terminal: `rails server`  (omit `-b 0.0.0.0`)
2. Open a regular browser window to `localhost:3000/` to visit the
   app's home page (note the `:3000` rather than `-3000`)

## Why the difference?

TL;DR: To tunnel HTTP traffic to port _port_ on Codio subdomain
_subdomain_`.codio.io`, point a browser at `https://`_subdomain_`-`_port_`.codio.io`.

For the curious: When developing in Codio, `localhost` doesn't work as
you'd expect, because it refers to the virtual machine hosted
at Codio in which you're developing.  But Codio arranges to do [HTTP
tunnelling](https://en.wikipedia.org/wiki/HTTP_tunnel) for you--if you form the
URL using _subdomain_`-`portnumber`.codio.io`, traffic will be
tunneled to port `portnumber` of the virtual machine running at `subdomain`.
