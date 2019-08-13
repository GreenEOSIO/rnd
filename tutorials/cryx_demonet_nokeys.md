# Cryptnox demonet installation log

## Bootstrapping

This is an installation log for a demo blockchain for Cryptnox
project. It's a fully functioning EOSIO blockchain with 5 producer
nodes. All private keys are truncated.

The `api01` server is a third-party machine that has enough RAM to
compile the system contracts. It's only used temporarily to bootstrap
the network and then removed from the blockchain. The "donkey" LXC
container was used previously for bootstrapping a testnet, so all
software is already in place.

```
ssh -A root@api01.eostest.xeos.me

cat >>/etc/lxc/dnsmasq.conf <<'EOT'
dhcp-host=donkey,10.0.3.80
EOT
systemctl restart lxc-net


zfs create -o mountpoint=/var/lib/lxc/donkey -o compression=lz4 zslow/donkey

lxc-create -n donkey -t download -- --dist ubuntu --release cosmic --arch amd64
lxc-start -n donkey
lxc-attach -n donkey

apt-get update && apt-get install -y git openssh-server net-tools
userdel -rf ubuntu
exit

cp -a .ssh/ /var/lib/lxc/donkey/rootfs/root/

ssh -A root@10.0.3.80

cd /var/local
wget https://github.com/EOSIO/eos/releases/download/v1.8.1/eosio_1.8.1-1-ubuntu-18.04_amd64.deb
apt install ./eosio_1.8.1-1-ubuntu-18.04_amd64.deb


## EOS compiler installation

cd /var/local
wget https://github.com/EOSIO/eosio.cdt/releases/download/v1.5.0/eosio.cdt_1.5.0-1_amd64.deb
apt install -y ./eosio.cdt_1.5.0-1_amd64.deb


## Boostrapping the network

# Reference: https://developers.eos.io/eosio-nodeos/docs/bios-boot-sequence
# Using the tutorial script: https://github.com/EOSIO/eos/tree/master/tutorials/bios-boot-tutorial

# A bugfix for custom token name is published in release/1.8m but not in master branch:
# https://github.com/EOSIO/eos/commit/94368b38c2da1384502104890be9fd91a91846d6

sudo apt install -y cmake clang git python3-numpy
mkdir /opt/src
cd /opt/src
git clone -b release/1.8.x https://github.com/EOSIO/eosio.contracts.git
cd ./eosio.contracts/
./build.sh

cd /opt/src
git clone https://github.com/EOSIO/eos.git
cd ./eos/tutorials/bios-boot-tutorial/

cleos create key --to-console

## Genesis initial key
Private key: 5KjpNLdWXyQ6PQ9UeY
Public key: EOS8mDTyHhRCLSsosVQmVDLWpPC3ve2CvLt79rYCQB6xYJ4LPsLXQ

## prod1
Private key: 5KZzwyzj4EqwjHQfT
Public key: EOS8Q4L7YjDEPyiHSqFjYwEPCTP9UeiXkQwTkPEvGZ86buY7Uhxyz

## prod2
Private key: 5KCCcfjtbw6ooBH9F
Public key: EOS618Hxf9VyXsSMygQ53PAvwNUgJbBQWioaWBG8sYsqrCDjuMfFr

## prod3
Private key: 5J2DjE7JLSkdBbsM5
Public key: EOS7GiJqjTHp3JSE7qiseA3BzEnUvF9L4acV1G6kTQbEoaaypeMDW

# prod4
Private key: 5Jh4UbgpTr1DkUXm5
Public key: EOS5GKiZvXPkEN83jxWLv1hB7z5Dz7UpHjRQFjQkXm1A6zNFkNJdo

# prod5
Private key: 5K6aFYoRp4TMwrY4q
Public key: EOS5b2psui7Kig2cJPGGx9xywAPvtgC5oo7jVHG6q24p5nMDqJLz5

## useraaaaaaaa
Private key: 5JAz2dc5aXWcEhqWB
Public key: EOS8Jj5HJzXkX4zkq9617ML7tAdTUdcdm4u5Lbk7UUTsPfJjoKcqy


cat >genesis.json <<'EOT'
{
  "initial_timestamp": "2019-08-13T00:00:00.000",
  "initial_key": "EOS8mDTyHhRCLSsosVQmVDLWpPC3ve2CvLt79rYCQB6xYJ4LPsLXQ",
  "initial_configuration": {
    "max_block_net_usage": 1048576,
    "target_block_net_usage_pct": 1000,
    "max_transaction_net_usage": 524288,
    "base_per_transaction_net_usage": 12,
    "net_usage_leeway": 500,
    "context_free_discount_net_usage_num": 20,
    "context_free_discount_net_usage_den": 100,
    "max_block_cpu_usage": 100000,
    "target_block_cpu_usage_pct": 500,
    "max_transaction_cpu_usage": 50000,
    "min_transaction_cpu_usage": 100,
    "max_transaction_lifetime": 3600,
    "deferred_trx_expiration_window": 600,
    "max_transaction_delay": 3888000,
    "max_inline_action_size": 4096,
    "max_inline_action_depth": 4,
    "max_authority_depth": 6
  },
  "initial_chain_id": "0000000000000000000000000000000000000000000000000000000000000000"
}
EOT

cat >accounts.json <<'EOT'
{
    "producers": [
        {"name":"prod1", "pvt":"5KZzwyzj4EqwjHQfTxBSchYot",
                                "pub":"EOS8Q4L7YjDEPyiHSqFjYwEPCTP9UeiXkQwTkPEvGZ86buY7Uhxyz"},
        {"name":"prod2", "pvt":"5KCCcfjtbw6ooBH9FAtmDc7D",
                                "pub":"EOS618Hxf9VyXsSMygQ53PAvwNUgJbBQWioaWBG8sYsqrCDjuMfFr"},
        {"name":"prod3", "pvt":"5J2DjE7JLSkdBbsM54ZGfv1p",
                                "pub":"EOS7GiJqjTHp3JSE7qiseA3BzEnUvF9L4acV1G6kTQbEoaaypeMDW"},
        {"name":"prod4", "pvt":"5Jh4UbgpTr1DkUXm5qDKaXe",
                                "pub":"EOS5GKiZvXPkEN83jxWLv1hB7z5Dz7UpHjRQFjQkXm1A6zNFkNJdo"},
        {"name":"prod5", "pvt":"5K6aFYoRp4TMwrY4qVdFvhJ",
                                "pub":"EOS5b2psui7Kig2cJPGGx9xywAPvtgC5oo7jVHG6q24p5nMDqJLz5"}
    ],
    "users": [
        {"name":"useraaaaaaaa", "pvt":"5JAz2dc5aXWcEhqW",
                                "pub":"EOS8Jj5HJzXkX4zkq9617ML7tAdTUdcdm4u5Lbk7UUTsPfJjoKcqy"}
    ]
}
EOT


# This will launch a fully functional network

python3 bios-boot-tutorial.py --cleos="cleos --wallet-url http://127.0.0.1:6666 " --nodeos=nodeos --keosd=keosd --contracts-dir="/opt/src/eosio.contracts/build/contracts" -a --public-key EOS8mDTyHhRCLSsosVQmVDLWpPC3ve2CvLt79rYCQB6xYJ4LPsLXQ --private-Key 5KjpNLdWXyQ6PQ9UeYqmW --num-producers-vote 5 --num-voters 1 --num-senders 1 --symbol CRYX


# Create an p2p node for public access 

mkdir -p /srv/cryx/etc

cat >/srv/cryx/etc/config.ini <<'EOT'
chain-state-db-size-mb = 8192
wasm-runtime = wabt
http-server-address = 127.0.0.1:8801
p2p-listen-endpoint = 0.0.0.0:9901
allowed-connection = any
access-control-allow-origin = *
verbose-http-errors = true
contracts-console = true
max-irreversible-block-age = -1
max-clients = 100
p2p-max-nodes-per-host = 100
plugin = eosio::chain_plugin
plugin = eosio::producer_api_plugin
plugin = eosio::chain_api_plugin
p2p-peer-address = 127.0.0.1:9001
p2p-peer-address = 127.0.0.1:9002
p2p-peer-address = 127.0.0.1:9003
p2p-peer-address = 127.0.0.1:9004
p2p-peer-address = 127.0.0.1:9005
EOT

cp /opt/src/eos/tutorials/bios-boot-tutorial/genesis.json /srv

cat >/etc/systemd/system/nodeos\@.service <<'EOT'
[Unit]
Description=nodeos
[Service]
Type=simple
ExecStart=/usr/bin/nodeos --data-dir /srv/%i/data --config-dir /srv/%i/etc
TimeoutStartSec=30s
TimeoutStopSec=300s
Restart=no
User=root
Group=daemon
KillMode=control-group
[Install]
WantedBy=multi-user.target
EOT

systemctl daemon-reload

/usr/bin/nodeos --data-dir /srv/cryx/data --config-dir /srv/cryx/etc --genesis-json=/srv/genesis.json

systemctl enable nodeos@cryx
systemctl start nodeos@cryx
journalctl -u nodeos@cryx -f

exit

iptables -t nat -A PREROUTING -d 85.17.224.203/32 -p tcp -m tcp --dport 9901 -j DNAT --to-destination 10.0.3.80:9901
```


## Bringing up producer servers

```
node1.uk.cryptnox.ch  54.36.163.132
node2.de.cryptnox.ch  51.38.126.237
node3.ca.cryptnox.ch  192.99.55.167
node4.pl.cryptnox.ch  54.37.235.45
node5.fr.cryptnox.ch  51.83.97.82
```

```
### common for all servers

cd /var/local
wget https://github.com/EOSIO/eos/releases/download/v1.8.1/eosio_1.8.1-1-ubuntu-18.04_amd64.deb
apt install ./eosio_1.8.1-1-ubuntu-18.04_amd64.deb

cat >/etc/systemd/system/nodeos\@.service <<'EOT'
[Unit]
Description=nodeos
[Service]
Type=simple
ExecStart=/usr/bin/nodeos --data-dir /srv/%i/data --config-dir /srv/%i/etc
TimeoutStartSec=30s
TimeoutStopSec=300s
Restart=no
User=root
Group=daemon
KillMode=control-group
[Install]
WantedBy=multi-user.target
EOT

systemctl daemon-reload

mkdir -p /srv/cryx/etc
cat >/srv/cryx/etc/genesis.json <<'EOT'
{
  "initial_timestamp": "2019-08-13T00:00:00.000",
  "initial_key": "EOS8mDTyHhRCLSsosVQmVDLWpPC3ve2CvLt79rYCQB6xYJ4LPsLXQ",
  "initial_configuration": {
    "max_block_net_usage": 1048576,
    "target_block_net_usage_pct": 1000,
    "max_transaction_net_usage": 524288,
    "base_per_transaction_net_usage": 12,
    "net_usage_leeway": 500,
    "context_free_discount_net_usage_num": 20,
    "context_free_discount_net_usage_den": 100,
    "max_block_cpu_usage": 100000,
    "target_block_cpu_usage_pct": 500,
    "max_transaction_cpu_usage": 50000,
    "min_transaction_cpu_usage": 100,
    "max_transaction_lifetime": 3600,
    "deferred_trx_expiration_window": 600,
    "max_transaction_delay": 3888000,
    "max_inline_action_size": 4096,
    "max_inline_action_depth": 4,
    "max_authority_depth": 6
  },
  "initial_chain_id": "0000000000000000000000000000000000000000000000000000000000000000"
}
EOT


cat >/srv/cryx/etc/logging.json  <<'EOT'
{
  "includes": [],
  "appenders": [{
      "name": "consoleout",
      "type": "console",
      "args": {
        "stream": "std_out",
        "level_colors": [{
            "level": "debug",
            "color": "green"
          },{
            "level": "warn",
            "color": "brown"
          },{
            "level": "error",
            "color": "red"
          }
        ]
      },
      "enabled": true
    }
  ],
  "loggers": [{
      "name": "default",
      "level": "warn",
      "enabled": true,
      "additivity": false,
      "appenders": [
        "consoleout"
      ]
    }
  ]
}
EOT


###  per-server specifics

###### prod1 #####

cat >/srv/cryx/etc/config.ini <<'EOT'
chain-state-db-size-mb = 8192
wasm-runtime = wabt
http-server-address = 127.0.0.1:8888
p2p-listen-endpoint = 0.0.0.0:9901
allowed-connection = any
access-control-allow-origin = *
verbose-http-errors = true
contracts-console = true
producer-name = prod1
signature-provider = EOS8Q4L7YjDEPyiHSqFjYwEPCTP9UeiXkQwTkPEvGZ86buY7Uhxyz=KEY:5KZzwyzj4E
max-irreversible-block-age = -1
max-clients = 100
p2p-max-nodes-per-host = 100
plugin = eosio::chain_plugin
plugin = eosio::producer_plugin
plugin = eosio::producer_api_plugin
plugin = eosio::chain_api_plugin
p2p-peer-address = api01.eostest.xeos.me:9901
p2p-peer-address = node1.uk.cryptnox.ch:9901
p2p-peer-address = node2.de.cryptnox.ch:9901
p2p-peer-address = node3.ca.cryptnox.ch:9901
p2p-peer-address = node4.pl.cryptnox.ch:9901
p2p-peer-address = node5.fr.cryptnox.ch:9901
EOT

/usr/bin/nodeos --data-dir /srv/cryx/data --config-dir /srv/cryx/etc --genesis-json=/srv/cryx/etc/genesis.json

systemctl enable nodeos@cryx
systemctl start nodeos@cryx
sleep 1
cleos get info


###### prod2 #####

cat >/srv/cryx/etc/config.ini <<'EOT'
chain-state-db-size-mb = 8192
wasm-runtime = wabt
http-server-address = 127.0.0.1:8888
p2p-listen-endpoint = 0.0.0.0:9901
allowed-connection = any
access-control-allow-origin = *
verbose-http-errors = true
contracts-console = true
producer-name = prod2
signature-provider = EOS618Hxf9VyXsSMygQ53PAvwNUgJbBQWioaWBG8sYsqrCDjuMfFr=KEY:5KCCc
max-irreversible-block-age = -1
max-clients = 100
p2p-max-nodes-per-host = 100
plugin = eosio::chain_plugin
plugin = eosio::producer_plugin
plugin = eosio::producer_api_plugin
plugin = eosio::chain_api_plugin
p2p-peer-address = api01.eostest.xeos.me:9901
p2p-peer-address = node1.uk.cryptnox.ch:9901
p2p-peer-address = node2.de.cryptnox.ch:9901
p2p-peer-address = node3.ca.cryptnox.ch:9901
p2p-peer-address = node4.pl.cryptnox.ch:9901
p2p-peer-address = node5.fr.cryptnox.ch:9901
EOT

/usr/bin/nodeos --data-dir /srv/cryx/data --config-dir /srv/cryx/etc --genesis-json=/srv/cryx/etc/genesis.json

systemctl enable nodeos@cryx
systemctl start nodeos@cryx
sleep 1
cleos get info

###### prod3 #####

cat >/srv/cryx/etc/config.ini <<'EOT'
chain-state-db-size-mb = 8192
wasm-runtime = wabt
http-server-address = 127.0.0.1:8888
p2p-listen-endpoint = 0.0.0.0:9901
allowed-connection = any
access-control-allow-origin = *
verbose-http-errors = true
contracts-console = true
producer-name = prod3
signature-provider = EOS7GiJqjTHp3JSE7qiseA3BzEnUvF9L4acV1G6kTQbEoaaypeMDW=KEY:5J2DjE7JLSkdB
max-irreversible-block-age = -1
max-clients = 100
p2p-max-nodes-per-host = 100
plugin = eosio::chain_plugin
plugin = eosio::producer_plugin
plugin = eosio::producer_api_plugin
plugin = eosio::chain_api_plugin
p2p-peer-address = api01.eostest.xeos.me:9901
p2p-peer-address = node1.uk.cryptnox.ch:9901
p2p-peer-address = node2.de.cryptnox.ch:9901
p2p-peer-address = node3.ca.cryptnox.ch:9901
p2p-peer-address = node4.pl.cryptnox.ch:9901
p2p-peer-address = node5.fr.cryptnox.ch:9901
EOT

/usr/bin/nodeos --data-dir /srv/cryx/data --config-dir /srv/cryx/etc --genesis-json=/srv/cryx/etc/genesis.json

systemctl enable nodeos@cryx
systemctl start nodeos@cryx
sleep 1
cleos get info


###### prod4 #####

cat >/srv/cryx/etc/config.ini <<'EOT'
chain-state-db-size-mb = 8192
wasm-runtime = wabt
http-server-address = 127.0.0.1:8888
p2p-listen-endpoint = 0.0.0.0:9901
allowed-connection = any
access-control-allow-origin = *
verbose-http-errors = true
contracts-console = true
producer-name = prod4
signature-provider = EOS5GKiZvXPkEN83jxWLv1hB7z5Dz7UpHjRQFjQkXm1A6zNFkNJdo=KEY:5Jh4UbgpTr1D
max-irreversible-block-age = -1
max-clients = 100
p2p-max-nodes-per-host = 100
plugin = eosio::chain_plugin
plugin = eosio::producer_plugin
plugin = eosio::producer_api_plugin
plugin = eosio::chain_api_plugin
p2p-peer-address = api01.eostest.xeos.me:9901
p2p-peer-address = node1.uk.cryptnox.ch:9901
p2p-peer-address = node2.de.cryptnox.ch:9901
p2p-peer-address = node3.ca.cryptnox.ch:9901
p2p-peer-address = node4.pl.cryptnox.ch:9901
p2p-peer-address = node5.fr.cryptnox.ch:9901
EOT

/usr/bin/nodeos --data-dir /srv/cryx/data --config-dir /srv/cryx/etc --genesis-json=/srv/cryx/etc/genesis.json

systemctl enable nodeos@cryx
systemctl start nodeos@cryx
sleep 1
cleos get info


###### prod5 #####

cat >/srv/cryx/etc/config.ini <<'EOT'
chain-state-db-size-mb = 8192
wasm-runtime = wabt
http-server-address = 127.0.0.1:8888
p2p-listen-endpoint = 0.0.0.0:9901
allowed-connection = any
access-control-allow-origin = *
verbose-http-errors = true
contracts-console = true
producer-name = prod5
signature-provider = EOS5b2psui7Kig2cJPGGx9xywAPvtgC5oo7jVHG6q24p5nMDqJLz5=KEY:5K6aFYoRp4TM
max-irreversible-block-age = -1
max-clients = 100
p2p-max-nodes-per-host = 100
plugin = eosio::chain_plugin
plugin = eosio::producer_plugin
plugin = eosio::producer_api_plugin
plugin = eosio::chain_api_plugin
p2p-peer-address = api01.eostest.xeos.me:9901
p2p-peer-address = node1.uk.cryptnox.ch:9901
p2p-peer-address = node2.de.cryptnox.ch:9901
p2p-peer-address = node3.ca.cryptnox.ch:9901
p2p-peer-address = node4.pl.cryptnox.ch:9901
p2p-peer-address = node5.fr.cryptnox.ch:9901
EOT

/usr/bin/nodeos --data-dir /srv/cryx/data --config-dir /srv/cryx/etc --genesis-json=/srv/cryx/etc/genesis.json

systemctl enable nodeos@cryx
systemctl start nodeos@cryx
sleep 1
cleos get info


###  api01 node ####
# after resync, kill nodeos processes launched by bootstrap

ps axf | grep max-irre | awk '{print $1}' | xargs kill

# Update nodeos config
cat >/srv/cryx/etc/config.ini <<'EOT'
chain-state-db-size-mb = 8192
wasm-runtime = wabt
http-server-address = 127.0.0.1:8888
p2p-listen-endpoint = 0.0.0.0:9901
allowed-connection = any
access-control-allow-origin = *
verbose-http-errors = true
contracts-console = true
max-irreversible-block-age = -1
max-clients = 100
p2p-max-nodes-per-host = 100
plugin = eosio::chain_plugin
plugin = eosio::producer_api_plugin
plugin = eosio::chain_api_plugin
p2p-peer-address = node1.uk.cryptnox.ch:9901
p2p-peer-address = node2.de.cryptnox.ch:9901
p2p-peer-address = node3.ca.cryptnox.ch:9901
p2p-peer-address = node4.pl.cryptnox.ch:9901
p2p-peer-address = node5.fr.cryptnox.ch:9901
EOT

systemctl restart nodeos@cryx
```

## Wallets and accounts

```
cleos wallet create -n useraaaaaaaa --to-console
# Password: "PW5HveSTt1j6bJiPnHQtkq"

cleos wallet import -n useraaaaaaaa --private-key 5JAz2dc5aXWcEhqWBVzs3TA

cleos wallet create -n prods --to-console
# Password: "PW5JfJvs5J4KpCMJyBMfMf"

cleos wallet import -n prods --private-key 5KZzwyzj4EqwjHQfTxBS
cleos wallet import -n prods --private-key 5KCCcfjtbw6ooBH9FAtm
cleos wallet import -n prods --private-key 5J2DjE7JLSkdBbsM54ZG
cleos wallet import -n prods --private-key 5Jh4UbgpTr1DkUXm5qDK
cleos wallet import -n prods --private-key 5K6aFYoRp4TMwrY4qVdF

cleos wallet unlock  -n prods --password PW5JfJvs5J4KpCMJyBMfMf

cleos multisig propose p1 '[{"actor": "prod1", "permission": "active"},{"actor": "prod2", "permission": "active"},{"actor": "prod3", "permission": "active"},{"actor": "prod4", "permission": "active"},{"actor": "prod5", "permission": "active"}]' '[{"actor": "eosio", "permission": "active"}]' eosio.token issue '{"to": "useraaaaaaaa", "quantity": "1000000.0000 CRYX", "memo": ""}' -p prod1

cleos multisig approve prod1 p1 '{"actor": "prod1", "permission": "active"}' -p prod1
cleos multisig approve prod1 p1 '{"actor": "prod2", "permission": "active"}' -p prod2
cleos multisig approve prod1 p1 '{"actor": "prod3", "permission": "active"}' -p prod3
cleos multisig approve prod1 p1 '{"actor": "prod4", "permission": "active"}' -p prod4
cleos multisig approve prod1 p1 '{"actor": "prod5", "permission": "active"}' -p prod5

cleos multisig exec prod1 p1 -p prod1


##### New accounts  ###########

cleos wallet unlock  -n useraaaaaaaa --password PW5HveSTt1j6bJiP

cleos system newaccount --stake-net "1.0000 CRYX" --stake-cpu "1.0000 CRYX" --buy-ram-kbytes 20 useraaaaaaaa cc32dninexxx EOS7txFiAr3fFzmQNUbxpnPV5ApYjq22igdYVYatFHgo1Vx6Vr553
cleos system newaccount --stake-net "1.0000 CRYX" --stake-cpu "1.0000 CRYX" --buy-ram-kbytes 20 useraaaaaaaa cc32dnineexp cc32dninexxx@owner EOS77qVt59tgxMos8Ru5Wqz7yLUwAT95v2pFf7CYWpS3CW7quE1we

cleos system newaccount --stake-net "1.0000 CRYX" --stake-cpu "1.0000 CRYX" --buy-ram-kbytes 1024 useraaaaaaaa escrowescrow cc32dninexxx@active

cleos transfer useraaaaaaaa cc32dninexxx "100 CRYX" "welcome"
cleos transfer useraaaaaaaa cc32dnineexp "100 CRYX" "welcome"

cleos system newaccount --stake-net "1.0000 CRYX" --stake-cpu "1.0000 CRYX" --buy-ram-kbytes 20 useraaaaaaaa switzerlanda EOS6NxStLYqpfLBna7KcARLJHPyBYmH8jAdn5LmQGgUX99AqzF1Mz EOS6esBNwa68TjmDDCUqW4HW2yicrqHxEEMSBoaLLj8kknR5u24M9

cleos transfer useraaaaaaaa switzerlanda "100 CRYX" "welcome"
```

## public API with history plugin

```
apt-get install -y nginx
rm /etc/nginx/sites-enabled/default
rm /etc/nginx/sites-available/default

cat >/etc/nginx/sites-available/cryptnox.xeos.me.conf <<'EOT'
server {
    listen 80;
    listen [::]:80;
    root /var/www/html;
    server_name cryptnox.xeos.me;
}
EOT

ln -s /etc/nginx/sites-available/* /etc/nginx/sites-enabled/

add-apt-repository ppa:certbot/certbot
apt-get update && apt-get install python-certbot-nginx

certbot --nginx -m cc32d9@gmail.com -d cryptnox.xeos.me

vi /etc/nginx/sites-available/cryptnox.xeos.me.conf
---------------------------------------------------------------------------
location / {
     if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        #
        # Custom headers and headers various browsers *should* be OK with but aren't
        #
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
        #
        # Tell client that this pre-flight info is valid for 20 days
        #
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain; charset=utf-8';
        add_header 'Content-Length' 0;
        return 204;
     }
     if ($request_method = 'POST') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
        add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
     }
     if ($request_method = 'GET') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
        add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
     }
}

    location /v1 {
        proxy_pass http://127.0.0.1:8889;
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;
    }
---------------------------------------------------------------------------


systemctl restart nginx

# API node

mkdir -p /srv/api/etc/

cat >/srv/api/etc/config.ini <<'EOT'
chain-state-db-size-mb = 8192
wasm-runtime = wabt
http-server-address = 127.0.0.1:8889
p2p-listen-endpoint = 0.0.0.0:9999
allowed-connection = any
access-control-allow-origin = *
http-validate-host = false
verbose-http-errors = true
contracts-console = true
max-clients = 100
plugin = eosio::chain_plugin
plugin = eosio::chain_api_plugin
plugin = eosio::history_plugin
plugin = eosio::history_api_plugin
filter-on = *
p2p-peer-address = node1.uk.cryptnox.ch:9901
p2p-peer-address = node2.de.cryptnox.ch:9901
p2p-peer-address = node3.ca.cryptnox.ch:9901
p2p-peer-address = node4.pl.cryptnox.ch:9901
p2p-peer-address = node5.fr.cryptnox.ch:9901
EOT



cat >/srv/api/etc/logging.json  <<'EOT'
{
  "includes": [],
  "appenders": [{
      "name": "consoleout",
      "type": "console",
      "args": {
        "stream": "std_out",
        "level_colors": [{
            "level": "debug",
            "color": "green"
          },{
            "level": "warn",
            "color": "brown"
          },{
            "level": "error",
            "color": "red"
          }
        ]
      },
      "enabled": true
    }
  ],
  "loggers": [{
      "name": "default",
      "level": "warn",
      "enabled": true,
      "additivity": false,
      "appenders": [
        "consoleout"
      ]
    }
  ]
}
EOT

/usr/bin/nodeos --data-dir /srv/api/data --config-dir /srv/api/etc --genesis-json=/srv/cryx/etc/genesis.json

systemctl enable nodeos@api
systemctl start nodeos@api
journalctl -u nodeos@api -f
```
