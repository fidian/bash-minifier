NONFUNCTIONAL - No minification happens yet!

This mostly tokenizes shell scripts, but doesn't quite do that right. After tokenization, some level of parsing needs to happen in order to properly change the contexts. From there, I believe the best thing to do is to send data to another function that will minify the files.

This bloats a single shell script to consume at least (untested) 50x the memory of just holding the shell script in memory. That's because it tokenizes and adds a lot of metadata to each token.  For example, take the classic fork bomb.

    :(){:|:;};:

After tokenization, there's an array that looks like this example. The actual tokens may change in the future.

    tokens=(
        'WORD,SOURCE.@1:1 :'
        'EMPTY_LIST,SOURCE.ARG.@1:2 ()'
        'BRACE_OPEN,SOURCE.ARG.@1:4 {'
        'WORD,BLOCK.@1:5 :'
        'UNKNOWN,BLOCK.ARG.@1:6 |'
        'WORD,BLOCK.ARG.@1:7 :;'
        'BRACE_CLOSE,BLOCK.ARG.@1:9 }'
        'WORD,SOURCE.ARG.@1:10 ;:'
        'EOL,SOURCE.ARG.@1:12 '$'\n'
    )

When split by the delimiters (the first comma, at symbol, colon, and space), you get the following.

| Token       | Context and Flags | Line | Col | Data  |
|-------------|-------------------|------|-----|-------|
| WORD        | SOURCE.           | 1    | 1   | :     |
| EMPTY_LIST  | SOURCE.ARG.       | 1    | 2   | \(\)  |
| BRACE_OPEN  | SOURCE.ARG.       | 1    | 4   | \{    |
| WORD        | BLOCK.            | 1    | 5   | :     |
| UNKNOWN     | BLOCK.ARG.        | 1    | 6   | \     |
| WORD        | BLOCK.ARG.        | 1    | 7   | :\;   |
| BRACE_CLOSE | BLOCK.ARG.        | 1    | 9   | \}    |
| WORD        | SOURCE.ARG.       | 1    | 10  | \;:   |
| EOL         | SOURCE.ARG.       | 1    | 12  | $'\n' |
