---
author: "Ben Tranter"
title: "How the Stripe CLI's Login Command Works"
date: "2024-01-17"
description: "It's better than ours"
---

At DigitalOcean, you can only log into our CLI `doctl` by creating and copying a personal API token from the Cloud control panel, and pasting it into the CLI when prompted by running `doctl auth init`.

I feel like it's possible to do better.

GitHub's CLI has a nice experience where they allow you to log in by running a command that prompts you to open a link and complete an OAuth flow. I prototyped this for DigitalOcean in 2021 or 2022, but it wasn't accepted. With an OAuth flow, you have to either expose the client ID and secret in the CLI, or use the implicit grant flow. Neither is ideal, but probably not a deal breaker. The deal breaker came when I decided to run a local server on HTTP (not HTTPS) localhost to handle the callback to the redirect URL and perform the token exchange. Too easy to intercept, so this was a no-go.

Concourse's `fly` CLI also has a way to log in without using OAuth, but I hate using Concourse so much that I couldn't bring myself to look into how this works.

At this point, I forgot about trying to improve the login experience, and moved onto other things.

---

Yesterday, fed up with Stripe's painfully slow web UI, I decided to try out their CLI to see if that could speed up my weekly workflow of preparing invoices. Following their getting started docs, I ran `stripe login`, and suddenly that desire I had to improve DigitalOcean's CLI came rushing back. Their experience is perfect. It just gives you a link to open where you hit confirm, and you're logged into the CLI. No OAuth or anything difficult.

I had to know how it worked.

Their CLI is open source, so this was easy to figure out. Here's how it works.

To start you submit a POST request to https://dashboard.stripe.com/stripecli/auth, with the URL parameter `device_name` set to the value that you want to associate with your API token.

```sh
$ curl -X POST https://dashboard.stripe.com/stripecli/auth\?device_name\=myclitest
# {"browser_url":"https://dashboard.stripe.com/stripecli/confirm_auth?t=ZDLO7tZJRXbbXpSdF7FAw1oZw8UUIYLX","poll_url":"https://dashboard.stripe.com/stripecli/auth/cliauth_POY8JoX14dgn4Z?secret=ZDLO7tZJRXbbXpSdF7FAw1oZw8UUIYLX","verification_code":"apple-secure-whooa-prompt"}
```

From there, have the CLI prompt the user to visit the returned `browser_url`, and display the `verification_code` to them, and instruct them to confirm the same code is visible on the page loaded after accessing the `browser_url`.

While the above happens, poll the `poll_url` every second. After 60 seconds have passed, the URL is no longer valid.

```sh
$ curl https://dashboard.stripe.com/stripecli/auth/cliauth_POY8JoX14dgn4Z?secret=ZDLO7tZJRXbbXpSdF7FAw1oZw8UUIYLX
```

Before the user has authorized the URL, the response is

```json
{"redeemed": "false"}
```

Once the user has authorized the URL, you'll get a single response that says

```json
{"redeemed":true,"account_id":"acct_<redacted>","account_display_name":"<redacted>","testmode_key_secret":"rk_test_<redacted>","livemode_key_secret":"rk_live_<redacted>","testmode_key_expiry":1713322623,"livemode_key_expiry":1713322623,"testmode_key_publishable":"pk_test_<redacted>","livemode_key_publishable":"pk_live_<redacted>"}
```

Note that the `expiry` fields are 90 days from the time of authentication (but just to the day, not the second), and don't appear to be configurable.

That's it!

While I probably don't have time to work on this anytime soon, I think it sets a great standard for how to conveniently log into a CLI.
