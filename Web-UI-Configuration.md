Similar to configuring Karafka, a few configuration options can also be used in Karafka Web.

Those options can be used to control things like topics names, frequency of data reports, encrypted data visibility, pagination settings, and more.

You can find the whole list of settings [here](https://github.com/karafka/karafka-web/blob/master/lib/karafka/web/config.rb).

You can configure Web UI by using the `#setup` method in your `karafka.rb`:

```ruby
Karafka::Web.setup do |config|
  # Report every 10 seconds
  config.tracking.interval = 10_000
end

Karafka::Web.enable!
```
