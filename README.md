# istio-mixer-authz-adapter

First of all, get go environment ready.

~~~ shell
cd /tmp
git clone https://github.com/JianshaoWu/istio-mixer-authz-adapter.git

mkdir -p $GOPATH/src/istio.io/ && \
cd $GOPATH/src/istio.io/  && \
git clone https://github.com/istio/istio

export ISTIO=$GOPATH/src/istio.io
export MIXER_REPO=$GOPATH/src/istio.io/istio/mixer

pushd $ISTIO/istio && make mixs
pushd $ISTIO/istio && make mixc

cp /tmp/istio-mixer-authz-adapter/authzadapter $MIXER_REPO/adapter/ -r

go generate $MIXER_REPO/adapter/authzadapter/
# go generate needs docker, may need root privilege, if so, GOPATH need pass for sudo as following
# sudo GOPATH=$GOPATH go generate $MIXER_REPO/adapter/authzadapter/
go build $MIXER_REPO/adapter/authzadapter/

cp $MIXER_REPO/adapter/authzadapter/config/authzadapter.yaml $MIXER_REPO/adapter/authzadapter/testdata/

go run $MIXER_REPO/adapter/authzadapter/cmd/main.go 45678

$GOPATH/out/linux_amd64/release/mixs server --configStoreURL=fs://${MIXER_REPO}/adapter/authzadapter/testdata

$GOPATH/out/linux_amd64/release/mixc check -s destination.service.host="testservice.svc.cluster.local",destination.namespace="test-namespace",request.path="/test",request.method="GET" --stringmap_attributes "request.headers=Authorization:Basic dGVzdENsaWVudDpzZWNyZXQ="

cd /tmp/istio-mixer-authz-adapter
CGO_ENABLED=0 GOOS=linux go build -a -v -o bin/authzadapter $MIXER_REPO/adapter/authzadapter/cmd/main.go

docker build -t mymixeradapter/authzadapter:latest .
# in case no remote repository
docker save -o authzadapter.tar mymixeradapter/authzadapter
docker load -i authzadapter.tar

kubectl -f authzadapter.yaml

~~~