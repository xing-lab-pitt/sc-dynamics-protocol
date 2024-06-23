# Tips on working remotely with slurm clusters

## Using VSCode tmux terminal
When connected to the cluster in remote mode, goto vscode terminal terminal UI on the bottom, find the plus sign button and click the dropdown button next to it. You can find a tmux option. Next time you login, follow the steps below to find your tmux sessions, or directly attach to one if you remember the id of the session.

To find and connect to tmux sessions, follow these steps:

### List all tmux sessions:
```sh
tmux ls
```

This command lists all currently running tmux sessions along with their session names and IDs.

### Connect to a specific tmux session:
To attach to a specific tmux session, use the following command:
```sh
tmux attach -t <session_name>
```
or
```sh
tmux attach -t <session_id>
```

Replace `<session_name>` or `<session_id>` with the name or ID of the session you want to connect to.

### Example Workflow:
1. List all tmux sessions:
   ```sh
   tmux ls
   ```
   Output might be something like:
   ```
   0: 1 windows (created Tue Jun 23 14:15:32 2024) [80x24]
   1: 2 windows (created Tue Jun 23 14:16:45 2024) [80x24]
   ```

2. Connect to a session with the name `0`:
   ```sh
   tmux attach -t 0
   ```

3. Connect to a session with the name `1`:
   ```sh
   tmux attach -t 1
   ```

If you know the name of the session you created, you can also use that directly. For example, if you named a session "work", you can connect to it using:
```sh
tmux attach -t work
```

These commands will help you manage and connect to your tmux sessions easily.


## Using TMUX (under remote server using ssh)
[Tmux Guide](https://hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)
Tmux’s authors describe it as a terminal multiplexer. Behind this fancy term hides a simple concept: Within one terminal window you can open multiple windows and split-views (called “panes” in tmux lingo). Each pane will contain its own independently running shell instance (bash zsh whatever you’re using). This allows you to have multiple terminal commands and applications running side by side without the need to open multiple terminal emulator windows.
On top of that tmux keeps these windows and panes in a session. You can exit a session at any point. This is called “detaching”. tmux will keep this session alive until you kill the tmux server (e.g. when you reboot). This is incredibly useful because at any later point in time you can pick that session up exactly from where you left it by simply “attaching” to that session.
If you’ve ever worked with remote servers or a Raspberry Pi over ssh you can guess where this will be useful: When you lose your ssh connection the tmux session will simply be detached but will keep running on the server in the background including all the processes that run within your session. To continue your session simply ssh to the server again and attach to the running session.
tmux is helpful when working on a remote machine but it shines just as much when you’re working locally. Not only for its window management features but also for the session handling. 

Jiho: Personally I find myself detaching from sessions when I’m switching context. I’ll just start a new session for my new task and attach to the old session whenever I want to continue with my old task.

### How to Install in remote server (ssh)
1. Create a source directory:
   ```sh
   mkdir -p /net/capricorn/home/xing/jipark25/src
   cd /net/capricorn/home/xing/jipark25/src
   ```
2. Download and compile ‘libevent’ ‘ncurses’ and ‘tmux’
   #### Downloading libevent
   ```sh
   wget https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz
   tar -xzf libevent-2.1.12-stable.tar.gz
   cd libevent-2.1.12-stable
   ./configure --prefix=$HOME/local
   make
   make install
   ```
   #### Downloading ncurses
   ```sh
   cd ~/local/src
   wget https://ftp.gnu.org/pub/gnu/ncurses/ncurses-6.3.tar.gz
   tar -xzf ncurses-6.3.tar.gz
   cd ncurses-6.3
   ./configure --prefix=$HOME/local
   make
   make install
   ```
   #### Downloading tmux
   ```sh
   wget https://github.com/tmux/tmux/releases/download/3.3a/tmux-3.3a.tar.gz
   tar -xzf tmux-3.3a.tar.gz
   cd tmux-3.3a
   ./configure CFLAGS="-I$HOME/local/include -I$HOME/local/include/ncurses" LDFLAGS="-L$HOME/local/lib" --prefix=$HOME/local
   make
   make install
   ```

### Basic tmux usage
- Start a new session: `tmux`
- List sessions: `tmux ls`
- Detach from a session: `Ctrl+b then d`
- Attach to a specific session: `tmux attach -t mysession`
- Kill a session: `tmux kill-session -t mysession`
- Attach to a session: `tmux attach -t 0`
- Make a new window: `Ctrl+b then c`
- Switch between windows:
  - `Ctrl+b then n`  # Next window
  - `Ctrl+b then p`  # Previous window
  - `Ctrl+b then [0-9]`  # Specific window

### Example Workflow
```sh
tmux
Long running command
Detach from a session: Ctrl+b d
Later you can go back to working session
tmux ls
tmux attach -t detached_session
```

### How It Works
When you run a tmux session on a remote server the processes you start within that session are executed on the remote server itself not on your local machine. Here’s how this setup works:
#### Local Machine:
- Your local machine (laptop or desktop) is used to initiate an SSH connection to the remote server.
- Once connected you start a tmux session on the remote server.
- Any commands or processes you run within tmux are executed on the remote server.

#### Remote Server:
- The remote server is a computer (or a cluster of computers) managed by your university or organization.
- The server continues to run the tmux session and all associated processes even if your local machine disconnects.