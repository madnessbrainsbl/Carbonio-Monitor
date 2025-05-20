import subprocess
import re
import sys
from datetime import datetime


USER = "zextras"
LOG_FILE = "path"
CRITICAL = {
    "mailbox": ["zmmailboxd", "start"],
    "config service": ["zmconfid", "start"]
}


def log_message(message: str):
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    with open(LOG_FILE, "a") as f:
        f.write(f"[{timestamp}] {message}\n")


def run_as_user(command: list):
    cmd = ["sudo", "-u", USER] + command
    try:
        subprocess.run(cmd, check=True)
        log_message(f"Ran: {' '.join(command)} as {USER}")
    except subprocess.CalledProcessError as e:
        log_message(f"ERROR running {' '.join(command)}: {e}")


def monitor_services():

    proc = subprocess.Popen(
        ["sudo", "-u", USER, "zmcontrol", "status"],
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        text=True,
        bufsize=1
    )

    if proc.stdout is None:
        log_message("ERROR: No output from zmcontrol status")
        return


    for raw_line in proc.stdout:
        line = raw_line.strip()
        if not line or line.startswith("Host") or line.startswith("--Checking"):
            continue


        clean = re.sub(r"^-+", "", line).strip()
        parts = clean.split()
        # Determine multi-word statuses
        if len(parts) < 2:
            continue
        if parts[-2] == "NOT":
            status = "NOT running"
            name = " ".join(parts[:-2])
        else:
            status = parts[-1]
            name = " ".join(parts[:-1])

        log_message(f"Service '{name}' status: {status}")


        key = name.lower()
        if key in CRITICAL and status.lower() != "running":
            cmd = CRITICAL[key]
            log_message(f"Service '{name}' is down. Attempting restart...")
            run_as_user(cmd)

    proc.wait()


if __name__ == "__main__":
    try:
        monitor_services()
    except Exception as e:
        log_message(f"Unexpected error: {e}")
        sys.exit(1)
