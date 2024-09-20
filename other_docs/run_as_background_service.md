# lsfgateway as a background service

In this document, we have provided an example on how to install lsfgateway as a background service in Ubuntu. 


## Add a New User for lsfgateway

### Step 1: Create a User Without Login Access

First, create a system user that does not have login access. Use the following command:
```sh
sudo adduser --system --shell /usr/sbin/nologin lsfgatewayuser
```

- The `--system` flag creates a system user.
- The `--shell` /usr/sbin/nologin flag ensures that the user cannot log into the system.

This will also create a home directory for the user at `/home/lsfgatewayuser`.


### Step 2: Copy Files to the Server

After downloading lsfgateway, move the binary to the appropriate directory:

```sh
sudo cp path/to/compiled/lsfgateway /usr/local/bin/lsfgateway
```

Next, set the correct ownership and permissions for the binary:
```sh
sudo chown lsfgatewayuser: /usr/local/bin/lsfgateway
sudo chmod 755 /usr/local/bin/lsfgateway
```
- `chown` sets the user as the owner of the file.
- `chmod 755` ensures that the file is executable by the owner and readable by others.


### Step 3: Create a Systemd Service File 

Navigate to the `/etc/systemd/system/` directory and create a new service file:

```bash
sudo nano /etc/systemd/system/lsfgateway.service
```

Here’s an example template for the service file:
```sh
[Unit]
Description=LSF Gateway Service
After=network.target

[Service]
User=lsfgatewayuser
Group=nogroup
ExecStart=/usr/local/bin/lsfgateway -f /home/lsfgatewayuser/config_file.yml -m monitoring
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

#### Explanation:

- **Description:** A brief description of the service.
- **After:** Ensures that the service starts after the network is up.
- **User:** The user under which the service will run.
- **Group:** The group under which the service will run.
- **ExecStart:** Specifies the command to start the lsfgateway binary, passing any required flags.
- **Restart:** Automatically restarts the service if it fails.

##### Some other values that could be used 
For future reference, here’s additional content you can use in the service configuration:

```bash
[Service]
WorkingDirectory=/path/to/working/directory
Restart=always
Environment=PATH=/usr/bin:/usr/local/bin
Environment=OTHER_ENV_VAR=value
```

- **Description:** A brief description of the service.
- **After:** Specifies that the service should start after the network is up.
- **ExecStart:** The command to start your Rust program. Replace /path/to/your/binary with the actual path to your binary.
- **WorkingDirectory:** The directory from which the program will run (optional).
- **Restart:** Ensures the service will restart automatically if it crashes. always is a good default.
- **Environment:** Environment variables needed by your program (optional).

### Step 4: Reload Systemd and Start the Service
After saving the service file, reload systemd to recognize the new service and start it:
```sh
sudo systemctl daemon-reload
sudo systemctl start lsfgateway.service
```

### Step 5: Enable the Service to Start on Boot

To make sure the service starts automatically on system boot, enable it:
```sh
sudo systemctl enable lsfgateway.service
```

### Step 6: Verify the Service
Finally, check the status of the service to confirm it's running as expected:
```
sudo systemctl status lsfgateway.service
```
