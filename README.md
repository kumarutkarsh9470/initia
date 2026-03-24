# INITIA
mkdir my-initia-project
cd my-initia-project
apt-get update
apt-get install -y lz4 curl wget tar
wget https://go.dev/dl/go1.25.8.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.25.8.linux-amd64.tar.gz

echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
wget https://github.com/initia-labs/weave/releases/download/v0.3.8/weave-0.3.8-linux-amd64.tar.gz
tar -xzf weave-0.3.8-linux-amd64.tar.gz
chmod +x weave
sudo mv weave /usr/local/bin/
export TERM=xterm-256color
script -q -c "weave version" /dev/null
script -q -c "weave init" /dev/null
