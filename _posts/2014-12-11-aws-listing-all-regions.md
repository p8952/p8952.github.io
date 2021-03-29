---
layout: "post"
title: "Listing EC2 instances in all regions"
date: "2014-12-11 00:00:00"
---

When working with EC2 instances across multiple regions I've found it's near
impossible to get a good overview of what is running where. This can be
especially annoying when you are automatically launching a number of short
lived instances.

To prevent me having to go through 9 different web pages to see what I
currently have running I found it easier to just use the API and list active
instances from the CLI.

Install dependencies:
```
$ gem install aws-sdk pmap
```

/usr/local/bin/aws-list:
```
#!/usr/bin/ruby

require 'aws-sdk'
require 'pmap'

def ec2(region = 'us-east-1')
	ec2 = AWS::EC2.new(
		access_key_id: ENV['AWS_ACCESS_KEY'],
		secret_access_key: ENV['AWS_SECRET_KEY'],
		region: region
	)
	ec2
end

def list_instances
	instances = []
	ec2.regions.peach do |region|
		ec2.regions[region.name].instances.peach do |instance|
			next if instance.status == :terminated
			instances << instance
		end
	end
	instances
end

list_instances.peach do |instance|
	puts "#{instance.id}\t\t#{instance.availability_zone}\t\t#{instance.status}\t\t#{instance.ip_address}\n"
end
```

Listing instances:
```
$ export AWS_ACCESS_KEY="ABCDEFGHIJKLMNOPQRSTUVWXYZ"
$ export AWS_SECRET_KEY="ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890"
$ aws-list
i-16b78754  eu-west-1a          running     54.77.218.113
i-0025e1e6  eu-west-1a          running     54.76.129.127
i-3926e2df  eu-west-1a          running     54.154.52.146
i-4924e0af  eu-west-1a          running     54.154.52.77
i-c424e022  eu-west-1a          running     54.72.131.127
i-0c25e1ea  eu-west-1a          running     54.154.51.140
i-9c25e17a  eu-west-1a          running     54.154.49.204
i-4b24e0ad  eu-west-1a          running     54.77.225.135
i-33e929f2  eu-central-1b       running     54.93.164.233
i-c324e025  eu-west-1a          running     54.76.98.165
i-3f26e2d9  eu-west-1a          running     54.154.47.126
i-8027e366  eu-west-1a          running     54.154.20.140
i-0d25e1eb  eu-west-1a          running     54.77.100.132
i-d718edd9  us-west-2c          running     54.149.35.63
i-0c2028e6  us-east-1a          running     54.164.193.104
i-5b95e54e  sa-east-1a          running     54.94.165.7
i-2dad38de  ap-northeast-1a     running     54.65.157.129
i-625a80af  ap-southeast-1a     running     54.169.195.201
i-a06e006f  ap-southeast-2a     running     54.66.184.34
i-2dbce5e5  us-west-1a          running     54.67.67.18
```
