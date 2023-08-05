- ## How to watch for changes in a file
- To watch a `.log` file for changes, you can use the `tail` command with the `-f` option. This will allow you to continuously monitor the file and display new lines as they are added.
- Here's an example command to watch a `.log` file named `example.log`:
- ```bash
  tail -f example.log
  ```
- This command will display the contents of the file and keep the output updated with any new lines that are appended to the file.
- You can stop the monitoring by pressing `Ctrl + C` in the terminal.
- ## The `tail` command
- The `tail` command provides several options that you can use to customize its behavior. Here are some commonly used options:
- `-n {{number}}` or `--lines={{number}}`: Displays the last {{number}} lines of the file. For example, `tail -n 10 example.log` will display the last 10 lines of `example.log`.
- `-f` or `--follow`: Continuously monitors the file for changes and displays new lines as they are added.
- `-c {{bytes}}` or `--bytes={{bytes}}`: Displays the last {{bytes}} bytes of the file. For example, `tail -c 100 example.log` will display the last 100 bytes of `example.log`.
- `-q` or `--quiet`: Suppresses the header that shows the file name when monitoring multiple files.
- `-s {{seconds}}` or `--sleep-interval={{seconds}}`: Specifies the sleep interval between checks for file changes when using the `-f` option. The default is 1 second.
- `-v` or `--verbose`: Displays the file name when monitoring multiple files.
  
  You can combine these options to suit your needs. For example, `tail -n 20 -f example.log` will display the last 20 lines of `example.log` and continuously monitor it for changes.
  
  To learn more about the `tail` command and its options, you can refer to its manual page by running `man tail` in your terminal.
- tags:: #[[TIL Series]], #[[CLI]]
-