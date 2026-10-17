# Text to PowerPoint Telegram bot

## Run locally

Install dependencies and set `BOT_TOKEN` in a local `.env` file (see
[.env.example](.env.example)):

```sh
python3 -m venv .venv
. .venv/bin/activate
python -m pip install -r requirements.txt
python bot.py
```

## Deploy both bots to one Ubuntu VM

This bot uses Telegram long polling, so the VM needs outbound internet access
but no public web server or inbound application port. The included
[systemd unit](deploy/txt-to-pptx.service) starts it on boot and restarts it if
it exits.

1. Install Python, venv support, and Git:

   ```sh
   sudo apt update
   sudo apt install -y git python3 python3-venv
   ```

2. Create a dedicated service account and application directory. Run the
   `useradd` command only if you have not already created this account for the
   other bot:

   ```sh
   sudo useradd --system --create-home --home-dir /opt/bots --shell /usr/sbin/nologin botuser
   sudo install -d -o botuser -g botuser /opt/bots/txt_to_pptx
   sudo -u botuser git clone https://github.com/O1ek5andr/txt_to_pptx.git /opt/bots/txt_to_pptx
   ```

3. Create a private environment file and put the token from BotFather in it:

   ```sh
   sudo install -m 600 /dev/null /etc/txt-to-pptx.env
   sudo nano /etc/txt-to-pptx.env
   ```

   Add `BOT_TOKEN=your-real-token`. Do not commit this file or copy a `.env`
   file into the repository.

4. Create the virtual environment, install dependencies, and install the
   service:

   ```sh
   sudo -u botuser python3 -m venv /opt/bots/txt_to_pptx/.venv
   sudo -u botuser /opt/bots/txt_to_pptx/.venv/bin/pip install -r /opt/bots/txt_to_pptx/requirements.txt
   sudo cp /opt/bots/txt_to_pptx/deploy/txt-to-pptx.service /etc/systemd/system/
   sudo systemctl daemon-reload
   sudo systemctl enable --now txt-to-pptx
   ```

5. Check status and logs:

   ```sh
   sudo systemctl status txt-to-pptx
   sudo journalctl -u txt-to-pptx -f
   ```

The German Articles bot can run on the same VM with its own service and token.
Don't run two polling processes with the same Telegram token.
