# Cosmovisor Upgrade Guide V7.2.x

{% hint style="warning" %}
❗️Please note that the current fxCore Testnet v7.2.x upgrade, the upgrade height is 14,369,500, and the upgrade countdown is [**Countdown Timer**](https://testnet.starscan.io/fxcore/block/14389000?chainId=fxcore).
{% endhint %}

Here is the guide to Cosmovisor upgrade `fxcored` preparation. This requires you to complete the upgrade preparation after the upgrade proposal is approved and before reaching the upgrade height.

> For more information on past upgrades and instructions, refer to [**Upgrade Versions**](../versions/).
>
> You may refer to this [**Starscan Countdown Timer**](https://testnet.starscan.io/fxcore/block/countdown/14389000?chainId=fxcore) which will countdown the time till the upgrade height.

> `cosmovisor` is a small process manager for Cosmos SDK application binaries that monitors the governance module for incoming chain upgrade proposals. If it sees a proposal that gets approved, cosmovisor can automatically download the new binary, stop the current binary, switch from the old binary to the new one, and finally restart the node with the new binary.

{% hint style="info" %}
**Go 1.21+** or later is required for the f(x)Core. Install `go` by following the [official docs](https://golang.org/doc/install).
{% endhint %}

## 1. Install Cosmovisor

```sh
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```

Set up the Cosmovisor environment variables. Creates the folder structure required for using cosmovisor.

{% hint style="warning" %}
if you have used cosmovisor before, you can skip this step. Or you can use `rm -rf $HOME/.fxcore/cosmovisor` to reset
{% endhint %}

```sh
git clone https://github.com/functionx/fx-core.git
cd fx-core

git checkout release/v6.0.x

make build
```

```sh
export DAEMON_NAME=fxcored DAEMON_HOME=$HOME/.fxcore DAEMON_POLL_INTERVAL=1s UNSAFE_SKIP_BACKUP=true
cosmovisor init ./build/bin/fxcored
mkdir -p $HOME/.fxcore/cosmovisor/upgrades/v6.0.x/bin/
cp ./build/bin/fxcored $HOME/.fxcore/cosmovisor/upgrades/v6.0.x/bin/
cosmovisor version
```

## 2. Install the fxcore release

{% hint style="info" %}
Releases can be found here [https://github.com/FunctionX/fx-core/releases](https://github.com/FunctionX/fx-core/releases)
{% endhint %}

<pre class="language-sh"><code class="lang-sh">git clone https://github.com/functionx/fx-core.git
cd fx-core
<strong>git pull &#x26;&#x26; git checkout v7.2.0-rc2
</strong>make build
</code></pre>

```sh
mkdir -p $HOME/.fxcore/cosmovisor/upgrades/v7.2.x/bin
cp ./build/bin/fxcored $HOME/.fxcore/cosmovisor/upgrades/v7.2.x/bin/
cp ./build/bin/fxcored $(go env GOPATH)/bin/
```

To check that you did this correctly, ensure your versions of `cosmovisor` are the same:

```sh
export DAEMON_NAME=fxcored DAEMON_HOME=$HOME/.fxcore DAEMON_POLL_INTERVAL=1s UNSAFE_SKIP_BACKUP=true
cosmovisor version
```

```
cosmovisor version: v1.5.0
10:29AM INF running app args=["version"] module=cosmovisor path=/root/.fxcore/cosmovisor/upgrades/v7.1.x/bin/fxcored
HEAD-v7.1.0-rc1
```

In addition, we have added the feature of the `doctor` command in the v4 version, which is used to check whether the environment you are currently running is correct. if you see the warning, please contact our technical support.

```sh
./build/bin/fxcored doctor
OR
fxcored doctor
```

{% hint style="info" %}
If the node has not been started, the output of the doctor command will shown "Blockchain Data" section is unavailable.
{% endhint %}

## 3. Start your node

To keep the process always running. If you're on linux, you can do this by creating a service.

```sh
sudo tee /etc/systemd/system/fxcorevisor.service > /dev/null <<EOF
[Unit]
Description=fxCore Daemon
After=network-online.target

[Service]
User=root
ExecStart=$(which cosmovisor) run start --home=/root/.fxcore
Restart=always
RestartSec=3
LimitNOFILE=infinity

Environment="DAEMON_HOME=/root/.fxcore"
Environment="DAEMON_NAME=fxcored"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_POLL_INTERVAL=1s"

[Install]
WantedBy=multi-user.target
EOF
```

{% hint style="info" %}
⚠️Before this, please make sure you have stopped `fxcored` and deleted the old `fxcored.service` file, if not, please execute the following command:
{% endhint %}

```sh
sudo systemctl stop fxcored
sudo rm -rf /etc/systemd/system/fxcored.service
```

Reload, enable and restart the node with daemon service file

```sh
sudo -S systemctl daemon-reload
sudo -S systemctl enable fxcorevisor
sudo -S systemctl restart fxcorevisor
```

## Troubleshooting

Checking working environment

```
fxcored doctor
```

Accessing logs

{% tabs %}
{% tab title="Entire Log" %}
```
journalctl -u fxcorevisor -f
```
{% endtab %}
{% endtabs %}