# linux notes

## first steps in the terminal

We can check we are using bash as a shell with `echo "${BASH_VERSION}"` If we do not see any output, we can enter a bash session by just typing the command `bash`

A useful basic command is `echo` This command outputs text to the screen. We wrap what we want to output to the screen in single or double quotes. If we are just typing text, we can use single quotes. These quotation marks do not make the text a string as they do in other programming languages. In bash, the quotations are related to expansions.

With the echo command, we specify an argument which is what we would like to output `echo 'hello world!'` The command along with the arguments makes a complete bash command.

The echo command finishes with a newline. We can remove this with the `-n` option like so `echo -n 'hello world!`

Another option we can use is `-e` which enables escape characters such as `\n` or `\t` An example of this is `echo -e 'bash is\ngreat!` This will use `\n` as a new line.

We can combine options - a quick way to do this is to put them all with one hyphen like so `echo -en 'hello\nworld!`

Another useful command is `ls` which lists the contents of a directory. We can use the `-a` option which will list all of the contents including hidden directories and folders. We can use the `-l` option to list more details about the contents of a directory and we can combine these two options using `ls -la` We can list the contents of a directory in time order using the `-t` option or in reverse order using the `-r` option.

When using `ls` we can specify which directory we want to list the contents of like so `ls -al /home/user/other`

We can use `--color=` to specify what we want bash to do with colours. The values we can use are `never`, `auto` and `always`. The `--` work with whole words whereas `-` works with single letters. We could use `--color=always` to use colours.

