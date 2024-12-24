# Motivations

Why would you want to leave your trusted IDE (Jetbrains RubyMine) to 
start development on ```vim```?

DHH explains it quite well in his inspirational podcast. Besides, this 
allow me to develop over ssh on a remote VM, which is good because 
the libraries (oracle database, service bus, etc) are really aligned with 
production environment.

## What do you need?

* a terminal (iTerm2 or alacritty);
* a session multiplexer (zellij or tmux);
* either ```nvim``` or a package-version of ```nvim``` like lazy-vim or similar.

The terminal is necessary as a shell container; the session handler provides 
the sessions utilities to pause/resume work and the terminal facility. 
Terminal is really necessary for git and run tests and it is not practical
to switch in/out from work session to invoke it (but it is possible).
