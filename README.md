# entheos-vagrant


## prerequisites

Vagrant Setup.


## Proxy setup if required.

set http_proxy=http://proxy.example.com:8888

set https_proxy=%http_proxy%

vagrant plugin install vagrant-proxyconf


set VAGRANT_HTTP_PROXY=%http_proxy%

set VAGRANT_HTTPS_PROXY=%http_proxy%

set VAGRANT_NO_PROXY=127.0.0.1,192.168.56.0/24,fashionpeaks.club,localhost




## Workspace setup

git clone https://github.com/CrazyEntheos/entheos-vagrant

cd entheos-vagrant

Note: Stage settings.xml in entheos-vagrant directory if maven proxy setting needed.


## SSL Certificate setup if required otherwise application will work on http.

stage the fashionpeaks.crt and fashionpeaks.key under ssl-certs


## Bring the workspace up and running.

vagrant up
