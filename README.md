rspec-dnsruby
=========

[![Build Status](https://travis-ci.org/vzctl/rspec-dnsruby.png?branch=master)](https://travis-ci.org/vzctl/rspec-dnsruby)

rspec-dnsruby is an rspec plugin for easy DNS testing. It is a fork of rspec-dns and uses dnsruby instead of the standard ruby library.

Differences from rspec-dns
--------------------------

* dependency on dnsruby and activesupport
* additional matchers: `at_least`, `in_authority`, `refuse_request`
* a bit different API for the chain methods because of dnsruby

Dnsruby was chosen because it gives a complete DNS implementation, including DNSSEC.

Installation
------------
If you're using bundler, add this line to your application's `Gemfile`:

```ruby
gem 'rspec-dnsruby'
```

Don't forget to run the `bundle` command to install.

Or install it manually with:

    $ gem install rspec-dnsruby

Usage
-----
RSpec DNS is best described by example. First, require `rspec-dnsruby` in your `spec_helper.rb`:

```ruby
# spec/spec_helper.rb
require 'rspec'
require 'rspec-dnsruby'
```

Then, create a spec like this:

```ruby
require 'spec_helper'

describe 'www.example.com' do
  it { should have_dns.with_type('TXT').and_ttl(300).and_data('a=b') }
end
```

Or group all your DNS tests into a single file or context:

```ruby
require 'spec_helper'

describe 'DNS tests' do
  it 'passes some tests' do
    'www.example.com'.should have_dns.with_type('TXT').and_ttl(300).and_data('a=b')
    'www.example.com'.should have_dns.with_type('A').and_ttl(300).and_address('1.2.3.4')
  end
end
```

### Chain Methods

Currently the following chaining methods are supported:

- at\_least
- in\_authority
- refuse\_request

Here's some usage examples:

```ruby
  it 'checks if recursion is disabled' do
    'google.com'.should have_dns.refuse_request
  end

  it 'checks if gslb subdomain is delegated to dynect' do
    'gslb.example.com'.should have_dns.in_authority.with_type('NS').and_name(/dynect/).at_least(3)
  end

  it 'checks number of hosts in round robin' do
    'example.com'.should have_dns.with_type('A').at_least(3)
  end
```

The other method chains are actually [Dnsruby](http://dnsruby.rubyforge.org/classes/Dnsruby/RR.html) attributes on the record. You can prefix them with `and_`, `with_`, `and_with` or whatever your heart desires. The predicate is what is checked. The rest is syntactic sugar.

Depending on the type of record, the following attributes may be available:

- address
- bitmap
- cpu
- data
- domainname
- emailbx
- exchange
- expire
- minimum
- mname
- name
- os
- port
- preference
- priority
- protocol
- refresh
- retry
- rmailbx
- rname
- serial
- target
- ttl
- type
- weight

If you try checking an attribute on a record that is non-existent (like checking the `rmailbx` on an `A` record), you'll get an error like this:

```text
Failure/Error: it { should have_dns.with_type('TXT').and_ftl(300).and_data('a=b') }
  got 1 exception(s): undefined method `rmailbx' for #<Dnsruby::RR::IN::A:0x007f66a0339b00>
```

For this reason, you should always check the `type` attribute first in your chain.

Configuring
-----------
All configurations must be in your project root at `config/dns.yml`. This YAML file directly corresponds to the Resolv DNS initializer.

For example, to directly query your DNS servers (necessary for correct TTL tests), create a `config/dns.yml` file like this:

```yaml
nameserver:
  - 1.2.3.5
  - 6.7.8.9
```

If this file is missing Resolv will use the settings in /etc/resolv.conf.

The full list of configuration options can be found on the [Dnsruby docs](http://dnsruby.rubyforge.org/classes/Dnsruby/Config.html).

### Configuring connection timeout

Connection timeout is to stop waiting for resolver.
If you want to wait over default timeout `1`,
you can change the timeout in spec files or spec_helpers like this:

```ruby
RSpec.configuration.rspec_dns_connection_timeout = 5
```

alternatively you can specify it in the `config/dns.yml` file:

```yaml
nameserver:
  - 1.2.3.5
  - 6.7.8.9
timeouts: 3
```

Contributing
------------
1. Fork the project on github
2. Create your feature branch
3. Open a Pull Request



License & Authors
-----------------
- Seth Vargo (sethvargo@gmail.com)
- Johannes Russek (jrussek@spotify.com)
- Alexey Lapitsky (lex@realisticgroup.com)

```text
Copyright 2012-2013 Seth Vargo
Copyright 2012-2013 CustomInk, LLC
Copyright 2013-2014 Spotify AB

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
