# oto_rollback

```
nano oto_rollback.sh
```
```
#!/bin/bash

# Log file
LOG_FILE="/var/log/stationd.log"

# Function to check if the log flow is broken
check_log_flow() {
    # Check for specific error messages
    grep -q -e "Error in VerifyPod transaction" -e "rpc error: code = Unknown desc" "$LOG_FILE"
}

# Check the log flow
if check_log_flow; then
    # If the log flow is broken, perform actions
    echo "Log flow is broken, initiating actions..."
    
    # Stop the stationd service
    systemctl stop stationd
    
    # Ensure the service is completely stopped
    while systemctl is-active --quiet stationd; do
        sleep 1
    done
    
    # Change to the tracks directory
    cd tracks
    
    # Perform the rollback operation
    go run cmd/main.go rollback
    
    # Restart the stationd service
    sudo systemctl restart stationd
    
    # Follow the log flow
    sudo journalctl -u stationd -f --no-hostname -o cat
else
    echo "Log flow is normal, no action taken."
fi

```

```
chmod +x oto_rollback.sh
```
```
./oto_rollback.sh
```
