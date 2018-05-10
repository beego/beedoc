---
name: Deployment with Systemctl 
sort: 2
---

# Systemctl

Systemctl command is a command used to manage and control services. It allows you to enable, disable, view, start, stop, or restart system services.
 
## Install beego application as a service

1. Pack your application using `bee pack` command. Copy the resultant `.tar.gz` file to target server.
		
2. Unpack 
                
		mkdir -p /usr/local/beepkg && cd "$_"
		tar -xvzf *.tar.gz
		rm -rf *.tar.gz
		
3. Configure service parameters

		cat <<'EOF' > /etc/systemd/system/beepkg.service
        [Unit]
        Description=beepkg
        AssertPathExists=/usr/local/beepkg
        
        [Service]
        WorkingDirectory=/usr/local/beepkg
        ExecStart=/usr/local/beepkg/beepkg
        
        ExecReload=/bin/kill -HUP $MAINPID
        LimitNOFILE=65536
        Restart=always
        RestartSec=5
        
        [Install]
        WantedBy=multi-user.target
        EOF

3. Install service 

        chmod +x beepkg
        chmod 644 /etc/systemd/system/beepkg.service
        systemctl daemon-reload
        systemctl enable beepkg
        systemctl start beepkg
        systemctl status beepkg

		
		



