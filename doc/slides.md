!SLIDE
# Continuous Integration with node.js using teamcity and chef #
Ben Lindsey

October 25th, 2012

!SLIDE bullets
# Vungle, Inc #

* Mobile video advertising platform
* Specialize in 15 second video trailers for other mobile apps
* 20 employees split between SF, London, and Pakistan
* Founded Summer 2011, AngelPad Fall 2012
* Small engineering team (3) following Agile Development

!SLIDE bullets
# mocha/should #

* async tests with many built-in reporters
* factory for creating common objects in mongo
* integration tests actually spin up a server in process
* npm test is your friend
* drop the whole database before running tests

!SLIDE code small
	@@@ javascript
	require('../lib/array');

	describe('Array', function() {
	  describe('#sortInsensitive', function() {
	    it('handles a zero length array', function() {
	      [].sortInsensitive().should.eql([]);
	    });

	    it('is non-iterable', function() {
	      for (var key in ['a', 'b']) {
	        key.should.not.equal('sortInsensitive');
	      }
	    });

	    it('is case insensitive', function() {
	      ['AC', 'Aa', 'B', 'a'].sortInsensitive()
	        .should.eql(['a', 'Aa', 'AC', 'B']);
	    });

	    it('handles attributes', function() {
	      var a = { title: 'foo' };
	      var b = { title: 'bar' };
	      var c = { title: 'CAT' };
	      [a, b, c].sortInsensitive('title')
	        .should.eql([b, c, a]);
	    });
	  });
	});

!SLIDE bullets
# Teamcity #

* mongo/redis instances for test and acceptance
* teamcity reporter from mocha - success/failure count
* automatic pulls from github, runs tests
* branches dev for development, master for production
* push button deploy to production servers

!SLIDE center

![Teamcity](/file/doc/teamcity.png)

!SLIDE bullets
# Chef #

* recipes for mongodb, redis, and nodejs from source
* npm install - use npm shrinkwrap
* deploy thru deploy_revision capistrano style with symlink to current
* send SIGUSR2 to cluster parent process for restart

!SLIDE code small 
	@@@ ruby
	deploy_revision "#{node[:vungle][:dir]}" do
	  action :deploy
	  repo 'git@github.com:Vungle/vungle.git'
	  revision node[:vungle][:branch]
	  user node[:vungle][:user]
	  group node[:vungle][:group]
	  shallow_clone true
	  rollback_on_error true

	  create_dirs_before_symlink([])
	  purge_before_symlink %w(log node_modules)
	  symlinks "log" => "log", "node_modules" => "node_modules"
	  symlink_before_migrate({})
	end

!SLIDE bullets
# Production #

* logs are in /mnt/log for extra storage
* we partition all of our impression/click/install data by day
* measure everything (mms, graphite, stathat, loggly, copperegg) 
* backpressure is ugly (TCP SYN flood, EMFILE, CPU/RAM)
* autoscaling with EC2 cloud watch

!SLIDE bullets
* [http://www.jetbrains.com/teamcity/](http://www.jetbrains.com/teamcity/)
* [http://www.opscode.com/chef/](http://www.opscode.com/chef/)
* [ben.lindsey@vungle.com](mailto:ben.lindsey@vungle.com)
* [@ben1mal](http://www.twitter.com/@ben1mal)
