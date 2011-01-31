Indy: A Log Archaeology Tool
====================================

Synopsis
--------

Log files are often searched for particular strings but it does not often treat the logs themselves as data structures.  Indy attempts to deliver logs with more powerful features by allowing the ability to collect segments of a log from particular time; find a particular event; or monitor/reflect on a log to see if a particular event occurred (or not occurred).

Installation
------------

To install Indy use the following command:

    $ gem install indy
    
(Add `sudo` if you're installing under a POSIX system as root)

Usage
-----

## Require Indy

    require 'indy'

## Specify your Source

### As a process or command

    Indy.search( {:cmd => 'ssh user@system "bash --login -c \"cat /var/log/standard.log\" "'} ).for(:severity => 'INFO')

### As a file

    Indy.search('logpath/output.log').for(:application => 'MyApp')

### As a string

    log_string = %{
        2000-09-07 14:07:41 INFO  MyApp - Entering application.
        2000-09-07 14:07:41 INFO  MyApp - Exiting application. }

    Indy.search(log_string).for(:message => 'Entering application')

## Log Pattern

### Default Log Pattern
  
The default search pattern follows this form:  
YYYY-MM-DD HH:MM:SS SEVERITY APPLICATION_NAME - MESSAGE

Which uses this regexp:
    /^(\d{4}.\d{2}.\d{2}\s+\d{2}.\d{2}.\d{2})\s+(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)\s+(\w+)\s+-\s+(.+)$/

and specifies these fields:  
    [:time, :severity, :application, :message]

For example:  
    Indy.search(source).for(:severity => 'INFO')
    Indy.search(source).for(:application => 'MyApp', :severity => 'DEBUG')

### Custom Log Pattern

If the default pattern is obviously not strong enough for you, brew your own.
To do so, specify a pattern and each of the match with their symbolic name.

    # HH:MM:SS SEVERITY APPLICATION#METHOD - MESSAGE
    custom_pattern = /^(\d{2}:\d{2}:\d{2})\s*(INFO|DEBUG|WARN|ERROR)\s*([^#]+)#([^\s]+)\s*-\s*(.+)$/

    Indy.search(source).with(custom_pattern,:time,:severity,:application,:method,:message).for(:severity => 'INFO', :method => 'allocate')

### Predefined Log Patterns

Several log formats have been predefined for ease of configuration. See indy/patterns.rb

    # Indy::COMMON_LOG_PATTERN
    # Indy::COMBINED_LOG_PATTERN
    # Indy::LOG4R_DEFAULT_PATTERN
    #
    # Example (Log4r)
    #  INFO mylog: This is a message with level INFO
    Indy.new(:source => 'logfile.txt', :pattern => Indy::LOG4R_DEFAULT_PATTERN).for(:application => 'mylog')

### Multiline log entries

By default, Indy assumes that log lines are separated by new lines. Any lines that don't match the active pattern are ignored. To enable multiline log entries you must do two things:

1. Use `Indy.new()` and include the `:multiline => true` parameter
2. Use a log entry regexp that does not use `$` and/or `\n` to define the end of the entry.

#### Multiline Regexp tips

* Use non-greedy matching when needed: `.*?` instead of `.*`
* Assuming your log entries do not include a unique line ending, you can use a zero-width positive lookahead assertion to verify that each line is followed by the start of a valid log entry, or the end of the string. e.g.: `(?=^foo|\z)`

Check out [Regexp Extensions](http://www.ruby-doc.org/docs/ProgrammingRuby/html/language.html#UN)

Example:

    # Given this log containing two entries:
    #
    # INFO MyApp - Multiline message begins here
    # and ends here
    # DEBUG MyOtherApp - Single line message
    
    severity_string = 'DEBUG|INFO|WARN|ERROR|FATAL'

    # single line regexp would be:
    #                  /^(#{severity_string}) (\w+) - (.*)$/
    multiline_regexp = /^(#{severity_string}) (\w+) - (.*?)(?=^#{severity_string}|\z)/

    Indy.new( :multiline => true, :pattern => [multiline_regexp, :severity, :application, :message], :source => MY_LOG)

### Explicit Time Format

By default, Indy tries to guess your time format (courtesy of DateTime#parse). If you supply an explicit time format, it will use DateTime#strptime, as well as try to guess.

This is required when log data uses a non-standard date format, e.g.: U.S. format 12-31-2000

    # 12-31-2011 23:59:59
    Indy.new(:time_format => '%m-%d-%Y %H:%M:%S', :source => LOG_FILE).for(:all)

## Match Criteria

### Exact Match

    Indy.search(source).for(:message => 'Entering Application')
    Indy.search(source).for(:severity => 'INFO')

### Exact Match with multiple parameters

    Indy.search(source).for(:message => 'Entering Application', :application => 'MyApp')
    Indy.search(source).for(:severity => 'INFO', :application => 'MyApp')

### Partial Match

    Indy.search(source).like(:message => 'Memory')

### Partial Match with multiple parameters

    Indy.search(source).like(:severity => '(?:INFO|DEBUG)', :message => 'Memory')

## Log Scopes

Multiple scope methods can be called on an instance. Use #reset_scope to remove scope constrints on the instance.

### Time Scope

    # After Dec 1
    Indy.search(source).after(:time => '2010-12-01 23:59:59').for(:all)

    # 20 minutes Around New Year's eve
    Indy.search(source).around(:time => '2011-01-01 00:00:00', :span => 20).for(:all)

    # After Jan 1 but Before Feb 1
    @log = Indy.search(source)
    @log.after(:time => '2011-01-01 00:00:00').before(:time => '2011-02-01 00:00:00')
    @log.for(:all)

    # Within Jan 1 and Feb 1 (same time scope as above)
    Indy.search(source).within(:time => ['2011-01-01 00:00:00','2011-02-01 00:00:00']).for(:all)

    # After Jan 1
    @log = Indy.search(source)
    @log.after(:time => '2011-01-01 00:00:00')
    @log.for(:all)
    # Reset the time scope to include entries before Jan 1
    @log.reset_scope
    # Before Feb 1
    @log.before(:time => '2011-02-01 00:00:00')
    @log.for(:all)

## Process the Results

A ResultSet is returned by #for and #like, which is an Enumerable containing a hash for each log entry.

    entries = Indy.search(source).for(:message => 'Entering Application')
    entries.first.keys
    # => [:line, :time, :severity, :application, :message]

    Indy.search(source).for(:message => 'Entering Application').each do |entry|
      puts "[#{entry.time}] #{entry.message}: #{entry.application}"
    end

LICENSE
-------

(The MIT License)

Copyright (c) 2010 Franklin Webber

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.