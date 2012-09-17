# EMPessimistic

*Pessimistic child processes for EventMachine.*

EventMachine provides both `EM.popen` and `EM::DeferrableChildProcess`, but none
of them are particularly useful in the case of your process exiting with an
error.

`EMPessimistic` provides `popen3` and a `DeferrableChildProcess` that caters for
failing child processes.

## API Docs

### `EMPessimistic.popen3(cmd, handler=nil, *args)`

Works like
[`EM::popen`](http://eventmachine.rubyforge.org/EventMachine.html#M000491).
Additionally, it sends data from the process' stderr to the handler's
`receive_stderr` method.

    require "em_pessimistic"
    require "eventmachine"

    class LsHandler < EventMachine::Connection
      def initialize(*args)
        puts "Initializing command"
      end

      def receive_data(data)
        puts "Received stdout: #{data}"
      end

      def receive_stderr(data)
        puts "Received stderr: #{data}"
      end

      def unbind
        puts "All done"
        EM.stop
      end
    end

    EM.run do
      EMPessimistic.popen3("ls -l", LsHandler)
      EMPessimistic.popen3("ls -l /oh/noes", LsHandler)
    end

### `EMPessimistic.::DeferrableChildProcess`

Works mostly like
[`EM::DeferrableChildProcess`](http://eventmachine.rubyforge.org/EventMachine/DeferrableChildProcess.html).

If the process spawned by `EMPessimistic::DeferrableChildProcess.open` exits
cleanly, the returned deferrable will invoke the `callback` with two arguments -
`data`, which is a string representing the process' stdout and `status`, which
is a [`Process::Status`](http://www.ruby-doc.org/core-1.9.3/Process/Status.html)
object.

If the process spawned by `EMPessimistic::DeferrableChildProcess.open` exits
with an error, the returned deferrable will invoke the `errback` with two
arguments - `data`, which is a string representing the process' stderr and
`status`, which is a
[`Process::Status`](http://www.ruby-doc.org/core-1.9.3/Process/Status.html)
object.

    require "eventmachine"
    require "em_pessimistic"

    EM.run do
      EM.next_tick do
        git = EMPessimistic::DeferrableChildProcess.open("git ls-tree master:")

        git.callback do |stdout, status|
          puts "Success! Stdout:\n#{stdout}"
          EM.stop
        end

        git.errback do |stderr, status|
          puts "Failure (exit code #{status.exitstatus}). Stderr:\n#{stderr}"
          EM.stop
        end
      end
    end

## Installing

`em_pessimistic` ships as a gem:

    $ gem install em_pessimistic

Or in your Gemfile:

    gem "em_pessimistic", "~> 0.1"

## Contributing

Contributions are welcome. To get started:

    $ git clone git://gitorious.org/gitorious/em_pessimistic.git
    $ cd em_pessimistic
    $ bundle install
    $ rake

When you have fixed a bug/added a feature/done your thing, create a
[clone on Gitorious](http://gitorious.org/gitorious/em_pessimistic) or a
[fork on GitHub](http://github.com/cjohansen/em_pessimistic) and send a
merge request/pull request, whichever you prefer.

Please add tests when adding/altering code, and always make sure all the tests
pass before submitting your contribution.