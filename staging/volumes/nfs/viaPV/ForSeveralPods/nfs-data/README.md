# NFS-exporter container with a file

This container exports /exports with index.html in it via NFS. Based on
../exports. Since some Linux kernels have issues running NFSv4 daemons in containers,
only NFSv3 is opened in this container (doesn't support NFSv4).

Available as `gcr.io/google-samples/nfs-server`
