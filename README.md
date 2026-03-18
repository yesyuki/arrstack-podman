# arrstack-podman
Tested on Fedora Linux 42 (Server Edition)
Gemini helped creating the quadlet. 

"Internal Root" (UID 0) is actually just the unprivileged host user who ran the command. It is safe, and it's the only way to ensure the internal container scripts can manage the /config volume without crashing on Fedora.
