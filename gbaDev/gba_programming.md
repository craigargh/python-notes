


## Debugger



To try:
- Use regular gdb instead of the bundled arm one
- Make a list of gdb commands and try them with gdb
- Use mgba's built-in debugger
- Make a list of mgba's built-in debugger commands + try them with gdb
- Use the built-in debugger in Visual Studio Code


### Connecting GDB to a Compiled GBA Game

Setup:

1. Does gdb need to be installed?
1. Install Ncurses library `apt install libncurses5` if you want to use the gdb bundled with devkitpro


Running:

1. Compile the game
1. Open VBA-M and load game
1. Start GDB server with `Tools > GDB > Break into GDB`
1. Start GDB either with `gdb` or the bundled with DevKitPro via the terminal `/opt/devkitpro/devkitARM/bin/arm-none-eabi-gdb`
1. Connect to GDB Server running in VBA-M `target remote localhost:55555`
1. Load the corresponding `.elf` file for the `.gba` file to load the symbols with `file <path_to.elf>`

### GDB Commands


View the code with `layout next`. Nothing will be displayed as the program is not running yet.

Start the program with `next`


Notes:
- `run` command does not work


### Using Debugger with Visual Studio Code

https://sausage-factory.games/dev-blog/Gameboy-Advance-Dev-Workflow/

