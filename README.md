## Configuring an Anaconda script to launch on startup

[This tutorial](https://github.com/torfsen/python-systemd-tutorial) is a good starting point; the below notes relate specifically to running a python script from within a user's anaconda environment on boot.

First, choose a preferred location for your script, for example:

```
mkdir -p ~/systemd/
```

In `~/systemd/service.py`:

```python
import time
import datetime

def main():
    while True:
        print(datetime.datetime.now())
        time.sleep(10)

if __name__ == '__main__':
    main()
```

Next, include the following in `~/systemd/on_start.sh`:

```bash
#!/bin/bash
source $HOME/miniconda3/etc/profile.d/conda.sh
conda activate base # change to your conda environment's name
# -u: unbuffered output
python -u service.py
```

In order to run the `source` command, specifying `bash` is necessary. You might need to change this to `anaconda3` (see [here](https://github.com/conda/conda/issues/7980#issuecomment-441358406) for details). Consult [this](https://github.com/torfsen/python-systemd-tutorial#stdout-and-stderr) for details on unbuffered output.


Then `chmod +x ~/systemd/on_start.sh` and test it.


In `~/.config/systemd/user/example.service`:

```
[Unit]
Description=Example Python Service

[Service]
Type=simple
WorkingDirectory=/home/ubuntu/systemd/
ExecStart=/home/ubuntu/systemd/on_start.sh

[Install]
WantedBy=default.target
```

Note that this specifies a working directory. You should now see `example.service` listed when running `systemctl --user list-unit-files`.

To test:

```bash
systemctl --user enable example.service
systemctl --user start example.service
```

Run `journalctl --user-unit example.service` (or `tail -f /var/log/syslog`) to observe output, and inspect status with `systemctl --user status example.service`.

**Note:** You can tail the output of `journalctl` with the `-f` flag: `journalctl --user-unit example.service -f`

To run the service on boot, without requiring the user to login:

`sudo loginctl enable-linger $USER`

This can be undone with `sudo loginctl disable-linger $USER`, and status can be queried via [this command](https://serverfault.com/questions/846441/loginctl-enable-linger-disable-linger-but-reading-linger-status).

To stop the service and prevent it from running on boot:

```bash
systemctl --user stop example.service
systemctl --user disable example.service
```
