# INITIA
1.mkdir my-initia-project
2.cd my-initia-project
3.apt-get update
4.apt-get install -y lz4 curl wget tar
5.wget https://go.dev/dl/go1.25.8.linux-amd64.tar.gz
6.sudo rm -rf /usr/local/go
7.sudo tar -C /usr/local -xzf go1.25.8.linux-amd64.tar.gz

8.echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
9.source ~/.bashrc
10.wget https://github.com/initia-labs/weave/releases/download/v0.3.8/weave-0.3.8-linux-amd64.tar.gz
11.tar -xzf weave-0.3.8-linux-amd64.tar.gz
12.chmod +x weave
13.sudo mv weave /usr/local/bin/
14.export TERM=xterm-256color
15.script -q -c "weave version" /dev/null
16.script -q -c "weave init" /dev/null
