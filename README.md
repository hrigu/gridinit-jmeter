# Gridinit::Jmeter

Tired of using the JMeter GUI or looking at hairy XML files?

This gem lets you write test plans for JMeter in your favourite text editor, and optionally run them on [Gridinit.com](http://gridinit.com). Here's some background on [why we think](http://www.slideshare.net/90kts/gridinit-jmeter) a DSL is necessary. Don't have Ruby? No problems, you can try out this DSL online in our [sandbox](http://sandbox.gridinit.com)

## Installation

Install it yourself as:

    $ gem install gridinit-jmeter

## Basic Usage

*Gridinit::Jmeter* exposes easy-to-use domain specific language for fluent communication with [JMeter](http://jmeter.apache.org/). As the name of the gem suggests, it also includes API integration with [Gridinit](http://gridinit.com), a cloud based load testing service.

To use the DSL, first let's require the gem:

```ruby
require 'rubygems'
require 'gridinit-jmeter'
```

### Basic Example
Let's create a `test` and save the related `jmx` testplan to file, so we can edit/view it in JMeter.

```ruby
test do
  threads 10 do
    visit 'Google Search', 'http://google.com'
  end
end.jmx
```

So in this example, we just created a test plan, with 10 threads, each of which visited the search page at Google. 

### Generating a JMeter Test Plan (JMX)
Note also how we called the `jmx` method of the test. Calling this method will write the contents of the JMeter test plan to screen (stdout) and also to a file like this.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2" properties="2.1">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="Test Plan" enabled="true">
      ...
    </TestPlan>
  </hashTree>
</jmeterTestPlan>
JMX saved to: jmeter.jmx
```

The file that is created can then be executed in the JMeter GUI. If you want to create the file with a different filename and/or path, just add the `file` parameter to the `jmx` method call like this.

```ruby
test do
  threads 10 do
    visit 'Google Search', 'http://altentee.com'
  end
end.jmx(file: "/tmp/my_testplan.jmx")
```

Windows users should specify a path like this.

```ruby
.jmx(file: "C:\\TEMP\\MyTestplan.jmx")
```

### Running a JMeter Test Plan locally
You can execute the JMeter test plan by calling the `run` method of the test like this.

```ruby
test do
  threads 10 do
    visit 'Google Search', 'http://altentee.com'
  end
end.run
```

This will launch JMeter in headless (non-GUI mode) and execute the test plan. This is useful for shaking out the script before you push it to the Grid. There are a few parameters that you can set such as the `path` to the JMeter binary, the `file` path/name for the JMX file, the `log` path/name to output JMeter logs and the `jtl` path/name for JMeter results like this.

```ruby
test do
  threads 10 do
    visit 'Google Search', 'http://altentee.com'
  end
end.run(
  path: '/usr/share/jmeter/bin/', 
  file: 'jmeter.jmx', 
  log: 'jmeter.log', 
  jtl: 'results.jtl')
```

### Running a JMeter Test Plan on Gridinit.com

As the gem name implies, you can also execute JMeter test plans on Gridinit.com using our API. To do so, you require an account and API token. If you don't know your token, sign in to the Grid and [generate a new token](http://gridinit.com/api).

To execute the test on the Grid, call the `grid` method on the test and pass it the API token like this.

```ruby
test do  
  threads 10 do
    visit 'Google Search', 'http://altentee.com'  
  end  
end.grid('OxtZ-4v-v0koSz5Y0enEQQ')
```

This will then provide you with a link to the live test results on the Grid like this.

``` 
Results at: http://prod.gridinit.com/shared?testguid=73608030311611e2962f123141011033&run_id=339&tags=jmeter&domain=altentee.com&cluster=54.251.48.129&status=running&view=
```

## Advanced Usage

### Blocks

Each of the methods take an optional block delimited by `do` and `end` or braces `{}`

Blocks let you nest methods within methods, so you can scope the execution of methods as you would in a normal JMeter test plan. For example.

```ruby
test do
  threads 100 do
    visit 'Home', 'http://altentee.com' do
      extract 'csrf-token', "content='(.+?)' name='csrf-token'"
    end
  end
end
```

This would create a new test plan, with a 100 user thread group, each user visiting the "Home" page and extracting the CSRF token from the response of each visit.

All methods are nestable, but you should only have one test method, and typically only one threads method. For example, it wouldn't make sense to have a test plan within a test plan, or a thread group within a thread group. You can have multiple thread groups per test plan though. This implies *some* knowlege of how JMeter works.

### Threads

You can use the `threads` method to define a group of users:

```ruby
threads 100
```

This methods takes 2 parameters: the number of threads and an optional parameters hash.

```ruby
threads 100
threads 100, {continue_forever: 'true'}
threads 100, {loops: 10}
threads 100, {ramp_time: 30, duration: 60}
threads 100, {
  scheduler: 'true', 
  start_time: Time.now.to_i * 1000,
  end_time:   (Time.now.to_i * 1000) + (3600 * 1000)
}
```

### Cookies

You can use the `cookies` method to define a Cookie Manager:

```ruby
test do
  cookies
end
```

This methods takes an optional parameters hash. This is based on the [HTTP Cookie Manager](http://jmeter.apache.org/usermanual/component_reference.html#HTTP_Cookie_Manager).

```ruby
test do
  cookies clear_each_iteration: false
end

test do
  cookies policy: 'rfc2109', clear_each_iteration: true
end
```

### Cache

You can use the `cache` method to define a Cache Manager:

```ruby
test do
  cache
end
```

This methods takes an optional parameters hash. This is based on the [HTTP Cache Manager](http://jmeter.apache.org/usermanual/component_reference.html#HTTP_Cache_Manager).

```ruby
test do
  cache clear_each_iteration: false
end

test do
  cache use_expires: true, clear_each_iteration: true
end
```

### Authorization

You can use the `auth` method to define an Authorization Manager:

```ruby
test do
  auth
end
```

This methods takes an optional parameters hash. This is based on the [HTTP Authorization Manager](http://jmeter.apache.org/usermanual/component_reference.html#HTTP_Authorization_Manager).

```ruby
test do
  auth url: '/', username: 'tim', password: 'secret', domain: 'altentee.com'
end
```

### Navigating

You can use the `visit` method to navigate to pages:

```ruby
  visit 'Google Search', 'http://altentee.com'
```

This method makes a single request and takes 3 parameters: the label, the URL and an optional parameters hash.

```ruby
visit 'Google Search', 'http://altentee.com'
visit 'Google Search', 'http://altentee.com', {
  method: 'POST', 
  DO_MULTIPART_POST: 'true'
}
visit 'Google Search', 'http://altentee.com', {
  use_keepalive: 'false'
}
visit 'Google Search', 'http://altentee.com', {
  connect_timeout: '1000',
  response_timeout: '60000',
}
visit 'View Login', '/login', {
  protocol: "https",
  port: 443
}
```

### Submitting a Form

You can use the `submit` method to POST a HTTP form:

```ruby
submit 'Submit Form', 'http://altentee.com/', {
  fill_in: {
    username: 'tim',
    password: 'password',
    'csrf-token' => '${csrf-token}'
  }
```

This method makes a single request and takes 3 parameters: the label, the URL and an optional parameters hash. The fill_in parameter lets you specify key/value pairs for form field parameters. You can also use the built in JMeter `${expression}` language to access run time variables extracted from previous responses.

### Think Time

You can use the `think_time` method to insert pauses into the simulation. This method is aliased as `random_timer`.

```ruby
think_time 3000
```

This method takes 2 parameters: the constant delay, and an optional variable delay. Both are specified in milliseconds. This is based on the [Gaussian Random Timer](http://jmeter.apache.org/usermanual/component_reference.html#Gaussian_Random_Timer). This timer pauses each thread request for a random amount of time, with most of the time intervals ocurring near a particular value. The total delay is the sum of the Gaussian distributed value (with mean 0.0 and standard deviation 1.0) times the deviation value you specify, and the offset value.

```ruby
# constant delay of 3 seconds
think_time 3000
# constant delay of 1 seconds with variance up to 6 seconds.
random_timer 1000,5000
```

### Response Extractor

You can use the `extract` method to extract values from a server response using a regular expression. This is aliased as the `web_reg_save_param` method. This method is typically used inside a `visit` or `submit` block.

```ruby
extract 'csrf-token', "content='(.+?)' name='csrf-token'"
```

This method takes 4 parameters: the selector (xpath or regex), the extracted value name, the PCRE regular expression and an optional parameters hash. The selector is optional and defaults to regex. You can specify regex or xpath extractors as follows.

```
visit 'Google', "http://google.com/" do
  extract 'button_text', 'aria-label="(.+?)"'
  extract :regex, 'button_text', 'aria-label="(.+?)"'
  extract :xpath, 'button', '//button'
end
```

This is based on the [Regular Expression Extractor](http://jmeter.apache.org/usermanual/component_reference.html#Regular_Expression_Extractor) and [XPath Extractor](http://jmeter.apache.org/usermanual/component_reference.html#XPath_Extractor)

```ruby
visit "Altentee", "http://altentee.com" do
  extract 'csrf-token', "content='(.+?)' name='csrf-token'"
  extract 'JSESSIONID', 'value="(.+?)" name="JESSIONSID"'
  web_reg_save_param 'VIEWSTATE', 'value="(.+?)" name="VIEWSTATE"'
  extract 'username', 'value="(.+?)" name="username"', {
    default: 'Tim Koopmans',
    match_number: 1
  }
  extract 'shopping_item', 'id="(.+?)" name="book"', {
    match_number: 0 # random
  }
end
```

### Response Assertion

You can use the `assert` method to extract values from a server response using a regular expression. This is aliased as the `web_reg_find` method. This method is typically used inside a `visit` or `submit` block.

```ruby
visit "Altentee", "http://altentee.com" do
  assert "contains", "We test, tune and secure your site"
end
```


This method takes 3 parameters: the matching rule, the test string, and an optional parameters hash. This is based on the [Response Assertion](http://jmeter.apache.org/usermanual/component_reference.html#Response_Assertion). 

```ruby
visit "Altentee", "http://altentee.com" do
  assert "contains", "We test, tune and secure your site"
  assert "not-contains", "We price gouge on cloud services"
  assert "matches", "genius"
  assert "not-matches", "fakers"
  assert "contains", "magic"
  assert "not-contains", "unicorns", {scope: 'all'}
end
```

## Roadmap

This is very much a work-in-progress. Future work is being sponsored by Gridinit.com. Get in touch with us if you'd like to be involved.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
