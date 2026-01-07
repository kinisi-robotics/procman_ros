Frequently Asked Questions {#procman_faq}
===========================

[TOC]

# How do I use bash aliases in procman? {#procman_faq_bash_aliases}

Bash aliases are shell-specific features that are only available when running commands through a bash shell. Procman executes commands directly using `execvp`, which does not support bash aliases.

To use bash aliases in procman commands, you have several options:

## Method 1: Use bash -c with eval (RECOMMENDED)

\code
cmd "my-command" {
    exec = "bash -c 'shopt -s expand_aliases && source ~/.bashrc && eval my_alias'";
    deputy = "deputy_id";
}
\endcode

In this approach:
1. `bash -c` runs the command through a bash shell
2. `shopt -s expand_aliases` enables alias expansion in non-interactive shells
3. `source ~/.bashrc` loads your bash configuration file containing the alias definitions
4. `eval my_alias` evaluates and executes the alias

**Note**: The `eval` is important because bash processes aliases during parsing, not execution. The eval forces a second parsing pass where the alias is expanded.

## Method 2: Use bash functions instead of aliases

Bash functions work more reliably than aliases in non-interactive shells:

\code
cmd "my-command" {
    exec = "bash -c 'source ~/.bashrc && my_function arg1 arg2'";
    deputy = "deputy_id";
}
\endcode

If you control the alias definition, consider converting it to a function:
\code
# In your ~/.bashrc, instead of:
# alias myapp="cd /path && ./run.sh"
# Use a function:
myapp() {
    cd /path && ./run.sh "$@"
}
\endcode

## Method 3: Use the full command directly

The most reliable approach is to avoid aliases entirely and use the actual command:

\code
cmd "my-command" {
    exec = "cd /path/to/myapp && ./run.sh --verbose";
    deputy = "deputy_id";
}
\endcode

Or wrap it in a bash -c if you need shell features:
\code
cmd "my-command" {
    exec = "bash -c 'cd /path/to/myapp && ./run.sh --verbose'";
    deputy = "deputy_id";
}
\endcode

## Method 4: Create a wrapper script

For complex commands, create a dedicated script file:

\code
# Create /usr/local/bin/myapp_wrapper.sh:
#!/bin/bash
source ~/.bashrc
eval my_alias

# Then use it in procman:
cmd "my-command" {
    exec = "/usr/local/bin/myapp_wrapper.sh";
    deputy = "deputy_id";
}
\endcode

## Important Notes

- **Environment Dependencies**: The deputy process inherits its environment from the terminal where it was started. Ensure your aliases are properly loaded.

- **eval is required**: When using aliases in non-interactive shells, you typically need to use `eval` to force bash to expand the alias.

- **Functions are better**: If you have the option, bash functions work more reliably than aliases for non-interactive execution.

- **Performance**: Sourcing configuration files adds overhead to command startup. For frequently-started commands, use wrapper scripts or the full command path.

- **Portability**: Hard-coding paths to configuration files (like `~/.bashrc`) may reduce portability. Consider using environment variables or wrapper scripts.

## Complete Example

Let's say you have an alias defined in your `~/.bashrc`:
\code
alias myapp="cd /opt/myapp && ./run.sh --verbose"
\endcode

To use this in procman:

\code
cmd "My Application" {
    exec = "bash -c 'shopt -s expand_aliases && source ~/.bashrc && eval myapp'";
    deputy = "localhost";
    auto_respawn = "false";
}
\endcode

## Better Alternative: Convert to Function

In your `~/.bashrc`, define it as a function instead:
\code
myapp() {
    cd /opt/myapp && ./run.sh --verbose "$@"
}
\endcode

Then use it in procman:
\code
cmd "My Application" {
    exec = "bash -c 'source ~/.bashrc && myapp'";
    deputy = "localhost";
    auto_respawn = "false";
}
\endcode

# How do I use bash functions in procman? {#procman_faq_bash_functions}

Bash functions work better than aliases for procman. Simply source your bash configuration and call the function:

\code
cmd "my-function" {
    exec = "bash -c 'source ~/.bashrc && my_function arg1 arg2'";
    deputy = "deputy_id";
}
\endcode

Functions are exported from sourced files and can be called directly without needing `eval`.

# Why don't my shell aliases work directly? {#procman_faq_why_no_aliases}

Procman executes commands directly using the system's `execvp` function, which doesn't go through a shell. This is done for efficiency and direct process management. Shell features like aliases, functions, and shell built-ins are only available when running commands through a shell interpreter like bash.

Additionally, bash aliases are only expanded during command parsing in interactive shells or when explicitly enabled with `shopt -s expand_aliases`. Even with this option, aliases defined and used in the same command line require `eval` for proper expansion.

To use these features, you must explicitly invoke the shell as shown in the examples above.

# Can I use other shell features? {#procman_faq_other_shell_features}

Yes! By running commands through bash (or another shell), you can use:

- **Pipes**: `bash -c 'cat file.txt | grep pattern'`
- **Redirections**: `bash -c 'command > output.txt 2>&1'`
- **Shell expansions**: `bash -c 'echo *.txt'`
- **Background jobs**: `bash -c 'command &'` (though procman manages processes, so this is rarely needed)
- **Shell built-ins**: `bash -c 'cd /path && pwd'`
- **Functions**: `bash -c 'source ~/.bashrc && my_function'`

Example with pipes and redirection:
\code
cmd "log-processor" {
    exec = "bash -c 'tail -f /var/log/myapp.log | grep ERROR > /tmp/errors.log'";
    deputy = "deputy_id";
}
\endcode

Example with shell glob expansion:
\code
cmd "cleanup" {
    exec = "bash -c 'rm -f /tmp/*.tmp'";
    deputy = "deputy_id";
}
\endcode
