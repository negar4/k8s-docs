# Fix etcd insufficient memers error

## Description

Prometheus creates a service monitor, which by default does not have certificate set up. ETCD is rejecting the request from node-exporter due to handshake error.

```text
$ kubectl logs -n kube-system etcd-dev-k8-master1.turnitvnet.eu --tail=10
{"level":"info","ts":"2022-07-05T07:44:38.159Z","caller":"mvcc/kvstore_compaction.go:57","msg":"finished scheduled compaction","compact-revision":3755939,"took":"45.478307ms"}
{"level":"warn","ts":"2022-07-05T07:45:00.769Z","caller":"embed/config_logging.go:169","msg":"rejected connection","remote-addr":"192.168.222.202:42614","server-name":"","error":"tls: first record does not look like a TLS handshake"}
{"level":"warn","ts":"2022-07-05T07:45:30.769Z","caller":"embed/config_logging.go:169","msg":"rejected connection","remote-addr":"192.168.222.202:55037","server-name":"","error":"tls: first record does not look like a TLS handshake"}
{"level":"warn","ts":"2022-07-05T07:46:00.769Z","caller":"embed/config_logging.go:169","msg":"rejected connection","remote-addr":"192.168.222.202:46290","server-name":"","error":"tls: first record does not look like a TLS handshake"}
{"level":"warn","ts":"2022-07-05T07:46:30.770Z","caller":"embed/config_logging.go:169","msg":"rejected connection","remote-addr":"192.168.222.202:59015","server-name":"","error":"tls: first record does not look like a TLS handshake"}
{"level":"warn","ts":"2022-07-05T07:47:00.769Z","caller":"embed/config_logging.go:169","msg":"rejected connection","remote-addr":"192.168.222.202:11336","server-name":"","error":"tls: first record does not look like a TLS handshake"}
{"level":"warn","ts":"2022-07-05T07:47:30.771Z","caller":"embed/config_logging.go:169","msg":"rejected connection","remote-addr":"192.168.222.202:24666","server-name":"","error":"tls: first record does not look like a TLS handshake"}
{"level":"warn","ts":"2022-07-05T07:48:00.769Z","caller":"embed/config_logging.go:169","msg":"rejected connection","remote-addr":"192.168.222.202:17037","server-name":"","error":"tls: first record does not look like a TLS handshake"}
{"level":"warn","ts":"2022-07-05T07:48:30.769Z","caller":"embed/config_logging.go:169","msg":"rejected connection","remote-addr":"192.168.222.202:24366","server-name":"","error":"tls: first record does not look like a TLS handshake"}
{"level":"warn","ts":"2022-07-05T07:49:00.769Z","caller":"embed/config_logging.go:169","msg":"rejected connection","remote-addr":"192.168.222.202:43707","server-name":"","error":"tls: first record does not look like a TLS handshake"}
```

## [Solution](https://github.com/prometheus-operator/prometheus-operator/issues/3815#issuecomment-767284696)

First of all we need to create a secret with all certificates data.
**Execute this command on the master node.**

```bash
kubectl -n monitoring create secret generic etcd-client-cert --from-file=/etc/kubernetes/pki/etcd/ca.crt --from-file=/etc/kubernetes/pki/etcd/healthcheck-client.crt --from-file=/etc/kubernetes/pki/etcd/healthcheck-client.key
```

Then we need to add [the secret name](https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/values.yaml#L2403) and [ETCD service monitor settings](https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/values.yaml#L1308-L1345) to prometheus values file.

Deploy prometheus release with new values.
