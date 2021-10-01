
Fetch the ignition file and proceed with the installation
~~~
coreos-installer install --insecure-ignition -I http://api.hetzner.sandbox1091.opentlc.com:22624/config/worker/config/worker /dev/vda
~~~
Approve the certs


Sep 30 20:10:32 systemd[1]: ignition-fetch-offline.service: Failed with result 'exit-code'.
Sep 30 20:10:32 ignition[825]: no config URL provided
Sep 30 20:10:32 systemd[1]: Failed to start Ignition (fetch-offline).
Sep 30 20:10:32 ignition[825]: reading system config file "/usr/lib/ignition/user.ign"
Sep 30 20:10:32 ignition[825]: parsing config with SHA512: 4c57568bc1b2ac89d2048d76378929e63322f1e3c8041ef3c27dac3b4f81f2632b88a5bf5f81cf778ae40d545e825999e8e2bf246f47e61407c4278161bd5f92
Sep 30 20:10:32 ignition[825]: failed to fetch config: unsupported config version
Sep 30 20:10:32 ignition[825]: failed to acquire config: unsupported config version
Sep 30 20:10:32 ignition[825]: Ignition failed: unsupported config version