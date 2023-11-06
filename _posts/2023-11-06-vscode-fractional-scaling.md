# Enable fractional scaling in VSCode on Ubuntu

I am currently using a laptop with a 2K 14" screen with Ubuntu 22.04 LTS. Linux' support of high-DPI displays is still lacking compared to Windows and especially macOS, however with a few easy tweaks we can solve this.

By default VSCode on Ubuntu does not support fractional scaling well, but it works great when launching it with the following flags:

```bash
code --enable-features=UseOzonePlatform --enable-features=WaylandWindowDecorations --ozone-platform-hint=auto "$@"
```

This command is quite long and it would be nice to be able to launch it by simply typing `code` in the terminal.
We can create a script that allows us to launch VSCode with the flags:

- Create a script in `~/bin/vscode-custom.sh`:
        
    ```bash
    nano ~/bin/vscode-custom.sh
    ```

- Paste the following in the script:

    ```bash
    #!/bin/bash
    code --enable-features=UseOzonePlatform --enable-features=WaylandWindowDecorations --ozone-platform-hint=auto "$@"
    ```

- Make the script executable:

    ```bash
    chmod +x ~/bin/vscode-custom.sh
    ```

- Edit `~/.bashrc` to add an alias to the command:

    ```bash
    nano ~/.bashrc
    ```

    and add the following line at the end of it:

    ```bash
    alias code="~/bin/vscode-custom.sh"
    ```

Now open a new terminal or execute `source ~/.bashrc` in the open one and type `code`. VSCode should be crisp clear!