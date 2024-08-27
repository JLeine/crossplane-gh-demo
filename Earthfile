VERSION 0.8
FROM ghcr.io/vfarcic/silly-demo-earthly:0.0.5
ARG --global registry=ttl.sh
ARG --global image=crossplane-gh-demo
WORKDIR /go-workdir

binary:
    COPY go.mod go.sum .
    COPY *.go .
    RUN GOOS=linux GOARCH=amd64 go build -o app
    SAVE ARTIFACT app

image:
    BUILD +binary
    ARG tag='latest'
    FROM scratch
    EXPOSE 8080
    CMD ["app"]
    ENV VERSION=$tag
    COPY +binary/app /usr/local/bin/app
    SAVE IMAGE --push $registry/$image:$tag $registry/$image:latest

manifests:
    ARG tag
    COPY k8s/appclaim.yaml k8s/appclaim.yaml
    RUN yq --inplace ".spec.parameters.image = \"$registry/$image:$tag\"" k8s/appclaim.yaml
    SAVE ARTIFACT k8s/appclaim.yaml AS LOCAL k8s/appclaim.yaml

all:
    ARG tag
    WAIT
        BUILD +image --tag $tag
    END
    BUILD +manifests --tag $tag
