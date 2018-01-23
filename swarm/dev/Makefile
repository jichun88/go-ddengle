.PHONY: build cluster test

default: build

build:
	go build -o bin/swarm    github.com/ethersocial/go-ddengle/cmd/swarm
	go build -o bin/geth     github.com/ethersocial/go-ddengle/cmd/geth
	go build -o bin/bootnode github.com/ethersocial/go-ddengle/cmd/bootnode

cluster: build
	scripts/boot-cluster.sh

test:
	go test -v github.com/ethersocial/go-ddengle/swarm/...
