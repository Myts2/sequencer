sequence
========

[![GoDoc](http://godoc.org/github.com/leolee192/sequencer?status.svg)](http://godoc.org/github.com/leolee192/sequencer) 

[![GoDoc](http://godoc.org/github.com/leolee192/sequencer/cmd/sequence?status.svg)](http://godoc.org/github.com/leolee192/sequencer/cmd/sequence)

`sequence` is a _high performance sequential log scanner, analyzer and parser_. It _sequentially_ goes through a log message, _parses_ out the meaningful parts, without the use regular expressions. It can achieve _high performance_ parsing of **100,000 - 200,000 messages per second (MPS)** without the need to separate parsing rules by log source type.

**If you have a set of logs you would like me to test out, please feel free to [open an issue](https://github.com/leolee192/sequencer/issues) and we can arrange a way for me to download and test your logs.**

### Motivation

Log messages are notoriusly difficult to parse because they all have different formats. Industries (see Splunk, ArcSight, Tibco LogLogic, Sumo Logic, Logentries, Loggly, LogRhythm, etc etc etc) have been built to solve the problems of parsing, understanding and analyzing log messages.

Let's say you have a bunch of log files you like to parse. The first problem you will typically run into is you have no way of telling how many DIFFERENT types of messages there are, so you have no idea how much work there will be to develop rules to parse all the messages. Not only that, you have hundreds of thousands, if not  millions of messages, in front of you, and you have no idea what messages are worth parsing, and what's not.

The typical workflow is develop a set of regular expressions and keeps testing against the logs until some magical moment where all the logs you want parsed are parsed. Ask anyone who does this for a living and they will tell you this process is long, frustrating and error-prone.

Even after you have developed a set of regular expressions that match the original set of messages, if new messages come in, you will have to determine which of the new messages need to be parsed. And if you develop a new set of regular expressions to parse those new messages, you still have no idea if the regular expressions will conflict with the ones you wrote before. If you write your regex parsers too liberally, it can easily parse the wrong messages.

After all that, you will end up finding out the regex parsers are quite slow. It can typically parse several thousands messages per second. Given enough CPU resources on a large enough machine, regex parsers can probably parse tens of thousands of messages per second. Even to achieve this type of performance, you will likely need to limit the number of regular expressions the parser has. The more regex rules, the slower the parser will go.

To work around this performance issue, companies have tried to separate the regex rules for different log message types into different parsers. For example, they will have a parser for Cisco ASA logs, a parser for sshd logs, a parser for Apache logs, etc etc. And then they will require the users to tell them which parser to use (usually by indicating the log source type of the originating IP address or host.)

Sequence is developed to make analyzing and parsing log messages a lot easier and faster.

### Performance

The following performance benchmarks are run on a single 4-core (2.8Ghz i7) MacBook Pro, although the tests were only using 1 or 2 cores. The first file is a bunch of sshd logs, averaging 98 bytes per message. The second is a Cisco ASA log file, averaging 180 bytes per message. Last is a mix of ASA, sshd and sudo logs, averaging 136 bytes per message.

```
  $ ./sequence bench scan -i ../../data/sshd.all
  Scanned 212897 messages in 0.78 secs, ~ 272869.35 msgs/sec

  $ ./sequence bench parse -p ../../patterns/sshd.txt -i ../../data/sshd.all
  Parsed 212897 messages in 1.69 secs, ~ 126319.27 msgs/sec

  $ ./sequence bench parse -p ../../patterns/asa.txt -i ../../data/allasa.log
  Parsed 234815 messages in 2.89 secs, ~ 81323.41 msgs/sec

  $ ./sequence bench parse -d ../patterns -i ../data/asasshsudo.log
  Parsed 447745 messages in 4.47 secs, ~ 100159.65 msgs/sec
```

Performance can be improved by adding more cores:


```
  $ GOMAXPROCS=2 ./sequence bench scan -i ../../data/sshd.all -w 2
  Scanned 212897 messages in 0.43 secs, ~ 496961.52 msgs/sec

  GOMAXPROCS=2 ./sequence bench parse -p ../../patterns/sshd.txt -i ../../data/sshd.all -w 2
  Parsed 212897 messages in 1.00 secs, ~ 212711.83 msgs/sec

  $ GOMAXPROCS=2 ./sequence bench parse -p ../../patterns/asa.txt -i ../../data/allasa.log -w 2
  Parsed 234815 messages in 1.56 secs, ~ 150769.68 msgs/sec

  $ GOMAXPROCS=2 ./sequence bench parse -d ../patterns -i ../data/asasshsudo.log -w 2
  Parsed 447745 messages in 2.52 secs, ~ 177875.94 msgs/sec
```

### Limitations

* `sequence` does not handle multi-line logs. Each log message must appear as a single line. So if there's multi-line logs, they must first be converted into a single line.
* `sequence` has only been tested with a limited set of system (Linux, AIX, sudo, ssh, su, dhcp, etc etc), network (ASA, PIX, Neoteris, CheckPoint, Juniper Firewall) and infrastructure application (apache, bluecoat, etc) logs. If you have a set of logs you would like me to test out, please feel free to [open an issue](https://github.com/leolee192/sequencer/issues) and we can arrange a way for me to download and test your logs.

### Usage

To run the unit tests, you need to be in the top level sequence dir:

```
go get github.com/leolee192/sequencer
cd $GOPATH/src/github.com/leolee192/sequencer
go test
```

To run the actual command you need to

```
cd $GOPATH/src/github.com/leolee192/sequencer/cmd/sequence
go run sequence.go
```

Documentation is available at [wiki](https://github.com/leolee192/sequencer/wiki).

### History

The project originated from [zentures/sequence](https://github.com/zentures/sequence) , which was iced in 2017. Since I tried and couldn't contact the original author for weeks, I decided to restart the project here in Nov 2019.
