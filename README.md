# Chess Evaluation in a Lambda Function

Run the stockfish chess engine in an aws lambda environment.  Analyze and annotate a .pgn file, with multiple games, and each game is processed in parallel.  You can run a 8 minute analysis on 200 games, and have the resulting PGN file emailed to you using SES in 8 minutes.

This works because lambda functions have their own real infrastructure underneath.  There is a file system, you can run binaries, the network stack works just fine - if only for a few hundred milliseconds!

I've constructed this service as a series of two functions, `analyze_games` and `analyze_game`.  `analyze_games` is a wrapper function that calls `analyze_game` in parallel for each game in the pgn file.  Games get fanned out to separate lambda instances, and fanned back in, once annotated.

The resulting, annotated .pgn file is collected by the original, invoking `analyze_games` function, which emails to them to you as an attachment (read up on setting up email addresses for AWS SES using your favorite search engine).  Other comments in the original .pgn file, such as click timings will be removed.

You need an AWS account to use this, and be familiar with setting up you environment, using the `serverless` framework etc.  This is not a tutorial.


### Example usage

Async, results emailed
`sls invoke -f annotate_games -p games/somegame.pgn -t Event`
will send email, leave off `-t Event` to block and receive output to stdout

I use it this way, having added the following to my `.bash_profile` (on OSX)

```
chesscheck() { pd=`pwd`; cd ~/Code/chessfunction; sls invoke -f annotate_games -p $pd/$1 -t Event; cd $pd; }
```

fire and forget

### Changing Dependencies

If you decide to hack on this and wish to add some different stuff, you will require this command to update deps:

```
$ pip3 install -t src/vendor -r aws_requirements.txt
```

### Notes/TODO

Currently the amount of time that each game is analyzed for is being stored in AWS Parameter store, you have to set up the parameter `/chessfunction/evaltime` with the aws cli:

```
$ aws ssm put-parameter --name /chessfunction/evaltime --value desired_value_in_min --type String
```

Afterwards, you can adjust this parameter using:

```
$ aws ssm put-parameter --name /chessfunction/evaltime --value desired_value_in_min --overwrite
```

It might be preferable to manage the email addresses to be used in this way as well. TODO

### Thanks

This fun project would not have been possible without the awesome [chess-annotator](https://github.com/rpdelaney-archive/python-chess-annotator) from `rpdelaney`, without which this would be just another boring serverless example project.  Thanks very much.  Also, thanks to the team at [Stockfish](https://stockfishchess.org/), who provided the UCI engine included here - I include the linux version `stockfish_20011801_x64` [downloaded here](https://stockfishchess.org/download/), in this repository.


